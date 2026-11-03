# Contributing

NexusKratos is in early development. Start with an issue describing the problem and intended outcome. Contributions to requirements, design, accessibility, documentation, tests, and implementation are welcome.

## Local setup

```sh
git clone https://github.com/NexusKratos/nexuskratos.git
cd nexuskratos
```

There is no application build, runtime, or test suite yet. Documentation changes should be checked for accurate status, valid relative links, and whitespace errors (`git diff --check`). Do not introduce fictional build commands or claim planned features work.

## Changes

Use a feature branch and pull request for community contributions. State what changed and how it was checked. Keep secrets, local paths, provider credentials, and personal data out of commits. Use a GitHub-provided noreply identity if you want to keep your email private. AI-assisted contributions remain the contributor's responsibility; disclose relevant assistance and actual validation.

Maintainers allocate the next SemVer in `VERSION`. `checkpoint/<VERSION>` identifies a source checkpoint; `v<VERSION>` is reserved for an accepted release. Checkpoint tags do not imply a runnable product or deployment. No automated workflows are installed at this stage.

## Reporting

Use issues for ordinary bugs and proposals. Send sensitive reports privately to the address in [SECURITY.md](SECURITY.md).
