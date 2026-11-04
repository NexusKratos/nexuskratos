# NexusKratos platform implementation plan

Dev mode: mvp (current repository default; no runtime exists).
Status: draft implementation design; documentation authorized 2026-11-04. Implementation explicitly on hold.
Written/updated: 2026-11-04. Owner: coordinating maintainer.
Execution rule: after a future implementation instruction, admitted tasks run back to back through their checks and close-out; this document alone authorizes no execution.

## Threat sketch

- Protect contributor machines, provider credentials, donated capacity, repository integrity, and community-private metadata.
- Entry points: web UI/API, GitHub webhooks, contributor protocol, repository content, task instructions, artifacts and provider output.
- Treat task authors, repositories, worker results and model output as potentially hostile, including inside a trusted community.
- Boundaries: browser/control service; community/community; coordinator/local supervisor; supervisor/executor; executor/provider; acceptance/GitHub merge.
- Authenticate remote access, authorize every object operation, validate inputs, and use parameterized queries and structured process arguments.
- Provider credentials remain local; repository code must not inherit host secrets, control credentials, or merge authority.
- Require durable fenced assignments, conservative capacity reservations, recoverable writes, and audit records without secrets.
- Real account data and credential-bearing runtime components require the sensitive-mode minimum and corresponding reviews before use; current documentation mode is not a runtime exemption.

## Product outcome and scope

A community converts a goal into dependency-aware tasks. Contributors choose trusted communities, compatible tools and enforceable limits. Work is assigned, executed, independently reviewed/tested, and accepted into GitHub with auditable evidence. The donor can pause or revoke participation locally at any time.

The first demonstration uses one community, one test repository and two distinct contributor identities. This is a validation fixture, not the final scope: multi-repository and multi-community operation, Codex and Claude Code adapters, planning/admin UI, contributor UX, self-hosting and Warboard integration remain required milestones below. No phase can be called the complete platform while these remain missing.

Anti-goals: a PR treated as a lock; a model claiming its own work passed; invented usage totals; a worktree advertised as a sandbox; secrets supplied to remote coordinators; a dependency considered complete before its accepted result exists.

Existing constraints: Apache-2.0 foundation; website stays separate; Warboard stays independently usable in its existing repository. No private Warboard source/history is read, copied or published by this plan. Federation between independently operated coordinators is a later separately designed capability, not required for multiple communities on one installation.

## Approach and planning decisions

Recommended: a modular control service with a durable transactional store and a separately distributed local contributor application. Prove the full assignment-to-acceptance path early, then expand product workflows. A UI-first approach hides execution risks until late. Independently deployed orchestration microservices add distributed coordination before workload evidence justifies them.

Retain the existing Rust recommendation for coordinator/contributor and TypeScript for the web interface. Propose PostgreSQL for authoritative coordination, with task transitions and an outbox committed together; no independent message broker required initially. Choose exact framework/crate versions after compatibility and dependency review in T01, not by inventing an existing toolchain. Use a machine-local durable transactional journal for reservations and recovery; T01 selects its implementation and migration contract.

Propose Linux as the first qualified executor backend, with explicit unsupported status elsewhere until their isolation/cancellation suites pass. T18 includes macOS and Windows feasibility and qualification work; no installer may imply equivalent guarantees without evidence. This sequencing is a planning assumption, not an owner decision to remove platforms.

Provider compatibility is a blocking feasibility dependency, not a promise: pin CLI/model/effort; verify unattended use, permitted authentication model, output formats, cancellation, accounting and applicable provider terms before enabling each adapter. Never silently substitute a tool or model. Record unsupported capabilities and blocked tasks visibly.

## Component and ownership boundaries

All paths below are planned, not scaffolded. The coordinator alone owns root manifests, integration, version allocation and release records, and approves/integrates shared protocol changes. The designated protocol worker owns scoped edits to protocol schemas, types and fixtures; there is only one protocol writer at a time.

