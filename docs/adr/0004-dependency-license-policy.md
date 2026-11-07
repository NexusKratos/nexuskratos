# 0004: Dependency license policy

Date: 2026-11-06. Status: accepted (owner decision, 2026-11-06; extended by owner decision 2026-11-07).

## Context

NexusKratos is Apache-2.0 and will ship binaries to contributor machines and to self-hosting operators. The coming feasibility work (T01) will pick third-party crates and tools for agent memory, sandboxing, headless CLI adapters and orchestration. Every one of those dependencies is redistributed with the product, so its license decides whether a community can freely use, modify, host and redistribute NexusKratos.

The owner first limited candidates to MIT and Apache-2.0, then widened the policy to include BSD-style permissive licenses. Two foundational candidates showed why: pgvector uses the PostgreSQL License, and the BSD family is common in mature infrastructure. On 2026-11-07, before this record merged, the owner also allowed three permissive licenses common in Rust dependency trees and allowed MPL-2.0 for unmodified dependencies only.

## Decision

**Allowed** for direct and transitive dependencies, whether linked, vendored or bundled:

| License | SPDX |
| --- | --- |
| MIT | `MIT` |
| Apache License 2.0, including the LLVM exception | `Apache-2.0`, `Apache-2.0 WITH LLVM-exception` |
| BSD 2-Clause and 3-Clause | `BSD-2-Clause`, `BSD-3-Clause` |
| Zero-Clause BSD | `0BSD` |
| ISC (BSD-style) | `ISC` |
| PostgreSQL License (BSD-style) | `PostgreSQL` |
| Unicode License v3 | `Unicode-3.0` |
| zlib License | `Zlib` |
| Community Data License Agreement, Permissive 2.0 | `CDLA-Permissive-2.0` |

A dual- or multi-licensed dependency is allowed when at least one offered option is on this list, and we use it under that option.

**Allowed with a condition: `MPL-2.0`, unmodified only.** A dependency under the Mozilla Public License 2.0 may be used exactly as its upstream publishes it. We do not edit, patch (including Cargo `[patch]` overrides or a vendored copy with changes) or fork any MPL-covered file. Changing one needs a fresh owner decision, because modified MPL files must be published under MPL. Binary releases list each MPL dependency, its version and where its source can be obtained, in the dependency license inventory that ships with them.

Every allowed license still requires keeping its copyright and license notices. The shipped license inventory carries them.

**Not allowed**: AGPL, GPL or LGPL code that is linked or vendored, SSPL, Business Source License (BSL/BUSL), Commons Clause additions, the Elastic License, "fair-code" or sustainable-use licenses, non-commercial licenses, proprietary or "all rights reserved" code, and code with no license.

**Verification rule**: read the license from the actual `LICENSE`/`COPYING` file or the manifest's SPDX field for the exact version adopted. Do not rely on a README badge, a hosting site's detected license, or memory. In a multi-crate repository, check each crate used. Record each new direct dependency's vetting under `docs/reports/`. Once code exists, a license checker in the build enforces this list mechanically; T01 selects and vets that checker.

## Pending owner decisions

These do not fall under "BSD-style permissive" and stay excluded until decided:

- GPL or LGPL tools used only as **separately executed programs**, never linked (for example bubblewrap, LGPL-2.1).
- Any other license T01's inventory finds that is not listed above. T01 brings the exact list to the owner.
- Licenses of **data and model weights** (embedding models, knowledge packs), which are separate from code licenses.

## Out of scope

Provider CLIs that contributors install themselves, such as Claude Code (proprietary) and Codex CLI (Apache-2.0), are external tools governed by their publishers' terms. NexusKratos does not redistribute them. Adapters must not modify or bundle them.

A proprietary tool, including the owner's own hexe, cannot be a required NexusKratos dependency unless the parts used are released under an allowed license.

## Alternatives rejected

- **MIT and Apache-2.0 only.** This would exclude PostgreSQL's own extension ecosystem and many mature BSD-licensed libraries, while adding no protection over BSD-style terms.
- **Excluding MPL-2.0 entirely.** It would force replacing some well-maintained crates. Its file-level copyleft only binds files we change, and the unmodified-only condition means we change none.
- **Allowing modified MPL-2.0 files without review.** Each modification creates a publication duty that should be a deliberate choice.
- **Allowing linked LGPL or GPL code.** This would impose copyleft obligations on self-hosters and distributors and complicate signed binary distribution.
- **Case-by-case review without a list.** This produces inconsistent decisions and cannot be enforced mechanically.

## Consequences

pgvector becomes an eligible candidate. Candidates already found to be AGPL (fletch, basic-memory, Windmill, Obelisk), BUSL (the Restate server) or Commons Clause (Task Master AI) are excluded. The Rust standard dependency tree (`unicode-ident` under `Unicode-3.0`, common certificate bundles under `CDLA-Permissive-2.0`) no longer blocks a first build. The license checker T01 selects must enforce the MPL-2.0 condition, for example by rejecting `[patch]` overrides or vendored edits of MPL crates. T01 must produce a dependency license inventory and bring the pending categories to the owner before adopting anything that needs them.
