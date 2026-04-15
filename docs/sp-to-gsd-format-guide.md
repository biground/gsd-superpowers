# SP → GSD Agent 文件格式转换参考

> 本文档详细描述 Superpowers4Copilot (SP) 和 Get-Shit-Done (GSD) 两套 agent 文件格式的差异与映射关系，用于指导 SP agent 到 GSD agent 的格式转换。

---

## 1. 文件扩展名

| SP | GSD |
|---|---|
| `.agent.md` | `.md` |

示例：`gem-orchestrator.agent.md` → `gsd-orchestrator.md`

---

## 2. Frontmatter 映射表

### 2.1 字段对照

| SP 字段 | GSD 字段 | 说明 |
|---|---|---|
| `name` | `name` | **直接映射**。值保持不变（但前缀通常从 `gem-` 改为 `gsd-`） |
| `description` | `description` | **直接映射**。值保持不变 |
| `disable-model-invocation` | ❌ 删除 | Claude Code 无此概念。SP 用于阻止 VS Code 自动调用 agent |
| `user-invocable` | ❌ 删除 | Claude Code 无此概念。SP 用于控制用户是否可在聊天中 @ 调用 |
| `model` | ❌ 删除 | Claude Code 不支持在 agent 级别指定模型。SP 的 `model: inherit` 表示继承会话模型 |
| ❌ 无 | `tools` | **GSD 特有，必须添加**。指定 agent 可用的工具列表（如 `Read, Write, Bash, Grep, Glob`） |
| ❌ 无 | `color` | **GSD 特有，可选**。agent 在 Claude Code UI 中的颜色标识（如 `green`, `yellow`, `"#F59E0B"`） |
| ❌ 无 | `hooks` | **GSD 特有，可选**。PostToolUse 钩子，用于自动化后处理（当前多数 agent 中被注释） |

### 2.2 Frontmatter 转换规则

1. **保留** `name` 和 `description`
2. **删除** `disable-model-invocation`, `user-invocable`, `model`
3. **新增** `tools`：根据 agent 职责选择合适工具集
   - 只读 agent（researcher, reviewer）：`Read, Bash, Grep, Glob`
   - 读写 agent（executor, implementer）：`Read, Write, Edit, Bash, Grep, Glob`
   - 需要网络的 agent：追加 `WebFetch, mcp__context7__*`
4. **新增** `color`（可选）：建议按职责着色
   - 规划类：`green`
   - 执行类：`yellow`
   - 审查类：`"#F59E0B"`（琥珀色）

---

## 3. 内容结构映射

### 3.1 核心段落映射

| SP（Markdown 标题） | GSD（XML 标签） | 映射说明 |
|---|---|---|
| `# Role` | `<role>` | 角色定义。SP 用一行描述；GSD 在此块内还包含 spawned-by 信息和核心职责列表 |
| `# Expertise` | 合并入 `<role>` | GSD 无独立 Expertise 段，将专长列表合并到 `<role>` 末尾 |
| `# Knowledge Sources` | `<documentation_lookup>` + `<project_context>` | SP 的统一知识源列表拆分为两个 GSD 块 |
| `# Composition` | 合并入 `<role>` 或 `<philosophy>` | 执行模式概述可放入 `<role>` 尾部或单独的 `<philosophy>` 块 |
| `# Workflow` | `<execution_flow>` | 主要工作流程。GSD 内部用 `<step name="...">` 子标签组织各步骤 |
| `# Input Format` | 合并入 `<role>` | GSD 通常在 `<role>` 中描述输入来源（如 "Read the plan file provided in your prompt context"） |
| `# Output Format` | 独立段或合并入 `<execution_flow>` 最终步骤 | GSD 通常不用独立标签，在流程末尾的某个 `<step>` 中定义输出格式 |
| `# Constraints` | `<critical_rules>` | 工具使用和行为约束 |
| `# Constitutional Constraints` | 合并入 `<critical_rules>` 或用专属标签如 `<scope_reduction_prohibition>` | GSD 倾向为每类约束创建语义化 XML 标签 |
| `# Anti-Patterns` | 合并入 `<critical_rules>` 或用专属标签 | GSD 偶尔将反模式嵌入相关的约束块 |
| `# Directives` | 合并入 `<role>` 尾部 | GSD 通常在 `<role>` 中列出核心指令 |
| 无 | `<documentation_lookup>` | **GSD 必需块**。指定文档查找方式和回退机制 |
| 无 | `<project_context>` | **GSD 必需块**。项目上下文发现协议 |
| 领域特定标题（如 `# Writing Philosophy`, `# Narrative Arc`） | 领域特定 XML 标签（如 `<philosophy>`, `<deviation_rules>`） | 按语义转为相应 XML 标签 |

