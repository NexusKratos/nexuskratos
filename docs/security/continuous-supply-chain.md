# Project certification, SBOMs and continuous vulnerability intelligence

> **Proposed official-community security policy — 2026-11-05.** No operational SBOM inventory, CVE ingestion service, automated blacklist, security-policy distribution or project certification engine exists today. Independent self-hosted registries may define different public policies; contributor daemon minimum safety and opt-in remain enforceable locally.

## Project registration and eligibility

Anyone may **propose** a legitimate open-source project and link **one or more** GitHub repositories. Proposal alone does not authorize building or imply endorsement. The official community validates repository owner-approved installation scopes, license/provenance, project governance, architecture revision, independent architecture reviews and baseline repository security checks before enabling task dispatch.

Approved projects become discoverable in the community marketplace; each *individual task* still requires policy-specific execution admission, qualified sandbox availability, approved architecture/plan revision, current SBOM/security evidence, resource reservations and repository/commit scope verification. Paid registration or managed review capacity cannot bypass admission.

Shared repositories used by multiple projects retain their own authority and scoped contribution policies. Cross-project tasks must obey the repository's single authoritative change/review process rather than competing project coordinators.

## Mandatory per-repository SBOM snapshots

Each participating repository must maintain a **complete-enough, versioned Software Bill of Materials** tied to source commit/build environment, including direct and transitive dependencies where the ecosystem permits resolution. Prefer standard **CycloneDX** or **SPDX** interchange with package identity (purl/CPE where applicable), exact resolved version, hashes/provenance, package relationships, applicable licenses, build toolchain/container/base OS packages, and completeness/unknown markers.

- Generate/verify an SBOM at onboarding, every dependency-affecting change, important build environment/toolchain change and each release artifact. Do not claim “continuously regenerate every repo” when no material input changed.
- Keep SBOM snapshots immutable per commit/build/release; publish or store only appropriate data. The project repository may retain reviewed snapshots; an artifact store can retain large digests with an indexed database referencing immutable hashes. No competing mutable architecture truth.
- Compare new dependencies, versions, lockfile changes and resolved transitive trees to prior inventories. Missing/incomplete/stale evidence **blocks relevant automated execution** according to the published policy.
- The local builder verifies actual resolved dependencies/environment against task-supplied SBOM evidence, not merely a repository-authored manifest or a self-reported clean CI check.
- A project-wide dependency graph aggregates each repository's snapshot with explicit edges and provenance; it must not silently merge unrelated owners/permission scopes.

## Continuously re-evaluate security status

The **Security Intelligence Service** receives signed/fetched, date-stamped upstream intelligence from qualified sources such as OSV, NVD/CVE, GitHub Security Advisories, CISA KEV, EPSS and ecosystem-specific malicious package registries. Treat upstream feeds as evidence, not blindly trusted truth; normalize identity/versions/affected ranges, deduplicate, record source freshness and handle corrections/withdrawn advisories.

Run a **daily full refresh/reconciliation** with more frequent urgent updates where supported. Re-evaluate all retained immutable SBOM snapshots when advisories or affected-range data change even if source commits did not. Maintain separate **`SecurityAssessment`** revisions bound to SBOM digest, source feed snapshots, policy version, affected package/version, severity/exploitability, evidence and disposition.

The policy engine should scope vulnerability controls to **specific affected versions/components**, not automatically blacklist entire project names. Confirm package identity/affected range, reachable exposure where demonstrable, CISA KEV/exploitation signals and risk-tier policy. Confirmed critical/actively exploited or malicious packages may trigger immediate quarantine/deny; other findings may enter timed review/remediation, never silently waived. Record false positives, independent adjudications, exception expiry, compensating controls and revalidation triggers.

## Propagation and contributor safety

~~~mermaid
flowchart LR
  Feeds["CVE, OSV, GHSA, KEV, EPSS, malicious packages"] --> Normalize["Normalize, dedupe and timestamp"]
  Normalize --> Index["Versioned dependency/SBOM impact graph"]
  Index --> Policy["Risk policy and signed blocklist delta"]
  Policy --> Coordinator["Task eligibility, affected projects, leases"]
  Coordinator --> Daemon["Authenticated local daemon policy"]
  Daemon --> Sandbox["Task admission / scoped terminate or quarantine"]
  Index --> Notify["Maintainers, reviewers and public-safe alerts"]
~~~

A critical updated assessment identifies affected projects/repositories/tasks/builds; the coordinator blocks new claims and sends authenticated revocation/stop instructions to impacted daemons. The trusted local supervisor independently enforces signed cached policy, timeouts, sandbox boundaries and an offline freshness deadline. Disconnected devices must not acquire indefinite execution permission: expiry/lease policy bounds residual work, and stale policy blocks *new* claims. Never claim instantaneous remote termination of an offline builder.

Do not run repository-controlled workflow files with credentials on a contributor host. Repository CI adds its own checks; the daemon independently qualifies execution, and the coordinator verifies trusted status/evidence attached to the *exact candidate/base* before protected PR acceptance. High-risk changes should invalidate affected approved architecture and task revisions through change-impact assessment; unaffected work may proceed.

## Data model and evaluation gates

Proposed entities: `Project`, `ProjectRepository`, `ArchitectureRevision`, `ProjectCertification`, `SBOMSnapshot`, `ComponentInstance`, `DependencyEdge`, `VulnerabilityAdvisoryRevision`, `SecurityAssessment`, `PolicyDecision`, `SecurityBlocklistRevision`, `ChangeImpactAssessment`, `TaskAdmissionReceipt`.

Mandatory tests: resolved transitive vulnerability not listed in manifest; same package multiple ecosystems/versions; false range and retracted CVE; new malicious package without a CVE; changed lockfile and unchanged repo commit but new advisory; affected/multiple repositories and independent project rights; signed policy rollback/expiry; offline builder lease expiry; malicious repo workflow claiming checks passed; private vulnerability report leaking through a public Warboard; remediation and scoped release of a block.

Before calling it production-ready, measure feed coverage/freshness, false positives/negatives, matching accuracy, coordinator update latency, and propagation time on qualified networks. A static SBOM plus a live advisory feed are complementary, not interchangeable.

Related: [security model](../security/threat-model.md), [planning architecture](../architecture/planning-and-orchestration.md), [project boundaries](../architecture/project-boundaries.md), and [implementation plan](../plans/2026-11-04-platform-implementation.md).