| Owner | Proposed boundary | Responsibility |
| --- | --- | --- |
| Protocol worker | `packages/protocol/` | Versioned schemas, generated Rust/TypeScript types, fixtures and compatibility tests |
| Control worker | `apps/control/` | Identity, policy, task graph, transactional scheduling, GitHub integration, acceptance and audit |
| Contributor worker | `apps/contributor/` | Local enrollment, journal, limits, execution supervisor, adapter processes and donor controls |
| Web worker | `apps/web/` | Planner/admin/contributor visibility; consumes protocol, never owns scheduling truth |
| Integration worker | `integrations/warboard/`, `tests/system/` | Read-only visibility adapter and end-to-end evidence |
| Coordinator | `docs/`, root manifests and release configuration | ADRs, task admission, reviews, compatibility matrix and close-out |

A Rust control process can serve the web assets; `apps/web` is a build boundary, not a mandatory extra deployed service. The contributor begins with a headless service and CLI; T15 supplies the donor-facing local interface and usable onboarding.

## Domain model and contracts

Use immutable identifiers for Community, Repository, Principal, ContributorDevice, PlanRevision, TaskRevision, Attempt, InstallationIncarnation, LeaseEpoch, Reservation, Artifact, Review, Acceptance and AuditEvent. Every scoped row carries its community identifier; repository membership and role grants are explicit. Never authorize a resource merely because its identifier is known.

A task revision records repository/base commit, dependency task-revision edges, bound acceptance IDs at admission, goal, allowed paths, acceptance commands, execution mode (`headless` or `human-assisted`), required capabilities, model constraints, limits and policy version. Changed requirements create a revision; they never silently mutate an active assignment. A draft dependency edge references an upstream TaskRevision before either task runs. On admission, bind that edge to its specific Acceptance ID and immutable result; a downstream task remains blocked until every edge has a valid binding. Never use issue/PR status as evidence. Reject cycles. Changes to an accepted prerequisite invalidate affected ready tasks and require a new explicit plan revision for already-running work.

Proposed protocol v1 operations (schemas and errors frozen by T02):

| Operation | Required inputs | Durable result / rejection |
| --- | --- | --- |
| enroll/revoke device | authenticated contributor, community grants, device public identity | scoped credential / revoked credential denied |
| offer/claim | device capabilities, task revision, idempotency key | attempt ID, monotonically increasing epoch, pinned base, lease deadline / incompatible, stale, unavailable |
| renew | attempt, epoch, progress sequence, requested extension | bounded new deadline / stale epoch, revoked, expired |
| report/checkpoint | attempt, epoch, sequence, content digest, evidence | append-only event/artifact reference / duplicate acknowledged, stale non-authoritative archive |
| submit | attempt, epoch, commit, artifact digests, execution receipt | immutable candidate / stale or invalid candidate rejected |
| review | separately assigned review attempt, candidate digest/commit, checks, verdict | immutable review evidence / wrong candidate or author identity rejected |
| accept | candidate, policy version, required evidence, expected repository base | acceptance record / missing evidence, changed base, revoked authority |
| attention/respond | attempt, incarnation/epoch, request ID, authorized respondent, response ID and deadline | durable response consumed once / expired, stale, unauthorized or duplicate |
| cancel | authorized actor, task/attempt, reason, idempotency key | cancellation intent and eventual stop receipt / denied |

Transport uses authenticated HTTPS request/response with polling and resumable event cursors first. Protocol carries major version, operation ID and error code. Unknown major versions fail closed. T02 fixtures define additive-field handling and exact size/rate limits. No serialized shell string is executed as a control instruction.

## State, fencing and recovery invariants

Task states: draft → ready → active → awaiting-review → accepted. Additional states: blocked, needs-attention, cancelled. Attempt states: offered → running → submitted; terminal alternatives: declined, failed, expired, cancelled. Review rejection creates a linked revision/rework attempt; it does not erase prior results. Readiness requires all dependency acceptances and compatible policy.

