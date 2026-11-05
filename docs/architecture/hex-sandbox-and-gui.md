# Proposed Hex-controlled multi-harness and GUI execution

> **Status: design discussion (2026-11-05), not an implementation decision.** This is a capability proposal for Nexus Kratos, not proof that Hex currently provides these protections. Do not install provider CLIs, run GUI tasks, expose a desktop, or accept untrusted projects until qualification is complete.

## Decision direction

Use a **single typed Nexus Kratos execution contract** with **multiple qualified harness adapters**. A provider CLI (OpenCode, Claude Code, Codex, or another supported native tool) is a replaceable adapter, not the policy or security boundary. Neither the CLI nor a GUI is trusted to enforce project admission, limits, leases, cancellation, GitHub write authority, or provider credential protection. The owner proposes **Hex** as the common controlled executor; its actual isolation guarantees and integration points must be verified before selection.

A task requests capabilities, not an arbitrary command line: `cli.headless`, `browser.headless`, `gui.virtual-desktop`, `human-assisted`, required model/provider/harness and platform. The local daemon matches the task against *both* approved platform policy and the donor's explicit opt-in and resource limits. Unsupported capability combinations are rejected, never silently downgraded.

## Containment boundary

~~~mermaid
flowchart TB
  Coordinator["Coordinator: signed task policy / fenced lease"] --> Daemon["Trusted local Rust daemon"]
  Donor["Donor local UI: consent, quotas, pause / revoke"] --> Daemon
  Daemon --> Select{"Qualified task capability?"}
  Select --> CLI["Version-pinned CLI harness adapter"]
  Select --> Browser["Browser automation adapter"]
  Select --> GUI["Virtual desktop / GUI adapter"]
  CLI --> Hex["Hex-managed isolated task environment"]
  Browser --> Hex
  GUI --> Hex
  Hex --> Workspace["Untrusted repo checkout + bounded artifacts"]
  Hex --> Receipts["Process, scan, resource and evidence receipts"]
  Broker["Trusted credential / provider broker"] -. "only explicitly authorized scoped calls" .-> CLI
  Broker -. "only explicitly authorized scoped calls" .-> Browser
  Broker -. "only explicitly authorized scoped calls" .-> GUI
  Receipts --> Daemon
  Daemon --> Coordinator
~~~

The **trusted side** contains device identity, provider credentials, lease journal, resource authority, audit records, safe UI access, secrets and GitHub permissions. The **untrusted side** contains all task code, dependency installers, model-suggested commands, subprocesses, browser pages, screenshots and GUI applications. No host home mount, personal session, SSH agent, cloud credentials, socket, clipboard or arbitrary host service is available to untrusted executors.

**Important caveat:** Running a provider-native CLI in the untrusted environment may require giving that process authentication material. That conflicts with the no-credentials boundary unless a provider-supported broker/delegation mode is proven. Each adapter must independently demonstrate the permitted authentication flow. Where impossible, mark it unsupported; do not move donor credentials into the workspace merely to make a demo run.

## GUI support: three separate modes

1. **Headless browser** (initial candidate): Playwright or equivalent in an isolated browser/process namespace; record safe screenshots and artifacts only when permitted. Prefer this over a full desktop for web tests and UI checks.
2. **Virtual desktop** (higher cost/risk): a dedicated ephemeral graphical session per attempt, such as isolated Wayland/Xvfb/virtual display with a narrowly scoped, authenticated viewer. Never control or record the donor's actual desktop. Disable host clipboard, shared folders, system D-Bus, USB passthrough and unrestricted local sockets.
3. **Human-assisted session** (explicit opt-in): local UI shows a pending-attention request with task identity, viewer session and deadline; authenticated donor can observe/approve or interact. The task is paused or cancelled on timeout and must not silently continue with a privilege increase.

A virtual display is **not** by itself a sandbox. The executor must additionally enforce process/user namespaces or equivalent OS isolation, filesystem and network policy, CPU/RAM/disk/time quotas, process-tree kill, browser egress restrictions and a controlled artifact bridge. A Linux-first implementation is a proposal only. macOS/Windows require separately qualified backends; feature availability must be truthful per platform.

## Isolation, security and observability requirements

