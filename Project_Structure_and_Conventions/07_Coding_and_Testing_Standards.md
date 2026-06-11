# 07. 編碼與測試規範 (Coding & Testing Standards)

為了確保 OpenStarry 能夠像一個工業級操作系統那樣穩定運行，我們制定了以下嚴格的開發規範。

## 1. TypeScript 嚴格模式 (Strictness)

所有 `tsconfig.json` 必須繼承自根目錄配置，並保持 `strict: true`。

*   **No Implicit Any:** 禁止隱式 `any`。如果真的不知道類型，請顯式使用 `unknown` 並在使用前進行類型收窄 (Type Narrowing)。
*   **Null Checks:** 必須處理所有可能的 `null` 或 `undefined`。

```typescript
// ❌ 錯誤示範
function getName(user) {
  return user.name; 
}

// ✅ 正確示範
function getName(user: IUser): string | null {
  if (!user) return null;
  return user.name;
}
```

## 2. 錯誤處理哲學：痛覺 (Error as Pain)

在 OpenStarry 中，錯誤不僅僅是異常，它是給 Agent 的反饋信號。

*   **不要吞掉錯誤:** 除非你有能力完全修復它，否則不要在低層級 `catch` 錯誤而不向上拋出。
*   **使用標準錯誤:** 優先使用 `@openstarry/shared` 中定義的 `AgentPainException` 或其子類。
*   **錯誤即輸入:** 在 Tool 執行層，捕獲的錯誤應被格式化為 `ToolResult` (status: error)，而不是導致進程崩潰。這樣 LLM 才能看到錯誤並嘗試自我修正。

```typescript
// ✅ Tool 內部實現
try {
  // ... 執行邏輯
} catch (error) {
  // 將異常轉化為失敗的執行結果，而非崩潰
  return {
    status: 'error',
    content: `操作失敗: ${error.message}。請檢查路徑是否存在。`
  };
}
```

## 3. 異步與併發

*   **Async/Await:** 全面使用 `async/await`，避免 Callback Hell。
*   **Promise.all:** 對於無依賴的並行操作（如同時讀取多個文件），應使用並行處理以提高效率。

## 4. 測試規範 (Testing Strategy)

我們使用 **Jest** 或 **Vitest** 作為測試框架。

### 4.1 單元測試 (Unit Tests)
*   **位置:** 測試文件應與源文件並列，命名為 `*.test.ts`。
*   **原則:** 測試純邏輯。對於外部依賴（如 LLM API, 文件系統），**必須** 使用 Mock。
*   **Mocking:** 使用依賴注入 (Dependency Injection) 模式，在測試時注入 `MockContext` 或 `MockFileSystem`。

### 4.2 整合測試 (Integration Tests)
*   **位置:** 放在 `tests/integration/` 目錄下。
*   **原則:** 測試多個組件的協同工作（例如：Core + Memory + MockLLM）。

### 4.3 測試覆蓋率
*   核心邏輯 (`packages/core/execution`) 要求 **90% 以上** 的分支覆蓋率。
*   工具插件要求測試其「成功」與「失敗」的兩種路徑。

## 5. 日誌規範 (Logging)

不要使用 `console.log`。必須使用 `context.logger` (來自 `@openstarry/sdk`)。

*   **DEBUG:** 詳細的內部狀態流動 (Loop tick, Context update)。
*   **INFO:** 關鍵生命週期事件 (Agent start, Tool call success)。
*   **WARN:** 可恢復的錯誤 (Tool call failed but handled)。
*   **ERROR:** 不可恢復的系統級錯誤 (Plugin crash, Out of memory)。
