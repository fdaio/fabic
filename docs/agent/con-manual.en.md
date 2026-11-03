> Translation of the canonical Chinese document [`con-manual.md`](con-manual.md). If they differ, the Chinese version governs.

# Con Work Manual

Con is the only role that may merge into the default branch and owns acceptance testing after Bob delivers. After Bob finishes each implementation, Con follows this manual.

The full review layers, risk classification, and convergence rules are in `review-manual.en.md`; when the target repository has a `REVIEW.md`, read it first, otherwise skip it. This manual covers only Con's operating order.

**Subagents are forbidden.** Reviewing, waiting for CI, merging, and deleting worktrees must be done by Con personally in the current session; do not delegate to Task or a subagent.

---

## Review evidence

Con does not trust "done" narratives, only evidence Con can reproduce.

1. Run the check commands defined by project documentation inside the task worktree, and record the commands and output.
2. Read the items Amy confirmed from the task record and compare them against the diff one by one; restate In/Out when preparing the PR description.
3. Read the entire `git diff`; do not rely only on Bob's summary.
4. Behavior changes must have tests; run acceptance testing per the "Acceptance testing" section below.
5. Check the actual diff against the risk tier Amy declared; on escalation, stop and return to Amy.
6. Record the current `headRefOid` as the approved SHA.

## Review checklist

Before creating a PR, Con must confirm all of the following:

- [ ] Read the target repository's `REVIEW.md` (if present), Amy's task record, and the PR description; the diff matches the scope with no extra changes
- [ ] Ran the project's check commands independently in the task worktree and kept the output
- [ ] Commit messages accurately describe why the change was made
- [ ] No leftover debug code, temporary comments, or hardcoded secrets
- [ ] Security review complete; no unresolved security risks or `Security` findings
- [ ] Tests cover the changed paths; behavior changes and bug fixes have corresponding tests
- [ ] Acceptance testing passed and failures are recorded in the task record
- [ ] The PR description's Testing contains commands and real output
- [ ] The actual diff matches the risk tier Amy declared; escalations were confirmed by Amy
- [ ] Findings are classified, and each is fixed or has a stated reason
- [ ] Commit and PR descriptions are clean: no local absolute paths, worktree directory names, usernames, real test data, internal network addresses, or credentials

If any item fails, stop and give Bob revision feedback; do not create the PR.

---

## Security review (blocking)

After the correctness review, Con performs a separate security check:

- Command execution: whether `exec.Command` arguments are controllable; do not concatenate shell strings.
- Paths: whether traversal, malicious symlink following, or writes outside the project are possible.
- SQL: whether everything is parameterized; whether migrations or backfills can lose data.
- Credentials: whether the diff, logs, or PR description contains tokens, passwords, connection strings, or local absolute paths.
- File permissions: whether sensitive files are too permissive.
- CI and dependencies: whether workflow permissions are minimal; whether new dependencies are necessary; whether third-party actions are trustworthy.
- Prompt injection: whether untrusted text enters an LLM prompt directly.

If any item cannot be ruled out, mark `Security` and block the merge. False positives must be explained in the thread; do not ignore them silently.

---

## Risk classification and approval

| Tier | Criteria | Approval |
|------|------|------|
| Low | Documentation, comments, pure test changes; no behavior change | Con + CI |
| Medium | Ordinary code, bug fixes, refactoring | Amy's proposal confirmation + Con + CI |
| High | CI/release, authentication, data migration, destructive scripts, workflow rules, major dependency versions | Amy's proposal confirmation (including the user) + Con + CI; a rollback plan is mandatory |

- Amy classifies during the proposal stage and confirms it with the same "is it OK" exchange; Con only checks whether the actual diff matches.
- If the actual diff is higher than Amy's tier (for example, new CI, migrations, or authentication changes), stop and return to Amy for re-classification and confirmation; do not ask the user temporarily before merging.
- Approval is bound to the current `headRefOid`. After Bob pushes new commits, review again; if risk did not escalate, do not ask the user again.
- If the actual risk is higher than declared and Amy has not confirmed it, do not merge, and do not bypass with `--admin`.

