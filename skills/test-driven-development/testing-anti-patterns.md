# 测试反模式

**何时加载本参考：** 编写或修改测试、添加 mock，或想给生产类加「仅测试用」方法时。

## 概述

测试必须验证**真实行为**，不是 mock 行为。mock 用来隔离依赖，不是被测对象。

**核心原则：** 测代码**做什么**，不要测 mock **做什么**。

**严格 TDD 能防止这些反模式。**

## 铁律

```
1. 绝不测 mock 行为
2. 绝不给生产类加仅测试用的方法
3. 未理解依赖就不要 mock
```

## 反模式 1：测 mock 行为

**违规：**
```typescript
// ❌ 差：测的是 mock 在不在
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```

**为何错：**
- 你在验证 mock 能用，不是组件能用  
- mock 在就过，不在就挂  
- 对真实行为一无所知  

**伙伴的纠正：**「我们是在测 mock 的行为吗？」

**修复：**
```typescript
// ✅ 好：测真实组件或不要 mock
test('renders sidebar', () => {
  render(<Page />);  // 不要 mock sidebar
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});

// 或若 sidebar 必须 mock 以隔离：
// 不要对 mock 断言——测 Page 在 sidebar 存在时的行为
```

### 门禁

```
在对任何 mock 元素断言之前：
  问：「我在测真实组件行为，还是只测 mock 在不在？」

  若只是测 mock 在不在：
    停——删掉断言或取消 mock

  改测真实行为
```

## 反模式 2：生产代码里仅测试用的方法

**违规：**
```typescript
// ❌ 差：destroy() 只在测试里用
class Session {
  async destroy() {  // 看起来像生产 API！
    await this._workspaceManager?.destroyWorkspace(this.id);
    // ... cleanup
  }
}

// 测试中
afterEach(() => session.destroy());
```

**为何错：**
- 生产类被测试专用代码污染  
- 若生产误调用很危险  
- 违反 YAGNI 与关注点分离  
- 混淆对象生命周期与实体生命周期  

**修复：**
```typescript
// ✅ 好：测试工具负责清理
// Session 无 destroy() —— 生产中无状态

// test-utils/
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}

// 测试中
afterEach(() => cleanupSession(session));
```

### 门禁

```
在给生产类加任何方法之前：
  问：「是否只有测试会用？」

  若是：
    停——不要加进生产类
    放进测试工具

  问：「这个类是否拥有该资源的生命周期？」

  若否：
    停——方法放错类
```

## 反模式 3：不理解就 mock

**违规：**
```typescript
// ❌ 差：mock 破坏了测试依赖的逻辑
test('detects duplicate server', () => {
  // mock 阻止了测试依赖的配置写入！
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));

  await addServer(config);
  await addServer(config);  // 应抛错——但不会！
});
```

**为何错：**
- 被 mock 的方法有测试依赖的副作用（写配置）  
- 过度 mock「求稳」反而破坏真实行为  
- 测试因错误原因通过或神秘失败  

**修复：**
```typescript
// ✅ 好：在正确层次 mock
test('detects duplicate server', () => {
  // 只 mock 慢的部分，保留测试需要的行为
  vi.mock('MCPServerManager'); // 只 mock 慢启动

  await addServer(config);  // 配置写入
  await addServer(config);  // 重复检测 ✓
});
```

### 门禁

```
在 mock 任何方法之前：
  停——先别 mock

  1. 问：「真实方法有哪些副作用？」
  2. 问：「本测试是否依赖其中任一副作用？」
  3. 问：「我是否完全理解本测试需要什么？」

  若依赖副作用：
    在更低层 mock（真正慢/外部的操作）
    或使用保留必要行为的测试替身
    不要 mock 测试依赖的高层方法

  若不确定依赖什么：
    先用真实实现跑测试
    观察实际需要什么
    再在合适层次加最少 mock

  危险信号：
    - 「我 mock 一下求稳」
    - 「可能慢，不如 mock」
    - 不理解依赖链就 mock
```

## 反模式 4：不完整的 mock

**违规：**
```typescript
// ❌ 差：部分 mock——只填你以为要的字段
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // 缺：下游会用到的 metadata
};

// 稍后：访问 response.metadata.requestId 时崩
```

**为何错：**
- **部分 mock 隐藏结构假设**——你只 mock 知道的字段  
- **下游可能依赖你没包的字段**——静默失败  
- **测试过、集成挂**—— mock 不完整，真实 API 完整  
- **虚假信心**——测试什么也没证明  

**铁律：** mock **完整**数据结构，如现实中所是，不要只 mock 当前测试立刻用到的字段。

**修复：**
```typescript
// ✅ 好：镜像真实 API 的完整度
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
  // 真实 API 返回的全部字段
};
```

### 门禁

```
在造 mock 响应之前：
  查：「真实 API 响应包含哪些字段？」

  行动：
    1. 从文档/示例看真实响应
    2. 包含系统下游可能消费的全部字段
    3. 验证 mock 与真实 schema 完全一致

  关键：
    若造 mock，你必须理解**整个**结构
    部分 mock 在代码依赖被省略字段时会静默失败

  不确定：包含文档列出的全部字段
```

## 反模式 5：集成测试成事后想法

**违规：**
```
✅ 实现完成
❌ 没写测试
「可以测了」
```

**为何错：**
- 测试是实现的一部分，不是可选后续  
- TDD 本会拦住这种情况  
- 没测试不能叫完成  

**修复：**
```
TDD 循环：
1. 写失败测试
2. 实现到通过
3. 重构
4. **再**声称完成
```

## mock 变得太复杂时

**警告信号：**
- mock 搭建比测试逻辑还长  
- 什么都 mock 才让测试过  
- mock 缺真实组件有的方法  
- mock 一变测试就挂  

**伙伴会问：**「这里真的需要 mock 吗？」

**考虑：** 用真实组件的集成测试往往比复杂 mock 更简单  

## TDD 如何防止这些反模式

**为何有帮助：**
1. **先写测试** → 被迫想清楚到底在测什么  
2. **看它失败** → 确认测的是真实行为，不是 mock  
3. **最少实现** → 不会塞进仅测试方法  
4. **真实依赖** → mock 前你已看到测试真正需要什么  

**若你在测 mock 行为，你已违反 TDD**——没先看对真实代码失败就加了 mock。

## 速查

| 反模式 | 修复 |
|--------------|-----|
| 断言 mock 元素 | 测真实组件或取消 mock |
| 生产类仅测试方法 | 移到测试工具 |
| 不理解就 mock | 先理解依赖，最少化 mock |
| 不完整 mock | 完整镜像真实 API |
| 测试当后补 | TDD，测试先行 |
| mock 过度复杂 | 考虑集成测试 |

## 危险信号

- 断言查 `*-mock` 测试 ID  
- 方法只在测试文件里被调用  
- mock 搭建占测试一半以上  
- 去掉 mock 测试就挂  
- 说不清为何需要 mock  
- 「先 mock 求稳」  

## 底线

**mock 是隔离工具，不是被测对象。**

若 TDD 显示你在测 mock 行为，你已经走偏了。

修复：测真实行为，或质疑是否根本不该 mock。
