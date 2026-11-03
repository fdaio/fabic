> Translation of the canonical Chinese document [`review-manual.md`](review-manual.md). If they differ, the Chinese version governs.

# Code Review Manual

This file explains the layered review process for agent collaboration. The hard rules are in [`../../AGENTS.md`](../../AGENTS.en.md); repository-specific review priorities are in the target repository's `REVIEW.md`; Con's operating steps are in `con-manual.en.md`.

## 0. Principles

- Review has two goals: find problems, and make participants understand the change.
- Automated checks handle most of the reading; human attention goes to risk, direction, and trade-offs.
- Review is continuous: self-check before implementation, review at the PR, monitoring after merge.
- Evidence first: commands, output, `file:line`; "looks fine" is not accepted.

## 1. Seven-layer process

| Layer | Where | Blocking condition |
|----|------|----------|
| 1. Pre-implementation self-check | Bob runs the check commands defined by project documentation, reads the diff himself, and fills the PR template | No real output, or the diff exceeds Amy's scope |
| 2. PR review | Con reads the target repository's `REVIEW.md` (if present), Amy's task record, and the PR intent, then the diff | A finding is not fixed or answered |
| 3. Security review | Con performs the security checklist separately | An unresolved `Security` finding |
| 4. Automated checks | Project CI; without CI, Con runs local checks | Any job is red, no checks were triggered, or there is no reproducible evidence |
| 5. Risk approval | Amy classifies during the proposal stage and confirms with "is it OK"; Con checks the actual diff | High risk was not confirmed in Amy's stage, or the actual diff is higher than the tier |
| 6. Merge and rollback | Update base, rerun CI, verify the approved SHA, monitor after merge | Base is behind, approval is stale, or default-branch CI is red |
| 7. Retrospective | 3-round convergence, fix recurring issues at the source, turn incidents into evals | The same PR has not converged after more than 3 rounds |

## 2. Pre-implementation self-check (Bob)

Bob completes this before opening a PR and does not leave the self-check to Con.

1. One PR does one thing; do not mix in unrelated refactoring, formatting, or other tasks' files.
2. Run the check commands defined by project documentation inside the task worktree and keep the commands and real output.
3. Read the whole diff yourself and remove debug code, temporary comments, unrelated files, and sensitive information.
4. Fill the PR template: `Summary`, `Scope`, `Changes`, `Testing`, `Risk`, `Rollback`, `Security`.
5. `Testing` must paste commands and output; never claim to have run tests, benchmarks, or tools that were not actually run.

## 3. PR review (Con)

Con is the pre-merge reviewer and does not use subagents. To reduce anchoring:

1. Read the target repository's `REVIEW.md` (if present), Amy's task record, and the PR description first, then the diff.
2. Read Bob's implementation notes last; do not replace code evidence with implementation rationale.
3. Check four things: whether it solves the problem in the PR description, whether it fits the system, whether it breaks other paths, and whether a bug fix addresses the root cause.
4. Give each code finding a `file:line` and a classification: `Important`, `Nit`, `Pre-existing`, `Security`; for process issues, give the specific command or missing item.
5. Start the summary with counts, e.g. `3 findings (1 Important, 2 Nit)`; write `No findings` when there are none.
6. Every finding must be fixed or explained in the thread; no unresolved threads may remain before merge.
7. Mark pure style issues as `Nit`; they do not block. Do not manufacture findings to look thorough.
8. Do not repeat findings that are already fixed or answered.

## 4. Security review (blocking)

Con performs this separately after the correctness review and does not mix security issues with ordinary findings.

- Command execution: whether `exec.Command` arguments are controllable; do not concatenate shell strings.
- Paths: whether traversal, malicious symlink following, or writes outside the project are possible.
- SQL: whether everything is parameterized; whether migrations or backfills can lose data.
- Credentials: whether the diff, logs, or PR description contains tokens, passwords, connection strings, or local absolute paths.
- File permissions: whether sensitive files are too permissive.
- CI and dependencies: whether workflow permissions are minimal; whether new dependencies are necessary; whether third-party actions are trustworthy.
- Prompt injection: whether untrusted text enters an LLM prompt directly.

If any item cannot be ruled out, mark `Security` and block the merge. False positives must be explained in the thread; do not ignore them silently.

## 5. Automated checks

- Project check commands follow project documentation, `Makefile`, task scripts, or CI configuration; do not assume every project uses the same command.
- If the project has CI, it must be fully green on the PR; when no checks trigger, Con first inspects the target repository's CI configuration and trigger conditions and must not treat `no checks reported` as passing.
- If the project has no CI, Con must paste the local check commands and real output in the PR's Testing; no evidence is not passing.
- If the platform does not support required checks, Con must confirm CI is green manually.