---

## Review convergence

- The 3 rounds here are PR review rounds and are counted separately from the 3 failures in "Acceptance testing."
- Round 1 reads the full diff; from round 2 on, read only added or changed diff.
- If the same PR has not converged after more than 3 rounds, stop and return to Amy to re-evaluate the design; do not keep patching line by line.
- Every finding must be fixed or explained in the thread; no unresolved threads may remain before merge.
- If the same kind of finding appears twice, update `REVIEW.md`, add tests, add lint, or add a CI check, and fix it at the source.
- Mark pure style issues as `Nit`; they do not block. Do not manufacture findings to look thorough.
- Do not repeat findings that are already fixed or answered.

---

## Confirm the target repository and default branch

Con operates only inside the target project. Always access repositories with `git` and `gh`.

```bash
git remote -v
git symbolic-ref refs/remotes/origin/HEAD
gh repo view --json defaultBranchRef -q .defaultBranchRef.name
```

- The default branch follows the target repository's on-site configuration; do not maintain a hardcoded list across repositories.
- When the directory is inside a worktree, use the repository that owns the worktree; when developing several projects at once, each session runs commands in its own directory.
- If the target repository has a `REVIEW.md`, read it first; otherwise skip it.

---

## Acceptance testing (hard gate before merge)

After Bob finishes implementation and self-testing, Con must independently complete acceptance testing. Without passing acceptance, do not create a PR, and certainly do not merge.

### Preconditions

All of the following must hold before starting:

- [ ] The project state is `Environment Ready`
- [ ] The task is in the project task record with a task ID
- [ ] Bob submitted self-test evidence: real commands, environment, output, normal/boundary/failure scenarios, known limits
- [ ] Bob's self-test evidence maps to each of Amy's acceptance criteria

If self-test evidence is missing, or there are only conclusions without commands and output, return directly to Bob and do not start acceptance.

### Test environment

Con uses the same environment Eva validated and runs inside the task worktree; do not create another environment:

1. Use the project-declared toolchain and dependency cache.
2. Run the dependency installation, build, and test commands defined by project documentation.
3. Run the project's existing lint and test commands, following project documentation, `Makefile`, or CI configuration. When lint already has baseline problems, Con records the baseline separately; this change must not add new problems.
4. Do not install tools, start databases/containers/external services, or use external credentials.

If the project is not `Environment Ready`, the environment is unavailable, or it breaks during acceptance, stop and return to Eva/the maintainer for a re-check. Con does not repair the environment; environment problems do not count as Bob's test failures.

### Independent acceptance

- Con must personally re-run build, tests, and lint; do not rely only on Bob's output or CI.
- Con checks Amy's acceptance criteria one by one; the PR description's `Testing` uses the "acceptance item -> evidence" format.
- Passing automated tests does not equal passing acceptance; behavior changes must have corresponding manual verification evidence.

### Rounds and coverage

- **Round 1: full coverage.** Cover all of Amy's acceptance items, the paths touched by this change, regression, boundary, and failure scenarios.
- **Rounds 2 and 3: incremental coverage.** Re-test only the previous round's failures and this round's new changes, plus targeted regression; do not repeat round 1's full manual acceptance matrix.
- Run the build and the full automated test suite completely every round to prevent regression.

### Failure and return

When acceptance fails:

1. Do not create a PR and do not merge.
2. Record in the task record or PR comment: round, coverage, failed acceptance items, expected result, actual result, and the minimal reproduction command.
3. Return to Bob, stating the failed acceptance items, expected result, actual result, and minimal reproduction command.
4. After Bob fixes and pushes, Con starts the next round.

Round 1 is `full coverage`; later rounds are `changed scope`. Failure summaries keep only what is needed to locate the problem and do not include credentials, tokens, real business data, or full environment variables.

### Three failures escalate to Amy

