# Repository security — 2026-11-04

## What was built

Configured native GitHub security for NexusKratos/nexuskratos. Three active rulesets preserve main history, require reviewed PRs, and prevent checkpoint/release tag rewriting or deletion. Secret scanning/push protection, dependency alerts/security updates, private vulnerability reporting, and restricted Actions permissions are enabled. Merge commits are enabled; squash/rebase merges are disabled to preserve checkpoint ancestry.

## Verification evidence

API readback confirmed every requested setting and ruleset field. Effective rules for main include deletion protection, non-fast-forward protection, and pull requests. No secret was inserted to test detection, and no destructive push was attempted. The website's native CodeQL setup run succeeded and its JavaScript analysis returned empty error/warning fields; that is not a guarantee that all possible vulnerabilities are absent. CodeQL is not configured on the documentation-only platform.

A focused independent Strong-tier review of the frozen intended rulesets found no blocker. Reviewer requested configuration: GPT-6.1 Sol xhigh; actual runtime telemetry was not available. API application/readback was performed by the coordinator, independently of that source-only review. Initial verification used strict JSON equality and flagged additional GitHub defaults, not a failed setting; all requested fields were then checked explicitly. Additional defaults include empty required-reviewer list, disabled dismissal restrictions, and extra approval for unattributed changes enabled.

## Assumptions made

One current administrator, no other maintainers. Require one approval for ordinary contributors, but permit repository administrators to bypass review only through a PR. This preserves a PR trail without pretending a self-merge received independent human approval. No fictitious CI status is required. Only GitHub-owned Actions are currently allowed, with full-SHA pinning; future third-party actions need a deliberate policy change.

## Concerns and observations

Organization-wide membership/2FA and Cloudflare settings are unchanged. No application code changed, so no application build or test suite was rerun. Native scanning is asynchronous and detection has limits. GitHub scanning configuration does not replace task acceptance or independent code review.

## Close-out confirmation and hygiene

Settings documentation is committed on a feature branch, tagged checkpoint/0.0.3, and landed through a protected PR using the documented sole-admin exception. The merge retains checkpoint ancestry. Source pushes do not deploy the website. No local builds, dependencies, fixtures, or servers were created. Native API receipts and the frozen policy are retained outside the repository; owner is the setup coordinator, next review when the repository policy changes. Source branch history is retained with its PR and checkpoint; no history or evidence was deleted.
