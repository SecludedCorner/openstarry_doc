# 15. 測試策略與基礎設施 (Testing Strategy and Infrastructure)

本文件詳述 OpenStarry 專案的測試分層策略、自動化基礎設施以及核心純淨性驗證機制。鑑於本專案採用微內核架構，測試的重點在於確保核心的穩定性與插件的隔離性。

## 1. 測試分層金字塔 (Testing Pyramid)

我們採用經典的測試金字塔模型，但針對 Agent 系統進行了適配：

### 1.1 單元測試 (Unit Tests) - 基石
*   **範圍：** 針對 `packages/core`, `packages/sdk`, `packages/shared` 中的每一個函數與類別。
*   **工具：** `Vitest` (因其對 Monorepo 與 TypeScript 的原生支持)。
*   **原則：**
    *   **Mock Everything:** 核心邏輯測試時，必須 Mock 掉所有的 IO 操作。
    *   **Pure Functions:** 鼓勵編寫純函數，易於測試輸入輸出。
*   **位置：** 與源碼同目錄，命名為 `*.test.ts`。

### 1.2 整合測試 (Integration Tests) - 核心交互
*   **範圍：** 測試 Core 與模擬插件 (Mock Plugins) 之間的交互。
*   **場景：**
    *   Core 是否能正確加載符合 SDK 規範的 Mock Plugin？
    *   Core 是否能正確處理 Plugin 拋出的異常？
    *   事件總線 (Event Bus) 消息是否正確路由？
*   **工具：** `Vitest` + 自研 `MockHost` 環境。

### 1.3 系統/端對端測試 (E2E Tests) - 模擬真實世界
*   **範圍：** 啟動一個真實的 Agent 實例 (使用 `apps/runner`)。
*   **場景：**
    *   **"Hello World" 測試：** Agent 啟動，加載 Stdio Listener，接收輸入，返回 Echo。
    *   **記憶測試：** 進行 3 輪對話，檢查 Context 狀態是否正確更新。
    *   **工具調用測試：** 使用 `fs` 工具寫入文件，並驗證文件是否存在。
*   **策略：** 使用 **"Replay" 機制**。錄製一次真實的 LLM 響應，在 CI 中重放，以避免消耗 Token 並確保測試確定性。

## 2. 核心純淨性強制機制 (Core Purity Enforcement)

由於 OpenStarry 強調 `packages/core` 不得依賴任何具體實現，我們在 CI 階段引入靜態分析：

### 2.1 依賴邊界檢查
*   **工具：** `dependency-cruiser` 或 `eslint-plugin-import`.
*   **規則：**
    *   `packages/core` **禁止 import** `plugins/*`。
    *   `packages/core` **禁止 import** `apps/*`。
    *   `packages/core` 只能依賴 `packages/sdk` 和 `packages/shared`。
*   **執行時機：** 每次 `git push` 前的 Git Hook 及 CI 流程。

### 2.2 構建產物檢查
*   檢查編譯後的 `dist/` 目錄，確保沒有意外將大型第三方庫 (如 `langchain` 或 `puppeteer`) 打包進 Core Bundle。Core 應該保持極度輕量。

## 3. 持續整合流水線 (CI Pipeline)

我們使用 GitHub Actions 構建自動化流水線：

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

## 4. 插件測試規範

對於第三方或官方插件開發者：

*   **MockHost 提供：** SDK 將提供一個 `TestAgentHost` 類別，模擬真實 Agent 環境。開發者無需啟動完整系統即可測試插件的 `onStart`, `onStop` 及工具調用邏輯。
*   **契約測試 (Contract Testing):** 確保插件嚴格遵守 `ITool` 或 `IProvider` 介面的輸入輸出規範。
