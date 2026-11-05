# Implementation planning handoff — 2026-11-04

Scope: improve the platform implementation plan, documentation only. Baseline `014decc6d1ca8c04ba860ceaedc545283edab397`; previous foundation history remains in [the foundation handoff](2026-11-04-foundation.md).

Read [the plan](../plans/2026-11-04-platform-implementation.md) and [proposed ADR 0003](../adr/0003-platform-planning-boundaries.md). The original 2026-11-04 draft described 19 tasks; the 2026-11-05 reconciliation now includes **25 tasks**, covering security scanners, independent multi-repository projects, moderated marketplace/Community Manager, GitHub-free donor onboarding, optional notifications and recognition, and signed framework distribution. Both versions describe milestone demonstrations, owners, dependencies and required evidence. They do not claim any implemented feature or passed runtime check.

The user explicitly prohibited implementation. Planning documentation and its normal source checkpoint are the only current authorized work. Do not scaffold, install dependencies, invoke providers, deploy, or start T01 because this plan exists. Continue planning revisions if requested; execution needs an explicit instruction lifting that hold.

First future execution order: T01 feasibility/technology decisions → T02 reviewed protocol → independent control and contributor branches as recorded in the plan. The new T20 scanner gate precedes T10 review/acceptance, and T21 repository scoping precedes T12 multi-repository integration. Community/UX and signed release work follow their explicit dependencies. Do not substitute credentials-in-workspace execution if a provider cannot meet the boundary. Resolve sensitive-runtime review scope before real accounts or credentials are used.

No build, service, provider task or installer was started as part of either documentation milestone. Website and Warboard repositories were not changed. Documentation checks and review dispositions are recorded in [the planning review/close-out record](../reviews/2026-11-04-platform-planning.md); source checkpoint is distinct from implementation acceptance.


## 2026-11-05 reconciliation handoff

PRs #3 and #4 have been merged into the framework repository. This PR #2 planning update deliberately keeps their README, architecture, diagrams, donor journey, project boundary and Community Manager design. The framework repository holds framework code/binaries only; each open-source project uses independently owned, explicitly authorized GitHub repositories. A separate community portal at `.com` is proposed, not deployed, and `.com` continues redirecting to `.dev`.

The 2026-11-04 independent reviews apply to the original 19-task text only. Updated validation and residual gates appear in [the later reconciliation record](../reviews/2026-11-05-pr2-reconciliation.md). The original review evidence must not be presented as a fresh review of changed 25-task scope.

No implementation approval, installer distribution, new project repositories, DNS change, or Cloudflare deployment is implied by merging this plan.
