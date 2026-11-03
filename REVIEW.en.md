> Translation of the canonical Chinese document [`REVIEW.md`](REVIEW.md). If they differ, the Chinese version governs.

# REVIEW.md

This file is written for reviewers (Con) of this handbook repository. It states only "what matters most in this repository, and what can be skipped."

The full process is in `docs/agent/review-manual.en.md`; the hard rules are in `AGENTS.en.md`.

## Repository goal and failure modes

fabic is a project-agnostic agent collaboration handbook. The most serious failures are not style problems but:

- Handbook rules contradict each other, or role handoff conditions are unclear.
- A process, command, or state the documentation claims cannot be executed in an actual project.
- The handbook contains personal, customer, internal project, credential, or local absolute path information.
- Examples contain real business data, or links point to files that do not exist.
- Rules, tools, or dependencies are added to look complete but cannot be maintained.
- Workflow rules are deleted, renamed, or tightened without recalibrating the affected manuals.

## Review priorities

1. Whether it is consistent with `AGENTS.md`; whether it exceeds the scope Amy confirmed; whether the actual diff is higher than Amy's risk tier.
2. Whether it still references deleted CLIs, databases, boards, project lists, or specific repositories.
3. Whether personal information, customer data, internal project names, credentials, internal network addresses, or local paths appear.
4. Whether examples use placeholders; whether relative links and file references exist.
5. Whether role manuals only add operating details without copying or rewriting hard rules in a conflicting way.
6. Whether the document is executable: are commands, states, and handoff conditions clear, and does it depend on tools that do not exist.

## High-risk paths

The following changes are high risk by default:

- `AGENTS.md`, `docs/agent/**`, `REVIEW.md` (the workflow rules themselves)
- `.github/**`
- Any change that deletes, renames, merges, or tightens role responsibilities
- Any change that introduces real project, customer, credential, or local environment information
- Any change that adds a runtime, database, board, CI, or third-party dependency (this repository should remain a pure handbook)

## Evidence requirements

- Documentation findings must give `file:line`; do not guess from impressions or file names. Process or evidence issues must give a concrete command or the missing item.
- "Checked" must include a command and real output; without output it is not evidence.
- Security findings must state the leak path or data impact.
- When there are no findings, write `No findings` explicitly; do not invent nits to look thorough.

## Finding classification

- `Important`: affects rule correctness, security, role handoffs, or handbook executability; must be fixed or given a clear reason before merge.
- `Nit`: style, naming, readability; does not block unless it hides a bug.
- `Pre-existing`: not introduced by this change; mark it and do not block unless it is a serious security issue.
- `Security`: security issue; must not remain unresolved at merge.

## Review rounds

- Round 1 reads the full diff.
- From round 2 on, read only the added or changed diff to avoid re-reading confirmed parts.
- If the same PR has not converged after more than 3 rounds: stop and return to Amy to re-evaluate the design; do not keep patching line by line.
- Do not repeat findings that are already fixed or answered.

## Known limitations

- This repository is a pure handbook repository; it has no runtime, database, or board.
- There is no automated security scanner; use the blocking security checklist in `docs/agent/review-manual.en.md` and scan for sensitive information manually before committing.
- There are no fresh-context subagents; Con reads the diff, Amy's task record, and this file first, then the implementation notes, to reduce anchoring.
- If the platform does not support required checks, Con must confirm CI is green manually.
