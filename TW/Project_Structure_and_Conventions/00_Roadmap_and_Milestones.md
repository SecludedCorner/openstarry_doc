# 00. 專案路線圖與里程碑 (Project Roadmap and Milestones)

本文件定義了 OpenStarry 從核心原型到完整生態系的實作路徑。此規劃旨在確保專案演進的有序性，並為開源貢獻者提供清晰的指引。

---

## 🏷️ 版本里程碑總覽 (Version Milestones)

| 版本 | 內容 | 狀態 |
|------|------|------|
| v0.1.0-alpha | MVP 基礎架構 (Plan01-04) | ✅ 完成 |
| v0.2.0-beta | 多通道傳輸 — WebSocket + HTTP (Plan05) | ✅ 完成 |
| v0.2.1-beta | Session 隔離 + SSE + TraceId (Plan05.1, 05.2, 05.5) | 📋 規劃中 |
| v0.2.2-beta | E2E 測試腳手架 (Plan05.4) | 📋 規劃中 |
| v0.3.0-beta | MCP 協議整合 (Plan06 + DevTools) | 📋 規劃中 |
| v0.3.5-beta | 管理層基礎建設 (Management Zone Infra) | 📋 規劃中 |
| v0.4.0-beta | 持久化與狀態恢復 (Plan07) | 📋 規劃中 |
| v0.5.0-beta | TUI Dashboard + 插件協調層 (Plan08-09) | 📋 規劃中 |

---

## 📅 階段一：創世紀 (Phase 1: Genesis)
**目標：** 建立 Monorepo 物理基礎與共用規範。
**狀態：** 🟡 進行中 (In Progress) — 1.3 CI/CD 尚未建立

- [x] **1.1 Monorepo 初始化**
    - [x] 建立 `apps/` 與 `packages/` 目錄結構。
    - [x] 配置 `pnpm-workspace.yaml` 與根目錄 `package.json`。
    - [x] 統一 TypeScript、ESLint 與 Prettier 規範。
- [x] **1.2 核心 SDK 與類型定義**
    - [x] 建立 `packages/sdk` 定義完整的**五蘊插件介面**。
        *   **色 (IUI)**: 介面與呈現 (Form/UI)。
        *   **受 (IListener)**: 感官監聽 (Perception/Input)。
        *   **想 (IProvider)**: 思考生成 (Cognition/LLM)。
        *   **行 (ITool)**: 工具執行 (Action/Will)。
        *   **識 (IGuide)**: 靈魂與人設 (Consciousness/Persona)。
    - [x] 建立 `packages/shared` 提供全局錯誤處理與日誌模組。
- [ ] **1.3 工程化與測試基建 (Engineering & Testing Infrastructure)**
    - [x] 配置單元測試框架 (Vitest)。 *(Plan02 Phase C — 56 項測試通過)*
    - [x] 純淨性檢查腳本 (`pnpm test:purity`)。 *(Plan03 Phase A4)*
    - [ ] 建立 CI/CD 流程 (GitHub Actions)。

## 🧠 階段二：意識內核 (Phase 2: The Conscious Kernel)
**目標：** 實現「無頭 (Headless)」、事件驅動的 `Agent Core`。
**狀態：** 🟢 已完成 (Done)

- [x] **2.1 執行迴圈 (The Loop)**
    - [x] 實作 `packages/core/execution` (基於事件隊列的 tick 機制)。
    - [x] ExecutionLoop 事件驅動重構：從 EventQueue pull 事件，狀態機含 WAITING_FOR_EVENT / SAFETY_LOCKOUT。 *(Plan02 Phase A)*
    - [x] InputEvent payload 標準化（source, inputType, data, replyTo）。 *(Plan02 Phase A4)*
- [x] **2.2 狀態與記憶管理**
    - [x] 實作 `packages/core/state` (支持 Snapshot 與持久化接口)。
    - [x] 實作 `packages/core/memory` (可插拔的上下文策略，如滑動窗口)。
