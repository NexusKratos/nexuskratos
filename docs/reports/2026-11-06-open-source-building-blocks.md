# Open-source building blocks survey — 2026-11-06

**Status: research record, not an adoption decision.** No dependency has been added and no code exists. Nothing here was installed or run; the findings come from reading repositories, license files and publisher documentation. T01 (feasibility) must re-verify the exact versions it adopts and run the adversarial fixtures before any choice becomes real.

**Filters applied:** licenses on the allowed list in [ADR 0004](../adr/0004-dependency-license-policy.md), a strong preference for Rust, and active maintenance as of 2026-11-06. Licenses marked *verified* were read from the repository's license file through the GitHub API, not from badges.

## 1. Provider terms: the constraint that shapes everything else

Both publishers' documentation was read directly on 2026-11-06.

**OpenAI Codex** ([CI/CD auth guide](https://learn.chatgpt.com/docs/auth/ci-cd-auth)). The guide calls ChatGPT-account automation "an advanced workflow for enterprise and other trusted private automation". It also says: "Do not use this workflow for public or open-source repositories", "Do not share the same file across concurrent jobs or multiple machines", and "The right way to authenticate automation is with an API key." **Consequence: Codex adapters use API keys only.**

**Anthropic Claude Code** ([legal and compliance](https://code.claude.com/docs/en/legal-and-compliance)).

Restrictions:
- "Advertised usage limits for Pro, Max, and Ultra plans assume ordinary, individual usage of Claude Code and the Agent SDK."
- "Anthropic does not permit third-party developers to … route requests through Free, Pro, Max, or Ultra plan credentials on behalf of their users. Moreover, developers may not collect, store, or intermediate Claude.ai credentials or session tokens."

Permissions:
- The same page states the policy does not "prevent an end user from signing in to the unmodified Claude Code binary with their own Claude subscription, including where a platform hosts Claude Code".
- Own API keys are fine if "billed to the key owner" and "not resold or intermediated".
- The page names contacting sales for questions about permitted authentication.

The consumer terms (not re-read in this pass) were reported to bar automated access except through an API key or where explicitly permitted.

**Consequences:**

1. API keys, or Bedrock or Vertex credentials, are the first-class path for both tools.
2. Subscription use, if offered at all, must be the contributor signing in to the unmodified binary themselves, as an explicit opt-in. Whether pooled community work counts as "ordinary, individual usage" is unresolved and needs written confirmation from Anthropic.
3. Subscription mode conflicts with the credential boundary. The clean isolation pattern is a trusted proxy that injects the real credential so the sandbox holds only a placeholder. That is fine for an API key the contributor owns, but for a subscription session token it would be "intermediating" it. In subscription mode the login therefore has to live inside the environment that runs untrusted repository code. That mode must be labelled as weaker isolation.

## 2. Agent memory

**Direction:** a small coordinator module of our own. No framework fits community-scoped, untrusted memory on PostgreSQL.

| Candidate | License | Language | Notes |
| --- | --- | --- | --- |
| pgvector + pgvector-rust | PostgreSQL License (extension); MIT/Apache (Rust client) | C, Rust | Fits the proposed PostgreSQL store. Row-level security can isolate communities. Eligible under ADR 0004. |
| fastembed-rs on `ort` | Apache-2.0; MIT/Apache | Rust | Local ONNX embeddings. `ort` 2.0 is still a release candidate, and model-weight licenses are separate. |
| rig (`rig-postgres`) | MIT | Rust | pgvector store plus LLM toolkit. Pre-1.0 with fast churn. |
| ai-memory (akitaonrails) | MIT | Rust | Design reference for coding-agent handoffs and prompt-injection notes. SQLite and single-writer, so not embeddable here. |
| xerj | Apache-2.0 (verified) | Rust | Search, MCP and memory engine with an Elasticsearch-compatible API, at release candidate rc.101. A separate service, and its `--insecure` dev mode has no auth. It could be a **contributor-local** context-retrieval tool. Not suitable as the community memory store. Commits are concentrated in one account, and its benchmark claims are unverified. |
| LanceDB, Qdrant | Apache-2.0 | Rust | Strong, but a second store or service beside PostgreSQL. |

Python frameworks (mem0, Letta, Graphiti, cognee) pass the license filter but do not provide community isolation or untrusted-memory handling. basic-memory is AGPL and excluded.

**Shape:**
- Rows carry `community_id`, repository, scope, source task and commit, and a trust level that is always untrusted for agent-written content.
- Retrieval combines full-text and vector search.
- Retrieved memory is injected only as quoted data.
- Promotion to trusted goes through review.

## 3. Sandboxing on contributor machines

| Candidate | License | Language | Platforms | Notes |
| --- | --- | --- | --- | --- |
| nono (nolabs-ai) | Apache-2.0 (verified) | Rust | Linux, macOS, WSL2 | Landlock + seccomp, a domain-allowlist proxy and placeholder-token credential injection. Its SECURITY.md says guarantees are "not yet stable" and production use is "not recommended". v0.81.0. |
| Strands Box (strands-agents) | Apache-2.0 (verified) | Rust | **Linux ARM64 and macOS Apple silicon only** (preview) | OS isolation plus default-deny Dogwood policies across shell, Python, egress gateway and MCP broker. Credential injection keeps secrets outside the agent, and it produces decision records. Usable as a Rust library. Strongest policy design seen, but **no x86-64 Linux build**. v0.2.1, created 2026-10. |
| sandlock (multikernel) | Apache-2.0 (verified) | Rust | Linux 6.12+ only | Landlock + seccomp user-notify, method/host/path HTTP ACL, copy-on-write filesystem, limits without cgroups. No credential injection found. The kernel floor excludes many distributions. v0.9.1. |
| microsandbox | Apache-2.0 (verified) | Rust | Linux with KVM, macOS | microVM tier with host-side secret substitution. Needs `/dev/kvm`, and nested virtualization on WSL2 is uncertain. v0.7.13. |
| Codex sandbox crates (openai/codex) | Apache-2.0 | Rust | Linux, macOS | Best reference code (bubblewrap + seccomp, network proxy), but internal crates with no stable API. |
| `landlock` crate | MIT OR Apache-2.0 | Rust | Linux | Primitive for a custom executor. |
| Bombini | Apache-2.0 (verified, top level) | Rust (Aya eBPF) | Linux 6.2+ with BPF LSM, x86-64 image | eBPF LSM monitoring and in-kernel blocking. Requires root or privileged containers and BPF LSM enabled at boot, so it is not suitable on donor machines. A possible defence-in-depth layer for operator-run hosts. The license of its eBPF program objects was not checked. |
| Firecracker / Cloud Hypervisor | Apache-2.0 | Rust | Linux with KVM | A later microVM tier. More assembly needed than microsandbox. |

Excluded: bubblewrap (LGPL-2.1, pending under ADR 0004 as a separate executable), ai-jail and birdcage (GPL-3.0), and uvm (Apache-2.0, a hobby bytecode VM that cannot run real repository toolchains).

**Direction:**
- A trusted Rust supervisor holds credentials and runs the egress proxy.
- The AI CLI runs entirely inside a Landlock + seccomp + namespace sandbox with a placeholder token, a clean environment, cgroup limits and process-tree kill.
- A microVM tier is selectable for untrusted communities.
- Repository-supplied agent configuration (settings files, hooks, MCP definitions) is treated as hostile.
- Box is the design to watch. If it ships x86-64 Linux, it becomes the leading candidate.

## 4. Headless CLI execution

| Candidate | License | Language | Notes |
| --- | --- | --- | --- |
| rust-code-agent-sdks (`claude-codes`, `codex-codes`) | Apache-2.0 (verified) | Rust | Typed output-format parsing for Claude Code, Codex and others, with version-drift warnings. 4 contributors and 28 stars. No process supervision. |
| Codex app-server (openai/codex) | Apache-2.0 | Rust | Native JSON-RPC path for Codex. Some methods are marked experimental. |
| Agent Client Protocol (Rust SDK) | Apache-2.0 | Rust | Possible future uniform protocol. Usage reporting not confirmed. |
| oneharness, agent-abstraction | MIT | Rust | Single-maintainer references for flag mappings. |

Excluded:
- The Claude Code binary and the TypeScript Agent SDK are proprietary. Claude Code is a user-installed external tool (ADR 0004).
- claude-squad is AGPL.
- coder/agentapi is archived.

The owner's hexe overlaps this area most fully, but it is proprietary. It can be used only if the needed parts are released under an allowed license (ADR 0004).

## 5. Planning and orchestration

**Direction:** build both halves.

**Planning:** no open project combines an AI-drafted dependency graph, human editing and a human publish gate. Borrow formats and ideas from these, all MIT and verified:
- Spec Kit
- Backlog.md
- Beads (Go)

Task Master AI is excluded because the Commons Clause blocks hosting it.

**Execution:** a lease table with an epoch column, where every write checks the current epoch in the same transaction as its outbox row. Candidates assessed:

| Candidate | License | Language | Notes |
| --- | --- | --- | --- |
| apalis 1.0 | MIT | Rust | Mature queue with workflows. Owns its schema; epoch fencing unverified. |
| Underway | MIT OR Apache-2.0 | Rust | Unreleased stale-attempt fencing on its main branch. A design reference. |
| graphile_worker_rs | MIT | Rust | Fine for peripheral jobs. |
| Rivet | Apache-2.0 (verified) | Rust engine, TypeScript-first SDK | Actor runtime backed by S3 (no database or message bus). It "separates the head from the hands": the agent loop runs outside the sandbox. A different storage model from the proposed PostgreSQL coordinator, with no fencing against untrusted remote workers. A pattern reference, not a v1 fit. Its JavaScript/Python agentOS sandbox cannot run arbitrary toolchains. |
| Temporal, Hatchet | MIT | Go/Rust SDK | Add a separate cluster or service. |

Restate (BUSL server), Windmill (AGPL) and Obelisk (AGPL) are excluded.

## Also checked and excluded

- **fletch (fwdai):** AGPL-3.0 (verified). It is an agentic IDE, not a building block.

## Not verified

- No candidate was installed or run against Claude Code or Codex.
- Process supervision inside `claude-codes` and `codex-codes`.
- Epoch fencing inside apalis and Underway.
- Licenses of individual crates inside multi-crate repositories beyond the root license file.
- Licenses of model weights.
- OpenAI consumer terms text (fetch blocked).
- Strands Box x86-64 plans.
- Bombini eBPF object licensing.
