# Free repository Warboard and premium hosted Warboard

> **Architecture and product proposal — 2026-11-05.** No public GitHub Pages board or paid hosted Warboard has been deployed. Warboard is independently developed and must remain optional; no deployment or DNS change is authorized by this design.

## Product split

**Free / open-source project board:** publish a sanitized, read-only Warboard to **GitHub Pages** associated with the project's own repository. The project can host its board without a Nexus Kratos-managed subscription or service dependency. The open-source board generator and its export schema must remain available to self-hosted registries.

**Premium / managed board:** optionally host a richer Warboard on a Nexus Kratos-managed service, with faster near-real-time updates, historical analytics, authenticated private views, cross-project management, alerts or other compute/storage-intensive functions. This is **cost-recovery**, not a prerequisite for participation, architecture approval, security certification, or receiving build tasks.

Both tiers must use the **same authoritative coordinator records** and published versioned/redacted event/snapshot contract. The Warboard itself is a view, not the authoritative task scheduler, security engine, review record or source of task truth.

## One project, one or multiple repositories

- A project can designate a **primary repository** (or separate documentation repository) to publish its project-wide GitHub Pages dashboard; do not generate conflicting project-wide boards independently from every linked repository by default.
- A project with multiple source repositories can link to repo-level status views, CI and PRs from the project-wide board. If individual repositories publish separate boards, they are scoped views, not independent project coordinators.
- Repository owners must explicitly authorize the Pages deployment and any Actions workflow. If Pages is unavailable or forbidden by policy, provide an exportable static artifact or self-hostable site without forcing users to buy hosted access.
- Projects remain independently owned; Nexus Kratos does not silently mutate a repository, turn on Pages, or add workflows without its authorized maintainer's consent.

## Update and publication model

~~~mermaid
flowchart LR
  Coord["Coordinator: canonical tasks and approvals"] --> Export["Versioned, privacy-filtered status snapshot"]
  Export --> Generator["Open-source static Warboard generator"]
  Generator --> GH["Authorized project repository CI / publishing job"]
  GH --> Pages["Free GitHub Pages dashboard"]
  Export --> Managed["Optional hosted Warboard ingestion"]
  Managed --> Premium["Premium live and historical dashboard"]
~~~

The free board is **automatically refreshed on approved events or a scheduled job**, subject to repository CI quotas, publisher availability and GitHub Pages deployment constraints. It is an **eventually consistent static snapshot**, not guaranteed real-time telemetry. Display the last successful update timestamp and stale-data indicator. Publish with an immutable snapshot version, schema revision and source revision so viewers can distinguish stale information from active tasks.

Recommended public data: project and architecture approval status, accepted architecture revision digest, milestone/task totals, anonymized queue and workflow states, relevant publicly visible PR links, scanner/check status summaries and aggregated contributor impact. Links to evidence can point at public review records. Project status must not disclose embargoed vulnerabilities.

**Never expose** reviewer/donor tokens, AI credentials, device identifiers, private email, raw task prompts, internal execution logs, unreviewed proprietary code, private infrastructure details, security exploit evidence or sensitive user information. Project policies define which fields are publishable; snapshots undergo allowlist-based field selection and redaction, not simply string replacement.

## Secure GitHub Pages publishing

- Use narrowly scoped publishing credentials with short lifetimes where possible; the board generator never receives donor provider tokens or coordinator merge credentials.
- Treat repository content, branch names, PR messages and build artifacts as untrusted display strings; sanitize output, enforce safe links and apply CSP as appropriate to limit XSS.
- A contributor PR must not be able to edit a privileged publishing workflow and immediately execute it with protected secrets. Use reviewed workflows and protected branch/publish policies.
- Avoid checking sensitive raw API payloads, telemetry or historical events into the publicly visible repository. Publish only a deliberately public snapshot.
- Changes to what data may be published require a policy review; a paid subscription does not authorize publication of private data.
- A forged static board is not proof of safety certification: display verifiable coordinator/attestation links or separately verifiable signed snapshot metadata. The underlying signed approval/review evidence remains authoritative.

## Premium managed service boundaries

- Managed hosting may offer live refresh, interactive filtering, retention, alert routing, private RBAC, audit exports and cross-repository insights, with transparent metering and spending caps.
- Free users retain access to the public board, essential safety status and open export format; **premium does not improve security approval authority**, and may not hide critical security warnings from free views.
- Hosted private dashboards require proper project/member authorization, data minimization, tenant isolation, deletion/export rights, notification preferences and explicit publishing consent.
- Choose hosting/domain (possibly under `nexuskratos.com`) later; the existing `.com` → `.dev` redirect is unchanged. The Warboard project remains separately maintained and usable without the Nexus Kratos coordinator.

## Required validation

1. A one-repository open-source project publishes a static board under its own repository Pages site without a paid service.
2. A project with three repositories creates exactly one designated project board and shows per-repository PR/check links with correct scopes.
3. Offline coordinator / blocked CI / exceeded quota results in a visible stale snapshot, not a false "all checks passed" state.
4. Attempted injection in issue titles, task names, PR messages and review text cannot execute scripts in the public board.
5. Private logs, tokens, device identifiers and embargoed findings cannot appear in generated Pages assets.
6. Unreviewed pull-request changes cannot obtain privileged publication tokens.
7. Premium service stops working: the coordinator and free GitHub Pages snapshots continue working independently.
8. Contributor/security status changes become visible in free summaries at next successful update; paid views do not alter acceptance rules.

Related: [system architecture](system.md), [review pool](independent-architecture-review.md), [sustainable free tier](trust-and-sustainability.md), and [platform implementation plan](../plans/2026-11-04-platform-implementation.md).
