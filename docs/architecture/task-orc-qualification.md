# Task Orc qualification and orchestration evaluation plan

> **Evaluation proposal — 2026-11-05.** No independent benchmark, distributed safety test or production qualification has been performed in this planning work. Task Orc is an owner-maintained *private, currently proprietary and unfinished* prototype; the owner is open to potentially publishing it later. **Do not copy its source or change its visibility without a separate explicit release/license decision.**

## Baseline established from current owner-maintained repository

Task Orc has a Rust deterministic plan validator, readiness selector, task risk/model floor checks, session affinity and write-path ownership validation, structured run/evidence records, and a durable local metadata-only Warboard outbox. Native Codex/Claude plugin packages wrap it; host coordinators retain dispatch and verification authority.

The latest observed `main` commit was `f1dea662e755faa633cf4bbfbeb4534fc68ccc7a` (2026-09-12). Its Linux GitHub Actions run `34721929106` reported **success** with **61 tests** across CLI (13), core (31), Warboard (17), plus formatting, Clippy and release build. This is historical *reported CI evidence*, not a fresh local rerun, model execution, live Warboard integration, performance/cost test, Windows/macOS proof, or sandbox qualification.

Confirmed constraints in its documented contracts:
- Plan contract schema v1, 512-task bound, 1 MiB JSON bound, explicit model/runtime capability policy, graph and write-path validation.
- `ready()` deterministically selects eligible work within a local parallelism cap; it does **not atomically claim** a distributed lease.
- Separate task acceptance requires declared verification and independent-review *records*, but text identifiers/references are not authenticated external evidence.
- SHA-256 plan content digest is *not* an authorization signature; sessions/groups are affinity hints, not distributed locks.
- Warboard outbox is observation-only and has bounded local publication/acknowledgment semantics, not authoritative acceptance.

## Principle to preserve

**AI plans and solves engineering problems. Deterministic software coordinates and enforces them.** Model calls may generate architecture, task decompositions, tests and reviews. Once an approved task DAG exists, no LLM should be required for routine ready-task selection, reservations, retry decisions, dependency enforcement, limits, fairness or work admission.

Do not increase task count merely to maximize parallel agents. Measure **coherent deliverables**: one independently verifiable engineering outcome per task; isolate only where risk, review boundaries, required context, conflicting file ownership or meaningful parallelism justify a split.

## Layer 1: deterministic, reproducible stress and property testing

Build a fixed-seed, inspectable test harness against Task Orc **and a neutral baseline**. Proposed test matrix:

| Category | Cases | Invariants |
| --- | --- | --- |
| Graph | Linear, wide DAG, diamond, nested fan-in/fan-out, unknown dependency, duplicate node, cycles; 1 to 512 tasks | No invalid plan accepted; no prerequisite skipped; readiness deterministic |
| Ownership and sessions | Same path/overlapping directories, case aliases, same session group and conflicting assignments | Concurrent non-ordered writers rejected; session affinity never changes acceptance |
| Capability policy | Unsupported model/effort/runtime, tier violations, team/messaging absence, downgraded provider | No silent substitution; unsupported plans blocked |
| Evidence | Forged review reference, missing/failed/skipped check, mismatched plan digest, stale revision | Cannot claim approval on unauthenticated evidence; report provenance limitations |
| Scheduling | Multiple ready queues, narrow CPU/model budgets, resource exhaustion, priorities and starvation | Capacity bounds; fair scheduling; stable output under identical state |
| Input safety | Malformed/oversized JSON, extreme path nesting, symlinks/FIFOs and invalid Unicode path assumptions | Bounded memory/CPU, no hang, reject unsupported path behavior |
| Recovery | Lost process, crash before/after checkpoint, duplicate result, retry storm, cancellation and expiry | No duplicate authoritative assignment, lost acceptance or infinite execution in distributed extension |
| Distributed coordination | Concurrent claims from multiple devices, lease expiry, fencing, network partition, stale submit and restore | Exactly one current authoritative attempt, idempotent recoverable transitions |

The existing CLI alone cannot satisfy distributed-lease tests; those become **new requirements** for the Nexus Kratos coordinator, not retroactive failure claims about Task Orc's advertised scope.

Use fixed fixtures, deterministic seeds, test oracle/invariants, repeatable configuration, timeouts and controlled clocks. Distinguish tests executed today from planned tests. Record exact source commit, toolchain, hardware, command, elapsed wall time, CPU time, peak RSS, memory allocations, throughput/latency distributions and exit statuses.

## Layer 2: representative engineering trials

Use 2–3 reproducible, explicitly permitted open-source or synthetic project tasks spanning different risk profiles: straightforward service feature, multi-repo integration and security/concurrency-sensitive work. Freeze the baseline code, architecture, acceptance tests, model availability, budgets and review policy for each comparison. Trial variants:

A. **Task Orc-assisted deterministic planning/coordination** with coherent task briefs and dependency/session rules.
B. **Simple baseline coordinator** using equivalent models, budget and explicit dependency tracking but without Task Orc's richer contract selection.
C. Optional later: improved Nexus Kratos deterministic scheduler after its distributed protocol is implemented.

Measure **end-to-end** completion time, total API tokens/actual provider charges vs estimates/unknown, implementation and independent-review effort, retries, context reconstruction, missed dependencies, overlapping edits, first-pass acceptance, security findings, accepted output quality and reproducibility. Report confidence intervals or ranges over repeated trials; do not claim savings without measurements or substitute subscription quota guesses for money spent.

## Qualification gates and decision

1. **Correctness gate:** all graph/contract/acceptance negative fixtures rejected; no premature work admission.
2. **Safety gate:** bounded malformed-input behavior and no credential access, unauthorized filesystem/host effect or unbounded subprocess. Execution sandbox is qualified separately.
3. **Distributed gate:** if adopted into Nexus Kratos, transactional claims, epoch fencing, authenticated receipts, crash/restore behavior and aggregate local reservations must pass multi-device fault injection.
4. **Performance gate:** publish 50th/95th/99th percentile latency and peak resource use for supported plan sizes. Thresholds must be established from measured candidate workload and hardware rather than fabricated numbers.
5. **Outcome gate:** compared with the baseline, Task Orc must demonstrate useful quality, reliability or cost benefit without regressions in mandatory safety; a win on token savings alone does not justify poorer software.
6. **Licensing and integration gate:** independently approve a public license, authorship/dependency scan and future import strategy *before* any private source is included in the Apache-licensed framework. Design principles may be implemented independently without copying the source.

Maintain a dated **qualification ledger** with exact commit digests, fixture and tool versions, results, reviewer independence, unresolved risks and yes/no recommendations. Until gates pass, Task Orc is a **candidate only**, not a selected or deployed Nexus Kratos subsystem.

## Implementation decision boundary

The coordinator should remain a deterministic service even if Task Orc is rejected. A future task may reuse its conceptual plan contract and evaluation methodology, but the exact interface/schema should be versioned for Nexus Kratos's immutable plan/architecture IDs, multi-repo scope and distributed policy.

Related: [planning architecture](planning-and-orchestration.md), [provider and knowledge interfaces](provider-and-knowledge.md), [platform implementation plan](../plans/2026-11-04-platform-implementation.md).
