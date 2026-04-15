# GSD install.js --copilot 模式完整分析

> 基于 `bin/install.js` 源码的逐行分析，非推测。

---

## 1. --copilot 模式下的安装目标目录

| 安装模式 | 目标目录 | 本地目录名 |
|---------|---------|-----------|
| **全局** (`--global`) | `~/.copilot`（可通过 `COPILOT_CONFIG_DIR` 环境变量或 `--config-dir` 覆盖） | N/A |
| **本地** (`--local`) | `<项目根>/.github` | `.github` |

关键代码路径：
- `getDirName('copilot')` → `'.github'`（本地安装目录名）
- `getConfigDirFromHome('copilot', isGlobal)` → `"'.copilot'"`（全局 homedir 下的配置目录）
- `getGlobalDir('copilot', ...)` → 优先级：`--config-dir` > `COPILOT_CONFIG_DIR` > `~/.copilot`

---

## 2. claudeToCopilotTools 映射表完整内容

```javascript
const claudeToCopilotTools = {
  Read: 'read',
  Write: 'edit',
  Edit: 'edit',
  Bash: 'execute',
  Grep: 'search',
  Glob: 'search',
  Task: 'agent',
  WebSearch: 'web',
  WebFetch: 'web',
  TodoWrite: 'todo',
  AskUserQuestion: 'ask_user',
  SlashCommand: 'skill',
};
```

**映射特点**：
- `Write` 和 `Edit` 都映射到 `'edit'`
- `Grep` 和 `Glob` 都映射到 `'search'`
- `WebSearch` 和 `WebFetch` 都映射到 `'web'`
- 存在多对一映射，安装时会去重（`[...new Set(mappedTools)]`）

**特殊映射逻辑**（`convertCopilotToolName()` 函数）：
- `mcp__context7__*` 前缀 → `io.github.upstash/context7/*`（MCP Context7 专用映射）
- 其他未知工具 → 转为小写（`claudeTool.toLowerCase()`）

**重要注释**：代码第 39 行明确标注：
> "Tool mapping applies ONLY to agents, NOT to skills (per CONTEXT.md decision)"

---

## 3. convertClaudeToCopilotContent() 函数的转换规则

该函数应用两类转换：**CONV-06（路径替换）** 和 **CONV-07（命令名转换）**。

### 3.1 CONV-06：路径替换

**全局安装（`isGlobal = true`）**：
| 原始路径 | 替换为 |
|---------|--------|
| `$HOME/.claude/` | `$HOME/.copilot/` |
| `~/.claude/` | `~/.copilot/` |

**本地安装（`isGlobal = false`）**：
| 原始路径 | 替换为 |
|---------|--------|
| `$HOME/.claude/` | `.github/` |
| `~/.claude/` | `.github/` |
| `~/.claude\n` | `.github/`（注意：处理了行尾的情况） |

**通用替换**（不区分全局/本地）：
| 原始路径 | 替换为 |
|---------|--------|
| `./.claude/` | `./.github/` |
| `.claude/` | `.github/` |

### 3.2 CONV-07：命令名转换

```javascript
c = c.replace(/gsd:/g, 'gsd-');
```

所有 `gsd:xxx` 格式的命令引用被转换为 `gsd-xxx` 格式。

### 3.3 Agent 引用中性化

```javascript
c = neutralizeAgentReferences(c, 'copilot-instructions.md');
```

`neutralizeAgentReferences()` 函数执行以下替换：
1. 将独立的 `Claude`（指代 agent 角色）替换为 `the agent`
   - 排除：`Claude Code`, `Claude Opus`, `Claude Sonnet`, `Claude Haiku`, `Claude native`, `Claude based`, `Claude-`
2. 将 `CLAUDE.md` 替换为 `copilot-instructions.md`
3. 移除与 `AGENTS.md` 冲突的指令

---

## 4. 文件复制逻辑

### 4.1 安装的目录/文件一览

| 安装内容 | 源路径 | 目标路径 | 说明 |
|---------|--------|---------|------|
| **GSD Skills** | `commands/gsd/*.md` | `<target>/skills/gsd-*/SKILL.md` | 每个命令变成一个 skill 目录 |
| **GSD 引擎** | `get-shit-done/` | `<target>/get-shit-done/` | 包含 bin, contexts, references, templates, workflows |
| **Agent 文件** | `agents/gsd-*.md` | `<target>/agents/gsd-*.agent.md` | **注意：文件名从 `.md` 改为 `.agent.md`** |
| **CHANGELOG** | `CHANGELOG.md` | `<target>/get-shit-done/CHANGELOG.md` | |
| **VERSION** | 动态生成 | `<target>/get-shit-done/VERSION` | 写入 package.json 中的版本号 |
| **copilot-instructions.md** | `get-shit-done/templates/copilot-instructions.md` | `<target>/copilot-instructions.md` | 通过 marker 机制合并插入 |
| **文件清单** | 动态生成 | `<target>/gsd-file-manifest.json` | 用于检测用户修改 |