- [x] **2.3 安全斷路器與錯誤反饋 (Safety & Correction)**
    - [x] 實作 Token 消耗上限與無限迴圈偵測 (Circuit Breakers)。 *(Plan02 Phase B — SafetyMonitor)*
    - [x] 實作「錯誤即痛覺」機制，將執行時錯誤轉化為 Context 反饋，觸發 Agent 自我修正。 *(Plan02 Phase B — 挫折計數器 + 重複調用偵測 + 錯誤級聯熔斷)*

## 🦾 階段三：肢體與感官 (Phase 3: Body and Senses)
**目標：** 構建完整的五蘊插件基礎設施與標準庫，建立安全邊界。
**狀態：** 🟡 進行中 (In Progress) — 核心插件已完成，進階功能待實作

- [ ] **3.1 核心加載協議 (Loading Protocol)**
    - [x] 實作 `PluginLoader`，支持**工廠模式 (Factory Pattern)** 初始化。 *(Plan01)*
    - [ ] 實作 `IPluginContext`：包含 Logger, Config, 以及**跨插件服務注入 (dependencies 欄位)**。
    - [ ] **實作迴路編織邏輯 (Dependency Wiring):** 根據 `20` 號文件，在加載時自動對接 OODA 迴路。
    - [x] **實作指令註冊表 (CommandRegistry):** 支援 CLI `run-tool` 與 Chat `/slash` 指令的映射與執行邏輯。 *(Plan01)*
    - [x] **實作動態插件載入 (Dynamic Loading):** CLI 支援 `agent.json` 的 `plugins[].path` 欄位，透過 `import()` 動態載入第三方插件。 *(Plan02 Phase A3)*
    - [x] 實作 `agent.json` 的運行時驗證器（Zod schema 驗證）。 *(v0.1.5 - Plan03 補強完成)*
- [ ] **3.2 協調層與註冊機制 (Coordination Layer)**
    - [x] **雙軌掃描機制 (Dual-Path Scanning):** 同時掃描系統與專案私有插件目錄。
    - [ ] **一鍵同步 (Sync):** 實作 `openstarry plugin sync` 指令，將官方 **`openstarry_plugin`** 倉庫內容同步至系統目錄。
    - [x] 在 Daemon 中實作 **Plugin Registry Service**，建立內存索引。
- [ ] **3.2 標準功能聚合插件庫 (Standard Aggregate Plugins)**
    > **開發位置：** `openstarry_plugin` 倉庫 (Ecosystem Repo)
    - [x] `@openstarry-plugin/standard-function-stdio`: 聚合 受(Stdio Listener) + 色(CLI Body) + 識(Default Guide)。 *(Plan01)*
    - [x] `@openstarry-plugin/provider-gemini-oauth`: 聚合 想(Gemini Provider)。PKCE + OAuth 2.0 認證。 *(Plan01)*
    - [x] `@openstarry-plugin/standard-function-fs`: 聚合 行(FS Tools) + 識(Path Scoping Policy)。 *(Plan01)*
    - [ ] `@openstarry-plugin/guide-mcp`: 聚合 識(MCP Guide)。賦予標準化通訊能力。
    - [ ] `@openstarry-plugin/guide-pain-mechanism`: 實作擬人化痛覺詮釋邏輯。
    - [x] `@openstarry-plugin/standard-function-skill`: 聚合 識(Skill Guide)。**核心依賴**：負責解析 Markdown 技能文件，是工作流與複雜 Agent 的基礎。 *(Plan03 Phase B1)*
- [ ] **3.4 插件開發體驗 (DX)**
    - [ ] 實作 `openstarry create-plugin` 腳手架。
    - [ ] 提供 `MockHost` 測試環境，支持插件的獨立開發與驗證。

> **🔒 安全備註：**
> 階段三的 `fs` 工具雖然不運行在 Docker 中，但必須通過 `packages/shared` 的路徑規範化組件，嚴格限制其操作範圍。這允許 Agent 執行「增刪資料夾」等必要操作，同時保護系統關鍵目錄。

