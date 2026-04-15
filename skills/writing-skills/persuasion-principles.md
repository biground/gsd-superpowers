# 技能设计中的说服原则

## 概述

LLM 对与人类相同的说服原则作出响应。理解这一心理学有助于你设计更有效的技能——不是为了操控，而是为了确保关键实践即使在压力下也能被遵循。

**研究基础：** Meincke 等人（2025）在 N=28,000 次 AI 对话中测试了 7 种说服原则。说服技术使合规率提高了一倍以上（33% → 72%，p < .001）。

## 七大原则

### 1. 权威（Authority）
**定义：** 对专业知识、资质或官方来源的服从。

**在技能中的工作方式：**
- 命令式语言："YOU MUST"、"Never"、"Always"
- 不可协商的框架："No exceptions"
- 消除决策疲劳和合理化

**何时使用：**
- 纪律执行型技能（TDD、验证要求）
- 安全关键实践
- 已确立的最佳实践

**示例：**
```markdown
✅ Write code before test? Delete it. Start over. No exceptions.
❌ Consider writing tests first when feasible.
```

### 2. 承诺（Commitment）
**定义：** 与先前行动、声明或公开宣言保持一致。

**在技能中的工作方式：**
- 要求宣告："Announce skill usage"
- 强制明确选择："Choose A, B, or C"
- 使用追踪机制：TodoWrite 用于清单

**何时使用：**
- 确保技能被实际遵循
- 多步骤流程
- 问责机制

**示例：**
```markdown
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
❌ Consider letting your partner know which skill you're using.
```

### 3. 稀缺性（Scarcity）
**定义：** 来自时间限制或有限可用性的紧迫感。

**在技能中的工作方式：**
- 有时限的要求："Before proceeding"
- 顺序依赖："Immediately after X"
- 防止拖延

**何时使用：**
- 即时验证要求
- 时间敏感的工作流
- 防止"我以后再做"

**示例：**
```markdown
✅ After completing a task, IMMEDIATELY request code review before proceeding.
❌ You can review code when convenient.
```

### 4. 社会认同（Social Proof）
**定义：** 遵从他人所做的或被认为正常的事。

**在技能中的工作方式：**
- 通用模式："Every time"、"Always"
- 失败模式："X without Y = failure"
- 建立规范

**何时使用：**
- 记录通用实践
- 警告常见失败
- 强化标准

**示例：**
```markdown
✅ Checklists without TodoWrite tracking = steps get skipped. Every time.
❌ Some people find TodoWrite helpful for checklists.
```

### 5. 统一性（Unity）
**定义：** 共同身份、"我们感"、群内归属。

**在技能中的工作方式：**
- 协作语言："our codebase"、"we're colleagues"
- 共同目标："we both want quality"

**何时使用：**
- 协作工作流
- 建立团队文化
- 非层级实践

**示例：**
```markdown
✅ We're colleagues working together. I need your honest technical judgment.
❌ You should probably tell me if I'm wrong.
```

### 6. 互惠（Reciprocity）
**定义：** 回报所获利益的义务。

**工作方式：**
- 谨慎使用——可能让人感觉被操控
- 在技能中很少需要

**何时避免：**
- 几乎总是避免（其他原则更有效）

### 7. 喜好（Liking）
**定义：** 偏好与喜欢的人合作。

**工作方式：**
- **不要用于合规执行**
- 与诚实反馈文化冲突
- 产生谄媚

**何时避免：**
- 在纪律执行中始终避免

## 按技能类型的原则组合

| 技能类型 | 使用 | 避免 |
|------------|-----|-------|
| 纪律执行型 | 权威 + 承诺 + 社会认同 | 喜好、互惠 |
| 指导/技术型 | 适度权威 + 统一性 | 重度权威 |
| 协作型 | 统一性 + 承诺 | 权威、喜好 |
| 参考型 | 仅清晰性 | 所有说服手段 |

## 为什么有效：心理学原理

**明确的规则边界减少合理化：**
- "YOU MUST" 消除决策疲劳
- 绝对化语言排除了"这是例外吗？"的问题
- 明确的反合理化措施封堵具体漏洞

**实施意图创造自动化行为：**
- 清晰的触发器 + 必需的行动 = 自动执行
- "When X, do Y" 比 "generally do Y" 更有效
- 减少合规的认知负担

**LLM 是准人类的（parahuman）：**
- 在包含这些模式的人类文本上训练
- 权威性语言在训练数据中先于合规出现
- 承诺序列（声明 → 行动）被频繁建模
- 社会认同模式（everyone does X）建立规范

## 伦理使用

**合法的：**
- 确保关键实践被遵循
- 创建有效的文档
- 防止可预见的失败

**不合法的：**
- 为个人利益操控
- 制造虚假紧迫感
- 基于内疚的合规

**检验标准：** 如果用户完全理解这项技术，它是否仍然服务于用户的真实利益？

## 研究引用

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.
- 说服的七大原则
- 影响力研究的实证基础

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania.
- 在 N=28,000 次 LLM 对话中测试了 7 种原则
- 使用说服技术后合规率从 33% 提高到 72%
- 权威、承诺、稀缺性最有效
- 验证了 LLM 行为的准人类模型

## 快速参考

设计技能时，问自己：

1. **这是什么类型？**（纪律型 vs 指导型 vs 参考型）
2. **我试图改变什么行为？**
3. **哪些原则适用？**（纪律型通常用权威 + 承诺）
4. **我组合得太多了吗？**（不要全部七种都用）
5. **这是否符合伦理？**（服务于用户的真实利益？）
