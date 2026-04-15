---
name: gsd:sp-tdd
description: "测试驱动开发——严格的 Red-Green-Refactor 纪律。"
argument-hint: "[feature or file]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
---

<objective>
以严格的红-绿-重构循环驱动开发。先写失败的测试，再写最少的代码让它通过，最后重构。

**铁律：没有失败的测试，就不能写生产代码。**

如果你没有看到测试失败，你就不知道它是否测试了正确的东西。
</objective>

<execution_context>
@~/.claude/get-shit-done/skills/test-driven-development/SKILL.md
</execution_context>

<process>
读取并遵循 test-driven-development SKILL.md 中的完整工作流。

### 红-绿-重构循环

**红（Red）— 写失败的测试：**
1. 写一个最小的测试，展示预期行为
2. 运行测试——**必须看到测试失败**
3. 确认失败原因正确（如"函数未定义"，而非"语法错误"）
4. 如果测试通过：修改测试或检查已有实现

**绿（Green）— 最少的代码：**
1. 写刚好足够让测试通过的代码——不多不少
2. 运行测试——必须通过
3. 如果新增了超出测试要求的代码：删除（YAGNI）

**重构（Refactor）— 清理代码：**
1. 改善代码结构、消除重复
2. 确保所有测试仍然通过
3. 不改变外部行为

### 纪律

- 在测试之前写了代码？**删掉，重新开始。**
- 不要把它留作"参考"，不要在写测试时"改编"它
- 从测试出发，全新实现
- 在想"就这一次跳过 TDD"？停下来——那是合理化借口
</process>