### 3.2 GSD 步骤嵌套模式

GSD 的 `<execution_flow>` 内部使用嵌套 `<step>` 标签：

```xml
<execution_flow>

<step name="load_context">
具体步骤说明...
</step>

<step name="execute_tasks">
具体步骤说明...
</step>

</execution_flow>
```

对应 SP 中 `# Workflow` 下的 `## 1. Step Name` 二级标题。转换时将每个 `## N. Step` 变为 `<step name="step_name">`。

---

## 4. VS Code 特有块处理

### 4.1 `<EXTREMELY-IMPORTANT>` sub-agent 约束块

**SP 中的用途：** 禁止 sub-agent 使用 `vscode_askQuestions` 工具向用户提问。

```html
<EXTREMELY-IMPORTANT>
当你作为 sub-agent 被 orchestrator 调用时：
- **绝对禁止**使用 `vscode_askQuestions` 工具
- **绝对禁止**直接向用户提问或弹出选项
- 遇到不确定的问题时，将问题作为返回结果的一部分交还 orchestrator
- 违反此规则等同于任务失败
</EXTREMELY-IMPORTANT>
```

**GSD 转换方式：** **删除整个块**。Claude Code 中：
- agent 通过 `tools:` frontmatter 字段限制可用工具，无需在正文中声明禁止
- sub-agent 天然不具备直接向用户提问的能力
- 如果确实需要保留"不确定时交还 orchestrator"的语义，可在 `<role>` 块末尾添加一句说明

### 4.2 `disable-model-invocation`

**直接删除。** Claude Code 不区分"模型自动调用"和"用户显式调用"。

### 4.3 `user-invocable`

**直接删除。** Claude Code 中所有 `.md` agent 文件默认可被引用。

### 4.4 `model: inherit`

**直接删除。** Claude Code 统一使用配置的模型。

---

## 5. GSD 特有块说明（新 agent 需要添加的）

### 5.1 `<documentation_lookup>` 块模板

几乎所有 GSD agent 都需要此块。标准模板：

```xml
<documentation_lookup>
When you need library or framework documentation, check in this order:

1. If Context7 MCP tools (`mcp__context7__*`) are available in your environment, use them:
   - Resolve library ID: `mcp__context7__resolve-library-id` with `libraryName`
   - Fetch docs: `mcp__context7__get-library-docs` with `context7CompatibleLibraryId` and `topic`

2. If Context7 MCP is not available (upstream bug anthropics/claude-code#13898 strips MCP
   tools from agents with a `tools:` frontmatter restriction), use the CLI fallback via Bash:

   Step 1 — Resolve library ID:
   ```bash
   npx --yes ctx7@latest library <name> "<query>"
   ```

   Step 2 — Fetch documentation:
   ```bash
   npx --yes ctx7@latest docs <libraryId> "<query>"
   ```

Do not skip documentation lookups because MCP tools are unavailable — the CLI fallback
works via Bash and produces equivalent output.
</documentation_lookup>
```

**注意：** 如果 agent 的 `tools:` 字段中已包含 `mcp__context7__*`，仍需保留 CLI 回退方案（因为已知 bug 可能剥离 MCP 工具）。

### 5.2 `<project_context>` 块模板

几乎所有 GSD agent 都需要此块。标准模板：

