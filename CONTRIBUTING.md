# Contributing

Use the shared Idarsi Python 3.12 environment for Ansible and Molecule commands:

```bash
export PATH="/home/arsi/.local/share/venvs/idarsi-ansible-testing/bin:$PATH"
```

Run the KVM smoke scenario from the role directory:

```bash
molecule test -s kvm-containerd
```

Keep KVM/Vagrant-specific settings in Molecule scenario files. Do not add
test-only VM configuration to production inventory examples.

## Blueprint applications

Application definitions belong under `iac_blueprint.kubernetes.applications`.
Every entry must have a DNS-compatible `name`, an explicit `state` of
`present` or `absent`, and a non-empty Kubernetes YAML `manifest`. Changes to
this interface must update `README.md` and include Molecule coverage for
creation, idempotence, and removal where applicable.
