# 15. Testing Strategy & Infrastructure

This document details the layered testing strategy, automation infrastructure, and core purity verification mechanism for the OpenStarry project. Given the project's microkernel architecture, testing focuses on ensuring core stability and plugin isolation.

## 1. Testing Pyramid

We employ a classic testing pyramid model, adapted for Agent systems:

### 1.1 Unit Tests — The Foundation
*   **Scope**: Targeting every function and class within `packages/core`, `packages/sdk`, and `packages/shared`.
*   **Tool**: `Vitest` (due to its native support for Monorepos and TypeScript).
*   **Principles**:
    *   **Mock Everything**: All I/O operations must be mocked during core logic testing.
    *   **Pure Functions**: Encourage the use of pure functions, making input/output testing straightforward.
*   **Location**: Placed alongside the source code, named `*.test.ts`.

### 1.2 Integration Tests — Core Interaction
*   **Scope**: Testing interactions between the Core and Mock Plugins.
*   **Scenarios**:
    *   Can the Core correctly load a Mock Plugin conforming to SDK specifications?
    *   Does the Core correctly handle exceptions thrown by Plugins?
    *   Are Event Bus messages routed correctly?
*   **Tool**: `Vitest` + custom `MockHost` environment.

### 1.3 System/End-to-End (E2E) Tests — Real-world Simulation
*   **Scope**: Launching a real Agent instance using `apps/runner`.
*   **Scenarios**:
    *   **"Hello World" Test**: Agent starts, loads the Stdio Listener, receives input, and returns an Echo.
    *   **Memory Test**: Perform 3 turns of dialogue and verify if the Context state is updated correctly.
    *   **Tool Call Test**: Use an `fs` tool to write a file and verify its existence.
*   **Strategy**: Utilize a **"Replay" mechanism**. Record a real LLM response once and replay it in CI to avoid consuming Tokens and ensure deterministic tests.

## 2. Core Purity Enforcement Mechanism

As OpenStarry emphasizes that `packages/core` must not depend on any concrete implementations, we introduce static analysis during the CI phase:

### 2.1 Dependency Boundary Checks
*   **Tool**: `dependency-cruiser` or `eslint-plugin-import`.
*   **Rules**:
    *   `packages/core` is **forbidden from importing** `plugins/*`.
    *   `packages/core` is **forbidden from importing** `apps/*`.
    *   `packages/core` can only depend on `packages/sdk` and `packages/shared`.
*   **Execution**: Part of Git Hooks before every `git push` and the CI pipeline.

### 2.2 Build Artifact Inspection
*   Inspect the compiled `dist/` directory to ensure that large third-party libraries (e.g., `langchain` or `puppeteer`) are not accidentally bundled into the Core Bundle. The Core must remain extremely lightweight.

## 3. Continuous Integration (CI) Pipeline

We build our automated pipeline using GitHub Actions:

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

## 4. Plugin Testing Standards

For third-party or official plugin developers:

*   **MockHost Provision**: The SDK will provide a `TestAgentHost` class to simulate a real Agent environment. Developers can test plugin `onStart`, `onStop`, and tool invocation logic without launching the full system.
*   **Contract Testing**: Ensures that plugins strictly adhere to the input/output specifications of the `ITool` or `IProvider` interfaces.
