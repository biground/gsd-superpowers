---
name: gsd:sp-subagent
description: "子代理驱动开发——每步派遣独立代理完成复杂多步骤任务。"
argument-hint: "[task description]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
---

<objective>
将复杂多步骤任务拆解为独立子任务，为每个子任务分派全新子代理执行，完成后进行两阶段审查（规格合规 + 代码质量）确保质量。

每个子代理拥有独立的全新上下文，避免上下文污染；审查环节保证交付物符合规格且代码质量达标。
</objective>

<execution_context>
@~/.claude/get-shit-done/skills/subagent-driven-development/SKILL.md
</execution_context>

<context>
Task description: $ARGUMENTS

子代理 prompt 模板位于技能目录中：
- implementer-prompt.md — 实现者代理
- spec-reviewer-prompt.md — 规格合规审查代理
- code-quality-reviewer-prompt.md — 代码质量审查代理
</context>

<process>

## 1. 分析与拆解

分析 $ARGUMENTS 描述的任务，将其拆解为可独立执行的子任务列表。每个子任务应有明确的输入、输出和验收标准。

## 2. 任务跟踪初始化

使用任务跟踪工具记录所有子任务及其状态（待执行 / 执行中 / 审查中 / 完成）。

## 3. 逐任务执行循环

对每个子任务执行以下循环：

### 3a. 派遣实现者子代理
- 分派一个全新子代理执行当前子任务
- 子代理提示词必须：聚焦单一任务、包含所有必要上下文、明确输出要求

### 3b. 规格合规审查
- 分派规格审查子代理，检查交付物是否满足验收标准
- 如不通过：将反馈返回实现者子代理修复

### 3c. 代码质量审查
- 分派代码质量审查子代理，检查代码风格、反模式、安全性
- 如不通过：将反馈返回实现者子代理修复

### 3d. 标记完成
- 两阶段审查均通过后，标记当前子任务为完成

## 4. 汇总

所有子任务完成后，汇总执行结果并报告给用户。

</process>
---
name: gsd:sp-subagent
description: "子代理驱动开发——每步派遣独立代理完成复杂多步骤任务。"
argument-hint: "[task description]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
---

<objective>
将复杂多步骤任务拆解为子任务，为每个子任务派遣全新的子代理，完成后进行两阶段审查（规格合规 + 代码质量）确保质量。

每个子代理拥有完整的上下文窗口，避免长对话中的上下文退化。
</objective>

<execution_context>
@~/.claude/get-shit-done/skills/subagent-driven-development/SKILL.md
</execution_context>

<process>
Execute the subagent-driven development workflow from the skill file end-to-end.

## 核心流程

1. **任务分析**：分析 $ARGUMENTS 中描述的任务，拆解为可独立执行的子任务
2. **子代理分派**：为每个子任务派遣独立子代理（实现者），提供自包含的指令和上下文
3. **规格合规审查**：实现完成后，派遣规格审查子代理验证输出是否符合需求
4. **代码质量审查**：通过规格审查后，派遣代码质量审查子代理检查代码质量
5. **迭代修复**：审查不通过时，将反馈交给新的子代理修复，再次审查
6. **完成确认**：所有子任务通过两阶段审查后，汇总结果

## 子代理 Prompt 模板

读取以下模板构建子代理指令：
- skills/subagent-driven-development/implementer-prompt.md
- skills/subagent-driven-development/spec-reviewer-prompt.md
- skills/subagent-driven-development/code-quality-reviewer-prompt.md
</process>
