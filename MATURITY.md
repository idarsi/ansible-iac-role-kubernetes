# Maturity assessment

Evidence updated 2026-11-15: `update/consolidated-local-kubernetes`, based on
`0829cb9` plus the focused negative-test working-tree repairs. The scorecard
retains the prior assessment's scores for the criterion-specific reasons below.
Scope is the existing README contract, not just API-port validation.

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
is retained without rerun because role implementation is unchanged.
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
| Functional: core convergence | 4 / 8 | Fresh Rocky 9 containerd baseline and historical Rocky 10 evidence; CRI-O, cluster and application coverage not freshly demonstrated. |
| Functional: lifecycle | 3 / 6 | Component and uninstall tasks/scenarios exist; full transition results unavailable. |
| Functional: platform/dependencies | 1.5 / 3 | Repository/runtime handling implemented; RHEL subscription/repository automation unverified. |
| Functional: failure/rerun | 1.5 / 3 | Assertions, creates guards and ownership checks; interruption/recovery evidence incomplete. |
| **Functional subtotal** | **10 / 20** | |
| Safety: preflight | 3 / 6 | Dedicated validators before mutations; 27 exact negative cases passed three lanes, but some normalization still precedes type validation and broader schema coverage remains incomplete. |
| Safety: secure behavior | 3 / 6 | Enforcing SELinux, GPG repositories, Helm checksums and no_log join token; comprehensive security verification unavailable. |
| Safety: destructive guardrails | 2.5 / 5 | Etcd/Helm/SELinux ownership markers and path checks; full preservation and cleanup execution evidence missing. |
| Safety: check/diff | 0 / 3 | No representative check/diff evidence or complete documented limitations. |
| **Safety subtotal** | **8.5 / 20** | |
| Convergence: idempotence | 2.5 / 5 | Fresh Rocky 9 containerd run reports zero changes on repeat; historical Rocky 10 evidence, not all claimed workflows. |
| Convergence: transitions | 2 / 4 | Lifecycle scenarios exist; comprehensive current results unavailable. |
| Convergence: recovery | 2 / 4 | Handlers and reset/ownership ordering implemented; recovery paths incompletely verified. |
| **Convergence subtotal** | **6.5 / 13** | |
| Tests: static quality | 3 / 3 | Reported base syntax/lint pass on three lanes; focused changed-file checks recorded for this change. |
| Tests: input validation | 2 / 4 | Validation passed all three lanes: positives, 27 exact negatives, both completeness checks and four platform-default assertions; broader schema and comprehensive validation-only non-mutation evidence remain incomplete. |
| Tests: functional verification | 3 / 6 | Fresh Rocky 9 baseline passed observable package, service, node, SELinux and configuration verification; wider matrix lacks current results. |
| Tests: idempotence | 2 / 4 | Fresh Rocky 9 automated idempotence passed with zero changes; incomplete feature coverage. |
| Tests: lifecycle/guardrails | 2 / 4 | Executable lifecycle/cleanup scenarios; missing comprehensive runtime results. |
| Tests: supported matrix | 1.5 / 3 | Fresh Rocky 9 and historical Rocky 10 results; RHEL 9/10 unverified. |
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
The current-lane Rocky 9 KVM pass at `856d417` is retained without repetition
because role implementation is unchanged.
The explicit partial scores above replace unsupported full-contract credit;
they do not claim that the compatibility fix regressed runtime functionality.

The final retest refreshes evidence without changing criterion scores. Input
validation and preflight safety retain 0.5 multipliers: the repaired negatives
are confirmed, but broader schema and comprehensive non-mutation coverage remain
incomplete. Core convergence, functional verification and both idempotence
criteria retain 0.5 multipliers: min/next add Ansible compatibility evidence for
the Rocky 9 containerd baseline, not the remaining runtimes or workflows.
Platform/dependency handling and supported-matrix criteria remain partial:
Rocky 10 evidence is historical and RHEL remains unverified. No new lifecycle,
recovery, check/diff or CI-enforcement evidence was supplied. The total remains
52.5/100, rounded down to 52%, with Alpha determined by the numeric band.
Successful test-environment destruction is not role-uninstall coverage.

## Smallest next steps and RC blockers

1. Add missing schema, comprehensive validation-only non-mutation and check/diff
   coverage; the 27 existing negative cases now pass all three lanes.
2. Verify lifecycle, preservation, CRI-O, cluster and application scenarios;
   verify RHEL with subscription-backed repositories or narrow support claims.
3. Complete the input/state reference, recovery guidance and contribution
   environment instructions; establish reproducible CI and release evidence.

No production-ready, Release Candidate, Stable or Mature claim is made.
