# 09. 可觀測性與鏈路追蹤 (Observability & Tracing)

本文件定義了 OpenStarry 系統中，如何實現標準化日誌 (Logging)、指標 (Metrics) 與鏈路追蹤 (Tracing)，這對於除錯多代理人協作系統至關重要。

## 1. 鏈路追蹤 (Tracing)

在多代理人環境下，一個用戶請求可能觸發多個代理人之間的 MCP 通訊。我們必須能夠追蹤整個請求鏈。

### 1.1 Trace Context (傳遞協議)
系統使用 `trace_id` 作為唯一標識符。

*   **TraceID 生成**: 由 `Orchestrator Daemon` 或 `Gateway` 在請求進入系統的第一刻生成。
*   **傳遞方式**: 在 MCP 消息封包的 `metadata` 欄位中攜帶。
    ```json
    {
      "source": "master_agent",
      "target": "worker_007",
      "metadata": {
        "trace_id": "tr-xxxx-yyyy-zzzz",
        "parent_span_id": "span-123"
      },
      "payload": { ... }
    }
    ```
*   **傳播責任**: Agent Core 收到帶有 `trace_id` 的事件時，必須將該 ID 綁定到當前執行的 `Execution Loop` 上下文中，並在產出的日誌與後續發送的消息中透傳該 ID。

---

## 2. 標準化日誌 (Structured Logging)

系統強制使用 **JSON 格式的結構化日誌**，以便於機器解析與自動化監控。

### 2.1 日誌字段規範
每條日誌應包含：
*   `timestamp`: ISO8601 格式。
*   `level`: `DEBUG`, `INFO`, `WARN`, `ERROR`。
*   `agent_id`: 當前代理人 ID。
*   `trace_id`: 關聯的鏈路追蹤 ID。
*   `module`: `Kernel`, `PluginLoader`, `SafetyMonitor` 等。
*   `msg`: 人類可讀的消息。
*   `context`: (選填) 額外的鍵值對數據。

### 2.2 關鍵事件記錄點
*   **LLM 調用**: 記錄 Prompt 摘要、Token 消耗、延遲。
*   **工具執行**: 記錄工具名稱、參數 (脫敏)、執行結果。
*   **狀態切換**: 記錄狀態機從 A 到 B 的切換。
*   **熔斷觸發**: 記錄熔斷原因與當前計數器值。

---

## 3. 指標收集 (Metrics)

用於監控系統健康度與成本。

*   **成本指標 (Cost)**: `total_tokens_used`, `cost_usd` (按模型計費)。
*   **性能指標 (Performance)**: `llm_latency_ms`, `tool_execution_time_ms`。
*   **健康指標 (Health)**: `loop_error_rate`, `active_agents_count`, `mcp_message_latency`。

---

## 4. 實現機制：一切皆插件 (Plugin-based Observability)

Agent Core 不直接連接日誌伺服器或 Prometheus。它透過 **`logger` 插件** 進行輸出。

*   **`ConsoleLoggerPlugin`**: 將 JSON 日誌輸出到 stdout/stderr (Daemon 會負責收集)。
*   **`FileLoggerPlugin`**: 將日誌寫入特定專案目錄。
*   **`OpenTelemetryPlugin`**: 將 Trace 與 Metrics 發送到後端分析平台 (如 Jaeger, Grafana Tempo)。

---

## 5. 總結

透過 TraceID 的全鏈路貫穿與 JSON 結構化日誌，我們實現了：
1.  **快速除錯**: 可以在海量日誌中精確撈出某次任務的所有步驟。
2.  **成本監控**: 即時掌握每個代理人、每個任務的花費。
3.  **效能瓶頸分析**: 找出哪個工具或哪個代理人反應最慢。
