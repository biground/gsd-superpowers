---
name: gsd:sp-brainstorm
description: "协作式头脑风暴——从想法到设计方案。在编码前使用。"
argument-hint: "[topic]"
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---

<objective>
通过结构化协作对话，将想法转化为完整的设计方案。
在编码前必须完成头脑风暴——这是 Superpowers 最核心的技能之一。
</objective>

<execution_context>
@~/.claude/get-shit-done/skills/brainstorming/SKILL.md
</execution_context>

<runtime_note>
**Copilot (VS Code):** Use `vscode_askquestions` wherever this workflow calls `AskUserQuestion`. They are equivalent — `vscode_askquestions` is the VS Code Copilot implementation of the same interactive question API. Do not skip questioning steps because `AskUserQuestion` appears unavailable; use `vscode_askquestions` instead.
</runtime_note>

<context>
Topic: $ARGUMENTS
</context>

<process>
读取并遵循 brainstorming SKILL.md 中的完整流程：

1. **探索上下文** — 检查项目文件、文档、最近提交，理解项目现状和约束
2. **澄清问题** — 逐个提问，每次提供选项；聚焦于理解目的、约束、成功标准
3. **提出方案** — 2-3 个带明确取舍的设计方案，附上推荐理由
4. **展示设计** — 可视化方案对比，逐部分展示并获取用户确认
5. **编写 spec** — 将选定方案落地为规格文档，保存到 `docs/superpowers/specs/` 目录
6. **Spec 自审** — 检查占位符、矛盾、歧义、范围问题
7. **用户审阅** — 请用户审阅 spec 文件后再继续
8. **过渡到实现** — 调用 writing-plans 技能创建实现计划

<HARD-GATE>
在展示设计方案并获得用户批准之前，不要编写任何代码、搭建任何项目脚手架，或采取任何实现行动。
</HARD-GATE>
</process>
