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
