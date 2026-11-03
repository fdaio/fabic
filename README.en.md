[简体中文](README.md) · [English](README.en.md) · [日本語](README.ja.md)

> Translation of the canonical Chinese document [`README.md`](README.md). If they differ, the Chinese version governs.

# fabic · Agent Collaboration Handbook

A project-agnostic handbook for agent collaboration. It defines the responsibilities, handoff conditions, review process, and merge rules for four roles: Amy, Eva, Bob, and Con.

This repository holds only the handbook and templates. It contains no runtime, database, board, or project-specific data.

## How to use

1. Read `AGENTS.en.md` first. It is the single source of all constraints for the team.
2. Read the matching manual under `docs/agent/` for your role.
3. When onboarding a project, the project keeps its own repository, issue tracker, CI, and check commands; this handbook only supplies collaboration rules.
4. Examples use placeholders only: `<project>`, `<repo>`, `<default-branch>`, `REQ-001`, `example.com`.

## Workflow

```text
New project onboarding
  -> Eva environment admission
  -> Amy clarifies and confirms
  -> Bob implements + self-tests (feature branch)
  -> Con acceptance testing
  -> Con opens PR
  -> Layered review + CI
  -> Merge to default branch
  -> Clean up worktree
```

- Environment unavailable: return to Eva or the maintainer; Bob does not start.
- Test failure: Con records it and returns the task to Bob.
- Three consecutive test failures: Con stops testing and reports to Amy; only Amy can start a new acceptance round.
- Risk escalation: Bob or Con stops and returns to Amy for re-classification; do not ask the user just before merging.

## Document index

| Document | Contents |
|------|------|
| `AGENTS.en.md` | Roles, handoffs, risk tiers, and hard merge rules |
| `docs/agent/amy-manual.en.md` | Amy: requirement clarification, six-point self-check, risk classification, judgment after three failures |
| `docs/agent/eva-manual.en.md` | Eva: environment admission, check commands, states, and failure handling |
| `docs/agent/con-manual.en.md` | Con: acceptance testing, review, PR creation, CI monitoring, merge, worktree cleanup |
| `docs/agent/review-manual.en.md` | Layered review, security review, risk approval, rollback, and retrospective |
| `REVIEW.en.md` | Review priorities and high-risk paths for this handbook repository |
| `.github/pull_request_template.md` | PR description template |

## Maintenance principles

- Keep the handbook decoupled from specific projects: do not write repository names, organization names, customer names, internal project names, or local paths.
- Do not record any personal, customer, internal project, credential, or real business data.
- Change rules in `AGENTS.md` first; put role operating details in `docs/agent/`.
- The Chinese version is canonical for translations; change it first, then sync the `.en.md` and `.ja.md` files.
- Any behavior a document claims must be traceable to a rule or to a project command run on site.

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for the full text.
