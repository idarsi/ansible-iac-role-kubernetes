# Testing

Molecule uses the shared Idarsi KVM backend and disposable virtual machines.
Activate the shared `ansible-current` environment described in
`idarsi-ansible-test-environments/docs/kvm-molecule.md`; do not install role
test dependencies into a role-local environment. Podman is not appropriate
for these KVM scenarios.

The `kvm-containerd` scenario uses Molecule's managed/default driver with the
shared create/destroy playbooks and platform-specific kickstart templates. The activated
shared `ansible-current` environment sets `IDARSI_TEST_ENVIRONMENT` to the
environment repository and provides pinned Ansible collections. Shared KVM
playbooks default to libvirt's `default` network. Rocky 9 uses
`IDARSI_TEST_KVM_INSTALL_SOURCE`; Rocky 10 uses
`IDARSI_TEST_KVM_INSTALL_SOURCE_ROCKY10`. Configured RHEL 9 and 10 entries use
`IDARSI_TEST_KVM_INSTALL_SOURCE_RHEL9` and
`IDARSI_TEST_KVM_INSTALL_SOURCE_RHEL10`, respectively. See the shared KVM guide
for host setup and prerequisites.

## Automated matrix

| Platform/image | Ansible or application versions | Molecule scenarios | Main coverage |
|---|---|---|---|
| Rocky Linux 9 / KVM | Kubernetes v1.34; versions in `requirements-ci.txt` | `validation`; `kvm-containerd` | Focused containerd KVM run passed convergence, idempotence, and verification |
| Rocky Linux 10 / KVM | Kubernetes v1.34 | `kvm-containerd` | Focused containerd KVM run passed convergence, idempotence, and verification |
| RHEL 9 / KVM | Kubernetes v1.34 | `kvm-containerd` | Configured only; unverified pending subscription-backed media and guest repository automation |
| RHEL 10 / KVM | Kubernetes v1.34 | `kvm-containerd` | Configured only; unverified pending subscription-backed media and guest repository automation |

## Scenario coverage

- `validation`: default, custom, and legacy API ports, blueprint precedence,
  inclusive port boundaries, and rejection of invalid types/ranges through the
  role entrypoint. API-port negative tests assert the exact failure and use a
  separate final play to detect unexpected acceptance despite `meta: end_host`.
  Existing Helm/reference/cleanup negative tasks later in `verify.yml` remain
  unreachable after its successful validation-only role invocation ends the
  play; those older tests must not be counted as executed coverage.
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

Only validation runs on ordinary pull requests. The practical Rocky `kvm-containerd`
baseline is configured for the scheduled or manually dispatched CI workflow on
a self-hosted runner labelled `kvm`; CI execution remains unverified. The
remaining KVM scenarios are manual.
KVM scenarios require the shared environment's documented KVM host setup,
hardware virtualization, libvirt, an active `default` network, and the
matching installation source. Scheduled/manual CI targets Rocky 9 and Rocky 10
separately with `--platform-name`. The scenario retains configured RHEL 9 and 10
entries for future/manual testing; these are not verified or working in the
current environment and require subscription-backed media plus guest repository
automation. An unfiltered `molecule test -s kvm-containerd` includes RHEL and can
fail without those prerequisites. An empty `install_source` does not skip a
platform. Rocky 9 uses
`IDARSI_TEST_KVM_INSTALL_SOURCE`, Rocky 10 uses
`IDARSI_TEST_KVM_INSTALL_SOURCE_ROCKY10`. RHEL media variables are reserved for
future/manual testing; configure guest repository access or registration
before attempting. The shared tooling does not download RHEL media or store
subscription credentials. KVM cannot run on
ordinary GitHub-hosted runners.

Every VM-backed scenario uses the shared managed/default KVM driver and shared
create/destroy playbooks; Vagrant is not involved. Bare `molecule reset` uses
the `default` scenario and fails because this role has no default scenario.
Reset a named scenario explicitly, for example `molecule reset -s kvm-containerd`
or `molecule reset -s kvm-crio`.

The verified evidence is focused Rocky Linux 9 and Rocky Linux 10
`kvm-containerd` runs passing convergence, idempotence, and verification,
including SELinux `/var/run/flannel` handling on Rocky Linux 9 (via semanage
equivalency) and `/run/flannel` handling on Rocky Linux 10. This evidence does
not cover every scenario or constitute a full-suite run. RHEL runtime coverage
remains unverified pending subscription/repository automation.

## Commands

### Ansible compatibility evidence

At base commit `325fefc`, the investigation reported syntax and lint passing
on the shared `ansible-min` (Core 2.15.13/Python 3.9), `ansible-current`
(Core 2.20.8), and `ansible-next` (Core 2.21.3) lanes. `molecule test -s validation`
passed current/next but failed min: a filtered scalar `set_fact` expression
converted the default integer port to a string before the numeric assertion.
The focused fix retains the port in the blueprint mapping, validates the raw
blueprint/legacy value, and avoids overwriting the legacy input with a scalar
fact. Other normalized fields were inspected; no other numeric scalar
normalization/type assertion pair was found.

