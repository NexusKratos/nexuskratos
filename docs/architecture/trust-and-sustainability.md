# Trust, release signing, registries, and sustainable free access

> **Proposed architecture and funding policy — 2026-11-05.** Documentation only: no trust root, certificate authority, release-signing service, billing service, hosted free tier, or paid product currently exists. Exact policies, providers, and limits require later decisions.

## Core principles

1. **Keep the framework open source.** The Apache-2.0 framework remains usable for independent self-hosting without licensing or subscription fees. Independent operators own their own registries, signing identities, policies, infrastructure and costs.
2. **Maintain a real free option in the official community.** People should be able to discover projects, propose or register projects, contribute donor resources and receive a bounded baseline of eligibility/architecture/security checking **at zero charge**. Fair-use limits, queues and sponsorship can protect sustainability. Publish actual limits rather than implying unlimited free compute.
3. **Never sell approval or compromise security.** Payment may buy more hosted processing capacity, priority within a published policy, retention, reporting and optional deeper audits; it never buys an automatic pass, bypasses minimum safety gates, greater repo authority, or privileged access to contributors.
4. **Separate identity from safety.** A valid OS signature proves software publisher/integrity, not a task's safety. A valid registry signature proves origin, not an independent security assessment.
5. **Local contributor security policy wins.** No coordinator, billing tier, registry or project can disable donor-approved isolation, credential protection, process cancellation or resource caps.

## Distinct signing and verification layers

| Layer | Who signs | What is verified | Limits |
| --- | --- | --- | --- |
| Windows/macOS distribution | Framework release publisher | Windows Authenticode-style publisher signing; macOS Developer ID signing and notarization for qualified releases | OS publisher validation does not prove registry approval |
| Linux / cross-platform distribution | Framework release publisher | Sign release manifest, hashes, SBOM, provenance and artifact identity; verify in installer/updater | A checksum alone is not authentic unless obtained through a trusted channel |
| Official registry authority | Official Nexus Kratos operator | An independently verifiable registry identity, declared operator/domain, key/validity period and policy issuer | TLS domain ownership is not equivalent to official registry identity |
| Independent registry authority | External operator | Its own operator identity and trust root, explicitly selected by contributor | Its signature does not confer Official Nexus Kratos status |
| Project/architecture authority | Authorized community/project roles | A specific architecture revision, repository scope, decision record and approvals bound to immutable digests | Community retains design authority; AI review evidence is not a substitute for approved governance |
| Work assignment and receipt | Registry/coordinator, separately enrolled daemon and reviewers | Versioned task/lease/attempt/epoch, scoped policy, immutable candidate hashes, evidence and verifier identities | Signatures authenticate origin; results still require independent verification |

The daemon must show **Official Nexus Kratos — cryptographically verified**, **Community-operated — independently verified identity**, or **Unverified / unknown**. Display the operator, endpoint, key fingerprint, policy source and whether execution was explicitly trusted; retain those facts in audit receipts. A forked UI can forge labels, so official identity must be independently checkable using a public verification specification and signed roots, not a cosmetic badge.

## Trust root and operational PKI

- Use an offline/hardware-protected root or equivalent controlled trust anchor for official delegations; daily task signing uses narrowly scoped intermediate/online keys, not the offline root.
- Keep **release signing**, **registry identity**, **security policy**, **device enrollment**, and **task/evidence signing** in separate key hierarchies with purpose binding. Never reuse Apple/Windows signing credentials as registry trust roots.
- Define cryptographic algorithm agility, key IDs, audience/domain/tenant binding, expirations, nonce/replay protection, delegation scopes and signed metadata canonicalization.
- Rotate/revoke keys; provide signed revocation/trust metadata with monotonic version/expiry and a documented recovery procedure for compromised roots. A stale trust snapshot cannot authorize unbounded offline execution.
- Define certificate/key issuance, transfer of registry ownership, expiration, disaster recovery and the behavior when trust metadata cannot be refreshed.
- Device trust is separate from registry trust: explicit donor opt-in, per-project consent, no privilege granted from an arbitrary public registry listing.
- Publish an open verifier and test vectors for verifying official registry status without trusting the official website's appearance alone.

