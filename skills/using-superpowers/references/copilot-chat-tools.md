# VS Code Copilot Chat 工具映射

技能使用 Claude Code 的工具名称。当你在 VS Code Copilot Chat 中执行技能时，请使用以下对应工具：

| 技能中引用的工具 | VS Code Copilot Chat 对应工具 | 说明 |
|---|---|---|
| `Read`（读取文件） | `read_file` | 指定起止行号读取文件内容 |
| `Write`（创建文件） | `create_file` | 创建新文件（不用于编辑已有文件） |
| `Edit`（编辑文件） | `replace_string_in_file` / `multi_replace_string_in_file` | 替换已有文件中的文本 |
| `Bash`（运行命令） | `run_in_terminal` | 在持久化终端会话中执行命令 |
| `Grep`（搜索内容） | `grep_search` | 按文本或正则表达式搜索文件内容 |
| `Glob`（搜索文件名） | `file_search` | 按 glob 模式搜索文件路径 |
| `Skill`（调用技能） | `read_file` | 用 read_file 读取技能的 SKILL.md 文件并遵循 |
| `Task`（子代理） | `runSubagent` | 分派子代理执行独立任务 |
| `TodoWrite`（任务跟踪） | `manage_todo_list` | 管理待办事项列表，跟踪多步骤任务进度 |
| `WebFetch` | `fetch_webpage` | 获取网页内容 |
| `WebSearch` | 无直接对应 | 使用 `fetch_webpage` + 搜索引擎 URL |
| `AskHuman`（向用户提问） | `vscode_askQuestions` | 结构化提问（选择题/开放题） |
| `EnterPlanMode` / `ExitPlanMode` | 无对应 | 直接在会话中操作 |

## 异步命令

| 工具 | 用途 |
|------|------|
| `run_in_terminal`（mode=async） | 在后台启动长时间运行的命令 |
| `send_to_terminal` | 向运行中的终端会话发送命令 |
| `get_terminal_output` | 从异步终端读取输出 |

## 额外工具

| 工具 | 用途 |
|------|------|
| `get_errors` | 获取文件的编译/lint 错误 |
| `semantic_search` | 在代码库中语义搜索相关代码 |
| `list_dir` | 列出目录内容 |
| `memory` | 管理持久化记忆笔记 |
| `view_image` | 查看图片文件 |

## 提问规范

在需要向用户提问或提供选项时，使用 `vscode_askQuestions` 工具，而非纯文本列出选项。这提供了结构化的选择界面。
