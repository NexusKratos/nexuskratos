# Handoff — license policy, building-blocks research, open-work view (2026-11-12)

Written at a usage-limit seam (Michel, relayed 2026-11-12: save context before the 04:00 reset). Nothing is running; no destructive action is pending.

## Repository state

- Branch `docs/dependency-license-policy-20261106`, pushed, open as [PR #8](https://github.com/NexusKratos/nexuskratos/pull/8). Base `main` at 875ca68 (PR #7 merged). Main requires PR + review; **merge is Michel's call.**
- Commits on the branch: 1654726 (ADR 0004 + survey, `checkpoint/0.0.5`), c7c21a2 (license extension, `checkpoint/0.0.6`), and this handoff (`checkpoint/0.0.7`).
- No code, dependency, deployment or provider tool was added or run. Implementation is still on hold.

## Verified done

- [ADR 0004](../adr/0004-dependency-license-policy.md), accepted by Michel: allowed `MIT`, `Apache-2.0` (incl. LLVM exception), `BSD-2-Clause`, `BSD-3-Clause`, `0BSD`, `ISC`, `PostgreSQL`, `Unicode-3.0`, `Zlib`, `CDLA-Permissive-2.0`; `MPL-2.0` **unmodified only** (owner decision 2026-11-07). Licenses verified from the real license file per exact version.
- [Building-blocks survey](../reports/2026-11-06-open-source-building-blocks.md): memory, sandboxing, headless CLIs, orchestration; provider terms (Codex automation API-key only; Claude subscription use for community work unresolved); seven owner-supplied repositories assessed.
- T01's future ADR reference changed from `0004-*` to the next free number.

## Not yet recorded in the repository (from conversation, 2026-11-07)

**Opengrep** (github.com/opengrep/opengrep), assessed read-only for the T20 scanner gate:
- Engine: **LGPL-2.1** (read from LICENSE), OCaml, Semgrep CE fork (Dec 2024), consortium-backed, active (v1.33.1 on 2026-10-29), Cosign-signed static binaries for Linux x86-64/ARM, macOS, Windows; JSON and SARIF output; intrafile cross-function taint.
- Fit: the SAST adapter, run as a **separate executable** in the isolated scanner sandbox. Eligible only if Michel allows GPL/LGPL tools as separate programs (pending decision 1 below).
- Rules are the real blocker: `opengrep/opengrep-rules` is Commons Clause (excluded) and near-dead; Semgrep registry rules carry the Semgrep Rules License (excluded); Trail of Bits rules AGPL-3.0 (excluded); elttam/semgrep-rules is MIT (allowed, small). A NexusKratos-owned rule pack under an allowed license would be needed.
- Not run or benchmarked. Suggested follow-up: add it to the survey report's scanner section once the GPL decision is made.

## Open owner decisions (also in BACKLOG)

1. GPL/LGPL tools as separately executed programs (bubblewrap, Opengrep).
2. Model-weight license policy.
3. Ask Anthropic in writing whether contributor subscriptions may serve community work.
4. Whether the needed parts of hexe can be released under an allowed license.
5. Merge PR #8 and, separately, lift the implementation hold (T01).

Hex-proposal questions surfaced as decisions in the open-work view (not yet in BACKLOG): identity of "Desk 4"; browser-only vs virtual desktop for the first release; microVMs for high-risk projects; soft budgets where no hard cap exists.

## Artifacts (outside the repository)

- Design walkthrough, updated to the 35-task plan, Hex proposal and license policy: https://claude.ai/code/artifact/a6002be5-244c-424b-bdce-bdd243f0c265 (shared by link).
- Open-work register, 49 items by category with status and dependencies: https://claude.ai/code/artifact/4aeb1386-f7aa-4ff9-bb33-0fc116b9c85a (private).
- Local sources were in this session's scratchpad (temporary); the published pages are the durable copies.

## Next steps, in order

1. Michel: decide GPL/LGPL-as-separate-program (unblocks bubblewrap and Opengrep).
2. Michel: merge PR #8.
3. If allowed, add Opengrep to the survey and BACKLOG; otherwise record it as excluded.
4. On Michel's go: start T01 feasibility.
