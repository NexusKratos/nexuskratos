# Repository security baseline

Date: 2026-11-04. Status: accepted for configuration.

## Context

The owner requested repository security for the new public platform and website repositories. Each currently has one administrator, no other maintainers, no existing rulesets, and no custom workflows. A practical baseline must protect contributions without requiring a nonexistent second maintainer or a nonexistent CI check.

## Decision

Require pull requests to the default branch, one approving review for ordinary contributors, dismissal of stale approvals, resolved review conversations, and merge commits. Repository administrators may bypass the approval gate only through a pull request; they cannot bypass it by directly pushing. This is an explicit solo-maintainer exception, not independent review evidence.

Keep separate active no-bypass rules preventing default-branch deletion/force-push and checkpoint/release tag updates/deletion. New tags remain possible; protected names are `checkpoint/*` and `v*`.

Enable secret scanning and push protection, Dependabot alerts/security updates, and private vulnerability reporting. Actions use read-only default tokens, cannot approve PRs, require approval for all external contributors, require full-SHA action pins, and allow GitHub-owned actions only until a maintainer deliberately adds a vetted exception.

Enable GitHub's default CodeQL setup for the website JavaScript using the standard query suite. The documentation-only platform has no supported code to scan yet. Do not require nonexistent CI statuses or claim a configured scan is a completed scan.

## Alternatives rejected

Requiring a second person's review without an administrator PR exception would block the sole maintainer. Allowing an administrator to bypass rules on direct pushes would remove the PR audit trail. Blocking all new tag creation would break the documented checkpoint process. Adding application tests or a deployment workflow is outside this settings task.

## Consequences

All future main-branch changes, including agent changes, use feature branches and PRs. Source pushes still do not deploy the website. Checkpoints are preserved with merge commits rather than squash/rebase. Adding third-party Actions requires a deliberate policy update. Organization membership, organization-wide 2FA enforcement, and Cloudflare settings are unchanged.
