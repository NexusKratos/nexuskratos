# NexusKratos

**Build together with AI.**

NexusKratos is an open-source platform being designed to help communities turn donated AI capacity into coordinated, reviewed software contributions across multiple repositories.

> **Status: early development.** This repository currently contains the project foundation and design direction. There is no runnable NexusKratos platform or contributor application yet.

## The idea

A contributor chooses which AI tools to use, which communities to trust, and how much capacity to contribute. The planned local application picks compatible tasks, runs them within those limits, and submits work through GitHub. Other contributors review and test the results.

The community can contribute implementation, review, QA, documentation, or a whole feature. GitHub remains the shared home for plans, issues, branches, pull requests, and accepted work.

## Four roles, one shared effort

| Role | Responsibility |
| --- | --- |
| Planner | Break a goal into tasks, dependencies, required skills, minimum model capabilities, repository context, and acceptance criteria. |
| Orchestrator | Assign compatible work, track leases and deadlines, coordinate handoffs, and route results to independent review. |
| Contributor | Donate bounded local AI capacity to build, review, test, or document. |
| Administrator | Manage communities, repositories, infrastructure, roles, and operating policy. |

## Planned foundations

- Multiple repositories and CLI adapters, beginning with the requirements of Codex and Claude Code; additional tools follow verified provider capabilities.
- Contributor-controlled task, time, concurrency, and usage limits. Dollar estimates must remain distinct from measured API spending and subscription quotas.
- Expiring assignment leases, explicit extensions, preserved partial work, and reassignment without allowing stale attempts to overwrite newer results.
- Independent reviews and QA attached to an exact commit, with acceptance criteria visible to everyone.
- Credentials remain on contributor machines; donated capacity does not mean sharing account credentials.
- A self-hostable core, with a clear protocol between the coordinator and contributor application.

## Warboard

Warboard is the visibility component being developed for the NexusKratos ecosystem: tasks, agents, plans, logs, and usage. Work on Warboard is ongoing. Its integration with NexusKratos is planned, not implemented, and Warboard is intended to remain independently usable.

## Get involved

- [Discuss an idea or report a problem](https://github.com/NexusKratos/nexuskratos/issues)
- [Read the architecture direction](ARCHITECTURE.md)
- [Read the contribution guide](CONTRIBUTING.md)
- [Website source](https://github.com/NexusKratos/website)
- General contact: [hello@nexuskratos.dev](mailto:hello@nexuskratos.dev)
- Private security reports: [security@nexuskratos.dev](mailto:security@nexuskratos.dev)

Website: **https://nexuskratos.dev**. The additional **nexuskratos.com** domain and both www variants permanently redirect to the primary site. The website is an introduction to the project; it is not a runnable NexusKratos platform.

## License

[Apache License 2.0](LICENSE).