### 4.2 **不安装的内容**（与 Claude 等 runtime 不同）

- ❌ 不安装 `hooks/`（不像 Claude/OpenCode 有 `settings.json` hooks）
- ❌ 不安装 `package.json`（CommonJS 模式标记）
- ❌ 不配置 statusline
- ❌ 不写入 `settings.json`

### 4.3 Skills 安装细节（`copyCommandsAsCopilotSkills()`）

1. **清理旧技能**：删除 `skills/` 下所有 `gsd-` 开头的目录
2. **递归处理** `commands/gsd/` 源目录
3. 对每个 `.md` 文件：
   - 生成 skill 名称：`gsd-<basename>`（子目录递归会连接：`gsd-sub-name`）
   - 创建目录 `skills/gsd-xxx/`
   - 通过 `convertClaudeCommandToCopilotSkill()` 转换内容
   - 通过 `processAttribution()` 处理 Co-Authored-By
   - 写入 `skills/gsd-xxx/SKILL.md`

### 4.4 Skill 前置数据转换（`convertClaudeCommandToCopilotSkill()`）

转换流程：
1. 先应用 `convertClaudeToCopilotContent()`（路径+命令名转换）
2. 提取 frontmatter 字段：`description`, `argument-hint`, `agent`
3. **CONV-02**：将 `allowed-tools` YAML 多行列表转为逗号分隔的单行字符串
4. 重构 frontmatter：
   ```yaml
   ---
   name: gsd-xxx
   description: ...
   argument-hint: "..."  # 如果存在
   agent: ...            # 如果存在
   allowed-tools: tool1, tool2, tool3  # 单行逗号分隔
   ---
   ```
5. **重要**：Skills 的工具名保持原始 Claude 格式，**不映射**到 Copilot 工具名

### 4.5 `copyWithPathReplacement()` 对 Copilot 的特殊处理

除了 `.md` 文件外，还会转换 `.cjs` 和 `.js` 文件中的路径和命令名，同样使用 `convertClaudeToCopilotContent()`。

---

## 5. skills/ 目录是否已被处理

### 结论：**GSD 的 `commands/` 被转换为 Copilot skills，但 GSD 项目本身没有独立的 `skills/` 源目录。**

详细说明：

- GSD 源码中的 `commands/gsd/*.md` 是命令文件，安装时被**转换**为 Copilot 的 `skills/gsd-*/SKILL.md` 格式
- GSD 源码的 `get-shit-done/` 引擎目录被整体复制到 `<target>/get-shit-done/`，其中包含 workflows、templates 等
- 如果用户有**独立于 GSD 的自定义 skills**（如 Superpowers 的 skills/），当前安装脚本**不会处理它们** — 它只处理 GSD 自带的 `commands/gsd/` 和 `agents/` 和 `get-shit-done/` 引擎

### 与 Superpowers 集成的影响

Superpowers 有自己的 `skills/` 目录结构（如 `skills/brainstorming/SKILL.md`）。GSD 的安装脚本**不涉及**这些文件 — 它只操作 `skills/gsd-*` 前缀的目录。这意味着两者可以在同一个 `skills/` 目录下共存，互不干扰。

---

## 6. Copilot Instructions Marker 机制

### 常量定义

```javascript
const GSD_COPILOT_INSTRUCTIONS_MARKER = '<!-- GSD Configuration — managed by get-shit-done installer -->';
const GSD_COPILOT_INSTRUCTIONS_CLOSE_MARKER = '<!-- /GSD Configuration -->';
```

### `mergeCopilotInstructions()` 的三种模式

| 场景 | 条件 | 行为 |
|------|------|------|
| **新建** | `copilot-instructions.md` 不存在 | 创建文件，内容 = `MARKER + gsdContent + CLOSE_MARKER` |
| **更新** | 文件存在，包含两个 marker | 替换 marker 之间的内容（保留 marker 前后的用户内容） |
| **追加** | 文件存在，但无 marker | 在文件末尾追加 marker 包裹的 GSD 内容 |

### 模板内容来源

模板文件路径：`get-shit-done/templates/copilot-instructions.md`

内容：
```markdown
# Instructions for GSD

- Use the get-shit-done skill when the user asks for GSD or uses a `gsd-*` command.
- Treat `/gsd-...` or `gsd-...` as command invocations and load the matching file from `.github/skills/gsd-*`.
- When a command says to spawn a subagent, prefer a matching custom agent from `.github/agents`.
- Do not apply GSD workflows unless the user explicitly asks for them.
- After completing any `gsd-*` command (or any deliverable it triggers), ALWAYS offer the user the next step.
```

### `stripGsdFromCopilotInstructions()`（卸载时使用）

- 如果找到 marker 对：删除 marker 之间的内容
- 如果删除后文件为空：返回 `null`（调用方会删除文件）
- 如果没有 marker：不做任何修改

