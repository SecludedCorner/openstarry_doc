# 14. 狀態管理與持久化機制 (State Management & Persistence)

本文件定義了 OpenStarry 系統如何實現「崩潰恢復 (Crash Recovery)」與「狀態一致性」。

設計遵循 **「核心定義狀態，插件負責存儲」** 的原則，確保 Agent Core 依然保持無狀態 (Stateless) 與輕量化，而將複雜的 IO 操作委託給專門的插件。

## 1. 核心概念：快照 (The Snapshot)

為了保持核心簡潔，Agent Core 在運行時維護一個純 JSON 的狀態對象。我們稱之為 **`AgentSnapshot`**。

### 1.1 `AgentSnapshot` 數據結構

這是 Core 與 Storage Plugin 之間的唯一契約。Core 只負責生成這個對象，不關心它如何被存儲。

```json
{
  "agent_id": "worker_007",
  "tick_index": 42,            // 邏輯時鐘，每執行一次循環 +1
  "timestamp": 1706582400000,  // 快照生成時間
  "status": "WAITING_FOR_EVENT", // 當前狀態機狀態
  "memory": {
    "short_term_history": [    // 當前 Context Window 內的對話
      { "role": "user", "content": "..." },
      { "role": "assistant", "content": "..." }
    ],
    "working_variables": {     // 代理人當前持有的變量
      "current_file_path": "/tmp/report.txt",
      "retry_count": 0
    }
  },
  "event_queue_backup": [      // 尚未處理的事件 (用於恢復現場)
    { "source": "mcp", "type": "task", "payload": "..." }
  ]
}
```

---

## 2. 架構設計：Storage Plugin

為了符合「一切皆插件」的原則，持久化層被抽象為一種新的插件類型。

### 2.1 新增插件類型：`storage`

*   **職責：** 實現 `AgentSnapshot` 的 CRUD 操作。
*   **接口定義 (Interface):**
    所有 `storage` 插件必須實現以下方法：
    *   `save(snapshot: AgentSnapshot): Promise<void>`
    *   `load(agent_id: string): Promise<AgentSnapshot | null>`
    *   `delete(agent_id: string): Promise<void>`

### 2.2 實現範例

*   **`FileStoragePlugin` (開發環境):** 將快照保存為 `.json` 文件 (如 `./data/snapshots/worker_007.json`)。
*   **`RedisStoragePlugin` (生產環境):** 將快照序列化後存入 Redis Key，支持極高的讀寫速度。
*   **`SQLiteStoragePlugin` (邊緣計算):** 存入本地 SQLite 資料庫文件，提供 ACID 事務保障。

---

## 3. 工作流程 (The Workflow)

### 3.1 啟動與恢復 (Hydration)

當 `Orchestrator Daemon` 啟動一個 Agent Core 行程時：

1.  **Core 啟動:** 實例化，加載 `PluginLoader`。
2.  **加載存儲插件:** `PluginLoader` 根據配置加載指定的 `storage` 插件 (例如 `RedisStorage`)。
3.  **請求狀態:** Core 調用 `await storage.load(my_agent_id)`。
4.  **狀態注入 (Hydration):**
    *   **若有存檔:** Core 將 `AgentSnapshot` 中的數據填充回自己的內存變量（恢復 `tick_index`, `history`, `event_queue`）。**代理人「復活」了，它記得崩潰前的一切。**
    *   **若無存檔:** Core 初始化為一個全新的空白狀態。

### 3.2 運行時持久化 (Runtime Persistence)

為了確保「嚴格的」數據安全性，採用 **"Save-After-Write" (處理後即存檔)** 策略。

在 **`01_Execution_Loop`** 的每個循環末尾：

1.  **事件處理完成:** LLM 思考完畢，工具執行完畢。
2.  **狀態更新:** Core 更新內存中的 `history` 和 `variables`。
3.  **觸發存檔:** Core 調用 `await storage.save(current_snapshot)`。
4.  **響應外部:** **只有在 `save` 成功返回後**，Core 才會將響應發送給 UI 或執行下一個 Action。
    *   *這確保了如果系統在響應前崩潰，重啟後不會丟失剛才的執行結果，也不會重複執行已保存但未響應的步驟（需配合等冪性設計）。*

---

## 4. 區分「狀態」與「記憶」

在架構中必須嚴格區分這兩個概念，以免混淆：

| 特性 | **狀態 (State / Persistence)** | **記憶 (Memory / RAG)** |
| :--- | :--- | :--- |
| **目的** | **生存** (Crash Recovery) | **知識** (Knowledge Retrieval) |
| **存儲內容** | 完整的上下文堆棧 (Stack), 變量 | 處理過的文檔, 經驗總結 |
| **存儲位置** | `storage` 插件 (Redis/SQL) | `RAG Engine` (Vector DB) |
| **生命週期** | 隨任務結束可刪除 (Ephemeral) | 永久保存 (Long-term) |
| **負責人** | **Agent Core 直接管理** | **Supporting Engine 外部服務** |

---

## 5. 總結

通過引入 `storage` 插件接口：

1.  **Core 保持了純淨:** 它不需要依賴任何特定的資料庫驅動，只需要調用 `save()` / `load()`。
2.  **極致的靈活性:** 用戶可以通過更換插件，將持久化層從本地文件無縫切換到雲端數據庫，而無需修改核心代碼。
3.  **高可靠性:** 配合 Daemon 的重啟機制，系統實現了「永不失憶」的健壯性。