## 👶 階段四：降生 (Phase 4: The First Breath)
**目標：** 實現第一個單機版 CLI Agent，驗證端到端流程。
**狀態：** 🟡 進行中 (In Progress) — Runner 可運行，端到端 LLM 驗證待 OAuth 登入後測試

- [x] **4.1 本地執行器 (Local Runner)**
    - [x] 實作 `apps/runner` 引導程序（純啟動 Bootstrap Runner），支持 `agent.json` 啟動。 *(Plan01)*
    - [x] 支持動態插件解析（path / node_modules 雙層策略，BUILTIN_FACTORIES 已移除，所有插件皆動態載入）。 *(Plan02 Phase A3)*
- [ ] **4.2 整合驗證**
    - [ ] 達成「感知 -> 思考 -> 工具調用 -> 修正 -> 回應」的完整閉環。*(需 OAuth 登入後手動測試)*

> **🏆 里程碑：v0.1 Alpha (MVP)**
> *   ✅ 能在 CLI 中運行單一 Agent。
> *   ✅ 使用 Gemini 作為 Provider（PKCE + OAuth 認證）。
> *   ✅ 能操作本地文件系統 (`fs` tool)。
> *   ✅ 具備基本記憶能力 (5 輪對話)。
> *   ✅ 安全熔斷機制（Token 預算、循環上限、錯誤級聯、挫折計數器）。
> *   ✅ 結構化 JSON 日誌（LOG_FORMAT=json, LOG_LEVEL 過濾）。
> *   ✅ agent.json Zod 運行時驗證（配置錯誤即時報錯）。
> *   ✅ 工具調用 Timeout（Promise.race 防卡死）。
> *   ✅ TraceID 機制（日誌追蹤一次完整處理週期）。
> *   ✅ Markdown 技能載入（standard-function-skill 插件）。
> *   ✅ guideFile 外部檔案引用（system_prompt 從 .md 載入）。
> *   ✅ 純淨性檢查腳本（pnpm test:purity）。
> *   ✅ 56 項單元測試通過（Vitest）。
> *   ⬜ 端到端 LLM 通話驗證（待手動測試）。

---

> **🚀 以上完成 Agent 核心。以下進入「代理人操作系統」與生態系階段。**

---

## 📡 階段 4.5：多通道傳輸 (Phase 4.5: Multi-Channel Transport)
**目標：** 驗證 IUI/IListener 分離架構的可擴展性，實作非 stdio 的通道。
**狀態：** 🟢 已完成 (v0.2.0-beta)

- [x] **4.5.1 WebSocket 傳輸插件**
    - [x] `@openstarry-plugin/transport-websocket`: WebSocket Listener + UI
    - [x] 支援多客戶端連線、定向回覆 (replyTo)
- [x] **4.5.2 HTTP Webhook 傳輸插件**
    - [x] `@openstarry-plugin/transport-http`: HTTP Listener + UI
    - [x] POST /api/input, GET /api/status, GET /api/response
- [x] **4.5.3 多 UI 同時輸出**
    - [x] TransportBridge 廣播機制驗證通過
    - [x] stdio + WebSocket 同時收到事件

> **🏆 里程碑：v0.2.0 Beta (Multi-Channel)**
> *   ✅ WebSocket 傳輸插件實作完成
> *   ✅ HTTP Webhook 傳輸插件實作完成
> *   ✅ 多 UI 同時輸出驗證通過

---

## 🔐 階段 4.6：實作週期一 (v0.2.1-beta)
**目標：** 解決多用戶隱私問題，優化傳輸效率，建立全鏈路追蹤。
**狀態：** 📋 規劃中

### Plan05.1: Session 隔離與訊息路由 🔴 最高優先級
- [ ] Listener 標記 `sessionId`
- [ ] Core 透傳 `sessionId` 到輸出事件
- [ ] UI 依據 `sessionId` 過濾推送
- [ ] 驗收：WebSocket 用戶 A 看不到用戶 B 的對話

