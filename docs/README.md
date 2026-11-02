# 手册索引

本目录保存角色手册与审查细则。入口规则见根目录 `../AGENTS.md`。

| 文档 | 作用 |
|------|------|
| `../AGENTS.md` | 角色、交接、风险与合并的硬规则 |
| `agent/amy-manual.md` | Amy：需求澄清、六项自检、风险定级、三次失败判断 |
| `agent/eva-manual.md` | Eva：环境准入、检查命令、状态与失败处理 |
| `agent/con-manual.md` | Con：测试验收、审查、PR、CI、合并、worktree 清理 |
| `agent/review-manual.md` | 分层审查、安全审查、风险审批、回滚与复盘 |
| `../REVIEW.md` | 本手册仓库的审查重点与高风险路径 |

## 阅读顺序

1. 新成员：`../README.md` -> `../AGENTS.md` -> 角色手册。
2. 接任务：`agent/amy-manual.md` -> `../AGENTS.md` 的 Bob 段 -> `agent/con-manual.md`。
3. 改手册：先读 `../REVIEW.md`，再按 `agent/review-manual.md` 自检。

## 约定

- 示例只使用占位符：`<project>`、`<repo>`、`<default-branch>`、`REQ-001`、`example.com`。
- 手册不绑定具体项目、仓库、组织、工具或本机路径。
- 规则冲突时以 `../AGENTS.md` 为准；角色手册只补充操作细节。
- 不把任务数据、客户数据、凭证或本机环境信息写入本仓库。
