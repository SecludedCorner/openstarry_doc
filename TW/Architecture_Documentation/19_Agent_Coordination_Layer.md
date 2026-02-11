# 19. 代理協調層 (Agent Coordination Layer)

本文件定義了 OpenStarry 系統中的「中樞神經」——協調層。它位於 Daemon 內部，負責管理所有插件的註冊、Agent 的通訊路由以及生命週期。

## 1. 核心職責

協調層 (Coordination Layer) 是 **Daemon** 進程中的一個邏輯模組，它承擔以下職責：

1.  **插件總機 (Plugin Registry Authority):**
    *   維護全系統所有可用插件的清單（掃描自 `~/.openstarry/plugins`）。
    *   響應 Agent 的加載請求，提供插件的物理路徑。
    *   處理 `openstarry plugin sync` 指令，從 `openstarry_plugin` 倉庫同步更新。

2.  **代理人註冊 (Agent Registry):**
    *   記錄所有活躍 Agent 的 PID、通訊端口 (Socket/Pipe) 與狀態。
    *   提供 `getAgent(id)` 接口，讓 CLI (`attach`) 或其他 Agent 能夠找到目標。

3.  **訊息路由 (Message Routing):**
    *   當一個 Agent 發出 `send("manager", "help")` 時，協調層負責解析 "manager" 是誰，並將訊息轉發過去。

## 2. 插件註冊流程

當開發者安裝一個新插件，或者系統啟動時：

1.  **掃描：** 協調層掃描插件目錄。
2.  **驗證：** 檢查 `plugin.json` 的合法性。
3.  **索引：** 將插件的 `id`, `version`, `capabilities` 存入內存資料庫 (如 LokiJS 或 SQLite)。
    *   索引範例：`Capability("weather") -> Plugin("openstarry-weather-v1")`。

## 3. Agent 啟動時的交互

1.  **請求：** 新啟動的 Agent Core 向協調層發送：「我需要 `standard/fs` 和 `gemini`」。
2.  **解析：** 協調層查詢索引，確認這些插件存在且版本兼容。
3.  **授權：** 協調層檢查 `agent.json` 的權限聲明，確認該 Agent 有權使用這些插件。
4.  **返回：** 協調層返回插件的入口文件路徑。
5.  **註冊：** Agent 成功加載後，向協調層回報：「我已上線，我是 `data-bot-01`，我具備 `analyze-data` 的能力」。

## 4. 實現技術

*   **通訊協議：** 使用 **gRPC** 或 **Named Pipes** (IPC) 進行 Core 與 Daemon 之間的高效通訊。
*   **存儲：** 使用輕量級嵌入式資料庫存儲註冊表資訊，確保重啟後不丟失。

---

## 5. 持久化與冷啟動策略 (Persistence & Boot Strategy)

### A. 物理持久化 (Physical Persistence)
透過 `openstarry plugin sync` 或 `add` 安裝的插件，其原始碼會**永久存放**於 `~/.openstarry/plugins/` 目錄下。除非手動刪除，否則重啟後它們依然存在。

### B. 註冊表索引 (Registry Indexing)
為了確保系統的高效能與一致性，協調層採用以下啟動策略：

1.  **快速掃描 (Fast Scan):** 每次 Daemon 啟動時，會先掃描插件目錄的**檔案變動時間 (mtime)**。
2.  **增量更新 (Incremental Refresh):** 
    *   若目錄無變動，則直接從 `registry.db` 讀取快取。
    *   若偵測到新資料夾或版本變更，則自動解析 `plugin.json` 並更新資料庫索引。
3.  **失效清理:** 若資料庫中記錄的插件在硬碟上已不存在，則自動將其從註冊表中移除。

### C. 運行時狀態
請注意，**「註冊表」的持久化不等於「插件實例」的持久化**。每次系統啟動，插件的代碼會根據 `18` 號協議重新加載並執行 `initialize()`。插件若需保存運行數據，必須透過 Core 注入的 `state` 介面（見文檔 06）進行。

---

## 6. 健康檢查與自我修復 (Health Check & Self-Healing)

協調層具備強大的容錯能力，能處理手動操作帶來的各種異常狀態。

### A. 手動放入 (Manual Drop-in)
當用戶直接將文件夾複製到 `~/.openstarry/plugins/` 時：
1.  **發現：** Daemon 的檔案監控 (Watcher) 或啟動掃描偵測到新目錄。
2.  **驗證：** 嘗試解析 `plugin.json`。
    *   **成功：** 自動註冊，狀態設為 `READY`。
    *   **失敗：** 狀態設為 `INVALID_MANIFEST`，並在 Dashboard 顯示黃色警告。

### B. 載入失敗 (Load Failure)
如果插件代碼有誤（如語法錯誤）導致 `initialize()` 崩潰：
1.  **隔離：** Loader 捕獲異常，防止 Daemon 崩潰。
2.  **標記：** 狀態設為 `QUARANTINED` (已隔離)。
3.  **報告：** 在 Dashboard 顯示紅色 ❌，並提供錯誤日誌查看選項。

### C. 依賴缺失 (Dependency Conflict)
如果插件 A 聲明依賴插件 B，但 B 不存在：
1.  **檢查：** 在構建依賴圖時發現斷鏈。
2.  **阻斷：** 拒絕載入插件 A，將其狀態設為 `UNSATISFIED`。
3.  **引導：** CLI 輸出明確建議：
    > "Plugin [A] requires [B]. Please run `openstarry plugin add B` or `openstarry plugin sync` to resolve."