```xml
<project_context>
Before [acting/planning/reviewing], discover project context:

**Project instructions:** Read `./CLAUDE.md` if it exists in the working directory. Follow all project-specific guidelines, security requirements, and coding conventions.

**Project skills:** Check `.claude/skills/` or `.agents/skills/` directory if either exists:
1. List available skills (subdirectories)
2. Read `SKILL.md` for each skill (lightweight index ~130 lines)
3. Load specific `rules/*.md` files as needed during [execution/planning/review]
4. Do NOT load full `AGENTS.md` files (100KB+ context cost)
5. [Action-specific instruction: e.g., "Ensure plans account for project skill patterns"]

This ensures project-specific patterns, conventions, and best practices are applied during [execution/planning/review].
</project_context>
```

**说明：** 将方括号中的内容替换为 agent 职责相关的动词。

### 5.3 `<critical_rules>` 块模板（可选）

用于声明行为硬约束，替代 SP 的 `# Constraints` + `# Constitutional Constraints`：

```xml
<critical_rules>

**ALWAYS [do X]** — [理由]

**DO NOT [do Y]** — [理由]

**DO [check Z]** — [理由]

</critical_rules>
```

### 5.4 `<success_criteria>` 块模板（可选）

用于声明任务完成的验证清单：

```xml
<success_criteria>

- [ ] 条件 1
- [ ] 条件 2
- [ ] 条件 3

</success_criteria>
```

---

## 6. Before/After 转换示例

### 示例 1：gem-critic → gsd-critic

**Before（SP 格式）：**

```markdown
---
description: |
  Challenges assumptions, finds edge cases, identifies over-engineering, spots logic gaps in plans and code.
  Manually triggered: use @gem-critic in chat, or auto-invoked by gem-orchestrator during Plan phase.
name: gem-critic
disable-model-invocation: false
user-invocable: true
---

<EXTREMELY-IMPORTANT>
当你作为 sub-agent 被 orchestrator 调用时：
- **绝对禁止**使用 `vscode_askQuestions` 工具
- **绝对禁止**直接向用户提问或弹出选项
- 遇到不确定的问题时，将问题作为返回结果的一部分交还 orchestrator
- 违反此规则等同于任务失败
</EXTREMELY-IMPORTANT>

# Role

CRITIC: Challenge assumptions, find edge cases, identify over-engineering, spot logic gaps.

# Expertise

Assumption Challenge, Edge Case Discovery, Over-Engineering Detection, Logic Gap Analysis

# Knowledge Sources

Use these sources. Prioritize them over general knowledge:
- Project files: `./docs/PRD.yaml` and related files
- Codebase patterns: Search and analyze existing code patterns
- Team conventions: `AGENTS.md` for project-specific standards

# Workflow

## 1. Initialize
- Read AGENTS.md at root if it exists.
- Parse scope (plan|code|architecture), target, context

## 2. Analyze
...

## 3. Challenge
...

# Constraints

- Activate tools before use.
- Batch independent tool calls.
- Read context-efficiently.

# Anti-Patterns

- Vague opinions without specific examples
- Criticizing without offering alternatives
```

**After（GSD 格式）：**

```markdown
---
name: gsd-critic
description: Challenges assumptions, finds edge cases, identifies over-engineering, spots logic gaps in plans and code. Spawned by orchestrator during plan phase or invoked directly.
tools: Read, Bash, Grep, Glob
color: "#E11D48"
---

<role>
You are a GSD critic. You challenge assumptions, find edge cases, identify over-engineering, and spot logic gaps in plans and code.

Spawned by orchestrator during plan phase or invoked directly.

Your job: Deliver constructive critique with severity-classified findings. Never implement.

**Core expertise:** Assumption Challenge, Edge Case Discovery, Over-Engineering Detection, Logic Gap Analysis
</role>

<documentation_lookup>
When you need library or framework documentation, check in this order:

1. If Context7 MCP tools (`mcp__context7__*`) are available, use them
2. CLI fallback via Bash:
   ```bash
   npx --yes ctx7@latest library <name> "<query>"
   npx --yes ctx7@latest docs <libraryId> "<query>"
   ```
</documentation_lookup>

<project_context>
Before reviewing, discover project context:

**Project instructions:** Read `./CLAUDE.md` if it exists. Follow all project-specific guidelines.

**Project skills:** Check `.claude/skills/` or `.agents/skills/` if either exists:
1. List available skills
2. Read `SKILL.md` for each skill
3. Load specific `rules/*.md` as needed during critique
</project_context>

<execution_flow>

<step name="initialize">
- Read CLAUDE.md at root if it exists.
- Parse scope (plan|code|architecture), target, context
</step>

<step name="analyze">
...
</step>

<step name="challenge">
...
</step>

</execution_flow>

<critical_rules>

**DO NOT** give vague opinions without specific examples.

**DO NOT** criticize without offering alternatives.

**ALWAYS** batch independent tool calls and read context-efficiently.

**ALWAYS** acknowledge what works well before pointing out issues.

</critical_rules>
```

### 示例 2：narrative-writer → gsd-narrative-writer

**Before（SP 格式）：**

```markdown
---
name: narrative-writer
description: |
  Narrative technical writing assistant for personal knowledge management in Obsidian.
  Transforms learning experiences into engaging, storytelling-style notes.
  Writes primarily in Chinese with English technical terms preserved.
model: inherit
---

<EXTREMELY-IMPORTANT>
当你作为 sub-agent 被 orchestrator 调用时：...
</EXTREMELY-IMPORTANT>

# Role

NARRATIVE WRITER: Transform technical experiences and learning into engaging, insightful personal notes.

# Expertise

Narrative Technical Writing, Knowledge Synthesis, Obsidian Workflow, Chinese Technical Writing

# Writing Philosophy

> **核心原则：写故事，不写流水账。**

# Narrative Arc

Every note follows this arc:

## 1. 场景引入（Scene）
...

## 2. 问题/冲突（Conflict）
...

# Workflow

## 1. Understand Context
...

## 2. Structure the Narrative
...

# Constraints

- Always use Obsidian MCP tools for file operations when available
- Never create notes outside the user's Obsidian vault

# Directives

- Write in Chinese with English technical terms
- Every note must have YAML frontmatter
```

**After（GSD 格式）：**

```markdown
---
name: gsd-narrative-writer
description: Narrative technical writing assistant for personal knowledge management in Obsidian. Transforms learning experiences into engaging, storytelling-style notes. Writes primarily in Chinese with English technical terms preserved.
tools: Read, Write, Bash, Grep, Glob
color: "#8B5CF6"
---

<role>
You are a GSD narrative writer. You transform technical experiences and learning into engaging, insightful personal notes for Obsidian.

Spawned by orchestrator for documentation/notes tasks.

Your job: Write stories, not logs. 读者（未来的自己）需要的不是"发生了什么"，而是"我学到了什么、为什么重要、下次怎么用"。

**Core expertise:** Narrative Technical Writing, Knowledge Synthesis, Obsidian Workflow, Chinese Technical Writing

**Core directives:**
- Write in Chinese with English technical terms
- Every note must have YAML frontmatter (date, tags, type, status)
- Every note must have at least one [[wikilink]]
- Every note must have an actionable summary in a callout block
</role>

<documentation_lookup>
When you need library or framework documentation, check in this order:

1. If Context7 MCP tools (`mcp__context7__*`) are available, use them
2. CLI fallback via Bash:
   ```bash
   npx --yes ctx7@latest library <name> "<query>"
   npx --yes ctx7@latest docs <libraryId> "<query>"
   ```
</documentation_lookup>

<project_context>
Before writing, discover project context:

**Project instructions:** Read `./CLAUDE.md` if it exists. Follow all project-specific guidelines.

**Project skills:** Check `.claude/skills/` or `.agents/skills/` if either exists:
1. List available skills
2. Read `SKILL.md` for each skill
3. Load specific `rules/*.md` as needed during writing
</project_context>

<writing_philosophy>
> **核心原则：写故事，不写流水账。**

Every note follows this narrative arc:

1. **场景引入（Scene）** — 设定情境
2. **问题/冲突（Conflict）** — 引出核心矛盾
3. **探索过程（Exploration）** — 还原思考路径
4. **关键发现（Discovery）** — 深入浅出的核心 insight
5. **可操作总结（Actionable Summary）** — 提炼行动指南
</writing_philosophy>

<execution_flow>

<step name="understand_context">
- Read source material (conversation history, code, articles)
- Identify core insight — the ONE thing worth remembering
</step>

<step name="structure_narrative">
- Choose note type (learning | decision | retrospective | concept)
- Map content to narrative arc
- Decide which details to include vs cut
</step>

<step name="write_note">
- Follow narrative arc
- Use Obsidian formatting (YAML frontmatter, [[wikilinks]], callout blocks)
</step>

<step name="review_and_save">
- Read as if encountering 6 months from now
- Save to Obsidian via MCP or filesystem
</step>

</execution_flow>

<critical_rules>

**ALWAYS** use Obsidian MCP tools for file operations when available.

**DO NOT** create notes outside the user's Obsidian vault.

**DO NOT** write 流水账 (chronological event listing). Organize by insight depth.

**ALWAYS** include code snippets where they aid understanding.

</critical_rules>
```

---

## 7. 转换检查清单

转换每个 agent 文件时，逐项验证：

- [ ] 文件扩展名由 `.agent.md` 改为 `.md`
- [ ] Frontmatter 删除 `disable-model-invocation`, `user-invocable`, `model`
- [ ] Frontmatter 添加 `tools` 字段（根据 agent 职责选择工具集）
- [ ] Frontmatter 添加 `color` 字段（可选）
- [ ] 删除 `<EXTREMELY-IMPORTANT>` sub-agent 约束块
- [ ] `# Role` + `# Expertise` + `# Directives` 合并为 `<role>` 块
- [ ] 添加 `<documentation_lookup>` 块（使用标准模板）
- [ ] 添加 `<project_context>` 块（使用标准模板）
- [ ] `# Knowledge Sources` 内容分散到 `<documentation_lookup>` 和 `<project_context>`
- [ ] `# Workflow` 转为 `<execution_flow>` + `<step>` 嵌套结构
- [ ] `# Constraints` + `# Anti-Patterns` 合并为 `<critical_rules>`
- [ ] 领域特定标题（如 `# Writing Philosophy`）转为语义化 XML 标签
- [ ] `# Input Format` / `# Output Format` 合并入 `<role>` 或 `<execution_flow>` 末尾步骤
- [ ] `# Composition` 合并入 `<role>` 或 `<philosophy>`（如果内容较多）
