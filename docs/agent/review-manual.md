# 代码审查手册

本文件说明 Agent 协作的分层审查流程。硬规则见 `../../AGENTS.md`，仓库特定的审查重点见目标仓库的 `REVIEW.md`，Con 的操作步骤见 `con-manual.md`。

## 0. 原则

- 审查有两个目标：找问题，以及让参与者理解改动。
- 自动检查负责大部分阅读；人的注意力留给风险、方向和取舍。
- 审查是连续的：实现前自检，PR 时审查，合并后监控。
- 证据优先：命令、输出、`文件:行号`；不接受“看起来没问题”。

## 1. 七层流程

| 层 | 落点 | 阻塞条件 |
|----|------|----------|
| 1. 实现前自检 | Bob 跑项目文档定义的检查命令，自读 diff，填 PR 模板 | 没有真实输出，或 diff 超出 Amy 的 scope |
| 2. PR 审查 | Con 读目标仓库的 `REVIEW.md`（如有）、Amy 任务记录、PR intent，再读 diff | finding 未修复或未答复 |
| 3. 安全审查 | Con 单独做一遍安全清单 | 有未解决的 `Security` finding |
| 4. 自动检查 | 项目 CI；没有 CI 时由 Con 执行本地检查 | 任一 job 红、没有触发检查，或没有可复现证据 |
| 5. 风险审批 | Amy 在方案阶段定级并随「是否 OK」确认；Con 核对实际 diff | high 未在 Amy 阶段确认，或实际 diff 高于定级 |
| 6. 合并与回滚 | 更新 base、重跑 CI、核对批准 SHA、合并后监控 | base 落后、批准过期、默认分支 CI 红 |
| 7. 复盘 | 3 轮收敛、同类问题修到源头、事故转 eval | 同一 PR 超过 3 轮仍不收敛 |

## 2. 实现前自检（Bob）

Bob 在开 PR 前完成，不把自检工作留给 Con。

1. 一个 PR 只做一件事；不混入无关重构、格式化或其它任务的文件。
2. 在任务 worktree 内运行项目文档定义的检查命令，保留命令和真实输出。
3. 自己完整读一遍 diff，删除调试代码、临时注释、无关文件和敏感信息。
4. 填 PR 模板：`Summary`、`Scope`、`Changes`、`Testing`、`Risk`、`Rollback`、`Security`。
5. `Testing` 必须贴命令和输出；没有跑过的测试、benchmark 或工具，不得声称跑过。

## 3. PR 审查（Con）

Con 是合入前的审查者；不使用 subagent。为了减少 anchoring：

1. 先读目标仓库的 `REVIEW.md`（如有）、Amy 的任务记录和 PR 描述，再读 diff。
2. 最后才读 Bob 的实现说明；不要用实现理由替代代码证据。
3. 检查四件事：是否解决 PR 描述的问题、是否适配系统、是否破坏其它路径、bug 修复是否处理根因。
4. 每条代码 finding 给出 `文件:行号` 和分级：`Important`、`Nit`、`Pre-existing`、`Security`；流程问题给出具体命令或缺失项。
5. 汇总以数量开头，例如 `3 findings (1 Important, 2 Nit)`；没有 finding 就写 `No findings`。
6. 每条 finding 必须修复，或在 thread 写明不修的理由；合并前不能有未解决 thread。
7. 纯风格问题标 `Nit`，不阻塞；不要为了显得认真而制造 finding。
8. 已修复或已回答的 finding 不重复提。

## 4. 安全审查（阻塞）

Con 在正确性审查之后单独做一遍，不把安全问题和普通 finding 混在一起。

- 命令执行：`exec.Command` 参数是否可控；不要拼接 shell 字符串。
- 路径：是否可能穿越、跟随恶意 symlink、写到项目外。
- SQL：是否全部参数化；迁移或回填是否可能丢数据。
- 凭证：diff、日志、PR 描述是否出现 token、密码、连接串或本地绝对路径。
- 文件权限：敏感文件是否过宽。
- CI 和依赖：workflow 权限是否最小；新增依赖是否必要；第三方 action 是否可信。
- Prompt injection：不可信文本是否直接进入 LLM prompt。

任一条无法排除 → 标 `Security`，阻塞合并。误报必须在 thread 写明理由，不能默默忽略。

## 5. 自动检查

- 项目的检查命令以项目文档、`Makefile`、任务脚本或 CI 配置为准；不要假设所有项目使用同一条命令。
- 如果项目有 CI，PR 上必须全绿；没有检查触发时，Con 先查目标仓库的 CI 配置和触发条件，不能把 `no checks reported` 当成通过。
- 如果项目没有 CI，Con 必须在 PR 的 Testing 中贴出本地检查命令与真实输出；没有证据不算通过。
- 如果平台不支持 required checks，Con 必须手动确认 CI 绿。

