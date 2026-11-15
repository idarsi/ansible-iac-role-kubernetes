# Maturity assessment

Matrix-only update 2026-11-15: `update/temporarily-exclude-rhel-tests`, based on
`1f32342`. RHEL 9/10 are temporarily excluded from executable runtime tests;
Rocky 9/10 remain. The production supported-OS contract is unchanged. Synthetic
RHEL validation facts establish defaults coverage only, not runtime evidence.
No new runtime results are claimed. All criterion scores below remain unchanged:
the supported-matrix criterion was already partial because RHEL was unverified,
and removing test entries does not close that gap. RC readiness remains
52%/Alpha; prior focused runtime results remain historical evidence.

Evidence updated 2026-11-15: `update/consolidated-local-kubernetes`, based on
`8919b8c` plus the scoped working-tree repairs. The scorecard
retains the prior assessment's scores for the criterion-specific reasons below.
Scope is the existing README contract, not just API-port validation.

Subsequent scoped repairs on this branch install `tar`/`gzip` before Helm
extraction, correct the uninstall result's loop-variable lookup, and restore
the `kvm-crio-component-absent` blueprint nesting and GPG URL. All nine targeted
Rocky 9 combinations now pass: application, uninstall and CRI-O component absence
on Core 2.15.13/Molecule 6, Core 2.20.8/Molecule 26.8 and Core 2.21.3/Molecule 26.8.
The prior min application pass was retained; eight follow-up runs completed
the matrix. Application reported converge ok=270/changed=44 and verify
ok=5/changed=0. Uninstall reported converge ok=154/changed=44 and verify
ok=10/changed=0: all 16 managed paths absent, unmanaged CNI preserved across
two uninstalls, initial sysctl reload and repeated skip passed. CRI-O reported
control-plane converge ok=198/changed=35 and worker ok=115/changed=22; verify
was ok=3/changed=0 on each host. Every command exited 0, with no failed or
unreachable hosts; owned VM/disk/key cleanup succeeded and unrelated resources
were untouched. The earlier uninstall verifier failure is superseded.
Application repeat-converge assertions passed, but no standalone Molecule
zero-change idempotence action ran. These are not full-suite results.
Readiness remains 52% for the criterion-specific reasons below.
RHEL `container-selinux` requires authorized matching BaseOS/AppStream access
managed by the environment owner; the exact guest failure remains unverified.
No repository provisioning or SELinux weakening is included.

**RC Readiness: 52% (52.5/100, rounded down). Maturity State: Alpha.**
The numeric band determines Alpha; no additional cap lowers it. Historical
Rocky 9/10 containerd convergence, idempotence, and verification evidence is
retained. Final validation passed Core 2.15.13, 2.20.8 and 2.21.3, including all
positive cases, 27 exact negative failures (24 API-port and three restored
Helm/reference/cleanup cases), both completeness checks and four platform-default
assertions. Min reported ok=368; current/next reported ok=396, each with
changed=0, failed=0 and rescued=27. Serialized min/next Rocky 9 containerd KVM
runs passed convergence (ok=71, changed=27), idempotence (ok=61, changed=0),
verification (ok=16, changed=0) and destruction; VM, disk and key cleanup was
confirmed for both. The historical current-lane Rocky 9 KVM pass at `856d417`
is retained as historical baseline evidence, not a retest of these repairs.
Other platforms and the full suite were not rerun.
This is not evidence for all runtimes, states, or platforms. There is
no recorded independent production approval.

Each score uses the maturity rubric's 0, 0.5, or 1 multiplier.

