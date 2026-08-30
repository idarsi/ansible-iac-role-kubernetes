# Testing

The initial test scenario uses Molecule with the Vagrant driver and the
libvirt provider. Vagrant creates a real Rocky Linux virtual machine on KVM;
the scenario does not run inside a Podman container.

## Scenario matrix

Scenario       | Platform             | Coverage
---------------|----------------------|-----------------------------
`kvm-containerd` | Rocky Linux 9, KVM | VM creation, host preparation, containerd, idempotence, and verification
`kvm-crio`     | Rocky Linux 9, KVM   | VM creation, host preparation, CRI-O, idempotence, and verification
`kvm-containerd-cluster` | Rocky Linux 9, KVM | Two-node control plane and worker cluster, CNI, idempotence, and verification
`kvm-crio-cluster` | Rocky Linux 9, KVM | Two-node control plane and worker cluster with CRI-O, CNI, idempotence, and verification
`kvm-containerd-uninstall` | Rocky Linux 9, KVM | Full Kubernetes uninstall and absence verification
`kvm-containerd-component-absent` | Rocky Linux 9, KVM | Declarative worker, CNI, and control-plane absence workflow
`kvm-crio-component-absent` | Rocky Linux 9, KVM | Declarative worker, CNI, and control-plane absence workflow with CRI-O

The `kvm-containerd` scenario also bootstraps a single Kubernetes control plane
with `kubeadm init`, installs the pinned Flannel CNI manifest, and verifies that
the node becomes `Ready`.

## Prerequisites

- KVM and libvirt are working with `virsh -c qemu:///system`.
- Vagrant is installed.
- The `vagrant-libvirt` Vagrant plugin is installed.
- The current user can access the system libvirt connection.

Install the CI dependencies in a dedicated Python virtual environment:

```bash
python -m pip install -r requirements-ci.txt
```

With the current Molecule and Ansible versions, expose the Vagrant plugin's
custom module explicitly when running the scenario:

```bash
export ANSIBLE_LIBRARY="$(python -c 'import molecule_plugins.vagrant, pathlib; print(pathlib.Path(molecule_plugins.vagrant.__file__).parent / "modules")')"
```

The command above intentionally uses the active Python environment to locate
the installed plugin module.

## Running the scenario

Run from the role directory:

```bash
molecule test -s kvm-containerd
```

Run the CRI-O runtime scenario:

```bash
molecule test -s kvm-crio
```

Run the two-node worker join scenario:

```bash
molecule test -s kvm-containerd-cluster
```

Run the two-node worker join scenario with CRI-O:

```bash
molecule test -s kvm-crio-cluster
```

Test the complete Kubernetes uninstall path:

```bash
molecule test -s kvm-containerd-uninstall
```

Test the declarative component-level absence workflow with containerd:

```bash
molecule test -s kvm-containerd-component-absent
```

Test the component-level absence workflow with CRI-O:

```bash
molecule test -s kvm-crio-component-absent
```

For faster iteration:

```bash
molecule converge -s kvm-containerd
molecule verify -s kvm-containerd
```

The scenario creates a VM in the system libvirt storage pool and removes it
when the full `molecule test` workflow completes.
