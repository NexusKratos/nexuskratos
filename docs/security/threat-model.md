# Nexus Kratos security architecture and scanner strategy

> **Proposed security controls — not yet implemented.** Public design; private vulnerability disclosures belong in the existing SECURITY.md process. Do not put exploit details, live secrets or exploitable deployment-specific configuration in this document.

## Security objectives and adversaries

Protect: donors' devices and data; provider tokens, account quota and budgets; GitHub repositories and maintainers; project trust/identity; software supply chain; coordinator and database; contribution evidence; private project metadata and logs.

Assume a hostile project suggestion, repository, PR, dependency, prompt, generated output, artifact, review or compromised contributor. An approved project is still untrusted *execution content*. Treat prompt injection as untrusted instructions; no model output is a permission grant.

Threats to validate: remote code execution, credential exfiltration, prompt-induced action bypass, poisoned toolchain/dependencies, malicious Git hooks, unbounded resource/cost consumption, symlink and path traversal, spoofed worker receipts, replayed expired leases, self-review and forged test status, webhook spoofing, CI cache/runner poisoning, compromised updates and badge manipulation.

## Trust zones

~~~mermaid
flowchart LR
  Community["Marketplace / user submissions (untrusted)"] --> API["Control API: identity, RBAC, rate limits"]
  API --> DB[("Transactional DB, audit + outbox")]
  API --> Coordinator["Task scheduler + policy"]
  Coordinator <-->|"authenticated, short-lived scoped protocol"| Supervisor["Trusted local supervisor"]
  Supervisor --> Broker["Runner/provider adapter; secrets stay here"]
  Supervisor --> Sandbox["Untrusted repository executor"]
  Sandbox -. "no host secrets, no merge token" .-> Broker
  Supervisor --> Evidence["Bounded artifacts + evidence"]
  Evidence --> Scan["Independent scans + QA + reviews"]
  Scan --> GitHub["Scoped GitHub App and protected PR"]
~~~

**Non-negotiable:** the GitHub publication/merge credential and private provider credentials are never passed into the untrusted repository executor. A Git worktree is not a sandbox. A container with the host Docker socket or ambient home directory is not a security boundary.

## Layered scanning pipeline

| Phase | Candidate controls | Enforcement / evidence |
| --- | --- | --- |
| Project onboarding | License and repository provenance checks, known-malware/abuse triage, dependency health, maintainership assessment | Human/community approval before discoverable autonomous work |
| Task admission | Task schema validation, allowed file/path/network profile, model/provider allowlist, scoped roles, risk tier | Reject invalid task or require explicit donor consent; no repository-supplied override |
| Before execution | Pin revision/toolchain, fetch integrity, trusted tool provenance; scan manifests, embedded scripts and suspicious files | Quarantine or block high-risk content before enabling an execution profile |
| During execution | OS sandbox, syscalls/privilege restrictions where qualified, CPU/RAM/disk/runtime quotas, egress allowlist, complete child-tree cancellation | Resource/kill receipts; security event triggers isolation and revocation |
| Candidate publication | Secret scanning (e.g. Gitleaks), SAST (e.g. Semgrep/CodeQL where supported), dependency vulnerability scan (e.g. OSV), IaC/container scan (e.g. Trivy), license policy, malicious paths/hooks check | Per-tool versioned machine-readable findings bound to candidate digest |
| Independent validation | Isolated QA commands, review from a distinct authenticated contributor, static checks, optional fuzz/security tests | No self-certification; raw bounded receipts tied to exact commit |
| Protected PR/merge | Required security checks, unresolved finding policy, branch protections, change-of-base/head revalidation, maintainer authorization | Merge denied if policy is unmet or evidence became stale |
| Release distribution | Signed provenance, reproducible/checksummed release artifacts, SBOM (e.g. Syft), dependency update policy, release scanning | Install verifier refuses unknown/unsigned/untrusted artifacts |

Tool names are examples for evaluation, not claims of integration, guaranteed accuracy or endorsement. Scan availability differs by language, license and hosted/self-hosted environment; use a maintained capability matrix. Scanner input is untrusted; run the scanners in isolated workers without donor secrets and with strict limits.

**Severity policy proposal:** block releases/merges for verified critical/high findings in reachable production paths; permit bounded, justified, time-limited exceptions with named maintainer/security approval and recorded compensating controls. Never suppress a finding merely because a model says "false positive." Require secret revocation even after a leak is removed.

## Execution isolation

