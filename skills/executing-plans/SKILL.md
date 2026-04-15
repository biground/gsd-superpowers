---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# 执行计划

## 概述

加载计划，批判性审查，执行所有任务，完成后报告。

**开始时宣告：** "我正在使用 executing-plans 技能来实施这个计划。"

**注意：** 告诉你的人类搭档，Superpowers 在有 subagent（子代理）访问权限时效果会好得多。如果在支持 subagent 的平台上运行（如 Claude Code 或 Codex），工作质量会显著提高。如果 subagent 可用，请使用 superpowers:subagent-driven-development 代替本技能。

## 流程

### 第 1 步：加载并审查计划
1. 读取计划文件
2. 批判性审查——识别对计划的任何疑问或顾虑
3. 如有顾虑：在开始之前向你的人类搭档提出
4. 如无顾虑：创建 TodoWrite 并继续

### 第 2 步：执行任务

对每个任务：
1. 标记为 in_progress
2. 严格按照每个步骤执行（计划已拆分为小步骤）
3. 按规定运行验证
4. 标记为 completed

### 第 3 步：完成开发

所有任务完成并验证后：
- 宣告："我正在使用 finishing-a-development-branch 技能来完成这项工作。"
- **必需子技能：** 使用 superpowers:finishing-a-development-branch
- 按照该技能的流程验证测试、展示选项、执行选择

## 何时停下来寻求帮助

**在以下情况立即停止执行：**
- 遇到阻塞（缺少依赖、测试失败、指令不明确）
- 计划有严重缺口导致无法开始
- 你不理解某个指令
- 验证反复失败

**宁可请求澄清，也不要猜测。**

## 何时回到前面的步骤

**在以下情况回到审查（第 1 步）：**
- 搭档根据你的反馈更新了计划
- 基本方法需要重新考虑

**不要硬闯阻塞** ——停下来并询问。

## 要点
- 先批判性审查计划
- 严格按照计划步骤执行
- 不要跳过验证
- 当计划提到技能时引用相应技能
- 遇到阻塞时停下来，不要猜测
- 未经用户明确同意，绝不在 main/master 分支上开始实施

## 集成

**必需的工作流技能：**
- **superpowers:using-git-worktrees** —— 必需：开始前设置隔离的工作区
- **superpowers:writing-plans** —— 创建本技能所执行的计划
- **superpowers:finishing-a-development-branch** —— 所有任务完成后收尾开发

## gem-orchestrator 委托

如果安装了 `gem-orchestrator`，对于包含多轮 wave 执行的复杂计划，可以考虑将执行委托给它：
- gem-orchestrator 擅长管理 wave 循环、自动分派子代理、集成检查
- 使用 `/sp-plan-exec` 时可以选择：自行按步骤执行（本技能）或委托给 gem-orchestrator
- gem-orchestrator 会自动调用 gem-reviewer 做每轮 wave 后的集成检查