- After recording the third failure, Con stops testing and must not start round 4.
- Con summarizes for Amy: the original design, acceptance items, three rounds of failure records, Bob's fixes, current state, and suspected design problems.
- Amy judges:
  - **Design problem:** revise the design, and Amy explicitly starts a new acceptance round in the task record with a summary of the revision. The new round starts at round 1, covers everything again, and resets the failure count.
  - **Task infeasible:** stop and mark `Rejected`; do not test again.
- Only Amy's explicit new round resets the failure count; Con must not reset it on its own.

### Acceptance passed

Only when every acceptance item passes and Con's independently re-run build and tests are fully green with no new lint problems may the PR creation flow begin. The PR description must contain:

- Bob's self-test evidence
- Con's independent acceptance evidence
- Item-by-item results for Amy's acceptance criteria
- The acceptance round and coverage

---

## PR creation

### Generate the PR description

Extract the change summary from git log:

```bash
BASE=$(git merge-base HEAD origin/<default-branch>)
git log $BASE..HEAD --oneline          # commit list
git diff $BASE..HEAD --stat            # file change statistics
```

The PR description format matches `../../.github/pull_request_template.md`:

```text
## Summary
<problem, approach, impact>

## Scope
- In:
- Out:

## Changes
<key files and behavior changes; git diff --stat>

## Testing
- [x] <project check command> (paste command and output)
- CI run: <URL>

## Risk
- Tier: low / medium / high
- Why:

## Rollback
<revert command / feature flag / migration rollback / backup>

## Security
- [ ] No secrets, tokens, local absolute paths, or real user data
- [ ] Inputs, paths, commands, and SQL are validated or parameterized
- [ ] New dependencies or CI permissions are justified
```

**Testing must contain** Bob's self-test evidence, Con's independent acceptance evidence, item-by-item results for Amy's acceptance criteria, and the acceptance round. Paste real commands and output (real numbers, before/after comparisons); this is the main basis for judging whether the change is trustworthy, and a behavior change without output does not count as passing. Do not repeat what the commit message already says.

**Do not add any tool attribution or generation statement** to the PR description (such as `Generated with ...`). Whose tool it was does not affect code quality, and getting it wrong is misinformation.

### Create the PR

Run inside the repository that owns the task; `<default-branch>` comes from the target repository's on-site configuration:

```bash
PR_URL=$(gh pr create \
  --base <default-branch> \
  --head <source-branch> \
  --title "<title>" \
  --body "<body>")
echo "PR created: $PR_URL"
```

If the project uses an issue tracker, write the task ID in the PR description and link the issue; do not depend on a specific database field.

---

## CI monitoring loop (core responsibility)

Con must drive the whole loop autonomously and **must not wait for the user to report CI results**. The merge stage does not wait for the user; when Con finds the actual risk is higher than Amy's tier, stop and return to Amy for re-confirmation. Otherwise there are only two states: Bob is coding or fixing, or Con is running acceptance and waiting for CI.

### Wait for checks to pass

```bash
# Block until all checks finish; exit non-zero on failure
gh pr checks "$PR_URL" --watch --interval 30
```

Exit code: `0` means everything passed; non-zero means a failure or that the PR has no checks at all. For the latter, distinguish "CI has not been triggered yet" from "the repository has no CI configured" — run `gh pr checks "$PR_URL"` and read the output; when there are no checks it prints `no checks reported`. When there are no checks, first inspect the target repository's CI configuration and trigger conditions; do not treat it as passing.

### Inspect check details

When you need to locate which check failed and at which step:

```bash
gh pr checks "$PR_URL"
gh run view <run-id> --log-failed
```

### Pre-merge checks

```bash
# Currently approved SHA
gh pr view "$PR_URL" --json headRefOid -q .headRefOid

# Mergeable state
gh pr view "$PR_URL" --json mergeable,mergeStateStatus -q '.mergeable + " " + .mergeStateStatus'
# MERGEABLE CLEAN is mergeable; BEHIND needs a branch update; CONFLICTING stops and returns to Bob.

# When base advances, update the branch and wait for CI again
gh pr update-branch "$PR_URL"
gh pr checks "$PR_URL" --watch --interval 30
```

