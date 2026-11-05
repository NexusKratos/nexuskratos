# Nexus Kratos: open-source contribution marketplace

> **Product vision / proposed design — 2026-11-05.** This document describes intended behavior, not a service available today.

## Product promise

**Install. Sign in. Pick a project. Set limits. Contribute.** A participant should not need to know Git, GitHub, pull requests, branches, or AI-agent invocation syntax. Nexus Kratos provides a trustworthy, inspectable path for volunteers to donate bounded AI execution, human review, testing, documentation, and engineering effort to open-source projects.

GitHub remains the **maintainer-facing source of truth for code and PRs**, not a requirement for the donor-facing experience. The platform mediates identity, publication, attribution, and reviews through a repository-scoped GitHub App. No participant should be asked to hand over a personal GitHub token or their AI-provider credentials.

## Participation journey

1. **Install** a signed, verified contributor binary/daemon and open its local web UI. Offer a portable CLI for advanced users.
2. **Create a Nexus Kratos account / sign in**, pair the installation to that account, inspect permissions, and choose optional notifications. A donor need not create or connect a GitHub account.
3. **Discover** suggested, approved, and active open-source projects in a community catalog; filter by domain, license, language, impact, risk, and need. Anyone may suggest a project, but suggestion is not approval.
4. **Select projects and contribution roles**, with a clear explanation of what tasks and repositories the device may execute. A project must be explicitly trusted/approved by the donor.
5. **Set resource and safety limits:** CPU, RAM, disk, concurrent jobs, available time windows, AI models/providers, maximum usage where measurable, security profile, network policy, and whether interactive approval is required. Show a human-readable *maximum exposure* summary before enabling automation.
6. **Enable contribution.** The daemon requests suitable work, receives short-lived leased task attempts, runs through an isolated executor, and continuously reports status. The donor may pause, revoke or reduce limits at any time.
7. **Inspect results and notifications** in the local UI, optional hosted account dashboard, Telegram or email. Show task history, receipts, costs (actual vs estimated vs unknown), reviews, PR status and accepted impact.
8. **Earn recognition** for verified, accepted contributions, helpful reviews, and sustained reliability—not raw token consumption or uncontrolled compute spending.

## Marketplace and community functions

- **Project suggestions:** title, problem statement, beneficiaries, license and repository provenance, scope, proposed maintainers, security/risk profile, expected contribution types, initial milestones.
- **Project review:** moderators/maintainers evaluate code-license compatibility, legitimacy, scope, abuse risk, repository permissions, dependency hygiene and sustainability before listing for autonomous contributions.
- **Lifecycle:** suggested → under review → approved → active → paused → archived; reasoned transitions, audit history and appeal path.
- **Discovery:** searchable categories, project pages, visible issue/task pool, dependency health, skill/capability needs and public review history.
- **Enrollment:** donor opt-in per project, per-device/contributor quotas, fair scheduling across projects, clear off-switch and device health.
- **Roles:** project proposer, planner/architect, coordinator, contributor-builder, reviewer/QA/security reviewer, project maintainer, **Community Manager** (onboarding, project discovery, announcements, recognition and moderation) and platform operator. These are scoped permissions, not informal titles. Community Managers have no automatic source-merge or security-override permissions.
- **Contribution receipts:** immutable task and attempt IDs, model/tool capability constraints, commit hashes, test receipts, scan outcomes, reviewers and acceptance status. Aggregate views must preserve the underlying evidence.
- **Visibility:** donated time and measured/estimated/unknown provider usage, resource utilization, active processes, work queue, blocked reasons, notification delivery and history.
- **Recognition:** badges, reputation, leaderboards where opted in, milestones and project acknowledgements. Prevent pay-to-win incentives, badge farming and exposure of private device data.

## Ownership of framework and projects

The `NexusKratos/nexuskratos` repository produces the **framework**—control plane, signed daemon releases, shared protocol and tooling. Each listed community project links to **one or multiple independently maintained GitHub repositories**, including repos in external organizations that explicitly grant the required GitHub App access. A project listing does not move project code into the framework repository, and approved project status does not grant blanket write permissions. Donors need not use GitHub directly.

For future public interfaces, propose `nexuskratos.dev` as the framework/developer site and `nexuskratos.com` as the community/project proposal and discovery site. The live `.com` redirect is unchanged and no community site exists yet. See [framework and project boundaries](../architecture/project-boundaries.md).

## Independent architecture assurance and free Warboard

Architecture belongs to the project's authorized architecture group, not the Nexus Kratos operator. For the official community, project implementation eligibility additionally requires a versioned architecture, its governance decision, and **multiple genuinely independent high-capability AI architecture reviews** with blocking contradictions resolved. Interested users may opt into donating higher-cost AI capacity as Architecture Reviewers. If qualified free reviewers are unavailable, the request waits; projects may alternatively pay for operator-funded independent review **under the same standards**. See [architecture review pool](../architecture/independent-architecture-review.md) and [cost-recovery design](../architecture/trust-and-sustainability.md).

The free public project Warboard is proposed as **automatically generated GitHub Pages content in the project's own selected repository**, with multi-repository aggregation and visible snapshot freshness. Premium managed Warboard may be hosted by Nexus Kratos for live/private/advanced features; a paid plan is not required to see the basic status. See [Warboard delivery](../architecture/warboard-delivery.md).

## Trust and transparency boundaries

A catalog entry, review score or badge never implies code is safe to run. Donor opt-in is independent of project approval; a donor can revoke an individual project without leaving the community. Each work request is pinned to a project, repository, task revision, model policy and permitted execution profile. New trust requirements trigger fresh consent.

Platform account and device credentials authenticate to the coordinator; a GitHub App with restricted installation permissions performs project publication. Provider keys/tokens stay on the donor's host, outside the untrusted repository executor. Reviewers and maintainers—not AI assertions—decide whether evidence satisfies acceptance criteria. Autonomous **execution** is distinct from autonomous **merge**.

For an MVP, centrally hosted identities and project catalog are acceptable if explicit export, deletion, self-host options and jurisdiction/privacy expectations are documented. Federation across independent coordinators is a later decision.

## Honest product status

No installer, marketplace, daemon, control plane or signup flow is released. The current public site is informational only. Designs here are candidates for product planning and technical validation, not implementation evidence.

Related: [contributor journey](contributor-journey.md), [system architecture](../architecture/system.md), [security model](../security/threat-model.md), [brainstorming](../brainstorming/2026-11-05-design-notes.md).
