ANSIBLE-IAC-ROLE-KUBERNETES
===========================
**COPYRIGHT** 2026 ^(ida|arsi)$ collective
**LICENSE** MIT License [LICENSE](LICENSE)
**AUTHORS**
- Arsi Atomi <arsi@atomi.sh>
- Arsi Atomi <arsi.atomi@valtori.fi>

Overview
========

This Ansible role provides a declarative way to install, configure, and remove
Kubernetes hosts using either containerd or CRI-O as the container runtime.

The role prepares the operating system, configures kernel and cgroup settings,
manages swap and firewalld, keeps SELinux enforcing by default, installs
Kubernetes from the official `pkgs.k8s.io` repository, and can bootstrap a
control plane, install Flannel, and join worker nodes.

The role uses declarative `present` and `absent` states for Kubernetes
components. The `install` and `uninstall` states provide the complete
installation and removal workflows.

> **Maturity Level: Beta**<br>
> The role's initial container-runtime, control-plane, Flannel, worker-join,
> SELinux, and uninstall workflows are implemented and covered by KVM-backed
> Molecule scenarios. Interfaces and supported Kubernetes combinations may
> still evolve before a stable release.

Supported Kubernetes versions are selected through the `kubernetes_version`
repository channel variable. The default channel is `v1.34`.

Supported operating systems:

Operating system                | Supported versions | Lifecycle reference
--------------------------------|--------------------|--------------------
Rocky Linux                     | 9, 10              | [Rocky Linux release guide](https://wiki.rockylinux.org/rocky/version/)
Red Hat Enterprise Linux        | 9, 10              | [Red Hat documentation](https://access.redhat.com/documentation/)

The automated KVM test matrix currently uses Rocky Linux 9. Rocky Linux 10 is
included in the role's supported-operating-system assertions and should be
validated in the target environment before production use.

Supported container runtimes:

- containerd, the default runtime, installed from Docker's official RPM
  repository.
- CRI-O, installed from the HTTPS CRI-O stable RPM repository for the selected
  Kubernetes channel (default `v1.34`). The default source is the upstream
  OpenSUSE repository; an HTTPS mirror and its HTTPS GPG key can be supplied
  together with `kubernetes_crio_repository_mirror_baseurl` and
  `kubernetes_crio_repository_mirror_gpgkey`.

Kubernetes packages are installed from the official
[Kubernetes package repository](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/).
The role does not install Kubernetes from distribution-provided Kubernetes
packages.

These operations are supported:

Operation                                      | State
-----------------------------------------------|-----------------
Install and configure the complete host        | install
Configure an already installed Kubernetes host | present
Remove Kubernetes configuration and packages   | uninstall
Remove Kubernetes from the host                | absent
Ensure the control plane exists                | kubernetes_control_plane_state: present
Remove the control plane                        | kubernetes_control_plane_state: absent
Ensure a worker has joined                     | kubernetes_worker_state: present
Remove the worker membership                    | kubernetes_worker_state: absent
Ensure the CNI exists                          | kubernetes_cni_state: present
Remove the CNI                                 | kubernetes_cni_state: absent
Apply or remove declared applications          | applications[].state: present/absent

The component-level `absent` states remove only the selected Kubernetes
component and preserve the container runtime and Kubernetes packages. Use
`state: uninstall` or `state: absent` for the complete host removal workflow.

The public root `iac_blueprint.kubernetes` is required. Legacy top-level
variables remain accepted as compatibility overrides when that root is also
provided; new inventories should use the blueprint fields.
Applications can be declared under `iac_blueprint.kubernetes.applications`.
The role accepts Kubernetes YAML manifests and Helm charts. Both are applied
or removed on the control plane. Keep application definitions explicit, pin
container image and chart versions, and store secrets outside manifests or
through the deployment's approved secret-management process.

Quick start
-----------

Install a single-node Kubernetes control plane with containerd and Flannel:

```yaml
---
- hosts: kubernetes_control_plane
  become: true
  roles:
    - role: ansible-iac-role-kubernetes
      state: install
  vars:
    iac_blueprint:
      kubernetes:
        control_plane_state: "present"
        cni_state: "present"
```

The default security posture is SELinux `enforcing`, swap disabled, systemd
cgroups enabled, and firewalld management enabled. Override these defaults only
when the target environment has an explicit and documented requirement.

To use CRI-O instead of containerd:

```yaml
---
- hosts: kubernetes_control_plane
  become: true
  roles:
    - role: ansible-iac-role-kubernetes
      state: install
  vars:
    iac_blueprint:
      kubernetes:
        container_runtime: "crio"
        control_plane_state: "present"
        cni_state: "present"
```

To join a worker to a control plane, run the role for the worker with the
control-plane inventory hostname:

```yaml
---
- hosts: kubernetes_workers
  become: true
  roles:
    - role: ansible-iac-role-kubernetes
      state: present
  vars:
    iac_blueprint:
      kubernetes:
        worker_state: "present"
        control_plane_inventory_hostname: "kubernetes-control-1"
```

Requirements
------------

- Rocky Linux 9 or 10, or RHEL 9 or 10.
- Ansible 2.15 or higher.
- Root privileges or passwordless privilege escalation on target hosts.
- Network access to the Kubernetes, container runtime, and Flannel package or
  release repositories.
- For the KVM Molecule scenarios: KVM, libvirt, Vagrant, and the
  `vagrant-libvirt` plugin.

The versions used by the automated test environment are documented in
[requirements-ci.txt](requirements-ci.txt). Install them in a dedicated Python
environment before running the tests locally.

Usage
=====

How to run a playbook with an inventory
---------------------------------------

Use a playbook and inventory that match the target environment:

```bash
ansible-playbook -i <inventory_file> <playbook_file> -K
```

Complete installation example
-----------------------------

```yaml
---
- hosts: kubernetes
  become: true
  roles:
    - role: ansible-iac-role-kubernetes
      state: install
  vars:
    iac_blueprint:
      kubernetes:
        control_plane_state: "present"
        cni_state: "present"
```

The `install` state prepares the host, installs the selected runtime and
Kubernetes packages, enables SELinux policy integration, bootstraps the
control plane when requested, installs Flannel, and joins a worker when its
worker state is enabled.

Declarative component workflow
------------------------------

The complete workflow can also be expressed as separate desired states:

```yaml
---
- hosts: kubernetes_control_plane
  become: true
  roles:
    - role: ansible-iac-role-kubernetes
      state: present
  vars:
    iac_blueprint:
      kubernetes:
        control_plane_state: "present"
        cni_state: "present"

- hosts: kubernetes_workers
  become: true
  roles:
    - role: ansible-iac-role-kubernetes
      state: present
  vars:
    iac_blueprint:
      kubernetes:
        worker_state: "present"
        control_plane_inventory_hostname: "kubernetes-control-1"
```

Application blueprint
---------------------

Each application requires a DNS-compatible `name`, a declarative `state`, and
a Kubernetes YAML `manifest`. The manifest may contain one or more Kubernetes
resources separated with `---`:

```yaml
iac_blueprint:
  kubernetes:
    applications:
      - name: nginx
        state: present
        manifest: |
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: nginx
            namespace: example
          spec:
            replicas: 2
            selector:
              matchLabels:
                app: nginx
            template:
              metadata:
                labels:
                  app: nginx
              spec:
                containers:
                  - name: nginx
                    image: docker.io/library/nginx:1.27
```

Set `state: absent` with the same manifest to remove the declared resources.
Applications are applied only on the control-plane host, after the control
plane is available. `state: absent` and `state: uninstall` remove declared
applications before removing the Kubernetes control plane.

Helm application:

```yaml
iac_blueprint:
  kubernetes:
    applications:
      - name: nginx
        type: helm
        state: present
        chart: nginx
        repo_url: https://example.org/helm-charts
        chart_version: 1.2.3
        release_name: nginx
        namespace: example
        values:
          replicaCount: 2
```

For a local chart, use an absolute `chart` path on the target control-plane
host and omit `repo_url` and `chart_version`. Helm is downloaded only over
HTTPS and is verified with the pinned architecture-specific
`kubernetes_helm_checksums` values. Override those role-controlled checksums
only with a separately verified release checksum; the role never downloads an
unauthenticated checksum from the mutable download endpoint.

Complete removal
----------------

The `uninstall` and `absent` states remove Kubernetes, the selected container
runtime, role-managed repositories and configuration, restore swap settings,
remove the control-plane and worker configuration, and clean role-managed
firewalld and SELinux entries. Use them only when the host should be removed
from the Kubernetes deployment.

The etcd data directory is removed during `absent`/`uninstall` only when its
root-owned external marker (`kubernetes_etcd_ownership_marker`, default
`/var/lib/.ansible-iac-role-kubernetes-etcd-owned`) proves that this role
created the directory. The marker is written only when `/var/lib/etcd` did not
already exist. An unmarked pre-existing `/var/lib/etcd` directory is
intentionally left untouched. Keeping the marker outside the data directory
also leaves the new etcd directory empty for kubeadm. SELinux fcontext
cleanup follows the same ownership rule: installation records only newly added
target/type pairs in a root-owned mode `0600` marker, and removal deletes only
those recorded matching rules before deleting the marker. If a recorded rule
is stale or has changed, removal fails safely and retains the marker.
Pre-existing contexts are never removed.

The `validate` state performs blueprint and cross-host validation only. It does
not install packages, alter files, restart services, or change cluster data.

## Testing

The KVM-backed Molecule scenarios and their prerequisites are documented in
[TESTING.md](TESTING.md). The scenarios use real Rocky Linux virtual machines
through libvirt/KVM rather than containerized test hosts.

Contributing
============

Contribution and local development guidance is available in
[CONTRIBUTING.md](CONTRIBUTING.md).