1. Claim and epoch increment are atomic in the authoritative store. At most one current execution attempt per task revision. Separate review tasks may run concurrently under their own leases.
2. Every worker-authoritative write checks installation incarnation, attempt, epoch, principal, task revision and current lease in the same transaction. Server time determines lease validity. An expired lease cannot renew; reassignment creates a new epoch.
3. Contributor computes a conservative local stop deadline using monotonic elapsed time and the received lease duration. Network loss never grants indefinite execution. Stop before lease uncertainty; reconnect/reconcile before resuming.
4. Heartbeats prove contact, not progress. Meaningful progress has a sequence, stage and evidence; repeated unchanged reports trigger a bounded stall policy.
5. Expiry fences new worker submissions immediately but cannot prove a remote process stopped. Replacement work and residual usage must remain separately recorded; preserve late artifacts without treating them as a current candidate.
6. Cancellation revokes further admission, stops the entire process tree, and reconciles usage/reservations only after termination is verified. A crash recovers journaled reservations conservatively; uncertain costs do not become zero.
7. An accepted result binds task revision, commit, base, policy, reviewer identities and evidence digests. New candidate commits invalidate approvals. A base change requires integration validation before merge.
8. DB transaction plus outbox handles external effects. Webhooks are authenticated, deduplicated and reconciled with GitHub state; retries must not create duplicate branches, PRs or acceptances.
9. Before serving after restore, generate a fresh cryptographically random installation incarnation that is never copied from the restored backup. Carry it in assignments, credentials and every worker write; reject prior incarnations even when epoch numbers match. Workers re-enroll/reconcile before writes. T03 must test repeated restoration of the same backup with old workers alive; an increment stored only in the restored database is insufficient.

Successful submission atomically verifies the live worker authority, seals the immutable candidate, retires the execution lease and enqueues coordinator-owned publication. Subsequent publication/review/acceptance uses current coordinator policy and its own authority, not the retired author lease. Author disconnect or subsequent lease time passage does not invalidate that candidate. Explicit task cancellation, new task revision or candidate replacement revokes eligibility while preserving evidence; replacement receives a new candidate ID and fresh reviews. Outbox processing rechecks eligibility before an external effect and reconciles any effect racing revocation.

Human-assisted requests carry a unique request ID, authorized respondent identity/role, bounded response deadline and expected incarnation/epoch. Enter needs-attention, stop active tool execution and retain its reservation until termination is confirmed; no provider process waits indefinitely for input. Response is authenticated, idempotent and consumed once. Resume only under a current lease, unchanged task revision and renewed local capacity admission; expired/stale responses are archived, never executed. Expiry requires a new attempt and an explicitly adopted safe checkpoint. Headless tasks fail with a structured needs-attention reason instead of silently becoming interactive. T02 fixes pause-time accounting and the deadline/lease interaction; T09 tests pause, response, expiry and cancellation races.

T02 fixes initial lease/renewal/stall/retry defaults and their permitted ranges in the protocol fixtures; no dependent implementation proceeds with unspecified timing semantics. T04 tests boundary instants using a controlled clock, including renewal racing expiry.

## Local trust, credentials and limits

Separate a trusted supervisor/provider adapter from the untrusted repository executor. The supervisor holds device/provider credentials in OS-protected local storage and never mounts its home or credential store into repository execution. Tasks cannot override the selected trust profile, tool paths or security controls. GitHub publication credentials stay in the control service; it imports a validated, bounded patch/bundle from the contributor and publishes an attempt-specific branch. Import is untrusted input: reject unsafe paths, hooks and unauthorized refs; do not run repository hooks during import.

T01 must demonstrate whether each provider CLI can operate across this boundary without giving repository commands its credential store. Evaluate a brokered provider path or isolated per-task credential context where needed, with an explicit residual-risk analysis. If a CLI cannot meet the promised boundary, that adapter is blocked; never relabel a worktree or credential-containing container as safe. No real provider credentials in feasibility fixtures.

The executor gets only approved workspace mounts and explicit network destinations, resource quotas and a bounded lifetime. No host Docker socket, ambient SSH agent or inherited control-service environment. Dependency fetch is an explicit policy step; unrestricted egress cannot be presented as an exfiltration barrier. Isolation backend selection and escape tests precede live donated work.