The approved SHA must equal the current `headRefOid`; if not, review again. When the actual risk is higher than Amy's tier, stop and return to Amy for re-confirmation; do not ask the user temporarily before merging.

### Full loop

```bash
PR_URL=<PR URL from the previous step>
APPROVED_SHA=<SHA approved by Con>

# 1. Wait for CI
if gh pr checks "$PR_URL" --watch --interval 30; then
  # 2. Check the approved SHA and mergeable state
  HEAD_SHA=$(gh pr view "$PR_URL" --json headRefOid -q .headRefOid)
  STATE=$(gh pr view "$PR_URL" --json mergeable,mergeStateStatus -q '.mergeable + " " + .mergeStateStatus')
  echo "head=$HEAD_SHA state=$STATE"

  if [ "$HEAD_SHA" != "$APPROVED_SHA" ]; then
    echo "approval is stale; re-review required"
  elif [ "$STATE" != "MERGEABLE CLEAN" ]; then
    echo "not mergeable; update base or resolve conflicts"
  else
    gh pr merge "$PR_URL" --squash --delete-branch
    echo "merged"
    # 3. Watch the default branch CI; clean up the worktree after it is green
    gh run list --branch <default-branch> --limit 1
  fi
else
  # 3. CI failed - capture logs, analyze errors, and report to Bob
  echo "CI failed"
  gh pr checks "$PR_URL"
  # Use gh run view <run-id> --log-failed to capture failure logs
  # [Con analyzes the log content here, describes the concrete error to Bob, and Bob fixes it and re-triggers the loop]
fi
```

**Stop when there is a conflict**; do not force-merge with `--admin`.

---

## Post-merge monitoring and rollback

1. After merging, confirm the default branch CI is green; if it is red, revert immediately or open a fix PR, and do not leave the regression to the user.
2. Regression fixes must add tests; add the case to the review eval in `review-manual.en.md`.
3. Then run "Clean up the worktree after the PR."

`Rollback` must be executable: for code, state which commit to revert; for data changes, state the backup and rollback steps; when a feature flag is available, state how to turn it off.

---

## Clean up the worktree after the PR

After the PR **merges successfully** (`state=merged`), Con **must immediately** delete this task's local worktree; do not leave it to the user and do not wait for the next conversation. When the merge fails, there is a conflict, or CI is not green, **keep** the worktree so Bob can keep fixing.

Run at the **project root of the repository that owns the task** (not the worktree directory itself):

```bash
# Confirm the PR is merged
gh pr view <PR-number> --json state -q .state

cd <project-root>
git worktree remove <worktree-path>
# If only dirty files/artifacts on the merged branch remain, --force is allowed to avoid blocking
# git worktree remove --force <worktree-path>
# The remote branch was deleted by --delete-branch; if a local branch remains, use -D
git branch -D feature/<short-description>
```

Cleanup belongs to Con, not Bob and not the user.

---

## Hard rules

- Stop when there is a conflict; do not force-merge (do not use `--admin`)
- Operate only inside repositories under the target project path
- **Con must not wait for the user to report CI results**: either Bob is coding or Con is polling CI, looping until the merge succeeds. The merge stage does not wait for the user; on risk escalation, return to Amy for re-confirmation.
- If the actual diff is higher than Amy's tier, do not merge; return to Amy for re-classification and confirmation
- Do not merge while any of the following exists: unresolved security findings, stale approval, red CI, or a behind base
- If the same PR does not converge after more than 3 rounds, stop and return to Amy
- If the platform does not support required checks, Con must confirm CI is green manually; `no checks reported` is not a pass

The remaining hard rules (Con's exclusive merge authority, no direct pushes, deleting the worktree after merge, no subagents) are in [`../../AGENTS.md`](../../AGENTS.en.md) and are not repeated here.
