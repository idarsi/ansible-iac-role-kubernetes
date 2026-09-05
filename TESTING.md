# Testing

Molecule uses Vagrant/libvirt and disposable Rocky Linux virtual machines. The
local shared Idarsi environment is used for Ansible tooling; Podman is not
appropriate for these KVM scenarios.

All KVM scenarios connect to the externally managed `default` network through
`qemu:///system`. Each platform's `provider_options` (the Molecule Vagrant
platform-level location) sets `management_network_name: "default"`,
`management_network_address: "192.168.122.1/24"`, and
`management_network_keep: true`. In vagrant-libvirt 0.11.2,
`management_network_keep` maps the management interface to
`always_destroy: false`; an existing network is therefore neither recorded as
created nor undefined during destroy. The network must already exist and be
active on the KVM runner. A dedicated stable network may be substituted
consistently in every KVM `molecule.yml` if the runner does not use `default`.

## Automated matrix

| Platform/image | Ansible or application versions | Molecule scenarios | Main coverage |
|---|---|---|---|
| Rocky Linux 9 / KVM | Kubernetes v1.34; versions in `requirements-ci.txt` | `validation`; `kvm-containerd` baseline on scheduled/manual KVM runner | Validation-only safety; runtime installation, bootstrap, idempotence and verification |
| Rocky Linux 10 / KVM | Kubernetes v1.34 | None currently | Supported, not automatically tested |
| RHEL 9/10 / KVM | Kubernetes v1.34 | None currently | Supported, not automatically tested |

## Scenario coverage

- `validation`: valid blueprint validation without host mutation; invalid schema,
  duplicate, and reference fixtures should be added as the scenario expands.
- `kvm-containerd`, `kvm-crio`: baseline runtime installation, bootstrap and
  idempotence.
- `kvm-containerd-cluster`, `kvm-crio-cluster`: worker references, CNI and
  multi-node convergence.
- `kvm-containerd-uninstall`: destructive cleanup in a disposable VM.
- `kvm-containerd-component-absent`, `kvm-crio-component-absent`: component
  absence states.
- `kvm-containerd-application`: manifest and Helm lifecycle and idempotence.
- `kvm-containerd-uninstall`: verifies role-owned etcd cleanup and removal of
  the external ownership marker while preserving unmarked data and
  ownership-safe SELinux cleanup on disposable hosts.

Only validation runs on ordinary pull requests. The practical `kvm-containerd`
baseline runs from the scheduled or manually dispatched CI workflow on a
self-hosted runner labelled `kvm`; the remaining KVM scenarios are manual.
KVM scenarios require hardware virtualization, libvirt, Vagrant, and the
`vagrant-libvirt` plugin, plus an active externally managed libvirt `default`
network on `qemu:///system`, so they cannot run on the ordinary GitHub-hosted
runner. Rocky Linux 10 and RHEL 9/10 remain supported but are not automatically
tested.

## Commands

```bash
export PATH="/home/arsi/.local/share/venvs/idarsi-ansible-testing/bin:$PATH"
ANSIBLE_ROLES_PATH="$(dirname "$PWD")" ansible-playbook --syntax-check -i 'localhost,' molecule/validation/converge.yml
ANSIBLE_ROLES_PATH="$(dirname "$PWD")" ansible-lint --profile production
molecule test -s validation
molecule test -s kvm-containerd
```

Before running a KVM scenario, verify the runner-side prerequisite without
letting Molecule manage it:

```bash
virsh -c qemu:///system net-info default
```

The command must report an active network. Vagrant may destroy the test domains;
the provider settings above ensure it does not manage the shared network.

Run `molecule converge` and `molecule verify` for faster iteration. The
pull-request CI job intentionally stops at syntax, production lint, and
validation; use the scheduled/manual workflow or a prepared KVM host for the
functional baseline and other KVM scenarios.
