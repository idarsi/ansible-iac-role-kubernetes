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
`IDARSI_TEST_KVM_INSTALL_SOURCE_ROCKY10`. RHEL 9 and 10 are temporarily
excluded from runtime tests, without changing `iac_supported_os` or the public
role contract. No RHEL media variables are needed by the active matrix.
See the shared KVM guide for host setup and prerequisites.

## Automated matrix

| Platform/image | Ansible or application versions | Molecule scenarios | Main coverage |
|---|---|---|---|
| Rocky Linux 9 / KVM | Kubernetes v1.34; shared Core 2.15.13 / 2.20.8 / 2.21.3 | `validation`; `kvm-containerd` | Three-lane validation passed; min/next KVM passed at `0829cb9` plus test repairs, historical current KVM pass retained at `856d417` |
| Rocky Linux 9 / KVM | Kubernetes v1.34; Core 2.15.13 / 2.20.8 / 2.21.3 | `kvm-containerd-application`; `kvm-containerd-uninstall`; `kvm-crio-component-absent` | All nine targeted combinations passed at `8919b8c` plus scoped repairs; details below |
| Rocky Linux 10 / KVM | Kubernetes v1.34 | `kvm-containerd` | Focused containerd KVM run passed convergence, idempotence, and verification |

RHEL 9/10 have no executable KVM platforms and no runtime coverage claim.
The retained RHEL kickstart templates are inactive fixtures for possible later
restoration. Synthetic RHEL facts in `validation/verify.yml` cover SELinux
defaults only, not execution on RHEL hosts.

## Scenario coverage

- Scoped follow-up repairs (nine targeted combinations passed): Helm ensures `tar` and
  `gzip` before extracting the checksum-pinned archive.
  `kvm-crio-component-absent` asserts the exact intended installation blueprint
  and a single HTTPS GPG-key URL before invoking the role.
  `kvm-containerd-uninstall` asserts that removing the existing sysctl file
  reloads sysctl successfully, then repeats uninstall and requires the reload
  to be skipped. The repaired uninstall verifier checks each managed file and
  marked etcd path individually, reports allowed parent remnants, and verifies
  an unmanaged CNI sentinel survives both removal passes. The earlier aggregate
  parent-directory absence verifier failed; that failure is superseded by the
  successful focused retest below, not by recursive deletion of shared trees.
- `validation`: default, custom, and legacy API ports, blueprint precedence,
  inclusive port boundaries, and rejection of invalid types/ranges through the
  role entrypoint. API-port negative tests assert the exact failure and use a
  separate final play to detect unexpected acceptance despite `meta: end_host`.
  The three older Helm/reference/cleanup negatives now run in a separate play
  after the successful validation-only invocation. Each checks the exact failed
  task and message before recording its identity; a final separate play requires
  all three identities in order and a count of three, even if unexpected
  acceptance invokes `meta: end_host`. Application/reference facts are reset
  between cases to prevent an earlier invalid input contaminating the next.
  Runtime confirmation passed all three lanes with 27 exact
  negative cases (24 API-port plus three older cases). The retained marker checks
  only establish marker absence, not comprehensive host non-mutation evidence.
- `kvm-containerd`, `kvm-crio`: baseline runtime installation, bootstrap and
  idempotence.
- `kvm-containerd-cluster`, `kvm-crio-cluster`: worker references, CNI and
  multi-node convergence.
- `kvm-containerd-uninstall`: destructive cleanup in a disposable VM.
- `kvm-containerd-component-absent`, `kvm-crio-component-absent`: component
  absence states.
- `kvm-containerd-application`: manifest and Helm lifecycle with repeat-converge
  assertions, not a standalone zero-change Molecule idempotence action.
- `kvm-containerd-uninstall`: verifies role-owned etcd cleanup and removal of
  the external ownership marker, unmanaged CNI preservation, sysctl reload
  behavior and SELinux marker removal on disposable hosts. This does not prove
  comprehensive unmarked-etcd or stale-SELinux-rule preservation behavior.

Only validation runs on ordinary pull requests. The practical Rocky `kvm-containerd`
baseline is configured for the scheduled or manually dispatched CI workflow on
a self-hosted runner labelled `kvm`; CI execution remains unverified. The
remaining KVM scenarios are manual.
KVM scenarios require the shared environment's documented KVM host setup,
hardware virtualization, libvirt, an active `default` network, and the
matching installation source. Scheduled/manual CI targets Rocky 9 and Rocky 10
separately with `--platform-name`. An unfiltered
`molecule test -s kvm-containerd` now selects exactly Rocky 9 and Rocky 10;
RHEL 9/10 are temporarily excluded, including from manual runtime selection.
An empty `install_source` does not skip a platform. Rocky 9 uses
`IDARSI_TEST_KVM_INSTALL_SOURCE`, Rocky 10 uses
`IDARSI_TEST_KVM_INSTALL_SOURCE_ROCKY10`. Restoring RHEL requires an explicit
matrix update plus authorized media and guest repository access. The shared
tooling does not download RHEL media or store subscription credentials. KVM cannot run on
ordinary GitHub-hosted runners.

Every VM-backed scenario uses the shared managed/default KVM driver and shared
create/destroy playbooks; Vagrant is not involved. Bare `molecule reset` uses
the `default` scenario and fails because this role has no default scenario.
Reset a named scenario explicitly, for example `molecule reset -s kvm-containerd`
or `molecule reset -s kvm-crio`.

The shared create and destroy playbooks are configured under
`provisioner.playbooks` in each KVM scenario's `molecule.yml`, a configuration
shape supported by both Molecule 6 and current Molecule. Run
`molecule syntax -s <scenario>` in the selected shared environment to parse
and validate a scenario without creating a VM. Molecule 6 does not provide a
`molecule config` command.

