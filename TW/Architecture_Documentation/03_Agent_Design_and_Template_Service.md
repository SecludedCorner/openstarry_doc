# 03. 代理人設計與模板服務 (Agent Design & Template Service)

本文件詳細描述了 OpenStarry 協調層中的「設計子系統」。它負責管理代理人的「基因藍圖」，即模板 (Templates)。

## 核心定位：動態設計工作坊

在 OpenStarry 架構中，**Agent Core 是執行者，Orchestrator 是管理者，而本服務則是「造物主」的工具台。**

它不僅僅是一個靜態的 JSON 文件數據庫，而是一個**運行時服務**，允許人類或高級代理人（如 Master Agent）通過 API 動態地創建、修改和實例化新的代理人角色。

---

## 功能職責

### 1. 模板管理 (Template Management)
負責存儲和版本控制所有的 Agent Template。一個 Template 定義了 Agent 的不可變屬性：
*   **System Prompt:** 核心人格與指令。
*   **Plugin Manifest:** 需要加載的插件列表（能力清單）。
*   **Default Configuration:** 默認的參數配置（如 LLM 型號、溫度）。

### 2. 設計接口 (Design Interface)
提供一組標準 API，供外部系統調用以進行設計工作：
*   `POST /templates`: 根據需求創建新模板。
*   `GET /templates/{id}`: 獲取模板詳情（供 Daemon 啟動 Agent 時使用）。
*   `PUT /templates/{id}/plugins`: 為現有模板增加新能力。

---

## 人機協同設計流程

我們提倡一種 **「由 AI 輔助的 AI 設計 (AI-Assisted AI Design)」** 模式：

1.  **人類意圖:** 管理員告訴主代理人：「我需要一個專門分析日誌的助手。」
2.  **主代理人推導:** Master Agent 的 LLM 分析需求，推導出該助手需要 `fs:read` 工具和 `log-analysis-plugin`。
3.  **調用設計服務:** Master Agent 調用 `AgentDesignerTool`，向本服務發送 API 請求。
4.  **模板生成:** 本服務驗證請求，生成一個新的 `LogAnalyst_Template_v1` 並存儲。
5.  **實例化:** Master Agent 隨後調用 Daemon，基於該新模板啟動一個工作代理人實例。

這種設計使得 OpenStarry 具備了自我擴展和進化的能力。