One supervisor arbitrates all participating communities on a device. Reserve aggregate concurrency, runtime and measurable usage before launch, transactionally. Units remain separate: measured tokens/API charges, estimates, subscription allowance and wall time. Unknown provider usage is displayed as unknown. A provider without a reliable monetary ceiling cannot offer a guaranteed hard dollar limit; users can select enforceable time/concurrency limits or decline that adapter. Reservations cover pending work and verified running children; reconcile after termination and late metering. T07 covers period rollover, reboot, duplicate receipts, concurrent claims and multiple local app instances. Cross-device account-wide ceilings are not guaranteed by a machine-local ledger; UI must say so.

## Review and merge authority

Contributor output is an untrusted assertion. A distinct authenticated contributor identity performs independent review; a new model session on the author's device alone is not proof of independence. Community policy records any stronger organization/model-family separation required. If an independent reviewer is unavailable, remain awaiting-review; do not self-approve. Independent execution is provenance, not a guarantee against collusion.

QA runs the declared commands against the pinned candidate in an isolated clean environment and records toolchain/environment identity, exit status and raw bounded evidence. Model-written summaries are not test receipts. Merge requires authorized maintainer action initially, with the policy engine independently rechecking required evidence and exact head/base. Automation may later execute that same policy, but does not acquire approval bypass rights. Merged/accepted GitHub commit IDs and the reviewed candidate are both retained. Validate the candidate integrated onto the expected base before merge, and require a GitHub mechanism that rejects a changed head/base (or a protected merge queue with equivalent checks). If that mechanism cannot be demonstrated, automatic acceptance is blocked. A base advancing after validation requires revalidation; checking the result after merge is additional evidence, never a substitute for the pre-merge gate.

## Milestones and task graph

Every task is planned, none started. `→` in Next means eligible only after the listed dependencies and required review pass. Owners refer to the boundaries above. Each task's evidence is a required future receipt, not a check claimed today.

