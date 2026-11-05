# Planning, architecture, and deterministic orchestration

> **Design proposal, 2026-11-05.** This is a specification target, not a shipped planner, approved architecture engine, released daemon, or authorization to execute tasks.

## Ownership and process boundaries

Nexus Kratos provides a self-hostable workflow and safety framework; it does **not** design or govern community-owned applications on their behalf. The project appoints its own architecture group, selects its own consensus/voting/approval procedure, and remains responsible for requirements, technology choices and source repositories. An official Nexus Kratos registry determines whether that project meets published *execution eligibility* criteria. Independent self-hosted registries may choose different admission policies; contributors retain local security authority everywhere.

Three independent responsibilities:

- **Project architecture group:** owns requirements, design and project-specific approval process. Multiple architects may collaborate on one authoritative architecture revision. Shared components across projects require compatible requirements and a single coherent design for that component, not competing instructions.
- **AI planning/review contributors:** voluntarily supply high-capability models (or qualified operator-provided paid capacity) to propose architecture, decompose approved goals, challenge assumptions, review feasibility and detect missing dependencies. Distinct reviewers must be independent from authors and the project under the configured assurance policy.
- **Deterministic coordinator:** owns approved revision references, task eligibility, task DAG, current attempts, leases, resource admission, priority, fairness, process state, check receipts and audit records. The coordinator must never need an LLM just to select the next eligible task.

A user may submit an existing architecture or plan, author one manually, or use donated/paid AI planning; all three paths enter the *same* versioned validation and approval pipeline. Neither an AI-generated plan nor a project owner's submission is authorization to dispatch tasks.

## Mandatory design and build gates

~~~mermaid
flowchart TB
  R["Project proposal, owner verification, repository scopes"] --> A["Requirements + immutable architecture revision"]
  A --> AR["Multiple independent architecture reviews"]
  AR --> D{"Blocking findings resolved?"}
  D -->|"No"| A
  D -->|"Yes"| G["Project governance approves revision"]
  G --> P["Plan and coherent task decomposition"]
  P --> PR["Independent plan review + deterministic validation"]
  PR --> E{"Approved task graph and security admission?"}
  E -->|"No"| P
  E -->|"Yes"| T["Distributed deterministic coordinator"]
  T --> B["Opted-in qualified builder daemons"]
  B --> Q["Candidate-bound QA, scans and independent code review"]
  Q --> X["Policy-gated acceptance / protected PR"]
~~~

Project admission, architecture validation, task admission, builder-machine execution safety and merge acceptance are distinct decisions. Architecture review evidence must bind the exact revision and review policy; every implementation task binds a specific accepted architecture revision and plan revision.

Substantive architectural or dependency changes trigger *change impact assessment*: identify affected components, tasks, repositories, checks, SBOMs and dependent results; suspend and revalidate **affected** work while independent unaffected tasks can proceed. Never silently update a running task's requirements.

## Planning output contract

A plan should carry: immutable ID/revision/digest; requirements and approved architecture references; objective and non-goals; project and authorized repository/base; dependencies; task owner/allowed paths; required model/provider capability; effort/context; task size rationale; security/risk tier; allowed parallelism and session affinity; limits/usage budget; expected inputs/outputs; acceptance tests, independent review floor; checkpoint/retry behavior; and versioned change-impact links.

**Coherent-task rule:** decompose into independently verifiable engineering outcomes, not token-sized fragments. Split when ownership boundaries, independent acceptance, external dependencies, risk isolation, or parallel scheduling justify the coordination cost. Merge tasks that share the same context, files and verification boundary when parallelism would increase conflicts/rework. A task may share an agent session with related tasks while still keeping distinct evidence and acceptance.

Deterministic validation checks DAG acyclicity, unknown dependencies, forbidden path overlaps, immutable revision binding, unsupported model effort, review-depth floors, capacity/concurrency ceilings, and all required receipts. A planner's natural-language explanation does not replace these checks.

## One daemon with voluntary specialized roles

The same distributed contributor runtime can expose **Architect**, **Planner**, **Independent Architecture Reviewer**, **Independent Plan Reviewer**, **Builder**, **QA/Security Reviewer**, and non-AI scanning/testing roles. Enrollment and usage limits are per-role, per-provider and per-project; users are not required to donate AI at all. Planning/review donors may supply more expensive high-reasoning models than builders. Paid operator-funded work must meet the same quality and independence requirements as the free reviewer pool.

Planning intelligence runs via a versioned provider adapter; provider secrets remain in the trusted local supervisor, never in arbitrary repository execution. The coordinator validates results, tracks cost and authority, and creates tasks; it never delegates policy authorization to model text. Provider subscriptions and API products are distinct and require a provider-specific permitted integration path.

## Task Orc as a *candidate*, not an adopted dependency

Task Orc is an owner-maintained Rust prototype with a **deterministic** plan validator, task-readiness selector, role/model/effort policy checks, dependency and write-path conflict detection, session-group controls, run/acceptance record checks, and a local Warboard observation outbox. Its existing Linux CI tests show a useful baseline, **not** production readiness, distributed leases, authenticated evidence, sandbox enforcement, provider execution, or performance/cost advantage.

The owner is open to potentially making Task Orc public, but its source is **currently private and proprietary**. This public architecture record deliberately contains **no copied Task Orc source** and grants no license. A future open-source release, code import or derived work needs an explicit license/provenance and integration decision. Nexus Kratos can independently implement the design principles in its own public framework.

Task Orc's `ready()` behavior is a *proposal*, not an atomic reservation. The distributed coordinator must add durable transactional claims, per-device leases and epochs, secure work assignment, idempotent reconciliation, multi-community fairness, cross-repo authority, crash recovery, resource reservations and authoritative signed check/review evidence. Local outbox receipts are transport bookkeeping, never independent proof of successful remote operation.

**Do not select Task Orc solely on its conceptual alignment.** Qualification against a baseline, adversarial fixtures and representative software projects is mandatory. See [evaluation plan](task-orc-qualification.md).

## Further specifications and open questions

- Decide architecture/plan schema versioning, immutable identifiers, reviewer identity/independence, and certification trust policy before authoring working services.
- Decide fairness, priorities, scheduling algorithm and affinity limits based on benchmarks, not assumptions that more parallel agents means faster delivery.
- Define paid/donated contributor model-usage accounting with actual/estimated/unknown units; no false hard caps for unmetered subscriptions.
- Keep single generic harness versus provider-native harnesses **open**. Provider capability qualification and safe isolation precede enabling either.
- Define project-level policy for human acceptance, critical security non-waivability and independently verifiable audit evidence.

Related: [implementation plan](../plans/2026-11-04-platform-implementation.md), [provider/knowledge strategy](provider-and-knowledge.md), [review pool](independent-architecture-review.md), [security controls](../security/threat-model.md).