- Isolated, task-specific workspaces with filesystem mount allowlists; no inherited home, SSH agent, cloud metadata or Docker socket; no arbitrary host IPC.
- Network denied by default where feasible; enable only destination classes required for declared dependency fetches or provider service calls through a trusted broker. Unrestricted outbound access invalidates exfiltration claims.
- Separate supervisor/provider-adapter execution from repository commands. Test real provider CLI behavior *before* committing to a boundary. An unsafe adapter remains disabled.
- OS-native resource and process-tree termination; enforce deadlines and revoke process capability before reporting cancellation as complete.
- Host-platform support is conditional on passing the same isolation, cancellation and recovery tests; Linux first is a validation proposal, not a guarantee.
- Secure auto-update: verify signer, version/rollback policy and artifact integrity; prompt for privilege escalation rather than silently elevating.
- Avoid passing GitHub write credentials to donors; validate and import bounded patches centrally, preventing unsafe paths, ref names and repository hooks.

## Publisher signing, registry authority, and independent architecture review

An OS-signed Windows or macOS release, notarized macOS binary, or authenticated Linux artifact verifies its publisher/integrity, **not the safety of every task**. Separately verify official registry identity, signed trust/policy metadata and scoped task authority using independent trust roots. The system must handle key purpose separation, rotation, revocation, expiry, malicious forks claiming "Official", replayed signatures, and stale offline trust data. HTTPS domain identity alone cannot confer official-registry status.

An approved architecture revision should be reviewed by **multiple genuinely independent qualified high-capability model reviewers**, opted into the Architecture Reviewer role. Conflict screening must exclude project-connected reviewers. The qualified reviewer may donate bounded model capacity in the free queue or be an operator-managed reviewer funded by a paid service, but both routes share the identical security and architecture admission gate. Signed review receipts identify exact architecture digest, reviewer's identity, approved model/version, findings and policy; a valid signature authenticates a claim, not the truth of its finding. See [reviewer design](../architecture/independent-architecture-review.md) and [trust/cost recovery](../architecture/trust-and-sustainability.md).

Public repository GitHub Pages Warboards must receive **only explicit sanitized data** from read-only exports, and repo publish workflows cannot inherit coordinator or donor credentials. Never expose private builder/device details or embargoed findings. The optional premium hosted Warboard is a visualization enhancement and has no merge, review or security bypass authority. See [Warboard delivery](../architecture/warboard-delivery.md).

## Protocol and coordinator

Device pairing uses a scoped, revocable device identity; HTTPS/mTLS or equivalent authenticated transport with rotation, replay protection and rate limits. Role grants are project/community-scoped; every object access is authorization checked server-side. Use idempotency keys, atomic lease-epoch fencing, expirations, immutable task revisions, secure webhooks with signature validation and reconciliation.

After restoring state, issue a new random installation incarnation and invalidate old worker authority even if the database rollback restored old lease epochs. Treat network partitions as uncertain execution, not permission to double-assign or claim completion.

## Budget/abuse protections

Enforce aggregate local concurrency, CPU/RAM/disk/time admission across all projects and reserve capacity before execution. Separate **measured API charges**, **estimates**, **subscription allowances**, and **unknown**. If a provider cannot guarantee a hard dollar/token cap, do not advertise one. Apply API throttling and queue fairness to prevent donation capture or denial of service. Guard project suggestions, signups, reputation and badges against spam/sybil attacks.

## Privacy and observability

Collect the minimum identity/device metadata needed to coordinate and audit; redact paths, prompts, tokens and raw logs by default. Use explicit retention and deletion/export options, compartmentalize private projects, and protect notification channels. Telegram/email must not include sensitive logs, patches or provider keys. Warboard consumes versioned, redacted events and remains optional.

## Required red-team / QA gates before real donated work

Attempt host-secret escape, malicious shell hooks, symlink traversal, exfiltration over allowed network, namespace/container breakout, spoofed review, stale lease replay, duplicate webhook, unbounded subprocess, resource exhaustion, provider-login expiry, update tampering, crafted scan outputs, restored-database replay and compromised GitHub installation tokens. Document threat, fixture, expected block, measured outcome, residual risk and responsible owner.

Read the existing [public security policy](../../SECURITY.md) for private disclosure channels. This document describes desired defense in depth; it does not replace incident response or the repository's existing branch/security controls.

## Continuous vulnerability assessment

One-time onboarding scans are not permanent approval. Require complete-enough commit/build-bound SBOM snapshots (including transitive dependencies), daily plus urgent advisory refreshes, and impact-scoped signed block decisions when a vulnerable or malicious component is detected. The coordinator blocks affected new claims; the local supervisor enforces its own sandbox and security policy. Offline execution cannot be stopped instantaneously and must be bounded by leases and policy freshness. See [continuous supply-chain intelligence](continuous-supply-chain.md).
