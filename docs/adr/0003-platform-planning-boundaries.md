# 0003: Platform planning boundaries

Date: 2026-11-04. Status: proposed design; implementation explicitly on hold.

## Context

The foundation describes components but no executable task graph, local credential isolation design, recovery contract or acceptance gates. The owner requested an improved implementation plan, with planning only.

## Decision

Use the [platform plan](../plans/2026-11-04-platform-implementation.md) as the proposed delivery sequence. Prove an isolated, scanner-gated end-to-end contribution/review loop before expanding the moderated marketplace and community administration. Retain a monorepo **for the Nexus Kratos framework only**, with a separate contributor executable, independent project code repositories, a separately deployed website and optional independently maintained Warboard. Projects may use one or several repositories, including external GitHub installations with narrowly scoped grants. Propose a modular Rust control service with PostgreSQL coordination, Rust contributor and TypeScript web interface. Keep provider secrets local and distinct from repository execution; feasibility must be demonstrated before enabling adapters. Review and acceptance bind exact candidate/base commits and authenticated identities. Community Manager moderation/engagement must not imply code-merge or security-exemption authority; ordinary donors should need no personal GitHub account.

## Alternatives rejected for this proposal

- UI-first delivery postpones the risks that determine whether the product can safely work.
- Coordinator microservices plus a broker multiply distributed-state failure modes before scale evidence requires them.
- GitHub issues/PRs as the assignment lock cannot express authoritative lease epochs and local capacity reservations.
- Mounting donor credentials into arbitrary repository execution does not satisfy the intended credential boundary.

## 2026-11-05 product reconciliation

The 25-task [updated plan](../plans/2026-11-04-platform-implementation.md) extends the earlier 19-task technical baseline with T20 scanner pipeline, T21 per-project repository links, T22 moderated project lifecycle/Community Manager, T23 GitHub-free onboarding, T24 opt-in notifications/badges and T25 signed binary distribution. These are **planned**, not delivered. `nexuskratos.dev` as developer/docs/download surface and `nexuskratos.com` as community/catalog surface are **future proposals only**. Existing .com→.dev redirect remains untouched.

The versioned runner interface is selected as a design boundary only; single shared harness versus native provider harnesses is intentionally **unresolved**. No dependency on a private runner is approved.

## Consequences

The first tasks are explicit feasibility and protocol work. Unsupported isolation, provider meters and platforms stay visible and block their guarantees. PostgreSQL adds an operator dependency; local execution needs its own durable journal. Exact library versions and backend choices require follow-on decision evidence. Nothing in this ADR authorizes implementation, provider calls, deployment, or access to Warboard private source.
