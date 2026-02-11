# 07. Coding & Testing Standards

To ensure that OpenStarry operates as reliably as an industrial-grade operating system, we have established the following rigorous development standards.

## 1. TypeScript Strictness

All `tsconfig.json` files must inherit from the root configuration and maintain `strict: true`.

*   **No Implicit Any**: Implicit `any` is prohibited. If a type is truly unknown, explicitly use `unknown` and perform type narrowing before use.
*   **Null Checks**: All potential `null` or `undefined` values must be handled.

```typescript
// ❌ Incorrect
function getName(user) {
  return user.name; 
}

// ✅ Correct
function getName(user: IUser): string | null {
  if (!user) return null;
  return user.name;
}
```

## 2. Error Handling Philosophy: Error as Pain

In OpenStarry, errors are not merely exceptions; they are feedback signals provided to the Agent.

*   **Do Not Swallow Errors**: Do not `catch` errors at a low level without re-throwing them unless you possess the capability to fully resolve them.
*   **Use Standard Errors**: Prefer the use of `AgentPainException` or its subclasses defined in `@openstarry/shared`.
*   **Error as Input**: At the tool execution layer, caught errors should be formatted as a `ToolResult` (status: error) rather than causing a process crash. This enables the LLM to perceive the error and attempt self-correction.

```typescript
// ✅ Internal Tool Implementation
try {
  // ... execution logic
} catch (error) {
  // Transform the exception into a failed execution result rather than a crash
  return {
    status: 'error',
    content: `Operation failed: ${error.message}. Please check if the path exists.`
  };
}
```

## 3. Asynchrony & Concurrency

*   **Async/Await**: Use `async/await` throughout to avoid "Callback Hell."
*   **Promise.all**: For independent parallel operations (e.g., reading multiple files simultaneously), use parallel processing to enhance efficiency.

## 4. Testing Strategy

We utilize **Jest** or **Vitest** as our testing framework.

### 4.1 Unit Tests
*   **Location**: Test files should be placed alongside source files and named `*.test.ts`.
*   **Principle**: Test pure logic. For external dependencies (e.g., LLM APIs, filesystem), Mocking is **mandatory**.
*   **Mocking**: Employ the Dependency Injection pattern to inject a `MockContext` or `MockFileSystem` during testing.

### 4.2 Integration Tests
*   **Location**: Placed within the `tests/integration/` directory.
*   **Principle**: Test the collaboration of multiple components (e.g., Core + Memory + MockLLM).

### 4.3 Test Coverage
*   Core logic (`packages/core/execution`) requires a minimum of **90%** branch coverage.
*   Tool plugins are required to test both "success" and "failure" paths.

## 5. Logging

Avoid using `console.log`. You must use `context.logger` (sourced from `@openstarry/sdk`).

*   **DEBUG**: Detailed internal state flow (Loop tick, Context update).
*   **INFO**: Key lifecycle events (Agent start, Tool call success).
*   **WARN**: Recoverable errors (Tool call failed but handled).
*   **ERROR**: Non-recoverable system-level errors (Plugin crash, Out of memory).
