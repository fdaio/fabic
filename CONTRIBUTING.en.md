> Translation of the canonical Chinese document [`CONTRIBUTING.md`](CONTRIBUTING.md). If they differ, the Chinese version governs.

# Contributing Guide

This repository is an agent collaboration handbook and stores only rules and templates. Read `AGENTS.en.md` first; it is the single source of all constraints for team members.

## What you can contribute

- Rule fixes: `AGENTS.md`, `docs/agent/**`, `REVIEW.md`
- Template improvements: `.github/**`
- Documentation fixes: typos, broken links, commands that cannot be executed

Out of scope: business code for specific projects, runtimes, databases, boards, and any real project, customer, credential, or local environment information.

## Submission flow

1. Open an issue first, describing the problem, impact, and expected result.
2. Create a feature branch from the default branch; do not modify the default branch directly.
3. One PR does one thing; do not mix in unrelated refactoring or formatting.
4. Fill in the PR description per `.github/pull_request_template.md`.
5. Wait for maintainer review; the maintainer performs the merge.

```bash
git switch -c feature/<short-description> <default-branch>
```

## Content rules

- Keep the handbook decoupled from specific projects: do not write repository names, organization names, customer names, or internal project names.
- Examples use placeholders only: `<project>`, `<repo>`, `<default-branch>`, `REQ-001`, `example.com`.
- Change rules in `AGENTS.md` first; put role operating details in `docs/agent/`.
- Any behavior a document claims must be traceable to a rule or to a project command run on site.
- Do not commit credentials, real business data, generated boilerplate, or local absolute paths.

## Checks

This repository has no CI and no automated tests. Documentation changes must at least:

- Scan for sensitive information: check the diff for credentials, private keys, connection strings, internal network addresses, or local absolute paths.
- Check links: new relative links must point to files that exist.
- Check rule consistency: the change does not conflict with `AGENTS.md` and does not rewrite role responsibilities.

Put the commands and real output in the PR's `Testing`.

## Commit messages

Write only "what changed and why." No running commentary, no unrelated logs, and no local absolute paths or real data.

## License

Contributions are licensed under the Apache License 2.0; see `LICENSE`.

## Code of conduct

By participating in this project you agree to [CODE_OF_CONDUCT.en.md](CODE_OF_CONDUCT.en.md).
