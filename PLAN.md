# Current work

Dev mode: mvp (default; no runtime exists).

## Public project foundation — authorized 2026-11-04

Status: done 2026-11-04 for this repository foundation; website work continues in its own repository. Written/authorized/updated: 2026-11-04.

User scope: set up the NexusKratos organization and main repository, plus a separate static website repository for Cloudflare. Execute authorized setup tasks in sequence; no platform implementation is part of this task.

Threat sketch: public documentation and static website, no user accounts or stored data. Avoid publishing secrets or private local information. Links and feature claims must be accurate. Cloudflare access remains a distinct capability from GitHub administration.

| Task | Owner | Boundary | Evidence | Next |
| --- | --- | --- | --- | --- |
| Organization and main repository foundation | Coordinator | Organization profile; this repository | GitHub API readback; document checks | Website verification |
| Static website | Website worker | Separate website repository public assets | Desktop/mobile browser checks | Source checkpoint |
| Verify and publish source | Coordinator | Both new repositories | Public metadata, pushed commit/tag parity | Hosting access if available |

Communication: native worker completion messages; coordinator integrates shared records sequentially. No worker commits or external publication. Cloudflare authentication refreshed successfully. Website deployment requires the Cloudflare GitHub App connection for the new organization; its separate repository tracks this external prerequisite.

## Repository security — authorized 2026-11-04

Status: done 2026-11-04 for the applied configuration; source records close through the protected PR workflow. Owner: coordinator. Scope: the two new public repositories only. Record security choices, enable native GitHub settings, commit the settings documentation, activate branch/tag rules, and verify API readback. No application code, dependencies, deployment, organization membership, or org-wide account policy changes. Threat scope: protect shared source and workflow permissions; preserve owner access through an audited PR exception.

## Platform implementation planning — authorized 2026-11-04

Status: planning document prepared; implementation on explicit user hold. Written/authorized/updated: 2026-11-04 (authorization covers documentation only).

Current plan: [Platform implementation](docs/plans/2026-11-04-platform-implementation.md). It defines the threat sketch, component ownership, protocol and recovery invariants, 19 dependency-ordered tasks, four milestones, reviews, acceptance evidence and unresolved feasibility gates. Proposed decisions are recorded in [ADR 0003](docs/adr/0003-platform-planning-boundaries.md).

This is the next product plan; earlier completed setup entries above remain historical. No implementation task is running. A future explicit implementation instruction is required to lift the user's planning-only hold.