### Plan05.2: HTTP SSE 支援 🟢 高優先級
- [ ] 新增 `GET /api/stream` 端點
- [ ] 使用 Server-Sent Events 取代輪詢
- [ ] 支援 EventSource 客戶端

### Plan05.5: 統一 TraceId 🔵 低優先級 (提前實作)
- [ ] Core 提供統一 `TraceId` 產生器
- [ ] 所有事件與日誌自動附加

## 🧪 階段 4.7：實作週期二 (v0.2.2-beta)
**目標：** 建立自動化驗證能力與測試腳手架。
**狀態：** 📋 規劃中

### Plan05.4: E2E 測試腳手架 🟡 中優先級
- [ ] `createTestAgent()` 工具函數
- [ ] 支援 MockLLM 插件
- [ ] 降低社群貢獻門檻

> **⚠️ 注意：Plan05.3 DevTools 延後至 Plan06 (MCP) 之後實作。**

---

## 🏰 階段五：守護進程 (Phase 5: The Orchestrator Daemon) — Plan07
**目標：** 實現進程級管理與持久化。
**狀態：** 📋 規劃中 (v0.4.0-beta)

- [ ] **5.1 守護進程核心 (Daemon)**
    - [ ] 建立 `apps/daemon`，管理多個 Agent 實例的生命週期。
    - [ ] 實作 **Agent 協調管理層 (Management Zone)** 中的調度邏輯。
- [ ] **5.2 API 網關與持久化**
    - [ ] 提供 HTTP/gRPC API 進行遠程管理。
    - [ ] 整合資料庫（SQLite/LevelDB）存儲 Agent 長期狀態。
- [ ] **5.3 安全與治理 (Security & Governance)**
    - [ ] 實作插件簽名與驗證機制 (確保載入的插件是受信任的)。
    - [ ] 實作運行時沙箱 (Sandbox) 策略 (如 Node.js vm 或 WASM 容器化)。
    - [ ] **實作環境隔離與資源配額 (Quota Management)**。
- [ ] **5.4 系統可觀測性與硬體抽象 (Observability & HAL)**
    - [x] 結構化 JSON 日誌，支援 agent_id / trace_id / module 欄位。 *(Plan02 Phase D)*
    - [ ] 整合 OpenTelemetry 或 Event Tracing，追蹤跨 Agent 的任務流動。
    - [ ] 實作日誌收集，支援 Dashboard 的視覺化呈現。
    - [ ] **實作硬體抽象層 (HAL) 標準感官數據流**。

## 🕸️ 階段六：分形社會 (Phase 6: Fractal Society) — Plan06
**目標：** 實現代理人協作與 MCP 協議。
**狀態：** 📋 規劃中 (v0.3.0-beta)
**前置條件：** Plan05.1 Session 隔離完成 ✓

> **進入檢查清單：**
> - ☐ Plan05.1 Session 隔離已實作
> - ☐ 多客戶端場景驗證通過
> - ☐ 訊息不會跨 Session 洩漏

- [ ] **6.1 MCP 深度整合 (Plan06)**
    - [ ] 實作 `packages/mcp-protocol`，讓 Agent 具備標準化互聯能力。
    - [ ] 支援多外部客戶端同時連接
- [ ] **6.2 DevTools 調試介面 (Plan05.3)**
    - [ ] 靜態 HTML 調試介面 `openstarry-devtools`
    - [ ] 左側：對話氣泡視圖
    - [ ] 右側：State/Context 檢視
    - [ ] 底部：事件過濾器
- [ ] **6.3 工作流引擎 (Workflow Engine)**
    - [ ] 實作 `WorkflowEngineTool`，支持 YAML 定義的任務編排。
    - [ ] 實作 Agent 動態孵化與銷毀機制 (Ephemeral Agents)。
- [ ] **6.3 跨 Agent 協作與因果調度 (Orchestration)**
    - [ ] 實現任務拆解與分發至不同子代理人的機制。
    - [ ] **實作基於因果鏈的事件調度 (Causality Chain)**。

