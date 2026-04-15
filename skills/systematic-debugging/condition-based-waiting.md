# 基于条件的等待

## 概述

不稳定测试（flaky test）通常使用任意延迟来猜测时机。这会产生竞态条件（race condition），导致测试在快速机器上通过但在高负载或 CI 环境中失败。

**核心原则：** 等待你真正关心的条件，而不是猜测它需要多长时间。

## 何时使用

```dot
digraph when_to_use {
    "测试使用了 setTimeout/sleep?" [shape=diamond];
    "正在测试时序行为?" [shape=diamond];
    "记录为什么需要超时" [shape=box];
    "使用基于条件的等待" [shape=box];

    "测试使用了 setTimeout/sleep?" -> "正在测试时序行为?" [label="是"];
    "正在测试时序行为?" -> "记录为什么需要超时" [label="是"];
    "正在测试时序行为?" -> "使用基于条件的等待" [label="否"];
}
```

**使用场景：**
- 测试中有任意延迟（`setTimeout`、`sleep`、`time.sleep()`）
- 测试不稳定（有时通过，在高负载下失败）
- 测试在并行运行时超时
- 等待异步操作完成

**不要使用：**
- 正在测试真正的时序行为（防抖、节流间隔）
- 如果使用任意超时，始终记录原因

## 核心模式

```typescript
// ❌ 修改前：猜测时机
await new Promise(r => setTimeout(r, 50));
const result = getResult();
expect(result).toBeDefined();

// ✅ 修改后：等待条件满足
await waitFor(() => getResult() !== undefined);
const result = getResult();
expect(result).toBeDefined();
```

## 快速参考模式

| 场景 | 模式 |
|------|------|
| 等待事件 | `waitFor(() => events.find(e => e.type === 'DONE'))` |
| 等待状态 | `waitFor(() => machine.state === 'ready')` |
| 等待数量 | `waitFor(() => items.length >= 5)` |
| 等待文件 | `waitFor(() => fs.existsSync(path))` |
| 复合条件 | `waitFor(() => obj.ready && obj.value > 10)` |

## 实现

通用轮询函数：
```typescript
async function waitFor<T>(
  condition: () => T | undefined | null | false,
  description: string,
  timeoutMs = 5000
): Promise<T> {
  const startTime = Date.now();

  while (true) {
    const result = condition();
    if (result) return result;

    if (Date.now() - startTime > timeoutMs) {
      throw new Error(`等待 ${description} 超时，已等待 ${timeoutMs}ms`);
    }

    await new Promise(r => setTimeout(r, 10)); // 每 10ms 轮询一次
  }
}
```

参阅本目录下的 `condition-based-waiting-example.ts` 了解完整实现，包含领域特定的辅助函数（`waitForEvent`、`waitForEventCount`、`waitForEventMatch`），来自实际调试会话。

## 常见错误

**❌ 轮询过快：** `setTimeout(check, 1)` - 浪费 CPU
**✅ 修正：** 每 10ms 轮询一次

**❌ 没有超时：** 如果条件永远不满足则无限循环
**✅ 修正：** 始终包含超时并提供清晰的错误信息

**❌ 过期数据：** 在循环前缓存了状态
**✅ 修正：** 在循环内调用 getter 以获取最新数据

## 何时任意超时是正确的

```typescript
// 工具每 100ms 触发一次——需要 2 次触发来验证部分输出
await waitForEvent(manager, 'TOOL_STARTED'); // 首先：等待条件
await new Promise(r => setTimeout(r, 200));   // 然后：等待定时行为
// 200ms = 100ms 间隔的 2 次触发——有文档记录且有合理理由
```

**要求：**
1. 首先等待触发条件
2. 基于已知时序（不是猜测）
3. 注释说明原因

## 实际影响

来自调试会话（2025-10-03）的数据：
- 修复了 3 个文件中的 15 个不稳定测试
- 通过率：60% → 100%
- 执行时间：快了 40%
- 不再有竞态条件