## Hosted community tiers (proposal, prices undecided)

| Capability | Free community path | Optional cost-recovery service |
| --- | --- | --- |
| Open-source framework and self-hosting | No framework license fee; operator supplies own infrastructure | Managed hosting, backups and operations if offered |
| Project suggestions and community participation | Free, with anti-spam and moderation controls | No paid bypass of project approval |
| Project registration and baseline checks | Free bounded quota/queue for genuine open-source projects; can be donor/community sponsored | Additional architecture model reviews, deeper audit reports or more frequent hosted scans |
| Required minimum security checks | Must be completed for a project/task to be eligible, regardless of ability to pay | Higher compute volume, extended retention, nonmandatory specialist reviews; **not less safety on free projects** |
| Builder participation | No participation charge; contributor provides local resources and approves limits | Optional managed capacity or paid hosted compute with clear caps |
| Security intelligence and SBOM rechecks | Essential vulnerability signals and blocks apply equally to free and paid participating projects | Optional historical analytics, reporting/export or higher-volume integrations |
| Badges and reputation | Based on verified accepted contributions/reviews, never spending | No pay-to-win privileges or purchased security certification |

A free project's processing can wait in a queue if it exceeds sponsored capacity, but **must not be dispatched to builders without completing mandatory checks**. Explicitly disclose quotas, wait policy, data retention, provider model costs and whether project owners must bring their own API keys; never silently charge contributors or promise free unlimited LLM tokens. Consider sponsorships, grants, donation pools, bring-your-own model/API quotas and opt-in community-funded scanning.

## Architecture and data contracts to design

`RegistryIdentity`, `TrustRoot`, `KeyDelegation`, `SignedPolicyRevision`, `FrameworkReleaseAttestation`, `ProjectApprovalAttestation`, `TaskManifestSignature`, `SecurityIntelligenceSnapshot`, `BillingAccount`, `ServiceUsageReceipt`, `FreeQuotaGrant`, `SponsorshipGrant`, and `ServiceEntitlement` need separate immutable identifiers, tenant scope, valid-from/expiry and audit trails. Payment records are distinct from security certification and must never grant authorization to skip checks.

The release client validates publisher, artifact digest, provenance and protocol compatibility; the daemon verifies registry trust and execution policy; the coordinator independently checks reviewer evidence. Avoid making any one signature or account a universal root of authority.

## Required design and test gates

- Windows signature verification and invalid/mismatched signer; macOS notarization/Developer ID and rejection scenarios; Linux signed manifest/artifact verification; signed auto-update rollback and freshness.
- Legitimate third-party registry and spoofed official domain; malicious fork claiming Official status; revoked or rotated root; expired intermediates; invalid trust metadata; offline stale registry policy; registry migration and wrong audience.
- Old/compromised device identity attempting stale lease; task manifest substitution; signature malleability/canonicalization; malicious reviewer attestation and PR evidence mismatch.
- Free project registering and passing minimum baseline without payment; exhausted free processing quota **queues or blocks** but never skips scans; paid account cannot buy architecture approval or suppress critical findings.
- Transparent cost/usage separation and sponsored quotas, anti-abuse limits, billing isolation, entitlement revocation and historical audit integrity.

## Boundaries and outstanding decisions

Choose certificate authorities/providers, signing toolchain, key storage/HSM, trust envelope and delegation representation during the design phase—not by inventing an implementation. Compare standard supply-chain attestations (e.g. in-toto/Sigstore-style approaches) against a bespoke PKI, with an explicit interoperability and recovery review. Independent deployments are free to use distinct policies but do not obtain the official trust root.

Official hosted service pricing, exact free quotas and funding source are **open decisions**. Keep basic community participation free, maintain transparent safety checks across tiers, and design a graceful capacity shortfall path without unsafe execution.

Related: [platform implementation plan](../plans/2026-11-04-platform-implementation.md), [security threat model](../security/threat-model.md), [marketplace vision](../vision/marketplace.md), and [framework/project boundaries](project-boundaries.md).
