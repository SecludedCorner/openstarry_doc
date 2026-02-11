# 15. 测试策略与基础设施 (Testing Strategy and Infrastructure)

本文档详述 OpenStarry 项目的测试分层策略、自动化基础设施以及核心纯净性验证机制。鉴于本项目采用微内核架构，测试的重点在于确保核心的稳定性与插件的隔离性。

## 1. 测试分层金字塔 (Testing Pyramid)

我们采用经典的测试金字塔模型，但针对 Agent 系统进行了适配：

### 1.1 单元测试 (Unit Tests) - 基石
*   **范围：** 针对 `packages/core`, `packages/sdk`, `packages/shared` 中的每一个函数与类别。
*   **工具：** `Vitest` (因其对 Monorepo 与 TypeScript 的原生支持)。
*   **原则：**
    *   **Mock Everything:** 核心逻辑测试时，必须 Mock 掉所有的 IO 操作。
    *   **Pure Functions:** 鼓励编写纯函数，易于测试输入输出。
*   **位置：** 与源码同目录，命名为 `*.test.ts`。

### 1.2 整合测试 (Integration Tests) - 核心交互
*   **范围：** 测试 Core 与模拟插件 (Mock Plugins) 之间的交互。
*   **场景：**
    *   Core 是否能正确加载符合 SDK 规范的 Mock Plugin？
    *   Core 是否能正确处理 Plugin 抛出的异常？
    *   事件总线 (Event Bus) 消息是否正确路由？
*   **工具：** `Vitest` + 自研 `MockHost` 环境。

### 1.3 系统/端对端测试 (E2E Tests) - 模拟真实世界
*   **范围：** 启动一个真实的 Agent 实例 (使用 `apps/runner`)。
*   **场景：**
    *   **"Hello World" 测试：** Agent 启动，加载 Stdio Listener，接收输入，返回 Echo。
    *   **记忆测试：** 进行 3 轮对话，检查 Context 状态是否正确更新。
    *   **工具调用测试：** 使用 `fs` 工具写入文件，并验证文件是否存在。
*   **策略：** 使用 **"Replay" 机制**。录制一次真实的 LLM 响应，在 CI 中重放，以避免消耗 Token 并确保测试确定性。

## 2. 核心纯净性强制机制 (Core Purity Enforcement)

由于 OpenStarry 强调 `packages/core` 不得依赖任何具体实现，我们在 CI 阶段引入静态分析：

### 2.1 依赖边界检查
*   **工具：** `dependency-cruiser` 或 `eslint-plugin-import`.
*   **规则：**
    *   `packages/core` **禁止 import** `plugins/*`。
    *   `packages/core` **禁止 import** `apps/*`。
    *   `packages/core` 只能依赖 `packages/sdk` 和 `packages/shared`。
*   **执行时机：** 每次 `git push` 前的 Git Hook 及 CI 流程。

### 2.2 构建产物检查
*   检查编译后的 `dist/` 目录，确保没有意外将大型第三方库 (如 `langchain` 或 `puppeteer`) 打包进 Core Bundle。Core 应该保持极度轻量。

## 3. 持续整合流水线 (CI Pipeline)

我们使用 GitHub Actions 构建自动化流水线：

```yaml
name: OpenStarry CI
on: [push, pull_request]

jobs:
  quality-gate:
    steps:
      - name: Linting
        run: pnpm lint
      - name: Type Checking
        run: pnpm tsc --noEmit
      - name: Purity Check
        run: pnpm check-dependency-boundaries

  test-core:
    needs: quality-gate
    steps:
      - name: Unit Tests
        run: pnpm test:unit --filter "@openstarry/core"

  test-integration:
    needs: test-core
    steps:
      - name: Integration Tests
        run: pnpm test:integration

  build-dry-run:
    steps:
      - name: Build All
        run: pnpm build
```

## 4. 插件测试规范

对于第三方或官方插件开发者：

*   **MockHost 提供：** SDK 将提供一个 `TestAgentHost` 类别，模拟真实 Agent 环境。开发者无需启动完整系统即可测试插件的 `onStart`, `onStop` 及工具调用逻辑。
*   **契约测试 (Contract Testing):** 确保插件严格遵守 `ITool` 或 `IProvider` 接口的输入输出规范。
    *   **契约测试 (Contract Testing):** 确保插件严格遵守 `ITool` 或 `IProvider` 接口的输入输出规范。
