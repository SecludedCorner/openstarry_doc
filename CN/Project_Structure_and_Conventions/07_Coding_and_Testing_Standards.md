# 07. 编码与测试规范 (Coding & Testing Standards)

为了确保 OpenStarry 能够像一个工业级操作系统那样稳定运行，我们制定了以下严格的开发规范。

## 1. TypeScript 严格模式 (Strictness)

所有 `tsconfig.json` 必须继承自根目录配置，并保持 `strict: true`。

*   **No Implicit Any:** 禁止隐式 `any`。如果真的不知道类型，请显式使用 `unknown` 并在使用前进行类型收窄 (Type Narrowing)。
*   **Null Checks:** 必须处理所有可能的 `null` 或 `undefined`。

```typescript
// ❌ 错误示范
function getName(user) {
  return user.name; 
}

// ✅ 正确示范
function getName(user: IUser): string | null {
  if (!user) return null;
  return user.name;
}
```

## 2. 错误处理哲学：痛觉 (Error as Pain)

在 OpenStarry 中，错误不仅仅是异常，它是给 Agent 的反馈信号。

*   **不要吞掉错误:** 除非你有能力完全修复它，否则不要在低层级 `catch` 错误而不向上抛出。
*   **使用标准错误:** 优先使用 `@openstarry/shared` 中定义的 `AgentPainException` 或其子类。
*   **错误即输入:** 在 Tool 执行层，捕获的错误应被格式化为 `ToolResult` (status: error)，而不是导致进程崩溃。这样 LLM 才能看到错误并尝试自我修正。

```typescript
// ✅ Tool 内部实现
try {
  // ... 执行逻辑
} catch (error) {
  // 将异常转化为失败的执行结果，而非崩溃
  return {
    status: 'error',
    content: `操作失败: ${error.message}。请检查路径是否存在。`
  };
}
```

## 3. 异步与并发

*   **Async/Await:** 全面使用 `async/await`，避免 Callback Hell。
*   **Promise.all:** 对于无依赖的并行操作（如同时读取多个文件），应使用并行处理以提高效率。

## 4. 测试规范 (Testing Strategy)

我们使用 **Jest** 或 **Vitest** 作为测试框架。

### 4.1 单元测试 (Unit Tests)
*   **位置:** 测试文件应与源文件并列，命名为 `*.test.ts`。
*   **原则:** 测试纯逻辑。对于外部依赖（如 LLM API, 文件系统），**必须** 使用 Mock。
*   **Mocking:** 使用依赖注入 (Dependency Injection) 模式，在测试时注入 `MockContext` 或 `MockFileSystem`。

### 4.2 整合测试 (Integration Tests)
*   **位置:** 放在 `tests/integration/` 目录下。
*   **原则:** 测试多个组件的协同工作（例如：Core + Memory + MockLLM）。

### 4.3 测试覆盖率
*   核心逻辑 (`packages/core/execution`) 要求 **90% 以上** 的分支覆盖率。
*   工具插件要求测试其「成功」与「失败」的两种路径。

## 5. 日志规范 (Logging)

不要使用 `console.log`。必须使用 `context.logger` (来自 `@openstarry/sdk`)。

*   **DEBUG:** 详细的内部状态流动 (Loop tick, Context update)。
*   **INFO:** 关键生命周期事件 (Agent start, Tool call success)。
*   **WARN:** 可恢复的错误 (Tool call failed but handled)。
*   **ERROR:** 不可恢复的系统级错误 (Plugin crash, Out of memory)。
    *   **ERROR:** 不可恢复的系统级错误 (Plugin crash, Out of memory)。