- Treat project manifests, pages, generated screenshots and tool output as hostile data, including prompt injection and exfiltration attempts.
- Deny host network and local management interfaces by default. Narrow, task-scoped outbound access needs explicit allowlists and redacted audit. Protect local metadata and cloud credential endpoints.
- Read-only executable images / pinned versions, signed release/provenance verification, dependency and secret scanning, known-vulnerability gating, and an ephemeral workspace. Scan outputs as untrusted.
- Separate privileged supervisor and renderer; expose GUI viewer only to a locally authenticated, short-lived channel, with session expiry and CSRF/origin protections.
- Per-run budgets: CPU, RAM, disk, concurrency, duration, model credits when enforceable, GUI/video bandwidth and capture retention. Unknown subscription usage must be labelled unknown, not reported as a guaranteed hard cap.
- Preserve lease epoch and process lineage across start, cancel, reconnect and crash. On revoked/expired lease, kill or quarantine all children, close viewer, revoke network/credential grants and retain bounded evidence for reconciliation.
- Telemetry: current capability/mode, harness/provider version, resource consumption, progress, explicit attention, start/stop reasons, signed or digested artifact/test/scan receipts, and redacted audit. Optional Warboard remains read-only and must not receive private screenshots or secrets.
- No direct contributor/agent merge authority. Submitted patches are validated and independently reviewed against an exact candidate/base before a scoped GitHub App publishes or accepts them.

## Qualification matrix (must test, not assume)

| Candidate | Primary task type | Key qualification blocker |
| --- | --- | --- |
| OpenCode adapter | Headless CLI / broad provider selection | Credential separation, pinned adapter/tool permissions, durable structured receipts |
| Claude Code adapter | Provider-native CLI | Supported unattended auth/terms, model controls, tool permission interception, cancellation |
| Codex adapter | Provider-native CLI | Supported unattended auth/terms, sandbox coordination, control of execution/tool calls |
| Browser automation | Headless web/UI validation | Isolation of browser and downloads, egress policy, artifact privacy, deterministic replays |
| Virtual GUI desktop | Apps requiring display | Display/session isolation, remote viewer auth, graphics dependencies, resource caps, full cleanup |

**Evaluation scenarios:** benign compile/test; malicious project attempts to read host secrets; malicious browser reaches localhost; subprocess escape/orphan after cancellation; provider authentication expiry; GPU/desktop denial-of-service; guest crashes; stale lease; screenshot privacy; network disconnect; poisoned artifacts; GUI viewer reconnect/replay. A failed security boundary blocks production support for that mode.

## Deliverables / acceptance criteria

- [ ] Define versioned `ExecutionCapability`, `HarnessAdapter`, `SandboxBackend` and `AttentionSession` contracts with clear adapter versus security responsibilities.
- [ ] Document a Hex feature inventory from a real prototype and independently qualify it against OS isolation, policy, credentials, artifact and kill requirements.
- [ ] Implement one safe Linux CLI fixture with untrusted source and no host credentials, full cancellation and evidence capture; prove isolation adversarially.
- [ ] Implement a *separate* headless-browser fixture with restricted network, no host desktop access and safe artifact retention.
- [ ] Evaluate a virtual-GUI prototype only after CLI/browser safety gates pass; require donor opt-in, viewer auth, resource caps, session expiry and isolated display.
- [ ] Publish a capability/OS/provider compatibility matrix and reject unsupported tasks before leasing them.
- [ ] Add immutable security and review receipts to the existing task/lease/PR acceptance workflow; preserve donor kill switch and optional Warboard visibility.

## Questions needing owner / engineering evidence

- What exact Hex runtime and interface will the daemon supervise? Is “Desk 4” a specific Hex component, version, or another product?
- Can all desired provider CLIs authenticate through a supported trusted broker without exposing full credentials to the untrusted executor?
- Do we need *interactive desktop GUI automation*, or only browser-based UI testing for the first usable release?
- Are Linux namespaces/seccomp/LSM sufficient for the chosen threat tier, or should high-risk marketplace projects require microVMs (e.g. Firecracker) instead?
- Can any supported provider or GUI workflow meet enforceable hard spend limits? If not, how should users opt into estimated or soft budgets?

Related: [system architecture](system.md), [workflow diagrams](diagrams.md), [brainstorming](../brainstorming/2026-11-05-design-notes.md), [threat model](../security/threat-model.md), [implementation plan](../plans/2026-11-04-platform-implementation.md).