| ID / milestone | Owner and files | Dependencies | Deliverable and acceptance evidence | Next |
| --- | --- | --- | --- | --- |
| T01 / M0 feasibility | Coordinator; `docs/adr/0004-*`, `docs/compatibility/` | Future implementation authorization | Pin toolchain/dependency choices; credential-isolation, cancellation and usage capability matrix for both providers; choose executor and journal; benign fixture evidence; fail closed on unsupported guarantees | T02 |
| T02 / M0 contracts | Protocol; `packages/protocol/`, `docs/protocol/` | T01 | Schemas and generated types; complete transitions/errors/timing/size limits; compatibility fixtures reject stale epochs, invalid revisions and unknown major versions | T03,T06 |
| T03 / M1 control identity | Control; `apps/control/identity`, `storage`, migrations | T02 | Community roles, device enrollment/revocation, transactional schema; cross-community authorization tests, migration/restore fixture | T04 |
| T04 / M1 coordinator | Control; `apps/control/scheduler` | T03 | Dependency readiness, atomic claims, renew/expiry/cancel and replay; races prove one current epoch; restart preserves decisions; no network calls inside DB lock | T05,T08 |
| T05 / M1 GitHub publication | Control; `apps/control/github`, `outbox` | T04 | App-scoped publication, authenticated webhooks and reconciliation; duplicate delivery/retry/revocation fixtures; bounded artifact import; no worker merge token | T10 |
| T06 / M1 local supervisor | Contributor; `apps/contributor/supervisor`, `journal` | T02 | Enrollment, local policy and crash recovery with fake adapter; exclusive local ownership, no duplicate process after restart; revoke/lease timeout stops children | T07 |
| T07 / M1 budget admission | Contributor; `apps/contributor/limits` | T06 | Durable aggregate reservations and reconciliation; simultaneous communities cannot oversubscribe; unknown usage never reported as measured zero | T08 |
| T08 / M1 isolated executor | Contributor; `apps/contributor/executor` | T01,T04,T07 | Qualified backend enforces mounts/egress/resources and process-tree cancellation; secret/path/symlink escape fixtures; supervisor remains outside workload | T09 |
| T09 / M1 provider adapters | Contributor; `apps/contributor/adapters/{codex,claude}` | T08 | Separate adapter conformance receipts for both tools: pinned identity, headless completion, login expiry, malformed output, input request/pause/resume/expiry races, timeout and cancellation; no silent fallback | T10 |
| T10 / M1 review/acceptance | Control; `apps/control/reviews`, `acceptance`; system tests | T05,T09 | Two contributor identities complete task→PR→independent review/QA→maintainer acceptance; stale commit/base, base-advance-during-merge and self-review rejected; author disconnect after valid submission preserves reviewability; failure recovery demo | T11,T13 |
| T11 / M2 planning workflow | Web + control sequential API ownership; `apps/web/plans`, `apps/control/plans` | T10 | Human-editable plans, task graph, role/capability constraints and revision preview; cycle detection; AI suggestions never authorize execution until planner publishes revision | T12 |
| T12 / M2 multi-repository work | Control; `apps/control/dependencies`, system fixtures | T11 | Goal spans two repos; downstream task pins accepted upstream artifact; partial failure/rework represented; no claim of atomic cross-repo merge | T14 |
| T13 / M2 multiple communities | Control + contributor with fixed protocol; membership/fairness modules | T10 | Role revocation, admission quotas and fair scheduling; one contributor spans two communities under one aggregate ledger; no cross-community data or authority leakage | T14 |
| T14 / M2 administration | Web; `apps/web/admin`, task/evidence views | T12,T13 | Repository onboarding, roles, policy changes, pause/cancel, review queues, reasons for blocked work, redacted audit export; keyboard/accessibility and permission tests | T15,T16 |
| T15 / M2 contributor experience | Contributor; local UI/onboarding/status modules | T14 | Choose communities/tools, limits and schedules; explain permissions; pause/revoke with stop receipt; needs-attention flow with deadline; accessible onboarding usability scenarios | T17 |
| T16 / M2 Warboard integration | Integration; `integrations/warboard` | T14 | Versioned read-only events/snapshot contract; replay/dedup/redaction tests; integration fixture and real compatible consumer verification; core works when Warboard is absent | T17 |
| T17 / M3 self-host operations | Coordinator + integration; `deploy/`, `docs/runbooks/`, system tests | T15,T16 | Install/upgrade/backup/restore/revoke walkthrough; restore invalidates stale work; TLS/secrets setup, metrics, retention/export and bounded log storage; no secret in export | T18 |
| T18 / M3 platform qualification | Contributor + integration; platform backends, `docs/compatibility/` | T17 | Linux qualification plus macOS/Windows feasibility and backend work; each supported platform runs identical isolation/cancellation/recovery suite; unresolved platform blockers remain explicit | T19 |
| T19 / M3 release qualification | Coordinator; release records/artifact manifests | T18 | Exact-candidate full checks, dependency review, installer provenance/checksums, high-deep reviews, pilot evidence and documented support matrix; release only if required scope and blockers resolved | Phase release |

M0 is a decision checkpoint, not a runnable product. M1 demonstrates the complete trusted contribution loop. M2 demonstrates community usefulness across repositories and contributors. M3 demonstrates recoverable self-hosting and a qualified distribution. No schedule estimate is implied by task count.

## Execution batches, reviews and communication

Future batches: B0 T01–T02; B1-control T03–T05; B1-local T06–T09; B1-integration T10; B2-planning T11–T12; B2-community T13–T14; B2-experience T15–T16; B3-operations T17–T18; B3-release T19. Small batches are deliberate at security/protocol/storage boundaries. Risk-class tasks receive deep review before dependent work; split a batch further when its changed-line/risk threshold requires it. Within a batch, dependent work can be provisional only if the governing coordinator permits it; accepted prerequisites always mean required reviews settled. Task Orc is not selected or invoked by this document.

After T02 is reviewed, control and contributor workers may proceed independently against the frozen protocol. Shared schema/root files have one writer. T11/T13 may run in parallel with distinct file ownership; integrate their common API changes sequentially. T15/T16 are separate consumers after T14. Never parallelize by giving two workers the same files.

