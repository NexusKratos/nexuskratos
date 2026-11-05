# Architecture knowledge and AI-provider compatibility

> **Proposed design — 2026-11-05.** The architecture daemon does not currently ship an embedded architecture catalog, nano model or universal provider adapter. Model names and integrations are candidates, not certified capabilities.

## Architecture knowledge embedded with the runtime

The architect/planning role needs fast local access to **a curated catalog of architectural approaches** rather than repeatedly downloading a public repository of long documents. Start with a signed/versioned knowledge pack compiled into the release or installed next to the binary, with independent updates after signature/integrity verification.

A proposed initial pack would contain **20–30 compared architectural methods**, but the count is a maintenance target rather than a quality metric. Cover modular monoliths, microservices, event-driven architectures, CQRS/event sourcing, hexagonal/ports-adapters, layered/clean architecture, service-oriented, serverless, pipelines/dataflow, batch/stream processing, actor systems, peer-to-peer, plugin architectures, offline-first, distributed state machines, and other patterns as justified by reviewed source material. Each entry needs:

- Name, version, source/attribution and license; constraints, assumptions and requirements fit.
- Operational trade-offs (consistency, scalability, latency, security, cost, maintainability, deployment complexity, failure recovery and testability).
- Applicability criteria, contraindications, alternatives, reference diagrams, validation questions and known anti-patterns.
- A compact token-budgeted summary and references for more detailed review; no universal “best architecture” claim.

The binary compares candidate approaches against **the project's requirements**, records reasoning/trade-offs, and makes suggestions. **Project architects still choose and approve the design.** Knowledge pack content is general guidance; it must not overwrite a frozen project-specific architecture or substitute for independent review.

### Retrieval versus a tiny model

First use deterministic metadata filters and a small local retrieval index to select relevant guidance. A quantized small/nano model may later summarize, classify requirements or improve retrieval, but only after accuracy, latency, RAM, license and hallucination benchmarks. **Do not treat a tiny local model as the architecture decision-maker.** Donated/paid qualified high-capability models perform hard reasoning and independent assessment. Knowledge packs must support offline use and separately signed updates, with expiration and rollback policy.

## Pluggable provider and authorization interface

A **model capability registry** tracks provider, model/version, supported context/input/output/tool calls, structured output, headless execution, cancellation, transport, authentication type, minimum capability tier, measured versus estimated usage and permitted billing modes. The architect, planner, reviewer and builder roles each declare their own approved model set and budgets.

Initial adapter candidates: **direct provider APIs**, OpenRouter, and official provider CLIs/SDKs or other expressly supported subscription integrations. Also allow future providers and local models through a typed adapter protocol. Potential high-end providers include OpenAI, Anthropic, Google Gemini and xAI/Grok, with further models admitted only after evidence. User-mentioned Astra and Fable may be **preferred planning candidates**, not hardcoded defaults or guarantees of third-party availability; benchmark them against independent architectural test cases and requalify versions over time.

**API billing and consumer subscriptions are not interchangeable.** Never infer “free API access” or third-party OAuth permission from a paid chat/coding subscription. For every adapter, verify current publisher documentation and terms for unattended operation, third-party integration, credential handling, resale/redistribution, rate limits, account delegation, model identity and cancellation. If a subscription mode is not clearly authorized and technically supported, mark it **unsupported** and offer an independently paid API path. No credential scraping, cookie/session harvesting or impersonation of proprietary clients.

Keep permissions local and provider-specific. Explicitly separate API key, officially sanctioned OAuth/SDK/CLI session and hosted operator compute, with no silent fallback from one billing channel to another.

## Scheduling and cost optimization

The deterministic scheduler matches the **task's capability requirements** to donor-supplied supported models/providers and certified adapter security properties. Prefer the lowest-cost *qualified* option within approved policies, not the cheapest untested model. Record model version, actual runtime/provider/effort, reasoning context and usage receipt; unknown usage remains unknown.

Planning, architecture review and code review should permit independent model families and provider diversity when the policy requires it. The model which authored the architecture cannot be its sole validator. Redundant independent reviews may use donated higher-tier capacity or optional paid operator capacity with exactly the same approval criteria.

## Qualifications before implementation

- Knowledge selection benchmarks: coverage, precision, false recommendations, comparisons, source traceability, and token/latency budgets versus reading full references.
- Nano-model trials: compare retrieval-only against retrieval plus tiny model using fixed cases; keep the nano model optional unless proven useful.
- Provider matrix: direct API, OpenRouter and approved subscription-native CLI paths with documented legal/billing/technical evidence (including changes over time).
- Adversarial provider tests: credential separation, malformed output, unsupported authentication, cancellation, exhausted limits, stale version and conflicting model capability claims.
- Model planning evaluations: multiple architecture families and tricky requirements; measure omitted constraints, unsupported assertions, reviewer agreement, cost, and rework.
- All model suggestions must be explicitly reviewable and leave a human-readable rationale and references.

Related: [planning and orchestration](planning-and-orchestration.md), [architecture review pool](independent-architecture-review.md), [implementation plan](../plans/2026-11-04-platform-implementation.md).
