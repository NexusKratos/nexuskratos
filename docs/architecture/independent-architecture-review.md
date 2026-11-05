# Independent architecture-review contributor pool

> **Product design proposal (2026-11-05).** Not a deployed service. These requirements apply to the official Nexus Kratos community; self-hosted instances may publish their own policies. Human project governance remains responsible for architecture decisions.

## Objective

A project may register one or more repositories, but **implementation tasks are not eligible for distribution** until the project has a settled, versioned architecture and passes the official community's published architecture and security review requirements. The platform certifies **review process compliance and execution readiness**, not that a given architecture is ideal or guaranteed bug-free.

The project retains its own group of architects, its decision process and design ownership. Nexus Kratos supplies independent assessment, evidence verification and quality gates, without choosing languages, frameworks or architectural patterns on behalf of the project.

## Architecture Reviewer as an opt-in contributor role

At account creation and subsequently in contributor settings, a participant may separately opt into **Architecture Reviewer**. This role is distinct from builder, architect for a project, code reviewer, community manager and administrator.

For each reviewer, maintain an explicit profile recording:
- Independent contributor/device identity and permitted registry/project scopes.
- Opt-in status, availability/schedule, concurrent-review limit and cancellation controls.
- Which high-capability AI providers/models the reviewer is willing and authorized to use, minimum supported tier, version and relevant review capabilities.
- Resource and **usage budget**: donated API tokens/charges where enforceable, time windows, estimated upper bounds and whether a cost ceiling can actually be guaranteed.
- Specialty areas (security architecture, distributed systems, supply chain, accessibility, performance, etc.) and a qualifications/evidence history without exposing private credentials.
- Declared conflicts of interest and required recusal; the coordinator must perform independent eligibility checks beyond a self-declaration.

Donor provider credentials remain local to the reviewer-controlled trusted supervisor, outside untrusted project documents/commands. Submitting a review never confers permission to push or merge project code. Every review task is independently leased, bounded and cancelable.

## Two funding/fulfilment paths, one review policy

| Property | Free community review queue | Paid Nexus Kratos-operated review |
| --- | --- | --- |
| Compute source | Opt-in independent reviewers donate high-capability model usage | The official operator pays for hosted models and reviewer execution capacity |
| Eligibility | Within published free fair-use quotas, queued until qualified capacity is available | Project requests a published paid capacity allocation; execute when qualified operator capacity is available |
| Project ownership | Retained by the project and its architects | Exactly the same |
| Security/architecture gate | Identical official minimum independent review and finding-resolution requirements | Identical gate; payment **cannot buy approval or bypass a blocker** |
| Model floor and review count | Same risk-tier policy | Same risk-tier policy |
| Outcome | Evidence-backed pass, needs-changes or fail/block | Evidence-backed pass, needs-changes or fail/block |
| Visibility | Queue position/status and estimated or unknown wait, if supported | Cost estimate/cap, usage receipt and status, without an approval guarantee |

Registration, normal participation and a bounded baseline of independent architecture and security assessment remain **free** for eligible official-community open-source projects. If the free reviewer pool is exhausted, requests remain queued rather than silently reducing review count, required model tier or independence. The operator may offer additional review capacity via paid cost recovery; neither a paid request nor a third-party reviewer can certify themselves.

Paid capacity is **the operator funding and operating independent reviews**, not the operator dictating project architecture. Operator-employed/operated reviewers must still be excluded if they authored the submitted architecture or have a disqualifying connection to the project.

Pricing, quotas, subsidies, reviewer qualifications and guarantees are open design questions. Never imply that using a particular subscription permits resale or shared unattended review capacity; each provider's terms and billing model must be verified before enabling it. No implicit charges.

## Redundant, genuinely independent reviews

