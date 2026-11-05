# Nexus Kratos design workshop — 2026-11-05

Status: **open brainstorming**, not an implementation approval or finalized ADR. This is a public, non-sensitive record of the project discussion. Specific exploit details, credentials and private infrastructure remain outside this repository.

## North star

Turn donated, user-bounded AI execution capacity into **secure, observable, reviewed open-source contributions**. A person installs the daemon, signs in, finds a project, chooses what they trust and what resources they can offer, and then the system handles compatible tasks automatically. GitHub becomes infrastructure behind the scenes—not a UI prerequisite.

## Ideas captured

| Area | Direction / concern | Current proposed treatment |
| --- | --- | --- |
| Open-source marketplace | Public suggestions, project discovery, approval, categories and visible impact | Curated project lifecycle with moderated submissions; opt-in per project |
| Contributor UX | GitHub should be invisible to ordinary donors | Account + local web UI; GitHub App on control plane; optional GitHub for maintainers |
| Installation | Easy signed binary/daemon and local web interface | Rust agent, headless core, local UI, portable CLI; verify per platform |
| Onboarding | User picks project and resources, then automation takes over | Guided trust/capability/security screen, explicit consent and local pause |
| Resource donation | CPU, RAM, disk, concurrency, running hours, tokens, provider budget | Transactional local reservations; hard limits only when enforceable |
| Capacity planning | Number of parallel projects tied to donated resources | Global max plus per-project quotas, fair scheduling and no overcommit |
| Task pool | PR-first tasks and bundles; dependencies and expiry | Immutable tasks/revisions, leased attempts, stale-epoch fencing, checkpoints |
| Architectural roles | Architects, orchestrators, builders, QA, security reviewers | Role-scoped permissions and independent review; accepted result is provenance-bound |
| Models | Minimum capability tier, approved lists and long/short complexity | Match capability, model, provider and effort with task; no silent fallback |
| Execution | Headless and unattended where feasible | Typed generic runner contract; explicit needs-attention path |
| Harness choice | One universal harness versus provider-native harnesses | **Deferred until after current documentation tasks**; qualify alternatives against security, UX and provider capability |
| Security | Scanners must protect donors, projects and code | Multi-phase scanning, sandbox, no host secrets, narrow GitHub App, PR policy |
| Review | Two-model review preference (high-tier judgment + low-tier mechanical) | Independent identities and candidate-bound test/scanner receipts; model diversity is a quality goal, not a substitute for human authority |
| Communication | Email, Telegram and web status | Consent-based notifications and redacted history; local UI is primary |
| Reputation | Badges and gamification | Award verified accepted work/reviews, not raw tokens; prevent farming |
| Warboard | Full visibility and telemetry | Optional independent read-only consumer; no mandatory dependency |
| Hosting | Self-hostable control plane, multi-project and multi-community | Modular core, versioned protocol; federation deferred |
| Marketing | README hero and diagrams clarify automated contribution | Static SVG and Mermaid with honest pre-alpha status |
| Project repository ownership | Framework repo should hold framework only; each community project has one or several repos | Explicit ProjectRepository mappings and scoped GitHub App installations; no code migration by default |
| Community Manager | Separate community engagement from technical project ownership | Scoped onboarding/moderation/comms/recognition without automatic merge or security permissions |
| Product domains | Consider .dev for framework and .com for community proposals and discovery | Proposed site roles; no live DNS/redirect changes |

## New owner guidance — 2026-11-05

- **Framework repository:** `NexusKratos/nexuskratos` is for the platform implementation, signed daemon distribution, protocols and documentation. Do not mix arbitrary community project implementation code into it.
- **Independent project repositories:** one project may target one or multiple repositories (within or outside the NexusKratos organization) through explicit permissions, leases and per-repository PRs.
- **Community Manager:** a scoped operational role for people, onboarding, suggestions, moderation, messages and recognition, distinct from maintainers and security reviewers.
- **Domain split (under consideration):** `nexuskratos.dev` would be the framework/developer site; `nexuskratos.com` would be the community portal for ideas, projects and participation. Current redirect remains live until separately approved deployment work.

See the [repository and domain boundary specification](../architecture/project-boundaries.md).

## Additional design discussion: 2026-11-05

- **Official status and software signing:** separate Windows/macOS publisher signing and Linux signed artifacts from official Nexus Kratos registry/trust-root signatures, policies, task identities, and evidence. Operators of open-source forks may sign their own registries; the official trust root is not shared. Verify identity cryptographically, not by a UI label.
- **Cost recovery with a permanent free option:** project proposal/registration, community participation and bounded baseline security/architecture checks remain free for eligible official-community projects. Optional paid volume or hosted services cannot purchase approval or bypass scanners.
- **Independent Architecture Reviewer specialization:** during account enrollment, contributors may opt into higher-cost high-capability model review work, set budgets and availability, and declare conflicts. Require multiple independent reviews for each frozen revision; author/maintainer-linked reviewers cannot count as independent. Unavailable volunteer capacity means a queue, not reduced requirements.
- **Paid review capacity:** an optional Nexus Kratos-operated service supplies the model and reviewer capacity when free volunteers are unavailable or the project chooses faster fulfilment, applying the **same independent review and blocking finding rules**.
- **Warboard delivery split:** free/public read-only project status generated as a static GitHub Pages board in a project-authorized repository (one designated aggregate board for multiple repositories). Optional premium Warboard hosted by Nexus Kratos can offer dynamic/private/advanced features, but the open-source coordinator never depends on it. Static boards must show freshness and never leak sensitive events.

See [review pool](../architecture/independent-architecture-review.md), [trust/free cost recovery](../architecture/trust-and-sustainability.md), and [Warboard delivery](../architecture/warboard-delivery.md).

