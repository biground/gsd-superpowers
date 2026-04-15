---
name: using-superpowers
description: Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation before ANY response including clarifying questions
---

<SUBAGENT-STOP>
如果你是作为子代理（subagent）被分派来执行特定任务的，请跳过此技能。
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
如果你认为哪怕只有 1% 的可能性某个技能（skill）适用于你正在做的事情，你就**必须**调用该技能。

如果某个技能适用于你的任务，你没有选择。你必须使用它。

这不可商量。这不是可选的。你不能为自己找理由绕过这一点。
</EXTREMELY-IMPORTANT>

## 指令优先级

Superpowers 技能会覆盖默认的系统提示（system prompt）行为，但**用户指令始终优先**：

1. **用户的显式指令**（CLAUDE.md、GEMINI.md、AGENTS.md、直接请求）——最高优先级
2. **Superpowers 技能**——在冲突处覆盖默认系统行为
3. **默认系统提示**——最低优先级

如果 CLAUDE.md、GEMINI.md 或 AGENTS.md 写了"不要使用 TDD"而某个技能写了"始终使用 TDD"，请遵循用户的指令。用户拥有控制权。

## 如何访问技能

**在 Claude Code 中：** 使用 `Skill` 工具。当你调用一个技能时，其内容会被加载并呈现给你——直接遵循即可。永远不要使用 Read 工具读取技能文件。

**在 Copilot CLI 中：** 使用 `skill` 工具。技能会从已安装的插件中自动发现。`skill` 工具的工作方式与 Claude Code 的 `Skill` 工具相同。

**在 Gemini CLI 中：** 技能通过 `activate_skill` 工具激活。Gemini 会在会话开始时加载技能元数据，并按需激活完整内容。

**在其他环境中：** 请查阅你所用平台的文档，了解技能是如何加载的。

## 平台适配

技能使用 Claude Code 的工具名称。非 CC 平台请参阅：`references/copilot-tools.md`（Copilot CLI）、`references/codex-tools.md`（Codex）获取工具对照表。Gemini CLI 用户通过 GEMINI.md 自动加载工具映射。

# 使用技能

## 规则

**在任何回复或操作之前，先调用相关或被请求的技能。** 哪怕只有 1% 的可能性某个技能适用，你都应该调用该技能进行检查。如果调用后发现该技能不适用于当前情况，你不需要使用它。

```dot
digraph skill_flow {
    "User message received" [shape=doublecircle];
    "About to EnterPlanMode?" [shape=doublecircle];
    "Already brainstormed?" [shape=diamond];
    "Invoke brainstorming skill" [shape=box];
    "Might any skill apply?" [shape=diamond];
    "Invoke Skill tool" [shape=box];
    "Announce: 'Using [skill] to [purpose]'" [shape=box];
    "Has checklist?" [shape=diamond];
    "Create TodoWrite todo per item" [shape=box];
    "Follow skill exactly" [shape=box];
    "Respond (including clarifications)" [shape=doublecircle];

    "About to EnterPlanMode?" -> "Already brainstormed?";
    "Already brainstormed?" -> "Invoke brainstorming skill" [label="no"];
    "Already brainstormed?" -> "Might any skill apply?" [label="yes"];
    "Invoke brainstorming skill" -> "Might any skill apply?";

    "User message received" -> "Might any skill apply?";
    "Might any skill apply?" -> "Invoke Skill tool" [label="yes, even 1%"];
    "Might any skill apply?" -> "Respond (including clarifications)" [label="definitely not"];
    "Invoke Skill tool" -> "Announce: 'Using [skill] to [purpose]'";
    "Announce: 'Using [skill] to [purpose]'" -> "Has checklist?";
    "Has checklist?" -> "Create TodoWrite todo per item" [label="yes"];
    "Has checklist?" -> "Follow skill exactly" [label="no"];
    "Create TodoWrite todo per item" -> "Follow skill exactly";
}
```

## 危险信号

这些想法意味着停下——你在为自己找借口：

| 想法 | 现实 |
|------|------|
| "这只是一个简单的问题" | 问题就是任务。检查是否有适用的技能。 |
| "我需要先了解更多上下文" | 技能检查在澄清问题**之前**进行。 |
| "让我先探索一下代码库" | 技能会告诉你**如何**探索。先检查技能。 |
| "我可以快速检查 git/文件" | 文件缺少对话上下文。先检查技能。 |
| "让我先收集信息" | 技能会告诉你**如何**收集信息。 |
| "这不需要正式的技能" | 如果技能存在，就使用它。 |
| "我记得这个技能" | 技能会更新。阅读当前版本。 |
| "这不算一个任务" | 行动 = 任务。检查技能。 |
| "这个技能太大材小用了" | 简单的事情会变复杂。使用它。 |
| "让我先做这一件事" | 在做任何事情**之前**检查技能。 |
| "感觉很有成效" | 无纪律的行动浪费时间。技能能防止这种情况。 |
| "我知道那是什么意思" | 了解概念 ≠ 使用技能。调用它。 |

## 技能优先级

当多个技能可能适用时，按以下顺序使用：

1. **先用流程类技能**（头脑风暴、调试）——这些决定了**如何**处理任务
2. **再用实现类技能**（前端设计、mcp-builder）——这些指导执行

"让我们构建 X"→ 先头脑风暴，再用实现类技能。
"修复这个 bug"→ 先调试，再用领域特定技能。

## 技能类型

**刚性**（TDD、调试）：严格遵循。不要因适应性调整而丢失纪律。

**柔性**（模式）：根据上下文调整原则。

技能本身会告诉你它属于哪种。

## 用户指令

指令说的是**做什么**，而不是**怎么做**。"添加 X"或"修复 Y"不代表跳过工作流。

## 可用 Agents

除了技能（skills）外，以下专业 agents 也可在工作流中调用：

| Agent | 用途 | 触发方式 |
|-------|------|----------|
| `gem-orchestrator` | 多 agent 编排，全局流程管理 | `@gem-orchestrator` 或作为主 agent 使用 |
| `gem-researcher` | 代码库探索、模式发现 | `@gem-researcher` 或由 orchestrator 分派 |
| `gem-planner` | DAG 计划生成 | `@gem-planner` 或由 orchestrator 分派 |
| `gem-implementer` | TDD 实现 + 端到端调试 | `@gem-implementer` 或由 orchestrator 分派 |
| `gem-reviewer` | 自动基线代码审查 | `@gem-reviewer` 或自动触发 |
| `gem-critic` | 批判性设计审查 | `@gem-critic`（手动触发）|
| `gem-documentation-writer` | 代码文档 | `@gem-documentation-writer` |
| `se-technical-writer` | 技术博客/教程/ADR | `@se-technical-writer` |
| `narrative-writer` | Obsidian 叙事笔记 | `@narrative-writer` |
| `code-reviewer` | 深度计划对齐审查 | `@code-reviewer` |

Agents 和 Skills 的关系：Skills 定义质量纪律（HOW），Agents 提供专业能力（WHO）。两者互补。
