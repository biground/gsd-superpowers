# Superpowers Integration

## 概述
本文档说明 GSD 与 Superpowers 的整合点，覆盖新增 Agent、增强能力、技能库与命令层面的对齐关系，便于开发者在执行与扩展工作流时快速定位能力入口。

## 新增 Agents

| Agent | 用途 |
|---|---|
| `gem-orchestrator` | 多 agent 编排核心：阶段检测、路由分发、结果综合与流程推进。 |
| `gem-narrative-writer` | Obsidian 叙事风格技术笔记写作，将技术经验沉淀为可复用知识。 |
| `gsd-critic` | 广范围批判性审查：覆盖 plan/code/architecture/design，强调目标反向验证与假设挑战。 |

## 增强的 GSD Agents

| Agent | 新增能力 |
|---|---|
| `gsd-planner` | 增加 DAG 调度（按 wave 分组并标注依赖）与 high/medium 任务 pre-mortem 风险分析。 |
| `gsd-executor` | 强化 TDD 纪律：当 `type="tdd"` 时强制 Red-Green-Refactor，并校验 RED/GREEN gate commit。 |
| `gsd-code-reviewer` | 引入双层审查：Baseline（quick+standard）与 Deep（跨文件 + 架构/计划对齐）。 |
| `gsd-debugger` | 增加轻量修复模式：根因明确且小范围变更可在同一上下文直接修复，降低切换成本。 |
| `gsd-doc-writer` | 增加 `narrative` 文档模式，采用 Scene→Conflict→Exploration→Discovery→Summary 叙事弧。 |
| `gsd-security-auditor` | 增加主动安全扫描模式，按 OWASP Top 10 执行补充检查并输出修复建议。 |
| `gsd-ui-researcher` | 增加组件框架检测建议，覆盖 shadcn/ui、Tailwind CSS、Headless UI。 |
| `gsd-ui-checker` | 增加 Tailwind 实践检查：内联 style、断点一致性、暗色模式覆盖、配置映射。 |
| `gsd-ui-auditor` | 增加组件框架一致性审计：CSS 方案混用、API 约定一致性、Tailwind 类名规范。 |

## Skills 技能库

| Skill | 用途 |
|---|---|
| `brainstorming` | 编码前的结构化头脑风暴与设计收敛。 |
| `dispatching-parallel-agents` | 对独立任务进行并行代理分派，提高吞吐。 |
| `executing-plans` | 在独立执行会话中按计划落地并带审查检查点。 |
| `finishing-a-development-branch` | 开发分支收尾：测试验证、合并/PR/清理决策。 |
| `narrative-writing` | 生成可分享、可长期复用的技术叙事笔记。 |
| `receiving-code-review` | 系统化处理人工审查反馈，强调先验证后实施。 |
| `requesting-code-review` | 在任务完成后请求结构化代码审查。 |
| `subagent-driven-development` | 复杂任务按子代理拆解执行，并进行两阶段审查。 |
| `systematic-debugging` | 以科学方法做根因定位，避免症状级修复。 |
| `test-driven-development` | 严格 Red-Green-Refactor 的 TDD 执行纪律。 |
| `using-git-worktrees` | 使用 Git Worktree 建立隔离工作区并安全验证。 |
| `using-superpowers` | 会话起始的技能发现与调用总入口。 |
| `verification-before-completion` | 声称完成前必须提供新鲜验证证据。 |
| `writing-plans` | 根据 spec/requirements 生成可执行多步骤计划。 |
| `writing-skills` | 以 TDD 思路编写、验证和迭代技能文档。 |

## 新增命令 (sp-*)

| 命令 | 用途 |
|---|---|
| `sp-brainstorm` | 协作式头脑风暴：在编码前将想法收敛为设计方案。 |
| `sp-branch-finish` | 分支收尾流程：验证测试并提供合并/推送/清理选项。 |
| `sp-code-review-recv` | 处理审查反馈：逐条验证、实施或有据反驳。 |
| `sp-parallel` | 并行代理调度：适用于 3+ 独立任务并发推进。 |
| `sp-subagent` | 子代理驱动开发：逐步分派并进行质量把关。 |
| `sp-tdd` | 测试驱动开发：强制 Red-Green-Refactor 纪律。 |
| `sp-worktree` | Git Worktree 管理：在隔离环境并行开发分支。 |

## 增强的命令

| 命令 | 新增引用 |
|---|---|
| `plan-phase` | 引入 `skills/writing-plans/SKILL.md` 与 plan-phase workflow；补充 Copilot 的 `vscode_askquestions` 运行时映射说明。 |
| `execute-phase` | 引入 `skills/executing-plans/SKILL.md` 与 execute-phase workflow；明确按参数激活 flag 的执行规则。 |
| `debug` | 引入 `skills/systematic-debugging/SKILL.md`；增加 `--diagnose`、session manager 续跑与 TDD mode 读取流程。 |
| `code-review` | 引入 `skills/requesting-code-review/SKILL.md` 与 code-review workflow；支持 `--depth`/`--files` 分级与精确 scope。 |

## 使用方式

### Claude Code

1. 通过 `/gsd:sp-*` 系列命令直接触发对应 Superpowers 能力。
2. 由 orchestrator 统一完成阶段判断与 agent 路由。
3. 在执行、调试、审查场景下，优先走 skill 驱动流程（TDD、debugging、review）。

### VS Code Copilot

1. 使用同名 `sp-*` 命令进入对应流程。
2. 工作流中出现 `AskUserQuestion` 时，按命令文档中的 runtime note 映射到 `vscode_askquestions`。
3. 命令层负责调度，具体实现与检查由对应 agent + skill 组合完成。
