> Translation of the canonical Chinese document [`eva-manual.md`](eva-manual.md). If they differ, the Chinese version governs.

# Eva Environment Admission Manual

Eva is responsible for preparing and validating the project environment. Eva is a process role defined by this handbook, executed by the current session per this manual; it is not an independent agent, not a subagent, and not invoked via `@Eva` or Task.

---

## Goal

Before implementation starts, prove that the project can build and test on this machine. If the environment cannot be prepared, mark the project `Unmaintainable`, notify the maintainer, and do not enter implementation.

---

## Scope

Use the toolchain and check commands declared by project documentation, `Makefile`, task scripts, or CI configuration. Common default commands:

| Ecosystem | Dependencies | Build | Test |
|------|------|------|------|
| Go | `go mod download` | `go build ./...` | `go test ./...` |
| Node | `npm ci` | `npm run build` | `npm test` |
| Python | `pip install -r requirements.txt` | project-declared build command | `pytest` |
| Rust | `cargo fetch` | `cargo build` | `cargo test` |

When the project declares no commands, run the ecosystem defaults; when they still cannot be determined, treat it as `Unmaintainable` and record the reason `unsupported project type`.

Explicitly unsupported:

- Projects that need containers, remote CI, or external services to build or test.
- Projects that need automatic installation or upgrade of system toolchains to pass.
- Projects that need automatic repair of build or test failures.
- Projects without reproducible check commands.

---

## Triggers

- When a new project is onboarded.
- Re-check after the environment is modified.
- Retry after failure.
- Run only one check per project at a time.

---

## Execution

Run at the project root, non-interactively, with a total timeout of 15 minutes per check:

1. Verify the project path exists and the declared toolchain is available.
2. Install dependencies per project documentation (only into project caches or dependency directories).
3. Run the build command.
4. Run the test command.

Execution constraints:

- Do not modify project files.
- Do not run `go mod tidy`, `go generate`, formatting, code generation, or other commands that rewrite the project.
- Dependencies go only into project caches; build and test use only cache directories.
- Do not start databases, containers, or external services.
- Do not record or print credentials, tokens, or full environment variables.

---

## Test environment used by Con for acceptance

`Environment Ready` also defines the environment Con uses for acceptance testing. Con works in the task worktree with the same project path, toolchain, and dependency cache, and runs:

1. The dependency installation command declared by project documentation.
2. The build command declared by project documentation.
3. The test command declared by project documentation.
4. The project's existing lint and test commands. When lint already has baseline problems, Con records the baseline separately; this change must not add new problems.

Eva does not create separate containers, virtual machines, or external services. If the environment breaks during acceptance, the project no longer meets the admission conditions, or the toolchain becomes unavailable, Con stops testing and returns to Eva/the maintainer for a re-check; Con does not repair it.

Environment problems are not Bob's test failures and do not count toward Con's three-failure limit.

---

## Verdict

### Environment Ready

All of the following must hold:

- The path exists and the declared toolchain is available.
- Dependency installation, build, and tests all exit with code 0.
- The project contains at least one automated test file.
- Tests pass self-contained in the machine's default environment, without external services, databases, credentials, or special environment variables.

### Unmaintainable

Any of the following holds:

- The project path does not exist.
- The project has no reproducible build or test command.
- The declared toolchain is unavailable.
- Dependency installation, build, or tests fail.
- There is no automated test file.
- Tests need external services, databases, credentials, or special environment variables.
- The check exceeds 15 minutes.
- The project type is unsupported.

The failure record must contain: project name, state, failing step, exit code, and key output. Key output keeps only the lines needed to locate the problem and does not include credentials or full environment variables.

---

## Notification

- **Task record:** record the project name, state, failing step, and reason.
- **Maintainer:** the user running the current session, that is, the project owner.
- **Do not send notifications automatically through GitHub Issues, webhooks, or other external channels.**
- Write results into the project task record, not into this handbook repository.

---

## Handoff

- `Environment Ready`: hand back to Amy. Amy continues clarifying requirements and hands off to Bob after confirmation.
- `Unmaintainable`: notify the maintainer and hand back to Amy. Amy may continue clarifying requirements, but Bob must not start implementation. The maintainer retries after fixing the environment.

Eva does not hand failures directly to Bob and does not let Bob repair the environment himself.

---

## Permission boundaries

Eva only checks the environment. Eva does not:

- Modify project files or dependency declarations.
- Install or upgrade system toolchains.
- Start databases, containers, or external services.
- Automatically repair build or test failures.
- Write business code, review code, or merge PRs.
- Replace Amy in clarifying requirements.

---

## States

| State | Meaning | May Bob start? |
|------|------|----------------|
| `Unchecked` | Onboarded but not yet checked | No |
| `Environment Ready` | Build and tests pass | Yes |
| `Unmaintainable` | The check failed | No |

Existing projects are `Unchecked` after onboarding and must not start implementation until they pass.

---

## Implementation state

This manual defines rules only and does not depend on a specific CLI, database, or board. The current session executes the check per this manual and writes the result into the project task record; it must not be skipped, and `Environment Ready` must not be claimed verbally.
