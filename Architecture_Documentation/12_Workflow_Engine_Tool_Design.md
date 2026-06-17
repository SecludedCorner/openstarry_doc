# 11. 能力插件設計：工作流引擎 (WorkflowEngineTool)

> ⚠️ **[2026-06-11 實作狀態標記 — v0.58.0-alpha 修復稽核]** 本文件與實際出貨的 workflow-engine plugin 有顯著差異：
> - **從未建造**：ApiCall/Database/Code/Agent 四種 node executor、`plugin.json` manifest 格式（系統中不存在此格式）；`workflow:execute` 的「立即返回 id 再輪詢」非同步模型（實際為同步完成、id 隨結果返回）
> - **實際實作**：YAML＋Zod schema＋Mustache 插值；step 類型為 `tool`/`service`/`llm`/`command`(stub)/`inference`
> - **v0.58.0-alpha 新增**：`loop` step（foreach/while＋強制 maxIterations 上限）與 opt-in 執行狀態落盤（`OPENSTARRY_WORKFLOW_STATE_DIR`，`getStatus` 跨進程存活）——補上了 MVP 的 loop/state 缺口
> - **v0.59.7 新增**：`workflow:status(executionId)` 工具已落地（`src/tool/workflow-status-tool.ts`，4 測試）——agent 可用 `workflow:execute` 回傳的 `executionId` 事後查該次執行的狀態/輸出/錯誤（設 `OPENSTARRY_WORKFLOW_STATE_DIR` 時跨重啟仍可查）。現存工具＝`workflow:execute` ＋ `workflow:status`。
> 權威規格＝plugin source（`openstarry_plugin/workflow-engine/src/`）及其 1800+ 行測試。

本文件描述了將宏觀、多步驟的任務編排能力，封裝成一個強大的 `Tool` 插件的設計。

## 核心定位：從「架構層」到「能力插件」

在「以代理人為中心」的模式下，複雜工作流的執行能力，不再由一個獨立的架構層來承擔，而是被**封裝成一個名為 `WorkflowEngineTool` 的工具插件**，供「主代理人」或其他被授權的高級代理人按需調用。

這使得「代理人核心」保持了純淨，它無需知道工作流如何執行，只需知道如何調用一個名為 `workflow:execute` 的工具。

---

## `WorkflowEngineTool` 設計

*   **插件類型：** `tool`

*   **提供的工具：**
    *   `workflow:execute(workflow_id: string, initial_payload: object)`: 執行一個預定義好的工作流。
    *   `workflow:status(execution_id: string)`: 查詢一個正在運行的工作流的狀態。

*   **`plugin.json` 示例：**
    ```json
    {
      "name": "Workflow-Engine-Tool",
      "type": "tool",
      "entryPoint": "./WorkflowEngineTool.js"
    }
    ```

*   **內部實現：**
    1.  **工作流定義：** 插件內部包含一個解析器，用於讀取工作流定義文件（例如 `./workflows/my_flow.yaml`）。這些文件描述了工作流的圖結構、節點和數據流。
    2.  **執行引擎：** 當 `workflow:execute` 被調用時，插件內部的執行引擎會：
        *   加載對應 `workflow_id` 的定義文件。
        *   創建一個工作流實例，並用 `initial_payload` 初始化其上下文。
        *   開始異步地、按照圖的依賴關係執行各個節點。
    3.  **節點執行器：** 插件內部會有多種類型的「節點執行器」。
        *   `ApiCallNodeExecutor`: 負責執行 HTTP API 調用。
        *   `DatabaseNodeExecutor`: 負責執行數據庫查詢。
        *   `CodeNodeExecutor`: 負責執行一小段數據轉換代碼。
        *   `AgentNodeExecutor`: **(關鍵)** 如果工作流中的一個節點需要一個代理人來完成，這個執行器會調用 `AgentManagerTool` 來創建一個臨時的「工作代理人」，並通過 MCP 與其通信來完成該節點的任務。
    4.  **返回結果：** `workflow:execute` 工具的調用可以被設計為異步的。它會立即返回一個 `execution_id`，代理人可以稍後使用 `workflow:status` 來查詢結果。或者，工作流執行完畢後，可以通過 `mcp:send` 將最終結果異步地發送回調用者。

---

## 優勢

*   **能力內聚：** 所有與工作流相關的複雜邏輯，都被完美地封裝在這一個 `Tool` 插件中。
*   **核心純淨：** 「代理人核心」完全無需關心工作流是如何被定義和執行的。
*   **按需使用：** 只有「主代理人」或被授權的高級代理人才會被加載這個 `WorkflowEngineTool`，普通的「工作代理人」則不需要，保持了輕量。