---

## 7. Agent 文件的 Frontmatter 字段转换逻辑

### `convertClaudeAgentToCopilotAgent()` 函数

1. 先对 body 应用 `convertClaudeToCopilotContent()`（路径+命令名+agent 引用中性化）
2. 提取 frontmatter 字段：`name`, `description`, `color`, `tools`
3. **CONV-04 + CONV-05**：工具名映射 + 去重 + JSON 数组格式
   - `tools` 字段从逗号分隔字符串 → 映射 → 去重 → `['tool1', 'tool2']` JSON 数组格式
4. 重构 frontmatter：
   ```yaml
   ---
   name: gsd-xxx
   description: ...
   tools: ['read', 'edit', 'execute', 'search', 'agent']
   color: cyan  # 如果原始有 color 字段则保留
   ---
   ```

### 文件重命名

```javascript
const destName = isCopilot ? entry.name.replace('.md', '.agent.md') : entry.name;
```

**所有 agent 文件在 Copilot 模式下从 `gsd-xxx.md` 重命名为 `gsd-xxx.agent.md`**。

### tools 映射示例

假设原始 Claude agent 有 `tools: Read, Write, Edit, Bash, Grep, Glob, Task, WebSearch`:
1. 映射：`['read', 'edit', 'edit', 'execute', 'search', 'search', 'agent', 'web']`
2. 去重：`['read', 'edit', 'execute', 'search', 'agent', 'web']`
3. 输出：`tools: ['read', 'edit', 'execute', 'search', 'agent', 'web']`

---

## 8. --insiders 标志

### **不存在 `--insiders` 标志**

搜索整个 `install.js`（6676 行），**没有**找到任何 `insiders` 相关的代码、参数或逻辑。

GSD 安装脚本不区分 VS Code 稳定版和 Insiders 版。安装目标完全由 `--config-dir` 或环境变量 `COPILOT_CONFIG_DIR` 决定。如果用户需要安装到 Insiders 特定的配置目录，需要手动使用 `--config-dir` 指定路径。

---

## 9. 卸载逻辑（--uninstall）

### Copilot 模式下的卸载步骤

运行 `npx get-shit-done-cc --copilot [--global|--local] --uninstall`

**按顺序执行**：

1. **删除 GSD skills**：遍历 `skills/` 目录，删除所有 `gsd-` 开头的子目录
2. **清理 copilot-instructions.md**：
   - 如果文件存在且包含 GSD marker → 删除 marker 之间的内容
   - 如果删除后文件为空 → 删除整个文件
   - 如果无 marker → 不修改
3. **删除 get-shit-done/ 目录**：保留 `USER-PROFILE.md`（如果存在）
4. **删除 GSD agents**：删除 `agents/` 下所有 `gsd-*.md` 文件
5. **删除 hooks**：尝试删除 GSD 特定的 hook 文件（虽然 Copilot 模式通常不安装 hooks）
6. **删除 package.json**：仅当内容是 `{"type":"commonjs"}` 时删除（Copilot 模式通常不创建）
7. **settings.json 清理**：如果存在，移除 GSD 相关的 statusline 和 hooks 配置
8. **删除 gsd-file-manifest.json**

### 不被删除的内容

- 用户自定义的 skills（非 `gsd-` 开头的目录）
- 用户自定义的 agents（非 `gsd-` 开头的文件）
- `copilot-instructions.md` 中 marker 之外的用户内容
- `get-shit-done/USER-PROFILE.md`（会被保留并恢复）

---

## 10. 安装流程总结（`install('copilot')` 执行顺序）

```
1. 保存用户修改的本地补丁 (saveLocalPatches)
2. 清理旧版遗留文件 (cleanupOrphanedFiles)
3. 安装 skills：commands/gsd/*.md → skills/gsd-*/SKILL.md
4. 安装引擎：get-shit-done/ → get-shit-done/（含 bin, contexts, references, templates, workflows）
   - 保留 USER-PROFILE.md
5. 安装 agents：agents/gsd-*.md → agents/gsd-*.agent.md
6. 安装 CHANGELOG.md → get-shit-done/CHANGELOG.md
7. 写入 VERSION → get-shit-done/VERSION
8. (跳过 hooks、settings.json、statusline — Copilot 不需要)
9. 写入文件清单 gsd-file-manifest.json
10. 报告本地补丁（如果有修改过的文件）
11. 扫描泄漏的 .claude 路径引用
12. 合并 copilot-instructions.md（通过 marker 机制）
13. 返回（无 settingsPath, 无 settings, 无 statuslineCommand）
```

### 返回值

```javascript
return { settingsPath: null, settings: null, statuslineCommand: null, runtime, configDir: targetDir };
```

Copilot 模式返回的 settingsPath 和 settings 均为 `null`，表明不涉及任何 settings.json 配置。