| Category / criterion | Score / maximum | Evidence and limitation |
|---|---:|---|
| Scope: purpose and boundaries | 3 / 3 | README describes host, runtime, cluster and application responsibilities. |
| Scope: inputs and states | 2.5 / 5 | Defaults and dispatch exist; full typed blueprint reference and precise present/install distinction are incomplete. |
| Scope: support contract | 2 / 4 | OS and version defaults explicit; RHEL and dependency compatibility evidence incomplete. |
| **Scope subtotal** | **7.5 / 12** | |
| Functional: core convergence | 4 / 8 | Focused baseline plus three-lane application, uninstall and CRI-O component passes; full cluster/platform contract remains unverified. |
| Functional: lifecycle | 3 / 6 | Three-lane application, CRI-O component and repeated uninstall transitions passed; remaining lifecycle/platform combinations lack current evidence. |
| Functional: platform/dependencies | 1.5 / 3 | Repository/runtime handling implemented; RHEL subscription/repository automation unverified. |
| Functional: failure/rerun | 1.5 / 3 | Assertions, creates guards and ownership checks; interruption/recovery evidence incomplete. |
| **Functional subtotal** | **10 / 20** | |
| Safety: preflight | 3 / 6 | Dedicated validators before mutations; 27 exact negative cases passed three lanes, but some normalization still precedes type validation and broader schema coverage remains incomplete. |
| Safety: secure behavior | 3 / 6 | Enforcing SELinux, GPG repositories, Helm checksums and no_log join token; comprehensive security verification unavailable. |
| Safety: destructive guardrails | 2.5 / 5 | Three-lane owned-etcd removal and unmanaged CNI preservation passed; unmarked-etcd, stale-rule and broader ownership guardrails still lack complete runtime evidence. |
| Safety: check/diff | 0 / 3 | No representative check/diff evidence or complete documented limitations. |
| **Safety subtotal** | **8.5 / 20** | |
| Convergence: idempotence | 2.5 / 5 | Fresh Rocky 9 containerd run reports zero changes on repeat; historical Rocky 10 evidence, not all claimed workflows. |
| Convergence: transitions | 2 / 4 | Application, CRI-O component and repeated uninstall transitions passed three lanes; other states/platforms remain incompletely verified. |
| Convergence: recovery | 2 / 4 | Handlers and reset/ownership ordering implemented; recovery paths incompletely verified. |
| **Convergence subtotal** | **6.5 / 13** | |
| Tests: static quality | 3 / 3 | Reported base syntax/lint pass on three lanes; focused changed-file checks recorded for this change. |
| Tests: input validation | 2 / 4 | Validation passed all three lanes: positives, 27 exact negatives, both completeness checks and four platform-default assertions; broader schema and comprehensive validation-only non-mutation evidence remain incomplete. |
| Tests: functional verification | 3 / 6 | Baseline and nine targeted combinations passed observable checks, including 16 absent managed paths and preserved CNI content; wider matrix lacks current results. |
| Tests: idempotence | 2 / 4 | Historical baseline zero-change idempotence plus application repeat assertions; no standalone application zero-change action and incomplete feature coverage. |
| Tests: lifecycle/guardrails | 2 / 4 | Three-lane application, CRI-O component and repeated uninstall checks passed; comprehensive destructive/recovery/platform coverage remains incomplete. |
| Tests: supported matrix | 1.5 / 3 | Prior Rocky 9 and historical Rocky 10 results; RHEL 9/10 unverified and temporarily excluded from runtime tests, not the support contract. |
| Tests: CI | 0.5 / 1 | PR static/validation and scheduled KVM jobs configured; current run and branch-protection enforcement unavailable. |
| **Tests subtotal** | **14 / 25** | |
| Documentation: operator | 2 / 4 | Usage/safety examples exist; complete schema and state semantics remain incomplete. |
| Documentation: tests/limitations | 2 / 2 | TESTING distinguishes fresh, historical and configured coverage and documents remaining gaps. |
| Documentation: contribution | 1 / 2 | Blueprint change-impact rules exist; environment instructions remain stale. |
| Documentation: metadata/release | 1 / 2 | License and CI pins exist; role metadata and release/compatibility history incomplete. |
| **Documentation subtotal** | **6 / 10** | |

## Change from the earlier summary

The previous README reported 80%/Beta against `cfdd685` plus an unspecified
working tree, without a criterion-level scorecard. A numerical per-criterion
delta cannot be reconstructed. This assessment preserves its focused Rocky
runtime evidence, but does not extrapolate that evidence to the full contract.
The minimum-Ansible validation and verifier failures are now resolved with
three-lane runtime evidence. The older negative tests were bypassed by successful
validation's end_host; this working tree moves them into a separate play and
requires exact task/message failures and all three recorded identities in a
final play. Three-lane validation and min/next Rocky 9 KVM now pass at
`0829cb9` plus the working-tree repairs.
The current-lane Rocky 9 baseline KVM pass at `856d417` is retained as historical
evidence, separate from the final nine targeted combinations.
The explicit partial scores above replace unsupported full-contract credit;
they do not claim that the compatibility fix regressed runtime functionality.

The final retest refreshes evidence without changing criterion scores. Input
validation and preflight safety retain 0.5 multipliers: the repaired negatives
are confirmed, but broader schema and comprehensive non-mutation coverage remain
incomplete. Core convergence and functional verification retain 0.5 multipliers:
the nine combinations add meaningful application, uninstall and CRI-O component
evidence, but do not establish the complete cluster/platform contract.
Lifecycle, transitions, destructive safety and lifecycle/guardrail testing also
retain 0.5: selected reverse/repeated transitions and CNI preservation now pass,
but the remaining states and ownership failure paths are not fully exercised.
Both idempotence criteria remain partial: application repeat assertions are not
a standalone zero-change action or full-feature idempotence coverage.
Platform/dependency handling and supported-matrix criteria remain partial:
Rocky 10 evidence is historical and RHEL remains blocked, unverified and was not
retried. Recovery, check/diff and CI-enforcement evidence did not improve.
No criterion crosses from partial to complete under the rubric. The total remains
52.5/100, rounded down to 52%, with Alpha determined by the numeric band.
Role-uninstall evidence now comes from its managed-path, package/service,
preservation and sysctl assertions, not merely test-environment destruction.

## Smallest next steps and RC blockers

1. Add missing schema, comprehensive validation-only non-mutation and check/diff
   coverage; the 27 existing negative cases now pass all three lanes.
2. Complete remaining lifecycle, ownership-failure, cluster and runtime/platform
   coverage, including standalone feature idempotence; verify RHEL with authorized
   matching repositories or narrow support claims. The nine focused combinations
   do not close these wider blockers.
3. Complete the input/state reference, recovery guidance and contribution
   environment instructions; establish reproducible CI and release evidence.

No production-ready, Release Candidate, Stable or Mature claim is made.
