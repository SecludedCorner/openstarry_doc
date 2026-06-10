# 02. 事件匯流排協議規格 (Event Bus & Schema Protocol)

本文件定義了 OpenStarry 內部與邊緣組件通訊的標準事件格式。所有從傳輸插件 (Transport) 流向核心 (Core) 的數據，以及核心產生的反饋，都必須遵守此協議。

## 1. 核心事件結構 (Core Event Envelope)

每個事件都必須被封裝在一個標準的信封對象中，以確保可追溯性與路由正確。

```typescript
interface IOpenStarryEvent {
  // 基礎元數據
  readonly id: string;            // UUID v4
  readonly timestamp: number;     // Unix Timestamp (ms)
  readonly traceId: string;       // 用於追蹤整個請求鏈條的唯一 ID
  
  // 路由信息
  readonly source: string;        // 來源組件 ID (例如 "transport-websocket-01")
  readonly sessionId: string;     // 會話 ID，用於多用戶隔離
  readonly priority: number;      // 0 (最高, 系統緊急) 到 5 (最低, 後台任務)
  
  // 載荷
  readonly type: EventType;       // 事件類型
  readonly payload: any;          // 具體數據內容
}
```

## 2. 事件類型枚舉 (EventType)

### 2.1 輸入類 (Input Events)
*   `INPUT:USER_MESSAGE`: 用戶輸入的純文本。
*   `INPUT:SYSTEM_COMMAND`: 系統級指令（如 `/stop`, `/reset`）。
*   `INPUT:SIGNAL`: 傳感器或外部 Webhook 的觸發訊號。

### 2.2 核心狀態類 (Kernel Events)
*   `KERNEL:TICK_START`: 執行迴圈開始一次迭代。
*   `KERNEL:STATE_CHANGE`: Agent 狀態切換（如 `IDLE` -> `EXECUTING`）。
*   `KERNEL:ERROR`: 核心標準化後的錯誤輸出。

### 2.3 執行類 (Execution Events)
*   `EXEC:TOOL_CALL`: Agent 請求執行工具。
*   `EXEC:TOOL_RESULT`: 工具執行後的結果反饋。

---

## 3. Payload 規範範例

### 3.1 `INPUT:USER_MESSAGE`
```json
{
  "type": "INPUT:USER_MESSAGE",
  "payload": {
    "text": "讀取最近的系統日誌",
    "mimeType": "text/plain",
    "metadata": { "platform": "discord" }
  }
}
```

### 3.2 `EXEC:TOOL_RESULT` (與痛覺機制相關的數據源)
```json
{
  "type": "EXEC:TOOL_RESULT",
  "payload": {
    "toolCallId": "call_abc123",
    "success": false,
    "error": {
      "code": "EPERM",
      "message": "Operation not permitted",
      "source": "fs-plugin"
    }
  }
}
```

## 4. 路由與傳播規則

1.  **優先級處理**: 核心隊列必須優先處理 `priority: 0` 的事件。
2.  **冪等性**: 核心應根據 `id` 丟棄在滑動窗口內重複收到的事件。
3.  **TraceId 透傳**: 任何由事件引發的新事件（例如 Input 引發的 ToolCall），必須繼承原始事件的 `traceId`。
