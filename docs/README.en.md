[简体中文](README.md) · [English](README.en.md) · [日本語](README.ja.md)

> Translation of the canonical Chinese document [`README.md`](README.md). If they differ, the Chinese version governs.

# Manual index

This directory holds the role manuals and review details. The entry rules are in the root [`../AGENTS.en.md`](../AGENTS.en.md).

| Document | Purpose |
|------|------|
| `../AGENTS.en.md` | Hard rules for roles, handoffs, risk, and merging |
| `agent/amy-manual.en.md` | Amy: requirement clarification, six-point self-check, risk classification, judgment after three failures |
| `agent/eva-manual.en.md` | Eva: environment admission, check commands, states, and failure handling |
| `agent/con-manual.en.md` | Con: acceptance testing, review, PR, CI, merge, worktree cleanup |
| `agent/review-manual.en.md` | Layered review, security review, risk approval, rollback, and retrospective |
| `../REVIEW.en.md` | Review priorities and high-risk paths for this handbook repository |

## Reading order

1. New members: `../README.en.md` -> `../AGENTS.en.md` -> role manual.
2. Taking a task: `agent/amy-manual.en.md` -> the Bob section of `../AGENTS.en.md` -> `agent/con-manual.en.md`.
3. Changing the handbook: read `../REVIEW.en.md` first, then self-check against `agent/review-manual.en.md`.

## Conventions

- Examples use placeholders only: `<project>`, `<repo>`, `<default-branch>`, `REQ-001`, `example.com`.
- The handbook is not bound to a specific project, repository, organization, tool, or local path.
- When rules conflict, `../AGENTS.md` governs; role manuals only add operating details.
- Do not write task data, customer data, credentials, or local environment information into this repository.