The earlier verified baseline evidence is focused Rocky Linux 9 and Rocky Linux 10
`kvm-containerd` runs passing convergence, idempotence, and verification,
including SELinux `/var/run/flannel` handling on Rocky Linux 9 (via semanage
equivalency) and `/run/flannel` handling on Rocky Linux 10. This evidence does
not cover every scenario or constitute a full-suite run. RHEL runtime coverage
remains unverified and temporarily excluded from the matrix.

## Commands

### Final nine-combination focused runtime evidence

At `8919b8c` plus scoped working-tree repairs, the tester confirmed all three
commands below passed in each shared lane:

```bash
molecule test -s kvm-containerd-application
molecule test -s kvm-containerd-uninstall
molecule test -s kvm-crio-component-absent
```

| Lane | Ansible Core | Molecule | Result |
|---|---|---|---|
| `ansible-min` | 2.15.13 | 6 | All three passed |
| `ansible-current` | 2.20.8 | 26.8 | All three passed |
| `ansible-next` | 2.21.3 | 26.8 | All three passed |

The earlier min application pass was retained because its code and fixtures
were unchanged; eight follow-up runs completed the matrix. Each command exited
0 with no failed or unreachable hosts. Scenario-owned VMs, disks and keys were
absent after successful teardown; unrelated resources were untouched.

| Scenario | Converge recap | Verify recap | Observable evidence |
|---|---|---|---|
| Application | ok=270, changed=44 | ok=5, changed=0 | Manifest/Helm lifecycle and repeat-converge assertions passed |
| Uninstall | ok=154, changed=44 | ok=10, changed=0 | All 16 managed paths absent; unmanaged CNI content preserved across two uninstalls; initial sysctl reload and repeated skip passed |
| CRI-O component absence | control plane ok=198, changed=35; worker ok=115, changed=22 | Each host ok=3, changed=0 | Repaired fixture and component absence checks passed |

The application run did **not** execute a standalone Molecule zero-change
idempotence action; its repeat assertions are narrower evidence. These nine
combinations are three scenarios across three lanes, not nine different
scenarios or a full-suite run. RHEL remains blocked and unverified, with no
retries in this retest; authorized matching guest repositories are still an
external prerequisite. Readiness remains 52%/Alpha for the criterion-specific
reasons in [MATURITY.md](MATURITY.md).

### Scoped Helm, uninstall and CRI-O fixture static checks

At `8919b8c` plus these scoped working-tree repairs, the following commands
passed in each existing shared `ansible-min`, `ansible-current` and
`ansible-next` environment (Core 2.15.13, 2.20.8 and 2.21.3):

```bash
export ANSIBLE_ROLES_PATH="$(dirname "$PWD")"
ansible-playbook --syntax-check -i 'localhost,' molecule/kvm-crio-component-absent/converge.yml
ansible-playbook --syntax-check -i 'localhost,' molecule/kvm-containerd-uninstall/converge.yml
ansible-lint --offline --profile production tasks/helm.yml tasks/uninstall.yml molecule/kvm-crio-component-absent/converge.yml molecule/kvm-containerd-uninstall/converge.yml
git diff --check
```

Each lint run processed 21 files with zero failures/warnings. Syntax checks
only warned that the localhost inventory did not contain the CRI-O scenario's
hosts. No runtime tests were performed during these static checks, and no
dependency/environment changes were made. Subsequent tester runtime results
are recorded above.

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
legacy fallback), the tester's earlier `molecule test -s validation` runs exited
0 on Core 2.15.13, 2.20.8 and 2.21.3. All positive cases, exactly 24 expected
negative cases with exact failure assertions, the completeness check and all
four platform-specific SELinux assertions passed. These platform assertions
validate defaults; they are not four-platform runtime verification.

The historical current-lane command at `856d417`
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

### Final focused retest for the negative-test repair

At `0829cb9` plus working-tree test repairs, the tester confirmed
`molecule test -s validation` exited 0 in all three shared lanes. Each passed
24 API-port and three restored Helm/reference/cleanup exact rejections, both
completion assertions and four platform-default assertions:

| Shared lane | Core | Verify recap |
|---|---|---|
| `ansible-min` | 2.15.13 | ok=368, changed=0, failed=0, rescued=27 |
| `ansible-current` | 2.20.8 | ok=396, changed=0, failed=0, rescued=27 |
| `ansible-next` | 2.21.3 | ok=396, changed=0, failed=0, rescued=27 |

Serialized `ansible-min` and `ansible-next` runs of
`molecule test -s kvm-containerd --platform-name k8s-containerd-rocky9` both
exited 0. Each reported convergence ok=71/changed=27, idempotence
ok=61/changed=0, and verification ok=16/changed=0. Destruction succeeded and
VM, disk and key cleanup was confirmed for both runs. The historical
current-lane Rocky 9 KVM pass at `856d417` was retained without rerun because
role implementation was unchanged at that assessment. No other matrix entries
or full suite were run in that earlier retest. These results strengthen focused
compatibility evidence, not broader
lifecycle/platform coverage; the maturity score remains 52% for the reasons in
[MATURITY.md](MATURITY.md).

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
guests on libvirt's default network. The active matrix requires only Rocky
installation sources.

The environment owner must provide authorized matching Red Hat BaseOS and
AppStream content, including `container-selinux`, before restoring RHEL runs. Repository
IDs can differ under Satellite, RHUI or EUS; no fixed IDs are enforced here.
The role and shared KVM launcher do not register guests or provision those
repositories. RHEL runtime tests are temporarily excluded; the exact guest
failure has not been diagnosed. Do not substitute foreign packages or disable
SELinux. The tester owns RHEL diagnostics and subsequent runtime verification.

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
