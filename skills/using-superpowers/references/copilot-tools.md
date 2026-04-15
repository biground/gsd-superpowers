# Copilot CLI 工具映射

技能使用 Claude Code 的工具名称。当你在技能中遇到这些工具名时，请使用你所在平台的对应工具：

| 技能中引用的工具 | Copilot CLI 对应工具 |
|-----------------|----------------------|
| `Read`（读取文件） | `view` |
| `Write`（创建文件） | `create` |
| `Edit`（编辑文件） | `edit` |
| `Bash`（运行命令） | `bash` |
| `Grep`（搜索文件内容） | `grep` |
| `Glob`（按名称搜索文件） | `glob` |
| `Skill` 工具（调用技能） | `skill` |
| `WebFetch` | `web_fetch` |
| `Task` 工具（分派子代理） | `task`（参见[代理类型](#代理类型)） |
| 多次 `Task` 调用（并行） | 多次 `task` 调用 |
| Task 状态/输出 | `read_agent`、`list_agents` |
| `TodoWrite`（任务跟踪） | `sql` 配合内置 `todos` 表 |
| `WebSearch` | 无对应工具——使用 `web_fetch` 配合搜索引擎 URL |
| `EnterPlanMode` / `ExitPlanMode` | 无对应工具——在主会话中操作即可 |

## 代理类型

Copilot CLI 的 `task` 工具接受 `agent_type` 参数：

| Claude Code 代理 | Copilot CLI 对应工具 |
|-------------------|----------------------|
| `general-purpose` | `"general-purpose"` |
| `Explore` | `"explore"` |
| 命名插件代理（如 `superpowers:code-reviewer`） | 从已安装的插件中自动发现 |

## 异步 shell 会话

Copilot CLI 支持持久化的异步 shell 会话，这在 Claude Code 中没有直接对应功能：

| 工具 | 用途 |
|------|------|
| `bash` 配合 `async: true` | 在后台启动长时间运行的命令 |
| `write_bash` | 向运行中的异步会话发送输入 |
| `read_bash` | 从异步会话读取输出 |
| `stop_bash` | 终止异步会话 |
| `list_bash` | 列出所有活跃的 shell 会话 |

## Copilot CLI 额外工具

| 工具 | 用途 |
|------|------|
| `store_memory` | 持久化代码库相关信息供后续会话使用 |
| `report_intent` | 更新 UI 状态栏显示当前意图 |
| `sql` | 查询会话的 SQLite 数据库（待办事项、元数据） |
| `fetch_copilot_cli_documentation` | 查阅 Copilot CLI 文档 |
| GitHub MCP 工具（`github-mcp-server-*`） | 原生 GitHub API 访问（issues、PRs、代码搜索） |
