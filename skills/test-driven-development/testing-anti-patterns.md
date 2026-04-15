# 测试反模式

**在以下情况加载此参考文档：** 编写或修改测试、添加 mock、或者想要向生产代码添加仅用于测试的方法时。

## 概述

测试必须验证真实行为，而非 mock 的行为。Mock 是隔离的手段，不是被测试的对象。

**核心原则：** 测试代码做了什么，而不是 mock 做了什么。

**严格遵循 TDD 可以预防这些反模式。**

## 铁律

```
1. 永远不要测试 mock 的行为
2. 永远不要向生产类添加仅用于测试的方法
3. 永远不要在不理解依赖关系的情况下使用 mock
```

## 反模式 1：测试 Mock 的行为

**违规做法：**
```typescript
// ❌ 坏：测试 mock 是否存在
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```

**为什么这是错的：**
- 你在验证 mock 是否正常工作，而不是组件是否正常工作
- 当 mock 存在时测试通过，不存在时失败
- 对真实行为毫无说明

**你的人类搭档的纠正：** "我们是在测试 mock 的行为吗？"

**正确做法：**
```typescript
// ✅ 好：测试真实组件或不要 mock 它
test('renders sidebar', () => {
  render(<Page />);  // 不要 mock sidebar
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});

// 或者如果必须 mock sidebar 来实现隔离：
// 不要断言 mock——测试 Page 在 sidebar 存在时的行为
```

### 守门函数

```
在断言任何 mock 元素之前：
  问自己："我在测试真实组件行为还是仅仅测试 mock 的存在？"

  如果是在测试 mock 的存在：
    停下来 - 删除断言或取消 mock 该组件

  改为测试真实行为
```

## 反模式 2：生产代码中的仅测试方法

**违规做法：**
```typescript
// ❌ 坏：destroy() 仅在测试中使用
class Session {
  async destroy() {  // 看起来像生产 API！
    await this._workspaceManager?.destroyWorkspace(this.id);
    // ... 清理
  }
}

// 在测试中
afterEach(() => session.destroy());
```

**为什么这是错的：**
- 生产类被仅用于测试的代码污染
- 如果在生产环境中被意外调用会很危险
- 违反 YAGNI 原则和关注点分离（separation of concerns）
- 混淆了对象生命周期和实体生命周期

**正确做法：**
```typescript
// ✅ 好：测试工具处理测试清理
// Session 没有 destroy() ——在生产环境中它是无状态的

// 在 test-utils/ 中
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}

// 在测试中
afterEach(() => cleanupSession(session));
```

### 守门函数

```
在向生产类添加任何方法之前：
  问自己："这个方法只在测试中使用吗？"

  如果是：
    停下来 - 不要添加
    把它放在测试工具中

  问自己："这个类拥有这个资源的生命周期吗？"

  如果不是：
    停下来 - 这个方法不属于这个类
```

## 反模式 3：不理解依赖就使用 Mock

**违规做法：**
```typescript
// ❌ 坏：Mock 破坏了测试逻辑
test('detects duplicate server', () => {
  // Mock 阻止了测试依赖的配置写入！
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));

  await addServer(config);
  await addServer(config);  // 应该抛出异常——但不会！
});
```

**为什么这是错的：**
- 被 mock 的方法有测试依赖的副作用（写入配置）
- 为了"安全"而过度 mock 破坏了实际行为
- 测试因错误原因通过或莫名其妙地失败

**正确做法：**
```typescript
// ✅ 好：在正确的层级 mock
test('detects duplicate server', () => {
  // Mock 慢的部分，保留测试需要的行为
  vi.mock('MCPServerManager'); // 只 mock 慢的服务器启动

  await addServer(config);  // 配置已写入
  await addServer(config);  // 检测到重复 ✓
});
```

### 守门函数

