---
name: gsd:sp-code-review-recv
description: "接收代码审查反馈——系统性处理审查意见并修复。与 code-review-fix（自动修复 REVIEW.md 中的问题）不同，此命令面向人工审查反馈，强调技术验证、逐条评估和有理有据的回应。与 code-review（请求审查）互补。"
argument-hint: "[PR number or review comments]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
---

<objective>
系统性处理代码审查反馈。逐条验证反馈的技术正确性，实施合理建议，有理有据地反驳不合理建议。

Arguments:
- PR number or review comments (optional) — PR 编号、审查链接或直接粘贴的审查意见。

核心原则：实施前先验证。假设前先提问。技术正确性优先于社交舒适度。

**与 gsd:code-review-fix 的区别：**
- `code-review-fix`：自动修复由 code-review 命令生成的 REVIEW.md 中的问题，面向机器审查结果，批量自动修复。
- `sp-code-review-recv`：系统性处理人工审查反馈（PR 评论、外部审查者意见），逐条技术验证，支持接受/反驳/讨论，强调审慎评估而非盲目实施。
</objective>

<execution_context>
@~/.claude/get-shit-done/skills/receiving-code-review/SKILL.md
</execution_context>

<context>
Review input: $ARGUMENTS

If $ARGUMENTS contains a PR number:
```bash
gh pr view "$ARGUMENTS" --comments
gh pr diff "$ARGUMENTS"
```

If $ARGUMENTS is empty, ask user to provide review feedback source.
</context>

<process>
严格遵循 execution_context 中技能文件定义的完整流程：

## 1. 获取审查反馈
从 PR 评论、粘贴的文本或其他来源收集完整的审查反馈。

## 2. 阅读与理解
完整阅读反馈，用自己的话复述需求。如有不清晰的条目，立即请求澄清——不要部分实施。

## 3. 验证与评估
对照代码库实际情况逐条检查：
- 对当前代码库在技术上是否正确？
- 是否会破坏现有功能？
- 当前实现的原因是什么？
- 审查者是否了解完整上下文？

## 4. 分类与排序
按优先级排序：阻塞性问题（崩溃、安全） → 简单修复（拼写、导入） → 复杂修复（重构、逻辑）。

## 5. 逐条处理
- 合理建议：实施修改，逐个测试，验证无回归
- 不合理建议：用技术理由反驳，引用代码和测试
- 不确定的：如实说明限制，请求指导

## 6. 验证结果
确认所有已接受的修改通过测试，提交修改摘要。
</process>