- Review a **frozen architecture revision** identified by repository/revision/commit and digest, scope, requirements, affected components, security profile, and required policy version.
- **At least two independent reviews** as the proposed minimum for the official community; require more by risk classification. The minimum count and independence threshold are policy-versioned design decisions before launch.
- Use qualified **high-capability models** for architecture evaluation; diversify model family/provider when availability and policy require it. The same reviewer rerunning one prompt is **not** independent redundancy.
- Require separate contributor/organization control and independently generated evidence. Detect conflicts involving project author, architect group, repository maintainer, task submitter, reviewer account, device or funding arrangement, to the extent verifiable. Do not assume separate API calls establish real independence.
- Each reviewer produces structured findings with severity, affected requirements/components, reproducible evidence where possible, model/provider identity and version, toolchain version, uncertainty, and proposed alternatives.
- The coordinator aggregates reviews by finding and design revision without falsely collapsing disagreement. Project architects resolve contradictions, record a decision or evidence-backed rebuttal, and resubmit a new revision for material changes.
- **Blocking findings remain blocking**, even on paid reviews. Model confidence is not proof; a material false positive may be formally adjudicated under published policy with auditable reasoning and independent checks. The project decides architectural trade-offs; the platform only enforces published eligibility/safety conditions.
- The architecture must be **formally accepted under the project's own declared governance policy**, then the official coordinator verifies mandatory reviews, absence of unresolved blocking findings and immutable evidence.
- Implementation tasks reference the resulting approved architecture revision. A material architecture or dependency change triggers impact analysis and revalidation of affected work, not a blanket revocation of unrelated tasks.

## Minimal lifecycle

~~~mermaid
flowchart TD
  Register["Project registers repos and architecture revision"] --> Validate["Check ownership, architecture completeness, scope and risk"]
  Validate --> Pool{"Select independent reviewer capacity"}
  Pool -->|"Free"| Queue["Wait for opted-in qualified volunteer reviewers"]
  Pool -->|"Paid"| Operator["Qualified operator-run reviewers and hosted compute"]
  Queue --> Review["Multiple independent high-tier model reviews"]
  Operator --> Review
  Review --> Findings["Consolidate evidence and blocking findings"]
  Findings --> Resolve["Project architects resolve or formally contest findings"]
  Resolve -->|"Material revision"| Validate
  Resolve --> Governance["Project governance records approval"]
  Governance --> Gate{"Published readiness + independence + security policy satisfied?"}
  Gate -->|"No"| Queue
  Gate -->|"Yes"| Eligible["Enable individually admitted builder tasks"]
~~~

The lifecycle must avoid an accidental paid bypass at any stage. When approval cannot be established, the project is not build-eligible, though its proposal can remain discoverable under the marketplace's public policy.

## Proposed entities / contracts

`ArchitectureRevision`, `ArchitectureReviewRequest`, `ArchitectureReviewPolicyRevision`, `ReviewerEnrollment`, `ReviewerCapability`, `ReviewerConflictDeclaration`, `ArchitectureReviewAttempt`, `IndependentReviewReceipt`, `ArchitectureFinding`, `FindingDisposition`, `ProjectArchitectureDecision`, `ArchitectureReadinessCertificate`, `ReviewFundingSource`, and `ReviewUsageReceipt`.

A review receipt must bind **reviewer identity, model/version, architecture revision digest, exact reviewed scope, policy version, time, toolchain/evidence digests and the signed or authenticated reviewer attestation**. Independent verification of receipts and model claims is required; signing authenticates a claim, not its truth.

The scheduler matches reviewer tier, specialties, capacity, conflicts, policy, cost controls and reviewer independence. A free or paid funding source is not itself permission to bypass those checks. The coordinator tracks successful completed reviews separately from abandoned/failed/duplicate attempts.

## Required validation scenarios

1. A project with no connected reviewers enters the free queue until **two independent** qualified volunteers are available.
2. A project selects paid capacity; operator-managed independent high-tier reviews run using explicitly approved budgets and produce the same gating evidence.
3. Paid service returns critical contradictions; the project **cannot** buy acceptance.
4. Two accounts sharing the same device/organization or declared project authorship cannot falsely count as independent reviewers under the configured assurance policy.
5. Reviewer cancels, provider quota expires, model downgrades, or spending becomes unknown: do not silently substitute a weaker review or claim a hard cost cap.
6. Review report targets an old architecture revision: reject its use as current approval.
7. Architecture changes affect only dependent tasks; stale review evidence cannot permit new task dispatch.
8. No qualified donated capacity: queue with honest status; no unlimited-free-compute claim, no admission without full checks.

## Related design records

[Trust signing and sustainable free access](trust-and-sustainability.md) · [Platform implementation plan](../plans/2026-11-04-platform-implementation.md) · [Security model](../security/threat-model.md) · [Marketplace](../vision/marketplace.md).

