# 纵深防御式校验

## 概述

因无效数据导致的 bug，只在一处加校验往往觉得够了。但那条路径可能被别的代码路径、重构或 mock 绕过。

**核心原则：** 在数据经过的**每一层**都校验。让 bug **结构上**不可能发生。

## 为何要多层

单层校验：「我们修了 bug」  
多层校验：「我们让 bug 不可能再发生」  

不同层抓住不同情况：
- 入口校验抓多数问题  
- 业务逻辑抓边界  
- 环境守卫防特定上下文下的危险  
- 调试日志在其他层失手时帮助取证  

## 四层

### 第 1 层：入口校验
**目的：** 在 API 边界拒绝明显无效输入

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory is not a directory: ${workingDirectory}`);
  }
  // ... proceed
}
```

### 第 2 层：业务逻辑校验
**目的：** 确保数据对本操作有意义

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ... proceed
}
```

### 第 3 层：环境守卫
**目的：** 在特定上下文阻止危险操作

```typescript
async function gitInit(directory: string) {
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ... proceed
}
```

### 第 4 层：调试埋点
**目的：** 取证用上下文

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... proceed
}
```

## 应用模式

发现 bug 时：

1. **追踪数据流** — 坏值从哪来？用在哪？  
2. **列出所有检查点** — 数据经过的每一处  
3. **每层加校验** — 入口、业务、环境、调试  
4. **逐层测** — 试着绕过第 1 层，确认第 2 层能抓住  

## 会话中的例子

Bug：空 `projectDir` 导致在源码目录 `git init`

**数据流：**
1. 测试 setup → 空串  
2. `Project.create(name, '')`  
3. `WorkspaceManager.createWorkspace('')`  
4. `git init` 在 `process.cwd()` 跑  

**四层：**
- 第 1 层：`Project.create()` 校验非空/存在/可写  
- 第 2 层：`WorkspaceManager` 校验 projectDir 非空  
- 第 3 层：`WorktreeManager` 在测试中拒绝 tmp 外 git init  
- 第 4 层：git init 前打栈  

**结果：** 1847 测试通过，bug 无法再复现  

## 关键洞察

四层都有必要。测试时各层抓到过其他层漏掉的问题：
- 不同路径绕过入口校验  
- Mock 绕过业务检查  
- 不同平台边界需要环境守卫  
- 调试日志发现结构性误用  

**不要停在一处校验。** 每一层都加检查。