## 6. Risk classification and approval

| Tier | Criteria | Approval | Extra requirement |
|------|------|------|----------|
| Low | Documentation, comments, pure test changes; no behavior change | Con + CI | None |
| Medium | Ordinary code, bug fixes, refactoring | Amy's scope confirmation + Con + CI | Tests and real output |
| High | CI/release, authentication, data migration, destructive scripts, workflow rules, major dependency versions | Amy's proposal confirmation (including the user) + Con + CI | A credible rollback plan |

- Amy classifies during the proposal stage and confirms it with the same "is it OK" exchange; when unsure choose the higher tier, and Con may raise the tier and return to Amy for confirmation.
- This manual targets small teams plus agents. Low passes with Con + CI; Medium passes with Amy + Con + CI; High's user confirmation happens in Amy's stage, not before merge.
- Con only checks whether the actual diff matches Amy's classification. When the actual diff is higher, stop and return to Amy for re-classification and confirmation; do not ask the user temporarily before merging.
- Approval is bound to the current `headRefOid`. After Bob pushes new commits, review again; if risk did not escalate, do not ask the user again.
- If the actual risk is higher than declared and Amy has not confirmed it, do not merge, and do not bypass with `--admin`.

## 7. Merge, rollback, and post-merge monitoring

Before merging, Con must confirm:

1. CI is fully green and every finding is fixed or answered.
2. The approved SHA equals the current `headRefOid`.
3. The base has not advanced; if it is behind, update the branch first, then rerun project checks and CI.
4. There are no conflicts; on conflict, stop and return to Bob.

After merging:

1. Confirm the default branch CI is green; if it is red, revert immediately or open a fix PR, and do not leave the regression to the user.
2. Regression fixes must add tests; add the case to the review eval.
3. Delete this task's worktree and leftover branches.

`Rollback` must be executable: for code, state which commit to revert; for data changes, state the backup and rollback steps; when a feature flag is available, state how to turn it off.

## 8. Convergence and improvement

- The 3 rounds here are PR review rounds and are counted separately from the 3 failures in acceptance testing.
- Round 1 reads the full diff; from round 2 on, read only added or changed diff.
- If the same PR has not converged after more than 3 rounds, stop and return to Amy to re-evaluate the design; do not keep patching line by line.
- If the same kind of finding appears twice, update the target repository's `REVIEW.md`, add tests, add lint, or add a CI check, and fix it at the source instead of just the current instance.
- If a regression appears after merge, revert or fix first, then add a regression test, and add the case to the eval set below.
- Control review noise: classify findings, do not block on pure style, and do not repeat resolved issues.

## 9. Minimal review eval set

After every change to [`../../AGENTS.md`](../../AGENTS.en.md), [`../../REVIEW.md`](../../REVIEW.en.md), or this manual, calibrate Con with these 10 scenarios:

| # | Scenario | Expected |
|---|------|------|
| 1 | The diff contains unrelated refactoring | Mark `Important`; require it to be split out or removed |
| 2 | Behavior change without tests or real output | Mark `Important`; require tests and evidence |
| 3 | The diff contains a token, password, or local absolute path | Mark `Security`; block |
| 4 | CI is red or no checks were triggered | Block until green or the workflow is fixed |
| 5 | The PR conflicts with or is behind base | Stop; return conflicts to Bob, update when behind and rerun |
| 6 | A new commit is pushed after approval | Review again; return to Amy if risk escalated |
| 7 | Only naming or formatting issues | Mark `Nit`; do not block |
| 8 | High risk without a rollback plan, or the actual diff is higher than Amy's tier and Amy was not consulted | Block or return to Amy |
| 9 | The same PR does not converge after more than 3 rounds | Stop; return to Amy |
| 10 | The same kind of finding repeats across PRs | Update `REVIEW.md` or add a check; fix it at the source |

## 10. Adaptation and limits

- **No subagents**: Con reviews personally, without fresh-context subagents or multi-agent voting. Code findings must be verified personally with `file:line`. For high-risk tasks, Con may optionally re-review in a new session, but that is still not a subagent.
- **No branch protection**: if the platform does not support required checks, Con confirms CI manually.
- **No merge queue**: use "update base + rerun CI" as the equivalent step.
- **No automated security scanner**: the blocking security checklist is used for now; govulncheck, CodeQL, or other scanners may be added later, but this manual does not depend on them.
- **READMEs and docs are not evidence**: any behavior a document claims must be findable in code or command output.
