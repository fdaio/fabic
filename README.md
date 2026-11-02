# fabic · Agent 协作手册

一套与具体项目无关的 Agent 协作手册。它定义 Amy、Eva、Bob、Con 四个角色的职责、交接条件、审查流程与合并规则。

本仓库只保存手册和模板，不包含运行时、数据库、看板或具体项目数据。

## 怎么用

1. 先读 `AGENTS.md`。它是团队成员的全部约束来源。
2. 按角色读 `docs/agent/` 下对应手册。
3. 接入项目时，项目继续使用自己的仓库、issue tracker、CI 和检查命令；本手册只提供协作规则。
4. 手册中的示例只使用占位符：`<project>`、`<repo>`、`<default-branch>`、`REQ-001`、`example.com`。

## 工作流

```text
新项目接入
  -> Eva 环境准入
  -> Amy 澄清并确认
  -> Bob 实现 + 自测（feature 分支）
  -> Con 测试验收
  -> Con 开 PR
  -> 分层审查 + CI
  -> 合入默认分支
  -> 清理 worktree
```

- 环境不可用：交回 Eva 或维护者，Bob 不开始实现。
- 测试失败：Con 记录并打回 Bob。
- 连续三次测试失败：Con 停止测试并汇总给 Amy；只有 Amy 能开启新一轮验收。
- 风险升级：Bob 或 Con 停止并回 Amy 重新定级；不在合并前临时问用户。

## 文档索引

| 文档 | 内容 |
|------|------|
| `AGENTS.md` | 角色、交接、风险分级与合并的硬规则 |
| `docs/agent/amy-manual.md` | Amy：需求澄清、六项自检、风险定级、三次失败判断 |
| `docs/agent/eva-manual.md` | Eva：环境准入、检查命令、状态与失败处理 |
| `docs/agent/con-manual.md` | Con：测试验收、审查、PR、CI、合并、worktree 清理 |
| `docs/agent/review-manual.md` | 分层审查、安全审查、风险审批、回滚与复盘 |
| `REVIEW.md` | 本手册仓库的审查重点与高风险路径 |
| `.github/pull_request_template.md` | PR 描述模板 |

## 维护原则

- 手册与具体项目解耦：不写仓库名、组织名、客户名、内部项目名或本机路径。
- 不记录任何个人、客户、内部项目、凭证或真实业务数据。
- 规则改动优先改 `AGENTS.md`；角色操作细节放 `docs/agent/`。
- 文档声称的行为必须能在规则或项目现场命令中找到依据。

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for the full text.