## 6. 风险分级与审批

| 等级 | 判定 | 审批 | 额外要求 |
|------|------|------|----------|
| Low | 文档、注释、纯测试改动，无行为变化 | Con + CI | 无 |
| Medium | 普通代码、bug 修复、重构 | Amy 范围确认 + Con + CI | 测试和真实输出 |
| High | CI/发布、认证、数据迁移、删除性脚本、工作流规则、依赖大版本 | Amy 方案确认（含用户）+ Con + CI | 可信回滚方案 |

- 定级由 Amy 在方案阶段完成，并随同一次「是否 OK」确认；不确定取高，Con 可以上调后交回 Amy 确认。
- 本手册面向小团队 + Agent。Low 由 Con + CI 放行；Medium 由 Amy + Con + CI 放行；High 的用户确认发生在 Amy 阶段，不在合并前。
- Con 只核对实际 diff 是否匹配 Amy 的定级。实际 diff 更高时，停止并交回 Amy 重新定级确认；不得在合并前临时问用户。
- 批准绑定当前 `headRefOid`。Bob 再 push 新 commit 后重新审查；风险未升级就不重新问用户。
- 实际风险高于声明且未回 Amy 确认，不得合并，也不得用 `--admin` 绕过。

## 7. 合并、回滚与合并后监控

合并前，Con 必须确认：

1. CI 全绿，所有 finding 已修复或答复。
2. 批准 SHA 等于当前 `headRefOid`。
3. base 没有前进；如果落后，先更新分支，再重跑项目检查和 CI。
4. 没有冲突；有冲突就停止，交回 Bob。

合并后：

1. 确认默认分支 CI 绿；红了立即 revert 或开 fix PR，不把回归留给用户。
2. 回归修复必须补测试；把案例加入 review eval。
3. 删除本次任务的 worktree 和残留分支。

`Rollback` 必须可执行：代码回滚写清 revert 哪个 commit；数据变更写清备份和回滚步骤；能使用 feature flag 的，写清关闭方式。

## 8. 收敛与改进

- 这里的 3 轮是 PR 审查轮次，和测试验收的 3 次失败分开计数。
- 第 1 轮看完整 diff；第 2 轮起只看新增或变化的 diff。
- 同一 PR 超过 3 轮仍未收敛：停止，交回 Amy 重新评估方案，不再逐行修补。
- 同一类 finding 出现两次：更新目标仓库的 `REVIEW.md`、加测试、加 lint 或加 CI 检查，修到源头，不只修当前实例。
- 合并后出现回归：先 revert 或修复，再补回归测试，并把案例加入下面的 eval 集。
- 审查噪声要控制：finding 分级，纯风格不阻塞，不重复提已解决的问题。

## 9. 最小 review eval 集

每次修改 `../../AGENTS.md`、`../../REVIEW.md` 或本手册后，用下面 10 个场景校准 Con：

| # | 场景 | 期望 |
|---|------|------|
| 1 | diff 包含无关重构 | 标 `Important`，要求拆出或删除 |
| 2 | 行为变化但没有测试或真实输出 | 标 `Important`，要求补测试和证据 |
| 3 | diff 出现 token、密码或本地绝对路径 | 标 `Security`，阻塞 |
| 4 | CI 红或没有触发检查 | 阻塞，直到绿或 workflow 修好 |
| 5 | PR 与 base 冲突或落后 | 停止；冲突交回 Bob，落后则更新并重跑 |
| 6 | 批准后又 push 新 commit | 重新审查；风险升级则回 Amy |
| 7 | 只有命名或格式问题 | 标 `Nit`，不阻塞 |
| 8 | High 风险没有回滚方案，或实际 diff 高于 Amy 定级且未回 Amy | 阻塞或回 Amy |
| 9 | 同一 PR 超过 3 轮仍不收敛 | 停止，交回 Amy |
| 10 | 同类 finding 在多个 PR 重复 | 更新 `REVIEW.md` 或加检查，修到源头 |

## 10. 适配与限制

- **没有 subagent**：Con 亲自审查，不做 fresh-context subagent 或多 Agent 投票。代码 finding 必须用 `文件:行号` 自己验证。高风险任务可选在新会话启动 Con 复核，但仍然不是 subagent。
- **没有 branch protection**：如果平台不支持 required checks，Con 手动确认 CI。
- **没有 merge queue**：用“更新 base + 重跑 CI”作为等价步骤。
- **没有自动安全扫描器**：当前使用阻塞式安全清单；后续可接 govulncheck、CodeQL 或其它扫描器，但不是本手册依赖。
- **README/文档不是证据**：文档声明必须能在代码或命令输出里找到对应行为。