Use native scoped subagents for future separable work and reviews, at most two implementation writers initially; reserve review capacity. Each handback names candidate, changed files, commands/results, unresolved findings and next dependency. Native messages report progress, not acceptance. If unavailable, run sequentially and state the limitation. No external messaging is implied. The coordinator maintains a review ledger with base/target hashes, requested/observed model, findings, fixes and final ruling. Never claim blind review from inherited context.

Apply current dev-mode review cadence, raising the effective floor for real sensitive data/credentials. Mechanical and deep checks close each batch; protocol/storage/security blockers hold their dependents. Milestone reviews follow the applicable mode; release review always gates release. Keep no more than three unruled batches per line, including inherited work; merge considers the union. Freeze each release candidate and restart affected checks after changes.

## Validation and release evidence

For each task: define the failure fixture first, demonstrate it fails, implement within ownership, run focused checks, then record exact candidate and commands. At each integrated boundary run the broad suite once on the frozen candidate; do not present reused receipts as independent execution. T01 selects actual toolchain commands, T02 establishes runnable commands; no command listed here is claimed to exist today.

Required system scenarios: simultaneous claim; renewal/expiry race; coordinator crash before/after commit; webhook duplicate/out-of-order/loss; worker network partition and clock jump; entire process-tree kill; credential/login expiry; budget rollover and unknown usage; cancellation during submission; old result after reassignment; changed candidate/base after review; malicious artifact paths; multi-community denial; repeated database restore with old workers alive; author disconnect after successful submit; human-assisted stale response; Warboard unavailable.

Evidence records distinguish source checks, simulated adapter checks, real provider runs, independent QA, accepted GitHub commits, installed artifacts and live deployment. Use test repositories/accounts with explicit scope for live checks; no paid calls are authorized by this planning request. Define concrete latency/capacity targets from a measured pilot workload before T19; record workload and environment rather than promise unsupported throughput.

Future task close-out: update docs, version, checkpoint and protected PR according to CONTRIBUTING. Future phase release is separate: all required reviews/evidence, advisory disposition, merge commit, release version/artifacts and install verification. This planning change produces only a documentation checkpoint, no runtime release.

## Decision register and remaining gates

| Decision/gate | Planning position | Evidence required before dependent implementation |
| --- | --- | --- |
| Control storage | PostgreSQL + transactional outbox proposed | T01 architecture ADR; T03 concurrency/migration evidence |
| Provider credential isolation | Trusted adapter outside repository executor | T01 per-provider feasibility and T08 adversarial fixtures; block unsafe adapter |
| Local hard budget | Only promise enforceable units; machine-local aggregate | T01 meter semantics and T07 crash/overshoot tests |
| Runtime data classification | Current docs mvp; sensitive minimum for real identity/credential data | T03 data inventory, minimization and applicable review cadence |
| Contributor platforms | Linux first; other platforms explicitly qualified | T18 support matrix, unresolved gaps cannot silently disappear |
| Review identity | Different contributor identity, exact candidate, policy-recorded independence | T10 authorization and provenance evidence |
| Warboard compatibility | Consumer of public versioned contract; no source migration | T16 compatible consumer evidence; unavailable integration remains blocked |
| Framework/version selection | Not installed or selected by a documentation-only task | T01 supported versions, license/advisory/maintenance review |

## Sources and provenance

Baseline: ARCHITECTURE.md, README.md, existing PLAN.md, PROGRESS.md, HANDOFF.md and ADRs 0001–0002 at repository commit `014decc6d1ca8c04ba860ceaedc545283edab397`. New choices above are planner proposals, not retroactively attributed to the owner.

Official references consulted 2026-11-04: [GitHub webhook guidance](https://docs.github.com/en/webhooks/using-webhooks/best-practices-for-using-webhooks) supports authenticated, deduplicated, asynchronously processed delivery; [GitHub installation authentication](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/authenticating-as-a-github-app-installation) describes installation-scoped permissions and expiring tokens; [PostgreSQL locking](https://www.postgresql.org/docs/current/explicit-locking.html) documents row locks available for transactional claim serialization. These support design mechanisms, not evidence that NexusKratos implements them.
