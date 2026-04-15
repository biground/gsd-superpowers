---
name: gsd:sp-parallel
description: "并行代理调度——3+ 个独立任务同时进行，最大化吞吐量。"
argument-hint: "[task list or description]"
allowed-tools:
  - Read
  - Bash
  - Task
---

<objective>
识别可并行执行的独立任务，同时分派多个专用子代理并行处理，最大化吞吐量。

适用于 3 个及以上互不依赖的任务需要同时推进的场景。每个代理的提示词必须聚焦、自包含、明确输出要求。
</objective>

<execution_context>
@~/.claude/get-shit-done/skills/dispatching-parallel-agents/SKILL.md
</execution_context>

<context>
Task list or description: $ARGUMENTS
</context>

<process>

## 1. 任务分析与依赖检测

分析 $ARGUMENTS，识别所有待执行任务。检测任务间的依赖关系，将可并行执行的独立任务分组。

如果存在依赖关系，按拓扑排序划分执行批次：同一批次内的任务可并行，批次之间串行。

## 2. 代理提示词准备

为每个独立任务准备自包含的子代理提示词，确保：
- **聚焦**：每个代理只负责一个明确的问题域
- **自包含**：所有必要上下文包含在提示词中，不依赖其他代理的输出
- **明确输出**：清晰定义期望的交付物和验收标准

## 3. 并行分派

使用 Task 工具同时分派所有独立子代理。跟踪每个代理的分派和完成状态。

## 4. 结果收集与冲突检测

等待所有并行代理完成，收集各自的执行结果。运行测试确认各代理的变更之间无冲突。

## 5. 汇总报告

汇总所有代理的执行结果，报告成功/失败状态。如有冲突，提供冲突详情和建议解决方案。

</process>