## 📦 階段七：視覺化與生態 (Phase 7: Ecosystem and UI)
**目標：** 提供 Web 圖形化管理與部署工具。
**狀態：** 🔴 待啟動

- [ ] **7.1 Dashboard 與多元交互層 (Interface)**
    - [ ] 建立 `apps/dashboard` (React/Next.js)，可視化監控所有 Agent。
    - [ ] **實作狀態投影 (State Projection) 儀表板**。
- [ ] **7.2 模板服務 (Agent Design Service)**
    - [ ] 提供 Agent 商店/模板下載功能，實現「目錄即協議」的極簡部署。

## 🖥️ 階段八：終端機演進 (Phase 8: CLI & TUI Evolution) — Plan08 / Plan09
**目標：** 實現「操作系統級」的終端機體驗，達成 User Scenario Guide 的願景。
**狀態：** 📋 規劃中 (v0.5.0-beta)

- [ ] **8.1 運行時儀表板 (Runtime Dashboard)**
    - [ ] 實作 `openstarry` 的 TUI 介面 (使用 Ink 或 Blessed)。
    - [ ] 整合 Daemon 狀態監控、資源圖表與即時日誌串流。
- [ ] **8.2 互動式設計器 (Interactive Designer)**
    - [ ] 實作 `openstarry design` 的上下文感知選單。
    - [ ] 建立「五蘊配置」的引導式 Wizard (Inquirer.js)。
- [ ] **8.3 無縫連線機制 (Seamless Attach)**
    - [ ] 實作 `openstarry attach` 的 Socket 連線邏輯。
    - [ ] 支援從 TUI 儀表板直接跳轉至 Agent 對話視窗 (按 'a' 鍵)。

---

## 📊 計畫依賴關係圖

```
                    ┌─────────────────────────────────────────┐
                    │           已完成 (v0.2.0-beta)           │
                    │  Plan01 → Plan02 → Plan03 → Plan04 → Plan05  │
                    └─────────────────────┬───────────────────┘
                                          │
              ┌───────────────────────────┼────────────────────────────────┐
              │                           │                                │
              ▼                           ▼                                ▼
        ┌──────────┐              ┌──────────────┐                  ┌──────────┐
        │實作週期一 │              │實作週期二     │                  │ Plan06   │
        │v0.2.1-beta│              │v0.2.2-beta   │                  │ MCP 整合  │
        │(Sess+SSE+ │              │(E2E 測試)     │                  │ (+DevTools)│
        │ TraceId)  │              └──────────────┘                  └────┬─────┘
        └────┬─────┘                                                      │
             │                                                            │
             │ (前置條件)                                                   │
             ▼                                                            ▼
        ┌──────────┐                                                ┌──────────┐
        │ Plan06   │────────────────────────────────────────────────▶│ Plan07   │
        │ MCP 整合 │                                                │ 持久化    │
        └──────────┘                                                └──────────┘
```

---

## 🟡 待討論議題 (Issues to Discuss)

以下議題尚未有明確決議，需要在開發過程中討論：

| 議題 | 狀態 | 相關計畫 |
|------|------|----------|
| WebSocket 認證機制 | 🟡 待討論 | Plan05.1 |
| 多實例負載均衡 | 🟡 待討論 | Plan07 |
| Session 過期策略 | 🟡 待討論 | Plan05.1 |
| SSE 斷線重連 | 🟡 待討論 | Plan05.2 |

### 詳細說明

1. **WebSocket 認證機制**
   - 是否需要 Token 認證？
   - 連線時驗證 vs 每次訊息驗證？
   - 與 Session 隔離的關係？

2. **多實例負載均衡**
   - 多個 Agent 實例如何共享 WebSocket 連線？
   - 是否需要 Redis PubSub？
   - Sticky Session 策略？

3. **Session 過期策略**
   - 多久沒活動後自動清理？
   - 是否需要心跳機制？
   - 過期時如何通知客戶端？

4. **SSE 斷線重連**
   - 客戶端斷線後如何恢復事件流？
   - 是否需要 Last-Event-ID？
   - 重連時如何補發遺漏事件？