```
在 mock 任何方法之前：
  停下来 - 先不要 mock

  1. 问自己："真实方法有什么副作用？"
  2. 问自己："这个测试依赖这些副作用中的任何一个吗？"
  3. 问自己："我完全理解这个测试需要什么吗？"

  如果依赖副作用：
    在更底层 mock（实际慢的/外部的操作）
    或者使用保留必要行为的测试替身（test double）
    不要 mock 测试依赖的高层方法

  如果不确定测试依赖什么：
    先用真实实现运行测试
    观察实际需要发生什么
    然后在正确的层级添加最少的 mock

  危险信号：
    - "为了安全，我先 mock 掉"
    - "这可能很慢，最好 mock 掉"
    - 在不理解依赖链的情况下使用 mock
```

## 反模式 4：不完整的 Mock

**违规做法：**
```typescript
// ❌ 坏：部分 mock——只包含你认为需要的字段
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // 缺少：下游代码使用的 metadata
};

// 之后：当代码访问 response.metadata.requestId 时崩溃
```

**为什么这是错的：**
- **部分 mock 隐藏了结构性假设** —— 你只 mock 了你知道的字段
- **下游代码可能依赖你没有包含的字段** —— 静默失败
- **测试通过但集成失败** —— Mock 不完整，真实 API 完整
- **虚假的信心** —— 测试对真实行为什么也没证明

**铁律：** Mock 完整的数据结构，按照它在现实中的样子，而不只是你当前测试用到的字段。

**正确做法：**
```typescript
// ✅ 好：完整地模拟真实 API 的响应
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
  // 真实 API 返回的所有字段
};
```

### 守门函数

```
在创建 mock 响应之前：
  检查："真实 API 响应包含哪些字段？"

  步骤：
    1. 检查文档/示例中的实际 API 响应
    2. 包含系统下游可能使用的所有字段
    3. 验证 mock 完全匹配真实响应的结构

  关键：
    如果你在创建 mock，你必须理解完整的结构
    部分 mock 在代码依赖被省略的字段时会静默失败

  如果不确定：包含所有已记录的字段
```

## 反模式 5：把集成测试当作事后补充

**违规做法：**
```
✅ 实现完成
❌ 没有写测试
"准备好测试了"
```

**为什么这是错的：**
- 测试是实现的一部分，不是可选的后续步骤
- TDD 本可以避免这个问题
- 没有测试就不能声称已完成

**正确做法：**
```
TDD 循环：
1. 写失败的测试
2. 实现让它通过
3. 重构
4. 然后才能声称已完成
```

## 当 Mock 变得过于复杂时

**警告信号：**
- Mock 的设置代码比测试逻辑还长
- 为了让测试通过而 mock 一切
- Mock 缺少真实组件所拥有的方法
- Mock 变更时测试就崩溃

**你的人类搭档的问题：** "我们真的需要在这里使用 mock 吗？"

**考虑：** 使用真实组件的集成测试往往比复杂的 mock 更简单

## TDD 如何预防这些反模式

**为什么 TDD 有帮助：**
1. **先写测试** → 迫使你思考你实际在测试什么
2. **看它失败** → 确认测试测的是真实行为，而非 mock
3. **最少的实现** → 不会悄悄引入仅用于测试的方法
4. **真实依赖** → 你在 mock 之前就能看到测试实际需要什么

**如果你在测试 mock 的行为，说明你违反了 TDD** —— 你在没有先对真实代码看到测试失败的情况下就添加了 mock。

## 快速参考

| 反模式 | 修复方法 |
|--------|----------|
| 断言 mock 元素 | 测试真实组件或取消 mock |
| 生产代码中的仅测试方法 | 移到测试工具中 |
| 不理解依赖就使用 mock | 先理解依赖关系，最少化 mock |
| 不完整的 mock | 完整地模拟真实 API |
| 测试作为事后补充 | TDD —— 先写测试 |
| 过于复杂的 mock | 考虑集成测试 |

## 危险信号

- 断言检查 `*-mock` 测试 ID
- 方法只在测试文件中被调用
- Mock 设置占测试 >50%
- 移除 mock 后测试就失败
- 无法解释为什么需要 mock
- "为了安全"而使用 mock

## 底线

**Mock 是隔离的工具，不是被测试的对象。**

如果 TDD 揭示你在测试 mock 的行为，那你就走错了。

修复方法：测试真实行为，或者质疑你为什么要 mock。
