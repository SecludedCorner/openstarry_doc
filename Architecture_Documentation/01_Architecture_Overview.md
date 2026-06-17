<!-- Status: CURRENT -->
<!-- Layer: 1-Engineering -->
<!-- Applies to: v0.34.0-alpha -->
<!-- Last verified: 2026-03-16 -->

> ✅ **[實作狀態 — v0.59.6 對代碼核對]** 本文描述「五個子系統」，但**只有前四個是 production-live**（有 import、有測試、實跑過）；第五個（支撐引擎 / Supporting Engines）是 v0.34 時代的**前瞻設計，至今未建造**。對代碼核實如下：
> - **1. 代理人核心（微內核）— 已實作。** `packages/core/src/`（`execution/`、`bus/`、`state/`、`agents/`、`vijnana/`、`vedana/`、`mano/`）。
> - **2. 代理人設計與模板服務 — 已實作。** SDK 介面 `packages/sdk/src/interfaces/` ＋ guide/template 插件（`openstarry_plugin/guide-character-init`、`guide-persistent`）。
> - **3. 插件基礎設施 — 已實作。** SDK 介面（ITool/IListener/IProvider）＋ 47 個插件目錄（`openstarry_plugin/`）。
> - **4. 編排器守護進程（Orchestrator Daemon）— 已實作。** `apps/channel/src/`（`registry-bridge.ts`、`registry-event-bus.ts`、`tools/register.ts`；模型＝daemon-attested-event，見 doc 57）。
> - **5. 支撐引擎（記憶/RAG + 安全護欄/Guardrails + 評測/Eval）— ⚠️ 未建造（aspirational）。** 47 個插件中**無**任何 `rag` / `vector-store` / `embedding` / `guardrail` / `policy` / `eval` 插件（grep 命中數＝0；命中的 `vector` 全是 LoopQualityVector、Lamport 向量時鐘、攻擊向量註解、crypto IV，非檢索向量）。反證：`openstarry_plugin/provider-lmstudio/src/index.ts:37` 以 `.filter((m) => !m.id.includes("embedding"))` **排除** embedding 模型＝無 RAG/檢索路徑。所指 doc 07/09 為設計論述（doc 07 §設計抉擇以 PostgreSQL/Jenkins 類比），非已落地代碼。
>
> 下方原文（含第 5 子系統與工作流第 4 步「支撐引擎」）保留為**歷史設計願景**；勿據此宣稱記憶/RAG、Guardrails、Eval 引擎已存在。

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

*   **5. 支撐引擎 (Supporting Engines):** ⚠️ **[未建造 — 前瞻設計，非 production；見頂部 v0.59.6 核對牌]**
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
