# Architecture direction

This is a proposed product boundary, not an implemented system.

## Components

- **Control application:** community planning, administration, repository membership, acceptance policies, and contributor visibility.
- **Coordinator:** dependency-aware task selection, assignment leases, deadline extensions, stale-attempt fencing, and independent review routing.
- **Contributor application:** locally enforced limits, trusted-community selection, isolated task execution, CLI adapters, and checkpoint reporting.
- **GitHub integration:** issues describe work; attempt-specific branches and draft pull requests hold contributions; accepted commits identify results.
- **Warboard:** independently usable observability component under active development; planned integration presents tasks, agents, plans, logs, and usage.

## Important boundaries

A pull request is not an exclusive assignment lock. The coordinator must own an explicit lease with an attempt identifier. Late results from an expired attempt must never overwrite a current attempt. Review and QA evidence must identify the exact commit reviewed.

Contributors retain provider credentials locally. The local application must enforce aggregate limits across communities, reserve capacity for in-flight work, and distinguish estimates from measured use. Subscription quotas are not interchangeable with API dollar balances. Provider compatibility and terms must be verified when each adapter is implemented.

Untrusted repository tasks need explicit trust selection and execution isolation. A separate Git worktree alone is not a security sandbox. Assignment, execution, and merge authority are separate permissions.

The project is intended to support self-hosting and multiple repositories. No language, framework, datastore, hosting architecture, or cross-community federation implementation is selected in this foundation task.

The static website is maintained separately in [NexusKratos/website](https://github.com/NexusKratos/website). It does not execute tasks or hold provider credentials.

## Recommended source layout

Keep the initial platform in this repository, with separately buildable applications: `apps/control` for planning, orchestration, and administration; `apps/contributor` for local build/review/QA execution; and `packages/protocol` for shared contracts. These are proposed boundaries, not directories or applications created by this setup.

Separate application permissions and release artifacts do not require separate repositories. A monorepo lets task/protocol changes update both applications in one reviewed change. Reconsider a repository split when independent teams, restricted source access, or mature independent release schedules make the coordination cost worthwhile. The website remains a separate repository by owner request; Warboard remains independently usable and in its existing repository for now.

## Recommended implementation technologies

Rust is recommended for the contributor agent, CLI adapters, and coordinator; TypeScript is recommended for the planner/admin web interface. These recommendations are not an implementation or a dependency selection. Rust suits local executable distribution and explicit failure handling, but does not replace process isolation, authorization, or cancellation design. The initial website stays plain static HTML/CSS/JavaScript.

## Recommended execution contract

Task execution should be headless by default. Each task declares `headless` or `human-assisted` execution, and each adapter must prove unattended operation and detectable completion. Provider login may require interactive onboarding; it must not become an invisible prompt inside an unattended task.

A task needing input enters an explicit needs-attention state with a reason and deadline. Cancellation, timeouts, and budget accounting must cover subprocesses. Retries are bounded; partial work is preserved across lease expiry and reassignment. Report current step, usage, logs, last meaningful progress, and deadline to the visibility layer. A heartbeat alone does not establish progress or success.

## GitHub-free contributor marketplace (2026-11-05 design update)

The newer product vision introduces a moderated project catalog, idea submissions, contributor accounts, optional notification channels and a local onboarding dashboard. A donor should **not** need a personal GitHub account to participate: the coordinator maps accepted work onto limited GitHub App permissions while presenting project enrollment, limits, task progress and recognized contributions through Nexus Kratos interfaces.

The local daemon enforces donor opt-in per project plus CPU, RAM, disk, concurrency, time, model/provider and trustworthy measurable usage limits. The marketplace can allocate work only inside those constraints; project approval is not permission to execute. Reputation reflects accepted evidence rather than tokens spent.

The full [system design](docs/architecture/system.md), [marketplace specification](docs/vision/marketplace.md), [user journey](docs/vision/contributor-journey.md), [security gates](docs/security/threat-model.md), [diagrams](docs/architecture/diagrams.md), and [design notes](docs/brainstorming/2026-11-05-design-notes.md) document proposed boundaries. The existing [implementation-plan PR #2](https://github.com/NexusKratos/nexuskratos/pull/2) remains a technical planning input to reconcile; it is not silently merged by this update.

The choice between one general-purpose runner harness and multiple provider-native harnesses is **deferred**. Use a typed, versioned adapter boundary in architecture meanwhile; do not hardwire a private runner or promise identical provider behavior. Security and compatibility fixtures, not preference alone, must decide implementation.
