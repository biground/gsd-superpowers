---
name: gsd:sp-worktree
description: "Git Worktree 管理——在隔离环境中并行开发多个分支。"
argument-hint: "[branch name or feature]"
allowed-tools:
  - Read
  - Bash
  - Grep
---

<objective>
使用 Git Worktree 创建共享同一仓库的隔离工作区，允许同时在多个分支上工作而无需频繁切换。

系统化的目录选择 + 安全验证 = 可靠的隔离环境。创建 worktree 后必须验证项目能正常构建和测试。
</objective>

<execution_context>
@~/.claude/get-shit-done/skills/using-git-worktrees/SKILL.md
</execution_context>

<context>
Branch name or feature: $ARGUMENTS
</context>

<process>

## 1. 确认需求

解析 $ARGUMENTS 获取目标分支名称或功能描述。如果只提供了功能描述，根据项目命名约定生成合适的分支名。

## 2. 创建 Worktree

使用 `git worktree add` 在合适的目录位置创建新的 worktree：
- 选择系统化的目录路径（如 `../<repo>-worktrees/<branch>`）
- 如果分支不存在，使用 `-b` 参数创建新分支

## 3. 安全验证

验证新 worktree 目录已被 `.gitignore` 正确忽略（`git check-ignore`），防止 worktree 目录被误提交。

## 4. 项目环境设置

在新 worktree 中运行项目安装和构建命令，确保开发环境就绪：
- 安装依赖（如 `npm install`、`pip install` 等）
- 运行构建验证
- 运行测试确认环境正常

## 5. 报告

输出 worktree 创建结果，包括：
- worktree 路径
- 分支名称
- 环境验证状态
- 后续使用提示

</process>
