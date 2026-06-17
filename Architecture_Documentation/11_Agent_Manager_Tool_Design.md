<!-- QUARANTINE NOTICE 2026-06-12 -->
> **⚠ 隔離標記（2026-06-12 蒸餾掃描）**：本文所述與現行實作不符，或描述未曾建造的系統——**請勿作為規格使用**。API 最高權威＝SDK 型別檔（`packages/sdk/src/`），行為權威＝測試套件。分類依據見 [DISTILLATION_LIST.md](../DISTILLATION_LIST.md)。
> 判定理由（掃描原文）：AgentManagerTool POSTing to a daemon management API at localhost:5050 — **此 REST 管理 API 仍為虛構**（真實 daemon 走 IPC，非 localhost:5050 REST）。
>
> **[v0.59.7 更新]** 但本文所述的「in-loop 代理人管理工具面」**已部分落地**，只是機制不同（daemon service ＋ ITool，非 REST POST）：
> - **生子**：`agent.spawnChild`（`@openstarry-plugin/agent-spawn`，v0.59.4）—— 經 `SERVICE_KEYS.DAEMON_SPAWN`，受 F-5 權限格 ＋ SEC-003。
> - **自省（agent:list/status）**：`agent.listChildren` / `agent.processTree`（`@openstarry-plugin/agent-introspect`，v0.59.7）—— 經 `SERVICE_KEYS.DAEMON_INTROSPECT`（唯讀）委派 daemon 的 `handleChildAgents`/`handleProcessTree`。
> 仍未建：本文的 localhost:5050 REST 管理面、以及對任意 agent 的 kill-by-id（需另立授權模型）。

# 10. 能力插件設計：代理人管理器

本文件詳細闡述了在最終架構下，`AgentManagerTool` 的設計和職責。

## 核心定位：決策者，而非執行者

在引入 `Orchestrator Daemon` 的最終架構中，`AgentManagerTool` 的角色被進一步提純。它不再親自執行創建 OS 行程的「髒活」，而是扮演一個**「決策接口」**的角色。

它負責將「主代理人」的 LLM 作出的「我需要一個新代理人」的**決策**，轉化為一個對 `Daemon` 的**標準化請求**。

---

## `AgentManagerTool` 設計

*   **插件類型：** `tool`

*   **提供的方法 (作為獨立的工具):**
    *   **`agent:start(agent_id: string, template_id: string)`**
        *   **功能：** **請求** `Orchestrator Daemon` 根據一個預定義的模板，創建並啟動一個新的工作代理人。
        *   **實現：** 此方法的 `execute` 邏輯現在非常簡單：
            1.  構造一個 JSON 請求體，如 `{ "agent_id": "...", "template_id": "..." }`。
            2.  向 `Orchestrator Daemon` 暴露的管理 API 端點（例如 `http://localhost:5050/agents`）發送一個 `POST` 請求。
            3.  等待 `Daemon` 的 API 返回結果（例如，包含成功狀態和 `agent_id` 的 JSON），並將其返回給 LLM。
    *   **`agent:stop(agent_id: string)`**
        *   **功能：** **請求** `Orchestrator Daemon` 停止並銷毀一個正在運行的工作代理人。
        *   **實現：** 向 `Daemon` 的 API 端點（例如 `http://localhost:5050/agents/{agent_id}`）發送一個 `DELETE` 請求。
    *   **`agent:status(agent_id: string)`**
    *   **`agent:list()`**
        *   **實現：** 同樣地，這些方法都會轉化為對 `Daemon` 管理 API 的 `GET` 請求。

---

## 總結

這種設計的優勢是顯而易見的：

*   **職責分離：**
    *   **主代理人 (LLM):** 負責最高層的業務決策（**Why & What** - 為什麼需要一個代理人，需要什麼樣的代理人）。
    *   **`AgentManagerTool`:** 負責將決策轉化為標準的 API 請求（**How to Ask** - 如何提出創建請求）。
    *   **`Orchestrator Daemon`:** 負責所有底層的、與操作系統相關的實現細節（**How to Do** - 如何真正創建和管理一個進程）。
*   **可擴展性：** 這種基於 API 的設計，未來可以輕鬆地將 `Daemon` 擴展為一個能夠在多台機器上分佈式地創建代理人的集群管理器，而 `AgentManagerTool` 的接口無需任何改變。
*   **安全性：** 所有創建/銷毀進程的權力都集中在 `Daemon` 一處，便於進行統一的審計和權限控制。