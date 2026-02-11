# 01. 理想的代理人架構：整體概述

**OpenStarry 本身即為一個完整的「代理人協調層 (Agent Coordination Layer)」解決方案。**

它不僅僅是一個 Agent 開發框架，更是一套用於構建、運行、管理和編排多智能體系統的操作系統級基礎設施。在此協調層之下，我們將系統解構為以下幾個關鍵子系統。

## 核心子系統 (Sub-systems)

這五個子系統如同五個器官，共同維持著這個協調層的運轉：

*   **1. 代理人核心 (Agent Core / Kernel):**
    *   **角色：** 「大腦與心臟」。
    *   **架構：** **微內核 (Microkernel)**。它只包含最基礎的執行迴圈 (Loop)、狀態機與事件總線。所有具體功能（包括檔案操作、網絡通訊、LLM 調用）都被移出內核，放入用戶空間（插件層）。
    *   **職責：** 一個純粹的、事件驅動的執行引擎。
    *   (詳情請參閱 `02_Headless_Agent_Core.md`)

*   **2. 代理人設計與模板服務 (Agent Design Service):**
    *   **角色：** 「基因庫與造物工坊」。
    *   **職責：** 管理代理人的「藍圖 (Templates)」。它定義了 Agent 擁有什麼性格、加載什麼插件。支持運行時動態創建新角色。
    *   (詳情請參閱 `03_Agent_Design_and_Template_Service.md`)

*   **3. 插件基礎設施 (Plugin Infrastructure):**
    *   **角色：** 「肢體與感官」。
    *   **職責：** 提供標準化的接口，讓 Core 可以掛載工具 (Tools)、通訊協議 (Listeners) 和 AI 模型 (Providers)。
    *   (詳情請參閱 `04_Plugin_Infrastructure.md`)

*   **4. 編排器守護進程 (Orchestrator Daemon):**
    *   **角色：** 「生命週期管理者 (Init Process)」。
    *   **職責：** 負責啟動、監控、重啟和終止 Agent 的 OS 行程。它是確保系統持久運行的基石。
    *   (詳情請參閱 `13_Orchestrator_Daemon_Design.md`)

*   **5. 支撐引擎 (Supporting Engines):**
    *   **角色：** 「後勤支援部門」。
    *   **職責：** 提供共用的後端能力，如：
        *   **記憶引擎 (RAG):** 知識檢索。
        *   **安全引擎 (Guardrails):** 權限與合規審查。
        *   **通訊策略:** 多協議適配與路由。
    *   (詳情請參閱 `07` 和 `09` 號文件)

## 協作工作流簡述

1.  **設計 (Design):** 人類或主代理人通過 **設計服務** 定義一個新的 Agent 模板。
2.  **孵化 (Spawn):** **Orchestrator Daemon** 讀取模板，拉起一個新的 OS 行程運行 **Agent Core**。
3.  **裝配 (Assemble):** 新的 Core 啟動 **插件基礎設施**，加載指定的工具與通訊插件。
4.  **運行 (Run):** Agent 開始在 **支撐引擎** (如記憶、安全) 的輔助下執行任務。
5.  **協作 (Collaborate):** 多個 Agent 通過標準協議 (如 MCP) 進行通訊，共同完成複雜目標。
