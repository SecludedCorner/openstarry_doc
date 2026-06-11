# OpenStarry 系統：完整啟動與任務流程

本文件詳細闡述了 OpenStarry 系統從「零啟動」到「完成一個複雜多代理人協作任務」的完整生命週期和工作流程。

## 核心流程總覽

整個系統的運作，是基於 `Orchestrator Daemon` 對「代理人核心」行程的生命週期管理，以及「代理人核心」之間透過 `MCP` 進行的智能協作。

---

### 階段一：系統引導與主代理人啟動

1.  **啟動 `Orchestrator Daemon`:**
    *   **啟動者：** 系統管理員。
    *   **動作：** 執行 `openstarry-daemon start` 命令。
    *   **結果：** `Orchestrator Daemon` 作為一個獨立的後台 OS 行程啟動。它成為整個 OpenStarry 系統的進程守護者。

2.  **`Daemon` 啟動「主代理人」:**
    *   **角色：** `Orchestrator Daemon`。
    *   **動作：** `Daemon` 根據其主配置文件，啟動一個新的 OS 行程，運行**「主代理人 (Master Agent)」**。
    *   **結果：** 「主代理人」的行程被創建，並由 `Daemon` 負責其健康監控和生命週期管理。

3.  **「主代理人」自我初始化:**
    *   **角色：** 「主代理人」行程 (一個 `Agent Core` 實例)。
    *   **動作：**
        *   實例化其內部的 **`PluginLoader`**。
        *   `PluginLoader` 掃描預設插件目錄（如 `~/.config/openstarry/system_plugins/`），加載所有為「主代理人」配置的核心能力插件（如 `UI_Listener`, `MCP_Listener`, `AgentManagerTool`, `WorkflowEngineTool`, `AgentDesignerTool` 等）。
    *   **結果：** 「主代理人」完全配置好，具備與人類交互、管理其他代理人、編排工作流等最高級能力，並進入待命狀態。

---

### 階段二：任務觸發與主代理人決策

4.  **接收外部事件 (任務觸發):**
    *   **角色：** 「主代理人」的 `Listener` 插件 (例如 `WhatsApp_Listener`)。
    *   **動作：** 一個人類用戶通過 WhatsApp 發送了一條新的請求（例如，「請幫我分析上季度的銷售數據並生成報告」）。`WhatsApp_Listener` 接收該消息。
    *   **結果：** `Listener` 將消息轉換為標準事件，推入「主代理人」的**內部事件隊列**。

5.  **「主代理人」決策與委派任務:**
    *   **角色：** 「主代理人」的「執行循環」和 LLM。
    *   **動作：**
        *   執行循環從隊列中取出事件，將其與上下文發送給 LLM。
        *   LLM 分析後，判斷這是一個需要專業數據分析能力的任務，決定需要啟動一個「工作代理人」來執行。
        *   LLM 輸出一個對 `AgentManagerTool` 的工具調用請求：`agent:start(template_id: 'DataAnalystAgent_v1', project_path: '/path/to/sales_data')`。

---

#### **階段三：工作代理人按需創建**

6.  **`AgentManagerTool` 請求 Daemon 創建工作代理人:**
    *   **角色：** `AgentManagerTool`。
    *   **動作：** 該工具的 `execute` 方法被調用。它向本地運行的 **`Orchestrator Daemon`** 發起一個 API 請求，請求創建一個基於 `DataAnalystAgent_v1` 模板的工作代理人。

7.  **`Daemon` 創建工作代理人行程:**
    *   **角色：** `Orchestrator Daemon`。
    *   **動作：**
        *   `Daemon` 向「代理人設計層」服務查詢 `DataAnalystAgent_v1` 模板的詳細配置。
        *   創建一個新的 OS 行程，用於運行「工作代理人」。
        *   將模板配置和 `project_path` 等啟動參數傳遞給這個新行程。
        *   將新行程納入自己的監控列表。

8.  **工作代理人自我初始化:**
    *   **角色：** 新的「工作代理人」行程 (一個 `Agent Core` 實例)。
    *   **動作：**
        *   實例化它自己的 `PluginLoader`。
        *   `PluginLoader` 根據傳遞的模板配置和 `project_path` (用於加載專案級插件)，加載該工作代理人所需的精確插件集（例如 `code_interpreter_suite`, `database_query_tool`, `MCP_Listener`）。
    *   **結果：** 一個輕量級、專門化的「數據分析代理人」啟動完畢，並進入待命狀態。

---

#### **階段四：跨代理人協作與任務完成**

9.  **主代理人分配具體任務:**
    *   **角色：** 「主代理人」的 LLM。
    *   **動作：** `agent:start` 工具成功返回後，LLM 知道工作代理人已就緒。它生成一個對 `mcp:send` 工具的調用，將具體的數據和分析指令發送給剛剛創建的工作代理人。

10. **工作代理人執行子任務:**
    *   **角色：** 「工作代理人」。
    *   **動作：**
        *   它的 `MCP_Listener` 接收到任務指令，並將其推入自己的事件隊列。
        *   它的執行循環調用自己的 LLM 和專業工具（如 `code_interpreter`）處理數據。
        *   完成後，通過 `mcp:send` 工具將分析結果（報告、圖表）發回給「主代理人」。

11. **主代理人整理並響應:**
    *   **角色：** 「主代理人」。
    *   **動作：** 收到工作代理人返回的結果，進行最終的整理和包裝。
    *   **結果：** 通過其 `UI_Listener`（例如 `WhatsApp_Listener` 的輸出接口）將最終報告發回給人類用戶。
