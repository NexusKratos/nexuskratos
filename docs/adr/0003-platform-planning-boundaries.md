# 0003: Platform planning boundaries

Date: 2026-11-04. Status: proposed design; implementation explicitly on hold.

## Context

The foundation describes components but no executable task graph, local credential isolation design, recovery contract or acceptance gates. The owner requested an improved implementation plan, with planning only.

## Decision

Use the [platform plan](../plans/2026-11-04-platform-implementation.md) as the proposed delivery sequence. Prove an end-to-end contribution/review loop before expanding administration. Retain a monorepo, separate contributor executable, independent website and Warboard. Propose a modular Rust control service with PostgreSQL coordination, Rust contributor and TypeScript web interface. Keep provider secrets local and distinct from repository execution; feasibility must be demonstrated before enabling adapters. Review and acceptance bind exact candidate/base commits and authenticated identities.

## Alternatives rejected for this proposal

- UI-first delivery postpones the risks that determine whether the product can safely work.
- Coordinator microservices plus a broker multiply distributed-state failure modes before scale evidence requires them.
- GitHub issues/PRs as the assignment lock cannot express authoritative lease epochs and local capacity reservations.
- Mounting donor credentials into arbitrary repository execution does not satisfy the intended credential boundary.

## Consequences

The first tasks are explicit feasibility and protocol work. Unsupported isolation, provider meters and platforms stay visible and block their guarantees. PostgreSQL adds an operator dependency; local execution needs its own durable journal. Exact library versions and backend choices require follow-on decision evidence. Nothing in this ADR authorizes implementation, provider calls, deployment, or access to Warboard private source.
