# 12. 運行時核心：編排器守護進程 (Orchestrator Daemon)

本文件詳細闡述了 `Orchestrator Daemon` 的設計，這是在我們最終架構中，負責管理所有代理人 OS 行程生命週期的關鍵後台服務。

## 核心定位：代理人系統的「init/systemd」

`Orchestrator Daemon` 是一個**常駐後台的、與代理人業務邏輯無關**的系統級服務。它不參與任何 LLM 的思考或任務的執行，其職責非常純粹和底層。

*   **類比：** 它就像 Linux 系統中的 `systemd` 或 `init` 進程，或者容器化世界中的 `Docker Compose` 或 `Kubernetes Kubelet`。
*   **職責：**
    1.  **進程生命週期管理：** 負責啟動、停止、監控和重啟所有「代理人核心」的獨立操作系統行程。
    2.  **基礎設施插件託管：** 負責加載 `infrastructure` 類型的插件，為代理人提供本地共享服務（如消息總線、狀態數據庫），確保環境的開箱即用與可替換性。
    3.  **資源管理：** 監控每個代理人行程的資源消耗（CPU、內存），並可以根據策略進行管理（例如，終止消耗過多的進程）。
    4.  **提供管理接口：** 暴露一組內部 API，供 `AgentManagerTool` 或外部管理工具調用，以程序化的方式管理代理人實例。

---

## 工作流程

### 1. 系統啟動

*   系統管理員在服務器上執行 `openstarry-daemon start` 命令，啟動守護進程。
*   `Orchestrator Daemon` 啟動後，它做的第一件事就是根據其主配置文件，啟動並開始監護**「主代理人 (Master Agent)」**的 OS 行程。

### 2. 代理人創建 (由 `AgentManagerTool` 觸發)

1.  「主代理人」的 `AgentManagerTool` 被調用，意圖啟動一個新的工作代理人。
2.  `AgentManagerTool` 的 `execute` 方法不再是自己創建進程，而是向本地運行的 `Orchestrator Daemon` 發起一個 API 請求。
    *   **請求示例：** `POST http://localhost:5050/agents`
    *   **請求體：**
        ```json
        {
          "agent_id": "data_worker_007",
          "template_id": "DataAnalystAgent_v1"
        }
        ```
3.  `Orchestrator Daemon` 接收到 API 請求後，執行以下操作：
    *   向「代理人設計層」查詢 `DataAnalystAgent_v1` 模板的配置。
    *   準備好啟動新代理人行程所需的環境變量和命令行參數。
    *   使用 `child_process.spawn()` 或類似的系統調用，創建一個新的、獨立的 OS 行程。
    *   將新行程的 `PID` 和 `agent_id` 記錄在自己的監控列表中。
    *   開始監控新行程的健康狀況（例如，通過心跳或 `PID` 是否存在）。

### 3. 代理人銷毀

*   當 `AgentManagerTool` 調用 `agent:stop(agent_id)` 時，它會向 Daemon 發起 `DELETE /agents/{agent_id}` 的 API 請求。
*   `Orchestrator Daemon` 收到請求後，負責安全地終止對應的 OS 行程並清理相關資源。

---

## 結論

引入 `Orchestrator Daemon` 是我們架構演進的最終形態。它完美地解決了多行程架構下的資源管理和生命週期監護問題，同時讓「主代理人」和 `AgentManagerTool` 的職責保持純粹——它們只負責**決策**（「我需要一個新代理人」），而將**執行**（「如何實際創建一個 OS 行程並監控它」）的髒活累活交給了這個專業的底層服務。