## Later discussion: architecture planning and deterministic Task Orc evaluation

- **Community architecture freedom:** project groups own architecture and may collaborate under their own declared decision rules. Official Nexus Kratos verifies revision-specific independent review, resolved blocking findings and task safety; it does not dictate the technology stack. One project may register multiple repositories; tasks can originate from multiple projects but permissions remain repository-scoped.
- **Planning as volunteer work:** Architect, Planner and Plan Reviewer are separately opted-in AI roles in the contributor daemon, with explicit provider/model/usage budgets. Projects may submit existing plans or use donated/paid planning capacity; AI never authorizes its own plan. Architecture and plan reviews precede implementation eligibility.
- **Knowledge efficiency:** distribute a compact signed local knowledge pack of architectural methods with explicit trade-offs and retrieval. Consider an optional tiny model for classification, not final design. Keep project-specific approved design and high-capability planning separate.
- **Provider ecosystem:** use a typed multi-provider interface for direct APIs, OpenRouter, verified subscription-native CLI modes, Gemini, Grok and other qualified models. Astra/Fable are preferred candidates for difficult planning subject to benchmark and availability; no provider's monthly subscription may be assumed to grant reusable API credits without current legal/technical verification.
- **Task Orc:** its Rust deterministic validation/readiness, coherent-task decomposition, model capability/risk floors, session affinity and bounded Warboard outbox are relevant, but it currently lacks distributed leases/authoritative execution. Owner welcomes possible future publication, yet it is still a private, unfinished, proprietary evaluation candidate. **Do not copy or publish its source** without a separate license/release decision. Qualify via reproducible stress/fault tests and comparative real engineering trials before adoption.
- **Supply-chain protection:** per-repo commit/build-bound SBOM snapshots with resolved transitive dependencies; normalized daily-plus-urgent CVE/advisory intelligence, targeted signed blocks and local daemon enforcement. Approval expires when relevant architecture or dependency evidence becomes invalid; only affected tasks need reassessment.

Full design records: [planning](../architecture/planning-and-orchestration.md), [provider knowledge](../architecture/provider-and-knowledge.md), [Task Orc qualification](../architecture/task-orc-qualification.md), [continuous supply chain](../security/continuous-supply-chain.md).

## Architectural concerns still needing evidence

1. **Credential separation:** can each provider CLI work headlessly when arbitrary repository commands cannot read its credentials? If not, disable that adapter or broker calls in a trusted process; do not claim a worktree is isolation.
2. **Model/harness compatibility:** unattended startup, auth expiry, API/CLI terms, cancellation, output parsing, cost accounting, safe adapter version pinning and tools allowed by provider.
3. **Trust in marketplace:** accepting a project suggestion is not enough to run code. Licenses, risk classification, moderation, donor trust and supply-chain provenance are separate gates.
4. **Identity/attribution:** GitHub-free donors should receive auditably attributed PR credit without exposing personal access tokens. Clarify bot authorship, legal attribution and account deletion.
5. **Review independence:** self-review, forged test receipts and same-model collusion must not unlock merge. Bind checks to exact candidate/head/base; ensure revalidation on changes.
6. **Isolation on platforms:** Linux likely first practical qualification; Windows/macOS support requires platform-specific equivalent guarantees and honest support matrix.
7. **Fairness/abuse:** guard against project monopolization, malicious prompts, donation denial-of-service, gaming badges, fake contribution accounts and paid-capacity manipulation.
8. **Leases/recovery:** crashes, duplicate webhooks, disconnected devices, late provider spend and expired tasks must preserve evidence without giving stale work authority.
9. **Scope:** test a complete contribution loop early; avoid a visually impressive marketplace with unproven secure execution. Keep catalog/user UX explicit acceptance scope, not an afterthought.
10. **Scanners:** choose scanner set and rules by language and risk; scanners themselves process untrusted input and need sandboxing. Track suppression ownership and expiry.
11. **Observability:** expose task states, progress, actual/estimated/unknown usage and redacted evidence; protect private content and enforce retention.
12. **Governance:** who may nominate projects, approve tasks, moderate, accept PRs, revoke devices and override security gates? Require scoped auditable authority.

## Proposed phased sequencing

**Foundation / feasibility:** finalize product contract, threat model and capability matrix; prove safe task execution, cancellation and credential boundaries with harmless fixtures.

**Secure contribution loop:** enrollment, task protocol/leases, isolated executor, provider adapters, artifact publication, independent QA, security scanning, protected PR acceptance.

**Marketplace beta:** project suggestions/moderation, account/device pairing, local contributor UI, browsing/opt-in, donations/limits, notifications, badges and audit displays.

**Multi-community + operations:** fairness, more platforms, self-host deployment, restore/recovery drills, Warboard adapter, production release qualification.

These phases are a discussion overlay. [Existing PR #2](https://github.com/NexusKratos/nexuskratos/pull/2) contains an earlier 19-task technical plan; reconcile both before merging and do not misrepresent its progress.

## Open decisions to close later

- Single general-purpose harness versus separate native provider harnesses; decision deferred at owner's request.
- Device authentication and account recovery requirements; data jurisdiction, deletion/retention and abuse moderation.
- Trust profiles, outbound network controls, per-project permission consent and first safe OS backend.
- MVP release scope: first one-community demonstration versus marketplace-assisted beta; criteria for "usable" and "public."
- Governance for automatic merge, if ever, and higher-assurance independent review policy.
- Notification provider integration, badge system design and moderation workflow.

## Non-decisions and exclusions

No runtime, installer, executable daemon, project catalog or marketplace has shipped. No paid provider calls, external service provisioning or Cloudflare deployment are implied by these documents. Do not use or name a private personal runner as a platform dependency unless the owner explicitly chooses it. Avoid writing private security findings to public design documents.
