# 12. 錯誤處理與自我修正 (Error Handling & Self-Correction)

本文件定義了 OpenStarry 核心 (Core) 中的異常攔截與反饋機制。核心的職責是確保系統的穩定性，並將異常轉化為可供上層插件（如 Guide）解析的標準化數據。

## 核心機制：異常攔截層 (Exception Interceptor Layer)

在 Agent Core 的執行迴圈中，必須存在一個顯式的攔截層，用於捕獲所有插件執行與內部處理過程中的異常。核心不對異常進行感性詮釋，僅進行技術性標準化。

### 1. 錯誤的標準化 (Normalization)

Core 負責捕獲底層異常（如 `ConnectionRefusedError`）並將其封裝為標準的 `ToolExecutionResult` 對象。這確保了無論底層發生何種錯誤，上層接收到的數據格式始終一致。

```typescript
type ToolExecutionResult = {
  success: boolean;
  data?: any;
  error?: {
    code: string;       // 標準化錯誤代碼，例如 "EPERM", "ENOENT"
    message: string;    // 原始錯誤訊息
    source: string;     // 引發錯誤的組件 ID
    suggestion?: string // 系統級的技術建議
  };
};
```

### 2. 反饋迴路流程 (The Feedback Loop)

1.  **Action Attempt:** 執行迴圈調用工具插件。
2.  **Execution Failure:** 插件執行失敗或拋出異常。
3.  **Interception:** Core 捕獲異常，提取特徵，轉化為 `error` 對象。
4.  **Cognitive Injection:** Core 將此標準化對象封裝為一條 `ToolMessage` 注入 Context。
5.  **Upstream Interpretation:** 由加載的 **識蘊 (Guide)** 插件決定如何向 LLM 詮釋這條訊息（例如將其轉化為負面回饋或重新規劃指令）。

---

## 錯誤分類學 (Taxonomy of Errors)

核心根據錯誤的性質將其分類，以決定不同的系統級行為：

### Type A: 邏輯與執行異常 (Execution Anomalies)
*   **定義：** 工具調用參數錯誤、找不到路徑、權限不足。
*   **核心行為：** 將錯誤數據化並反饋給 LLM 處理。

### Type B: 瞬態環境異常 (Transient Environmental Anomalies)
*   **定義：** 網絡超時、API 限流、資源暫時不可用。
*   **核心行為：** 執行自動重試機制 (Backoff Retry)。

### Type C: 致命系統崩潰 (Fatal System Failures)
*   **定義：** 內存溢出、核心邏輯 Bug、安全熔斷觸發。
*   **核心行為：** 立即終止進程並觸發守護進程 (Daemon) 報警。

---

## 防禦性機制：挫折計數器 (Frustration Counter)

為了防止執行迴圈陷入無效的重複嘗試，核心內建了技術性的保護措施：

1.  Core 維護一個計數器，記錄連續失敗的 Tool 調用次數。
2.  如果計數器超過預設閾值（如 5 次），核心將觸發安全干預。
3.  **干預動作：** 強制在隊列中插入系統級指令，要求 Agent 停止當前路徑。

> **註記：** 核心僅提供「連續失敗」這一事實的偵測，關於「挫折感」或「痛覺」的擬人化詮釋，應完全由外部 Guide 插件實現。
