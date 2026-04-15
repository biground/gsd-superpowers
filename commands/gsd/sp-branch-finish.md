---
name: gsd:sp-branch-finish
description: "完成开发分支——清理、squash、合并准备，确保分支干净交付。"
argument-hint: "[branch name]"
allowed-tools:
  - Read
  - Bash
  - Grep
---

<objective>
完成开发分支的收尾工作。验证测试通过后，提供结构化的合并/推送/清理选项，确保分支干净交付。

Arguments:
- branch name (optional) — 要完成的分支名称。省略时使用当前分支。

流程概要：验证测试 → 呈现 4 个选项（本地合并 / 推送 PR / 保持现状 / 丢弃） → 执行选择 → 清理 worktree。
</objective>

<execution_context>
@~/.claude/get-shit-done/skills/finishing-a-development-branch/SKILL.md
</execution_context>

<context>
Branch: $ARGUMENTS (optional branch name; defaults to current branch if empty)

If $ARGUMENTS is non-empty:
```bash
git checkout "$ARGUMENTS"
```

Current state:
```bash
git branch --show-current
git status --short
git log --oneline -5
```
</context>

<process>
严格遵循 execution_context 中技能文件定义的完整流程：

## 1. 验证测试
运行项目测试套件，确认全部通过。测试失败则停止，不进入后续步骤。

## 2. 确定基础分支
检测当前分支的 merge-base（main 或 master），确认目标分支。

## 3. 呈现 4 个选项
准确呈现以下选项，不添加额外说明：
1. 本地合并到 <base-branch>
2. 推送并创建 Pull Request
3. 保持分支现状（稍后处理）
4. 丢弃此工作

等待用户选择。

## 4. 执行用户选择
按技能文件中定义的每个选项的具体步骤执行。

## 5. 清理 Worktree
选项 1、4 时清理 worktree；选项 2、3 时保留。
</process>
---
name: gsd:sp-branch-finish
description: 完成开发分支——验证测试、呈现合并选项、清理 worktree，确保分支干净交付。
argument-hint: "[branch name]"
allowed-tools:
  - Read
  - Bash
  - Grep
---

<objective>
完成开发分支的收尾工作。验证测试通过后，提供结构化的合并/推送/清理选项。

参数：
- 分支名（可选）——要完成的分支。省略时使用当前分支。

流程：验证测试 → 确定基础分支 → 呈现 4 个选项（本地合并 / 推送 PR / 保持现状 / 丢弃） → 执行选择 → 清理 worktree。
</objective>

<execution_context>
@~/.claude/get-shit-done/skills/finishing-a-development-branch/SKILL.md
</execution_context>

<context>
Branch: $ARGUMENTS

如果 $ARGUMENTS 为空，使用当前分支：
```bash
BRANCH=$(git branch --show-current)
```

如果指定了分支名，切换到该分支：
```bash
git checkout "$ARGUMENTS"
BRANCH="$ARGUMENTS"
```
</context>

<process>
严格遵循 execution_context 中技能文件定义的完整工作流。以下为关键步骤摘要：

## 1. 验证测试
运行项目测试套件，确认全部通过。如果测试失败，停止——不继续到下一步。

## 2. 确定基础分支
```bash
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

## 3. 呈现选项
准确呈现 4 个结构化选项：
1. 本地合并回基础分支
2. 推送并创建 Pull Request
3. 保持分支现状
4. 丢弃此工作

## 4. 执行选择
根据用户选择执行对应操作。选项 4（丢弃）需要用户输入 "discard" 确认。

## 5. 清理 Worktree
选项 1、4 清理 worktree 和分支。选项 2 保留 worktree。选项 3 不做任何清理。

**红线：**
- 测试失败时绝不继续
- 合并后必须再次验证测试
- 未经确认绝不删除工作成果
- 未经明确请求绝不 force-push
</process>
