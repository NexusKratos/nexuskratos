# Framework, project repositories, and public surfaces

> Design update — 2026-11-05. **Framework/repository separation is the intended architecture.** The two-domain product split remains a **proposal**, not a change to live routing. No framework executable, marketplace, or project provisioner exists yet.

## Three independent layers

1. **Nexus Kratos framework** — `NexusKratos/nexuskratos` is the source repository for the platform itself: control plane, contributor daemon and installer/release pipelines, versioned protocol, adapters, shared infrastructure contracts, platform docs and tests. This repository **is not the default work repository for every community project**.
2. **Community projects** — every approved project references **one or more independently owned source repositories**. Projects may live in the NexusKratos GitHub organization or an external organization that opts in. The coordinator tracks explicit `Project ↔ Repository` associations, task-to-repository targets, dependency edges and distinct authorization scopes. A single project can involve multiple repos, while two projects must not implicitly share permissions simply because they reference the same repo. Project source, issues, PRs, release branches and accepted changes remain in **their own repositories**, not in the framework repository. Cross-repo work requires explicit scoped grants and coordinated, auditable integration; no promise of atomic multi-repository merges.
3. **Contributor marketplace/community** — user identities, project proposals and moderation, opt-in, contributor capacity preferences, notifications and recognition belong in the application/control plane and community interface. They are **not stored as commits or pull requests in the framework repo**. GitHub integration serves the code-facing project repositories while contributors interact with the Nexus Kratos product.

## Proposed domain responsibilities

| Domain | Proposed purpose | What exists currently |
| --- | --- | --- |
| **nexuskratos.dev** | Framework/developer home: architecture, open-source framework source, signed installer and binary releases, install guides, protocol/API docs, SDKs, self-hosting, security disclosures | Informational static website deployed, with no download/service interface yet |
| **nexuskratos.com** | Community and discovery home: propose projects, browse approved work, contributor identity/profile, project enrollment, moderation, task history, optional badges, community updates | Redirects to `nexuskratos.dev`; **no independent community application** |
| **github.com/NexusKratos/nexuskratos** | Canonical platform framework source repository | Public docs and planning; no runnable framework |
| **Project GitHub repositories** | Their own code, issues, branches, PRs, review and releases | Created/maintained by each project's authorized owners; the framework does not take ownership implicitly |

The split is an information-architecture and future product-hosting proposal. **Do not modify redirects, DNS, TLS, Workers routes, authentication callbacks, or release links based on this design alone.** Record deployment/migration details and obtain explicit owner authorization before shipping the `.com` surface.

The existing website source is currently `NexusKratos/website`. It can remain a separately deployed static framework site; a future community application may need its own source and deployment lifecycle. This documentation does not create a new website repo or decide its stack.

## Relationship model

~~~mermaid
flowchart TB
  subgraph Framework["NexusKratos framework repository"]
    Core["Control plane + scheduler + protocol"]
    Binary["Signed daemon / installer releases"]
  end
  subgraph Experience["Future public product surfaces"]
    Dev["nexuskratos.dev: framework docs + downloads"]
    Com["nexuskratos.com: community + discovery"]
  end
  subgraph Projects["Independent open-source project repositories"]
    P1["Project A → repository A"]
    P2["Project B → repositories B1 + B2"]
  end
  Donor["Contributor installs daemon"] --> Dev
  Dev --> Binary
  Donor --> Com
  Com --> Core
  Binary --> Core
  Core --> GitHubApp["Scoped GitHub App permissions"]
  GitHubApp --> P1
  GitHubApp --> P2
~~~

## Proposed domain model

- `Project`: stable ID, approved project owner, proposal, status, maintainers, catalog metadata, contributor eligibility, community policy and trust profile.
- `ProjectRepository`: links project ID to repository ID, host, installation/authorization reference, allowed paths, publication policy, synchronization status and explicitly authorized operations. A project may have multiple links; adding a new link needs fresh ownership verification and donor trust review.
- `TaskRevision`: targets one repository/commit or an explicitly declared multi-repo plan; never assumes the framework repo. Each executable attempt has a pinned base and scoped permissions.
- `ContributorEnrollment`: explicit project IDs, device, allowed providers/models, resource reservations, role/capability constraints and revocable consent; no GitHub account required for ordinary donors.
- `FrameworkRelease`: signed binary + checksums/provenance, API/protocol compatibility matrix and platform support claims. This is separate from each project's own code releases.

## Community Manager role

A **Community Manager** operates within **one or more explicitly assigned communities**, not all projects by default. It is distinct from project maintainer, platform administrator and technical reviewer.

**May** (subject to scoped policy): guide member onboarding, triage project proposals, curate discovery metadata, moderate conversation, manage announcements/notification campaigns, coordinate events, help contributors find suitable projects, and recommend recognition/badges with audit trails.

**Must not automatically**: execute or merge project code, approve security exemptions, bypass independent review, access donor provider credentials, grant their own extra permissions, modify platform infrastructure or install a GitHub App into another owner's repositories.

For the MVP, Community Manager and Community Moderator may share a role preset with separate fine-grained permissions and future migration. An approved project still requires a project maintainer to authorize repo linkage, and a contributor must independently opt in before code is run. Require explicit authorization and audit records for moderation and reputation actions.

## Operational implications / acceptance criteria

1. A contributor can enroll in two unrelated projects, one with two repositories, while viewing those projects separately in the catalog.
2. All three repositories require individually validated installation scopes; one approved repository does not confer access to the others.
3. A task targeted at a project repo cannot write to the framework repository unless that repository is explicitly designated as a target project and separately authorized.
4. Project reviewers and maintainers remain scoped to their project and repo; Community Managers cannot grant merge authority by editing catalog metadata.
5. The installed binary has a verifiable framework release identity independent of individual project commits and PRs.
6. The platform handles revoked GitHub App installations, renamed/transferred repositories, missing branches, repo deletion and cross-repo dependency failure without silently assigning code to an unrelated repo.
7. Before a `.com` launch, verify DNS/routing, session/cookie scope, callback origins, account recovery, phishing resistance, CSRF/CORS, privacy, and coherent cross-domain navigation. **No implicit shared session cookies across `.dev` and `.com`.**

## Status

**Decided in discussion:** separate framework source from project repositories; projects may use one or many repos; include a Community Manager role with permissions separate from maintainers and administrators.

**Still proposed:** `.dev` as developer/framework home and `.com` as community home. Current redirects stay untouched until a later explicit launch decision.