The tester subsequently confirmed API-port regressions passed all three lanes,
including exactly 24 expected negative cases. Current/next completed validation
(`ok=336`, no failures); min reached `ok=291`, `rescued=24`, then failed the
Flannel verifier because its namespaced defaults mapping eagerly templated
unrelated sibling references on Core 2.15. The verifier now loads defaults under
their original names, resolving those references without duplicating defaults.
All four platform assertions use the direct SELinux-context variable; no other
namespaced defaults references were found in Molecule scenarios.
After the verifier adjustment and stronger blueprint-negative isolation (a valid
legacy fallback), the tester's final `molecule test -s validation` runs exited
0 on Core 2.15.13, 2.20.8 and 2.21.3. All positive cases, exactly 24 expected
negative cases with exact failure assertions, the completeness check and all
four platform-specific SELinux assertions passed. These platform assertions
validate defaults; they are not four-platform runtime verification.

The final current-lane command
`molecule test -s kvm-containerd --platform-name k8s-containerd-rocky9`
also exited 0: convergence reported 27 changes, idempotence reported zero
changes, and verification reported 16 successful tasks with zero failures.
Destroy succeeded; the scenario's VM, disk and keys were absent afterward.
Unrelated VMs and the preexisting `.ansible_async/` directory were unchanged.
Other platforms, runtimes and the full suite were not rerun; prior Rocky 10
results remain historical evidence, not results for this working tree.

Run `molecule test -s validation` after activating each shared environment;
do not install or change dependencies to match the CI pins. Local current and
min intentionally differ from `requirements-ci.txt`, which pins the next lane.
CI is not yet a three-lane compatibility matrix. Port tests validate inputs,
not custom-port cluster connectivity; kubeadm bind-port configuration is
outside this focused change. The tester owns Molecule and KVM execution.

## Preflight

Run these checks before starting a KVM scenario. They fail early with a
specific explanation when the shared test environment, role lookup, or libvirt
network is not ready:

```bash
test -n "${IDARSI_TEST_ENVIRONMENT:-}" || { echo "Activate the shared ansible-current environment first" >&2; exit 1; }
test -d "$IDARSI_TEST_ENVIRONMENT" || { echo "IDARSI_TEST_ENVIRONMENT does not exist: $IDARSI_TEST_ENVIRONMENT" >&2; exit 1; }
command -v ansible-playbook >/dev/null || { echo "ansible-playbook is missing from the active environment" >&2; exit 1; }
command -v molecule >/dev/null || { echo "molecule is missing from the active environment" >&2; exit 1; }
test -f "$(dirname "$PWD")/ansible-iac-role-kubernetes/tasks/main.yml" || { echo "Role is not located beside its expected ansible-iac-role-kubernetes path" >&2; exit 1; }
virsh --connect qemu:///system net-info default | grep -q 'Active:[[:space:]]*yes' || { echo "libvirt default network is not active" >&2; exit 1; }
```

For Rocky Linux KVM guests, also verify that an installation source is
configured before starting the scenario:

```bash
test -n "${IDARSI_TEST_KVM_INSTALL_SOURCE:-}" || { echo "Set IDARSI_TEST_KVM_INSTALL_SOURCE to the Rocky Linux 9 installation source" >&2; exit 1; }
test -n "${IDARSI_TEST_KVM_INSTALL_SOURCE_ROCKY10:-}" || { echo "Set IDARSI_TEST_KVM_INSTALL_SOURCE_ROCKY10 to the Rocky Linux 10 installation source" >&2; exit 1; }
```

Use `default` for the network check because the shared KVM playbooks create
guests on libvirt's default network. RHEL scenarios additionally need the
RHEL-specific media and guest repository access described above.

```bash
source /path/to/idarsi-ansible-test-environments/.venv/ansible-current/bin/activate
ANSIBLE_ROLES_PATH="$(dirname "$PWD")" ansible-playbook --syntax-check -i 'localhost,' molecule/validation/converge.yml
ANSIBLE_ROLES_PATH="$(dirname "$PWD")" ansible-lint --profile production
ANSIBLE_ROLES_PATH="$(dirname "$PWD")" molecule test -s validation
ANSIBLE_ROLES_PATH="$(dirname "$PWD")" molecule test -s kvm-containerd --platform-name k8s-containerd-rocky9
ANSIBLE_ROLES_PATH="$(dirname "$PWD")" molecule test -s kvm-containerd --platform-name k8s-containerd-rocky10
```

Before running a KVM scenario, verify the runner-side prerequisite without
letting Molecule manage it:

```bash
virsh --connect qemu:///system net-info default
```

The command must report an active network. Molecule manages the disposable test
domains; the shared lifecycle leaves the shared network untouched.

Run `molecule converge` and `molecule verify` for faster iteration. The
pull-request CI job intentionally stops at syntax, production lint, and
validation; use the scheduled/manual workflow or a prepared KVM host for the
functional baseline and other KVM scenarios.
