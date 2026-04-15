---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---

# 请求代码审查

分派 superpowers:code-reviewer 子代理（subagent）在问题蔓延之前捕获它们。审查者获得的是精心构建的评估上下文——而非你的会话历史。这使审查者专注于工作成果而非你的思考过程，同时保留你自己的上下文以便继续工作。

**核心原则：** 尽早审查，频繁审查。

## 何时请求审查

**必须：**
- 子代理驱动开发（subagent-driven development）中每个任务完成后
- 完成主要功能后
- 合并到 main 之前

**可选但有价值：**
- 卡住时（获取新视角）
- 重构之前（基线检查）
- 修复复杂 bug 之后

## 如何请求

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 分派 code-reviewer 子代理：**

使用 Task 工具，类型为 superpowers:code-reviewer，填充 `code-reviewer.md` 中的模板

**占位符：**
- `{WHAT_WAS_IMPLEMENTED}` - 你刚构建的内容
- `{PLAN_OR_REQUIREMENTS}` - 它应该做什么
- `{BASE_SHA}` - 起始提交
- `{HEAD_SHA}` - 结束提交
- `{DESCRIPTION}` - 简要描述

**3. 根据反馈采取行动：**
- 立即修复 Critical（严重）问题
- 在继续之前修复 Important（重要）问题
- 记录 Minor（次要）问题，稍后处理
- 如果审查者有误，进行反驳（附带理由）

## 示例

```
[刚完成任务 2：添加验证函数]

你：让我在继续之前请求代码审查。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[分派 superpowers:code-reviewer 子代理]
  WHAT_WAS_IMPLEMENTED: 会话索引的验证和修复函数
  PLAN_OR_REQUIREMENTS: docs/superpowers/plans/deployment-plan.md 中的任务 2
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: 添加了 verifyIndex() 和 repairIndex()，支持 4 种问题类型

[子代理返回]:
  优点：架构清晰，测试真实
  问题：
    重要：缺少进度指示器
    次要：魔法数字 (100) 用于报告间隔
  评估：可以继续

你：[修复进度指示器]
[继续任务 3]
```

## 与工作流的集成

**子代理驱动开发（Subagent-Driven Development）：**
- 每个任务完成后进行审查
- 在问题叠加之前捕获它们
- 修复后再进入下一个任务

**执行计划（Executing Plans）：**
- 每批（3 个任务）之后进行审查
- 获取反馈，应用修复，继续

**临时开发（Ad-Hoc Development）：**
- 合并前进行审查
- 卡住时进行审查

## 危险信号

**绝不要：**
- 因为"很简单"就跳过审查
- 忽略 Critical（严重）问题
- 在 Important（重要）问题未修复时继续
- 与合理的技术反馈争辩

**如果审查者有误：**
- 用技术理由反驳
- 展示证明其有效的代码/测试
- 请求澄清

参见模板：requesting-code-review/code-reviewer.md

## Gem Agent 审查替代

如果安装了 `gem-reviewer`，可以作为自动基线审查的替代路径：
- **gem-reviewer**：自动运行，每个 wave 后触发。覆盖安全扫描、lint、build、PRD 合规。只报告 Critical/Important。
- **code-reviewer**（原有）：深度审查，关注计划对齐、架构设计、代码模式。适合重大步骤完成后。

推荐组合：gem-reviewer 做每次 wave 的自动基线 → code-reviewer 做关键里程碑的深度审查。
