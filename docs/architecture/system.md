# Proposed system architecture

> **Design proposal, 2026-11-05.** The current project has documentation and a static public website, not a deployed platform or released binary. This complements the existing ARCHITECTURE.md and the unmerged technical implementation-plan PR.

## Design principles

1. **Donor experience is GitHub-free.** GitHub is a delivery and governance integration behind the control plane, not a front-door UX.
2. **Trust boundaries are enforceable.** Local owner policy outranks task instructions and community preferences; remote coordinators do not receive provider credentials.
3. **Automated execution, governed acceptance.** A generated patch never self-certifies as safe or approved.
4. **Task leases, not PR locks.** Task revisions, attempts, monotonic lease epochs, idempotency and fencing are first-class.
5. **Transparent operations.** Display real progress, observable resource use and evidence; unknown usage is explicitly unknown.
6. **Modular and self-hostable.** Prefer one transactional control service to premature microservices, with a versioned protocol.
7. **Pluggable runners.** Keep the runner/harness choice open until compatibility, policy and security testing; do not hardwire a named runner.

## Logical components

~~~mermaid
flowchart TB
  subgraph Community["Community / public interface"]
    Catalog["Project catalog, ideas, badges"]
    Portal["Web portal / account / governance"]
  end
  subgraph Control["Control plane — proposed Rust service"]
    Auth["Identity / device enrolment / policy"]
    Plan["Planner and task graph"]
    Sched["Capacity-aware scheduler + lease fencing"]
    Gate["Reviews / security / acceptance engine"]
    Out["Outbox, audit, notifications"]
    DB[("PostgreSQL: transactional truth")]
    GH["Scoped GitHub App"]
  end
  subgraph Donor["Contributor machine"]
    Local["Local web UI + trusted Rust daemon"]
    Limit["Reservations / kill switch / device journal"]
    Adapter["Versioned runner/provider adapters"]
    Box["Isolated untrusted executor"]
  end
  Obs["Warboard / optional observability consumer"]
  Git["GitHub: issues / branches / PRs / merges"]
  Catalog <--> Portal
  Portal <--> Auth
  Auth <--> DB
  Plan <--> Sched
  Sched <--> DB
  Sched <--> Local
  Local --> Limit --> Adapter --> Box
  Local --> Out
  Out <--> DB
  Out --> Gate --> GH --> Git
  Out -. "redacted events" .-> Obs
~~~

### Control plane

- **Identity and project catalog:** accounts, contributors/devices, per-project trust/membership, proposals, review, approved catalog and recognition.
- **Planning:** architecture proposals and immutable plan/task revisions, dependencies, acceptance criteria, required capability/model tier and explicit approval to publish a task plan.
- **Scheduling:** fair opportunity across approved projects; match policy/capability and resource budgets; atomic claim, short lease, extension, cancellation, progress/stall tracking, recovery and stale-epoch fencing.
- **Review and GitHub integration:** validated artifact ingestion, attempt-scoped branch and PR publication, independent security/QA/code review, merge-policy evaluation, webhook reconciliation and immutable acceptance evidence.
- **Transactional state:** PostgreSQL for durable identity, policy, task, lease and audit state; transactional outbox for GitHub and notification side effects; no broker until justified.
- **Web:** TypeScript account, catalog, maintainer and dashboard interface consuming versioned APIs. Hosted portal should not impersonate the local resource-enforcement UI.

### Contributor plane

- **Signed Rust daemon:** installation/enrolment, version and device capability checks, updater policy, local web UI, remote protocol, local pause/revoke and safe shutdown.
- **Local supervisor and durable journal:** reserves CPU/memory/disk/concurrency/provider budgets across projects; persists uncertain attempts and crash recovery. No remote server can override local hard limits.
- **Runner/provider adapter boundary:** typed task contract and capabilities; pinned executable/model/version when applicable; verify unattended behavior, cancellation, identity and usage semantics. The choice between a common harness and several provider-native harnesses is deliberately deferred.
- **Executor:** isolated untrusted workspace with mount/egress/privilege quotas, timeouts, complete process-tree termination, and no direct access to daemon/provider/GitHub credentials.
- **Results:** bounded signed/digested artifacts, scan and test receipts, progress/checkpoints and auditable status. A contributor cannot merge code.

### GitHub and observability

The GitHub App uses only the permissions each project's owners explicitly grant. The coordinator publishes a candidate PR from a validated bounded artifact; checks and policies must bind to the exact commit and base. Maintainers keep final merge authority until safe automatic-merge policy is proven. A contributor's identity is represented in platform receipts and PR attribution without forcing a personal GitHub login.

Warboard remains independently developed and optional. Publish versioned, redacted events to an observability adapter; the core must function if that consumer is unavailable.

## Core contracts and failure guarantees

- **Device:** device identity + installation incarnation; revocation invalidates further authoritative writes. Server backups/restores issue a new incarnation to fence old devices.
- **Task:** immutable revision, project/repository, base commit, dependencies, acceptance criteria, paths, allowed models, resource profile and required execution/review policy.
- **Attempt:** task revision, device, lease epoch, deadline, expected base and idempotency key; one authoritative current execution attempt per revision.
- **Evidence:** artifact digest, exact candidate commit, toolchain, scan/QA results, independent reviewer identities, source timestamps, and actual vs estimated vs unknown usage.
- **Acceptance:** never inferred from a PR being open, CI being green, or an LLM assertion; verify policy, review independence and latest head/base at merge.

Network partitions, duplicate webhooks, local crashes, machine reboot, stale work after reassignment and delayed receipts are normal design cases, not exceptions. On uncertainty, fail closed for new work and preserve partial artifacts for reconciliation. Local resource controls still work offline.

## Proposed source boundaries

- apps/control — API, catalog, scheduler, state, governance, GitHub App and notification outbox
- apps/contributor — Rust daemon, local UI service, journal, adapters and isolated executor
- apps/web — hosted catalog/maintainer/dashboard UI
- packages/protocol — versioned cross-component schema and fixtures
- integrations/warboard — optional event/snapshot adapter
- tests/system — multi-device end-to-end and adversarial scenarios

These paths are **proposed**, not present as implementations today. For detailed implementation task graph and proofs, evaluate [PR #2](https://github.com/NexusKratos/nexuskratos/pull/2) and reconcile its sequencing with the newer marketplace/user-account onboarding goals before merge.

Related: [marketplace](../vision/marketplace.md), [journey](../vision/contributor-journey.md), [security](../security/threat-model.md), [diagrams](diagrams.md).
