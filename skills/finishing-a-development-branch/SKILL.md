---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
---

# 完成开发分支（Finishing a Development Branch）

## 概述

通过提供清晰的选项并处理所选工作流，引导开发工作的收尾。

**核心原则：** 验证测试 → 呈现选项 → 执行选择 → 清理。

**开始时宣布：** "我正在使用 finishing-a-development-branch 技能来完成这项工作。"

## 流程

### 步骤 1：验证测试

**在呈现选项之前，验证测试通过：**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with merge/PR until tests pass.
```

停止。不要继续到步骤 2。

**如果测试通过：** 继续步骤 2。

### 步骤 2：确定基础分支（Base Branch）

```bash
# Try common base branches
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或者询问："这个分支是从 main 分出来的——对吗？"

### 步骤 3：呈现选项

准确呈现以下 4 个选项：

```
Implementation complete. What would you like to do?

1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)
4. Discard this work

Which option?
```

**不要添加额外说明** ——保持选项简洁。

### 步骤 4：执行选择

#### 选项 1：本地合并（Merge Locally）

```bash
# Switch to base branch
git checkout <base-branch>

# Pull latest
git pull

# Merge feature branch
git merge <feature-branch>

# Verify tests on merged result
<test command>

# If tests pass
git branch -d <feature-branch>
```

然后：清理 worktree（步骤 5）

#### 选项 2：推送并创建 Pull Request

```bash
# Push branch
git push -u origin <feature-branch>

# Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

然后：清理 worktree（步骤 5）

#### 选项 3：保持现状

报告："保留分支 <name>。Worktree 保存在 <path>。"

**不要清理 worktree。**

#### 选项 4：丢弃

**先确认：**
```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>
- Worktree at <path>

Type 'discard' to confirm.
```

等待精确确认。

如果确认：
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

然后：清理 worktree（步骤 5）

### 步骤 5：清理 Worktree

**对于选项 1、2、4：**

检查是否在 worktree 中：
```bash
git worktree list | grep $(git branch --show-current)
```

如果是：
```bash
git worktree remove <worktree-path>
```

**对于选项 3：** 保留 worktree。

## 快速参考

| 选项 | 合并 | 推送 | 保留 Worktree | 清理分支 |
|------|------|------|---------------|----------|
| 1. 本地合并 | ✓ | - | - | ✓ |
| 2. 创建 PR | - | ✓ | ✓ | - |
| 3. 保持现状 | - | - | ✓ | - |
| 4. 丢弃 | - | - | - | ✓（强制） |

## 常见错误

**跳过测试验证**
- **问题：** 合并损坏的代码，创建失败的 PR
- **修复：** 在提供选项之前始终验证测试

**开放式问题**
- **问题：** "接下来该怎么做？" → 模糊不清
- **修复：** 准确呈现 4 个结构化选项

**自动清理 worktree**
- **问题：** 在可能需要时移除 worktree（选项 2、3）
- **修复：** 仅在选项 1 和 4 时清理

**丢弃时不确认**
- **问题：** 意外删除工作成果
- **修复：** 要求输入 "discard" 确认

## 红旗（Red Flags）

**绝对不要：**
- 在测试失败时继续
- 合并时不对结果验证测试
- 未经确认就删除工作成果
- 未经明确请求就 force-push

**始终要：**
- 在提供选项之前验证测试
- 准确呈现 4 个选项
- 选项 4 需要输入确认
- 仅在选项 1 和 4 时清理 worktree

## 集成

**被以下技能调用：**
- **subagent-driven-development**（步骤 7）——所有任务完成后
- **executing-plans**（步骤 5）——所有批次完成后

**配合使用：**
- **using-git-worktrees** ——清理由该技能创建的 worktree
