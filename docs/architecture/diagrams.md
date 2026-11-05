# Architecture and workflow diagrams

> These Mermaid diagrams illustrate **proposed** architecture. GitHub renders Mermaid in Markdown; text descriptions remain the source of truth for accessibility.

## Product experience

~~~mermaid
flowchart LR
  Install["Install signed binary"] --> Login["Sign in / pair device"]
  Login --> Marketplace["Browse and select open-source projects"]
  Marketplace --> Limits["Set budgets, models, trust, security"]
  Limits --> Auto["Enable automated contributions"]
  Auto --> Review["Independent reviews, QA and scans"]
  Review --> Value["Accepted PRs, badges, notifications"]
  Auto -. "pause / revoke" .-> Limits
~~~

## Responsibility split

~~~mermaid
flowchart TB
  People["Donors, planners and maintainers"] --> UI["Marketplace + account UI"]
  UI --> Control["Control plane: project, task graph, policy, scheduler"]
  Control --> DB[("PostgreSQL + durable outbox")]
  Control <--> Device["Local Rust daemon: policy, journal, kill switch"]
  Device --> Runner["Trusted generic runner adapter"]
  Runner --> Sandbox["Isolated execution: untrusted repository"]
  Device --> Evidence["Receipts + bounded artifact"]
  Evidence --> Review["Independent QA / security / model review"]
  Review --> GH["GitHub App: candidate PR / protected merge"]
  Control -. "redacted events" .-> Observability["Optional Warboard"]
~~~

## Task authority and evidence

~~~mermaid
stateDiagram-v2
  [*] --> Draft
  Draft --> Ready: Plan approved / dependencies accepted
  Ready --> Active: Atomic claim and lease epoch
  Active --> AwaitingReview: Candidate sealed and submitted
  Active --> Blocked: Needs attention / policy violation
  Active --> Ready: Lease expired; fenced reassignment
  AwaitingReview --> Accepted: Exact commit + independent evidence + merge gate
  AwaitingReview --> Ready: Rework as linked revision
  Blocked --> Ready: Authorized resolution
  Active --> Cancelled: Authorized revoke
  Accepted --> [*]
  Cancelled --> [*]
~~~

## Security gates

~~~mermaid
flowchart LR
  Catalog["Moderated project"] --> Admission["Trust + limits + model admission"]
  Admission --> Isolation["Isolated execution and local kill switch"]
  Isolation --> Intake["Validate artifact / secrets / static scans"]
  Intake --> QA["Independent tests + review"]
  QA --> Merge{"Policy + exact head/base valid?"}
  Merge -->|"Yes"| Publish["Protected PR acceptance"]
  Merge -->|"No"| Rework["Block / remediation / human approval"]
  Rework --> QA
~~~

## Failure containment

~~~mermaid
flowchart TB
  Claim["Task claimed with attempt + epoch"] --> Run["Local execution under resource reservation"]
  Run -->|"report/renew"| Live["Coordinator checks epoch, lease, identity"]
  Live -->|"current"| Run
  Live -->|"expired / revoked / stale"| Fence["Reject authoritative writes"]
  Run -->|"local crash / network partition"| Stop["Stop or quarantine children; preserve journal"]
  Fence --> Recovery["Preserve late artifacts, never auto-accept"]
  Stop --> Recovery
  Recovery --> Reconcile["Reconcile cost, revoke authority, safe retry"]
~~~

## Reading order

For behavioral details: [contributor journey](../vision/contributor-journey.md). For service contracts: [system architecture](system.md). For threats/scanners: [security model](../security/threat-model.md). For unresolved alternatives: [brainstorming](../brainstorming/2026-11-05-design-notes.md).
