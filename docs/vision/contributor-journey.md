# Contributor journey and operational states

> **Proposed experience, not implemented.** The target is a zero-GitHub onboarding path while retaining auditable GitHub-based project delivery.

## First-run workflow

~~~mermaid
flowchart LR
  A["Install signed daemon"] --> B["Open local web UI"]
  B --> C["Sign in and pair device"]
  C --> D["Browse approved projects"]
  D --> E["Opt in to projects and roles"]
  E --> F["Set enforceable limits and trust"]
  F --> G["Review permissions"]
  G --> H["Enable automatic contribution"]
  H --> I["Live status, pause, alerts"]
~~~

**First-run UX:** provide sane conservative defaults, supported-platform checks, clear per-provider login steps, a permission preview and a dry-run/self-test before the first real task. Do not advertise a hard monetary spending cap if the provider cannot enforce one. Never start a task simply because the user viewed a project.

### Local settings, per device

| Control | Initial behavior | Important rule |
| --- | --- | --- |
| CPU, memory, disk | Explicit caps and idle/quiet-hour policy | Executor enforces OS limits; validate support per platform |
| Concurrency | Global maximum plus per-project allocation | Reservations cover in-flight and uncertain work |
| Runtime/schedule | Per-task deadline, daily availability, pause | Local off-switch must work during coordinator outage |
| Provider/model | Explicitly approved tools, models and capability tiers | Never silently switch provider or model |
| Usage | Tokens/actual API charges where measurable; estimates separate | Subscription and approximate usage cannot be hard-guaranteed |
| Trust | Per-project allowlist, network and filesystem policy | No project can relax host security by task instruction |
| Notifications | Opt-in dashboard/email/Telegram; per-category preferences | Redact secrets and sensitive task content |

## Autonomous contribution cycle

~~~mermaid
sequenceDiagram
  actor Donor
  participant UI as Local UI
  participant Agent as Trusted daemon
  participant Coord as Coordinator
  participant Exec as Isolated executor
  participant Git as GitHub App / PR
  participant QA as Review and QA
  Donor->>UI: Approve projects and limits
  UI->>Agent: Persist policy
  Agent->>Coord: Enroll / advertise capacity
  Coord-->>Agent: Compatible task and fenced lease
  Agent->>Exec: Start constrained attempt
  Exec-->>Agent: Evidence and bounded artifact
  Agent->>Coord: Report/checkpoint/submit
  Coord->>Git: Publish attempt branch and draft PR
  Coord->>QA: Assign independent review and scans
  QA-->>Coord: Candidate-bound receipts
  Coord->>Git: Gate merge through maintainer policy
  Coord-->>Agent: Accepted/rework/blocked status
  Agent-->>UI: Status and recognition
  Donor->>UI: Pause / revoke anytime
~~~

**Interaction model:** headless by default; a task needing human input enters an explicit needs-attention state with reason, deadline, and safe pause/termination. This must not leave a provider process running indefinitely or implicitly approve spending.

## Participation and project state machines

Project: suggested → review → approved → active → paused/archived. Rejection and appeal are recorded.

Contributor device: unpaired → paired → eligible → running → paused/disconnected/revoked. Revoked devices cannot claim or submit authoritative work. Uncertain execution status after a crash stays visible until reconciled.

Task: draft → ready → active → awaiting-review → accepted, with blocked/needs-attention/cancelled and rejected/rework paths. Expired attempts cannot overwrite newer work; prior artifacts remain preserved for audit.

## Visibility and notifications

The local UI is operationally complete without Telegram or email. The hosted dashboard may aggregate multiple devices, subject to explicit data minimization. Show project and task, phase, start/stop, resource reservation, provider/model, actual vs estimated vs unknown usage, exit/scan/test receipts, PR status, blocking reason and actionable next steps. Never send raw provider keys or unrestricted task logs to notifications.

Notification categories: task accepted, review requested, task blocked, attention required, lease expired, quota reached, project security warning, security revocation and milestone recognition. Allow mute and quiet hours, but security-critical notices remain available in the local audit history.

## Definition of done for user experience

A new donor with no GitHub account can install and verify an artifact, create an account, connect an allowed provider locally, browse projects, opt into one, specify caps, finish a safe simulated task, view its evidence, pause/revoke, and remove the device. Before claiming production readiness, repeat this with a real isolated task, independent review and an authorized GitHub PR; verify accessibility and failure scenarios.
