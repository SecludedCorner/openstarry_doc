# 00. 專案路線圖與里程碑 (Project Roadmap and Milestones)

本文件定義了 OpenStarry 從核心原型到完整生態系的實作路徑。此規劃旨在確保專案演進的有序性，並為開源貢獻者提供清晰的指引。

---

## 🏷️ 版本里程碑總覽 (Version Milestones)

| 版本 | 內容 | 狀態 |
|------|------|------|
| v0.1.0-alpha | MVP 基礎架構 (Plan01-04) | ✅ 完成 |
| v0.2.0-beta | 多通道傳輸 — WebSocket + HTTP (Plan05) | ✅ 完成 |
| v0.2.1-beta | Session 隔離 + SSE + Health Check (Plan05.1, 05.2, 05.5-①) | ✅ 完成 (Cycle 1, 2026-02-10) |
| v0.2.2-beta | Metrics/Logging + Error Handling (Plan05.5-②③) | ✅ 完成 (Cycle 2, 2026-02-11) |
| v0.3.0-beta | MCP 協議整合 (Plan06) | ✅ 完成 (Cycle 3, 2026-02-11) |
| v0.3.5-beta | 管理層基礎建設 (Management Zone Infra) | 📋 規劃中 |
| v0.4.0-beta | Runtime Sandbox MVP (Plan07) | ✅ 完成 (Cycle 5, 2026-02-11) |
| v0.4.1-beta | Sandbox Restart + Worker Pool (Plan07.1) | ✅ 完成 (Cycle 6, 2026-02-11) |
| v0.4.2-beta | Import Analysis + PKI Verification (Plan07.2) | ✅ 完成 (Cycle 7, 2026-02-11) |
| v0.4.3-beta | Audit Logging + Module._load (Plan07.3) | ✅ 完成 (Cycle 8, 2026-02-11) |
| v0.5.0-beta | TUI Dashboard MVP (Plan08) | ✅ 完成 (Cycle 9, 2026-02-11) |
| v0.6.0-beta | DevTools Plugin + E2E Testing (Plan11) | ✅ 完成 (Cycle 12, 2026-02-12) |
| v0.7.0-beta | DevTools Debugging + E2E Framework | ✅ 完成 (Cycle 12, 2026-02-12) |
| v0.8.0-beta | Daemon Mode MVP (Plan12) | ✅ 完成 (Cycle 13, 2026-02-12) |
| v0.9.0-beta | Seamless Attach (Plan13) | ✅ 完成 (Cycle 14, 2026-02-12) |
| v0.10.0-beta | MCP Resources + OAuth 2.1 (Plan06-P3) | ✅ 完成 (Cycle 15, 2026-02-12) |
| v0.11.0-beta | Multi-client Attach & Session Management (Plan14) | ✅ 完成 (Cycle 16, 2026-02-12) |
| v0.12.0-beta | MCP Sampling & Advanced Protocol Extensions (Plan06-P4) | ✅ 完成 (Cycle 17, 2026-02-12) |
| v0.13.0-beta | SDK Context Extensions & Provider Integration (Plan15) | ✅ 完成 (Cycle 18, 2026-02-12) |
| v0.14.0-beta | Security Hardening & Quality Polish (Plan16) | ✅ 完成 (Cycle 19, 2026-02-12) |
| v0.15.0-beta | Plugin Developer Experience (Plan17) | ✅ 完成 (Cycle 20, 2026-02-12) — 970 tests |
| v0.16.0-beta | Plugin Sync & System Plugin Directory (Plan18) | ✅ 完成 (Cycle 21, 2026-02-12) — 1009 tests |
| v0.17.0-beta | Plugin Dependency Wiring & Cross-Plugin Services (Plan19) | ✅ 完成 (Cycle 22, 2026-02-12) — 1067 tests |
| v0.18.0-beta | Workflow Engine MVP (Plan20) | ✅ 完成 (Cycle 23, 2026-02-12) — 1104 tests |
| v0.19.0-beta | Web-based Remote Attach (Plan21) | ✅ 完成 (Cycle 24, 2026-02-12) — 1132 tests |
| v0.20.0-beta | Plugin Marketplace MVP (Plan22) | ✅ 完成 (Cycle 25, 2026-02-13) — 1330 tests |
| v0.20.1-beta | Windows 跨平台修復 + Attach UX 改善 (Hotfix) | ✅ 完成 (Cycle 25-26, 2026-02-13~14) — 1339 tests |
| v0.28.0-alpha | IVolition v1 + Safety Hardening (Plan28) | ✅ 完成 — 1722 tests |
| v0.29.0-alpha | ILoopQualityMonitor + IConfidenceAuditor (Plan29) | ✅ 完成 — 1757 tests |
| v0.30.0-alpha | LoopQualityMonitor Layer 3 (Plan30) | ✅ 完成 |
| v0.31.0-alpha | AuditContext + ThresholdAuditor + Audit Trail (Plan31) | ✅ 完成 |
| v0.32.0-alpha | **Tenet #7 Absolute Purity + Tenet #9 Partial Fix (Plan32)** | ✅ 完成 — 1803 tests, 4 新 plugin, 34 total |

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
        *   **色·輸入 (IListener)**: 感官監聽 (Sensory/Input)。
        *   **想 (IProvider)**: 認知引擎 (Cognition/LLM)。
        *   **行 (ITool)**: 工具執行 (Action/Will)。
        *   **識 (IGuide)**: 我執框架 (Identity/Persona)。
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

- [x] **3.1 核心加載協議 (Loading Protocol)**
    - [x] 實作 `PluginLoader`，支持**工廠模式 (Factory Pattern)** 初始化。 *(Plan01)*
    - [x] 實作 `IPluginContext`：包含 Logger, Config, 以及**跨插件服務注入 (dependencies 欄位)**。 *(Plan19)*
    - [x] **實作迴路編織邏輯 (Dependency Wiring):** 根據 `20` 號文件，在加載時自動對接 OODA 迴路。 *(Plan19 — topologicalSort, cycle detection, AgentCore.serviceRegistry)*
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
**目標：** 解決多用戶隱私問題，優化傳輸效率，建立連線健康檢查。
**狀態：** 🟢 已完成 (Cycle 1, 2026-02-10) — 118 tests, QA PASS, Architect PASS WITH NOTES

### Plan05.1: Session 隔離與訊息路由 ✅
- [x] ISession / ISessionManager 介面 (SDK) + SessionManager 實作 (Core)
- [x] InputEvent.sessionId 欄位 (可選，向下相容)
- [x] IPluginContext.sessions 暴露 ISessionManager 給插件
- [x] Default session (`__default__`) 自動建立，無法銷毀
- [x] WebSocket session-aware routing (per-session 廣播)
- [x] 17 個新增測試覆蓋 session isolation 場景

### Plan05.2: HTTP SSE 支援 ✅
- [x] 新增 `GET /api/events` SSE 端點
- [x] EventSource 相容格式 (text/event-stream)
- [x] Session-scoped 事件過濾
- [x] SSE heartbeat 定時發送
- [x] 與現有 HTTP webhook listener 共存
- [x] 11 個新增測試覆蓋 SSE 場景

### Plan05.5-①: 連線健康檢查 ✅
- [x] WebSocket protocol ping/pong (server-initiated)
- [x] missedPongs 計數，超過 staleThreshold 斷開
- [x] HTTP SSE heartbeat 定時檢查
- [x] HealthCheckConfig 可配置 (enabled, intervalMs, staleThreshold)
- [x] 6 個新增測試覆蓋 health check 場景

> **🏆 里程碑：v0.2.1-beta**
> *   ✅ 多客戶端 session 隔離 (WebSocket + HTTP)
> *   ✅ HTTP SSE 即時串流
> *   ✅ 連線健康檢查 (ping/pong + heartbeat)
> *   ✅ 118 tests, 11 packages, snapshot saved

---

## 🧪 階段 4.7：實作週期二 (v0.2.2-beta)
**目標：** 建立可觀測性基礎建設與標準化錯誤處理。
**狀態：** 🟢 已完成 (Cycle 2, 2026-02-11) — 165 tests, QA PASS, Architect PASS WITH NOTES

### Plan05.5-②: Metrics / Logging 基礎建設 ✅
- [x] MetricsCollector (Core): increment / gauge / getSnapshot / reset
- [x] Logger.time() 方法 (performance.now() 計時)
- [x] METRICS_SNAPSHOT 事件類型
- [x] /metrics slash command (AgentCore)
- [x] Transport 插件 console.log → createLogger 遷移
- [x] 19 個新增測試

### Plan05.5-③: 標準化錯誤處理 ✅
- [x] ErrorCode const (12 個錯誤碼)
- [x] TransportError / SessionError / ConfigError 類別
- [x] ES2022 Error cause chain 支援
- [x] 既有 2-arg 建構子向下相容
- [x] 16 個新增測試

> **🏆 里程碑：v0.2.2-beta**
> *   ✅ MetricsCollector 可觀測性基礎
> *   ✅ 標準化錯誤層級 (ErrorCode + cause chain)
> *   ✅ Transport 插件結構化日誌
> *   ✅ 165 tests, 11 packages, snapshot saved

---

## 🔗 階段 4.8：MCP 協議整合 (v0.3.0-beta)
**目標：** 實現 OpenStarry Agent 與外部 MCP Server 的標準化互聯。
**狀態：** 🟢 已完成 (Cycle 3, 2026-02-11) — 86 tests, Architect PASS WITH NOTES

### Plan06: MCP Client Plugin ✅
- [x] `@openstarry-plugin/mcp-client` 主要實作
  - [x] 通用 MCP 傳輸層抽象 (stdio, SSE)
  - [x] StdioClientTransport 實作 (cross-platform: Windows/Unix)
  - [x] SSEClientTransport 實作
  - [x] RPC 通訊層 (JSONRPCClient)
- [x] MCP Tool Bridge
  - [x] `mcp:` namespace 機制
  - [x] `tools/list` → OpenStarry Tool Registry 映射
  - [x] `tools/call` 執行與結果回傳
- [x] MCP Prompt Bridge
  - [x] Slash command (`/mcp-prompt-name`) 映射
  - [x] `prompts/list` 清單
  - [x] `prompts/get` 取得完整 prompt 內容
- [x] `@openstarry-plugin/mcp-server` 伺服器實作
  - [x] StdioServerTransport 實作
  - [x] MCP Server Protocol 處理 (initialize, tools/list, tools/call, prompts/*)
  - [x] JSON-RPC 回複機制
  - [x] 配套完整錯誤處理
- [x] `@openstarry-plugin/mcp-common` 共用型別
  - [x] MCP Protocol 介面定義
  - [x] Transport 抽象
  - [x] RPC 通訊型別
- [x] 86 項單元測試
  - [x] MCP Client: 53 tests
  - [x] MCP Server: 33 tests

> **🏆 里程碑：v0.3.0-beta**
> *   ✅ MCP Client Plugin 完整實作
> *   ✅ MCP Server Plugin 完整實作
> *   ✅ Stdio 與 SSE 傳輸支援
> *   ✅ Tool 與 Prompt Bridge 互聯
> *   ✅ 86 tests, 3 新增 packages, snapshot saved

---

## 🔒 階段 4.9：Runtime Sandbox (v0.4.0-beta ~ v0.4.3-beta)
**目標：** 實現完整的運行時沙箱機制，確保插件安全隔離與控制。
**狀態：** 🟢 已完成 (Cycles 5-8, 2026-02-11) — 442 tests total, QA PASS, Architect PASS WITH NOTES

### Plan07: Runtime Sandbox MVP (Cycle 5) ✅
**目標：** 建立 Worker 執行緒隔離與基礎簽名驗證機制。
- [x] SandboxManager 與 Worker 執行緒管理
  - [x] 單一 Worker 初始化與運行
  - [x] RPC 通訊機制 (cross-thread MessagePort)
  - [x] Timeout 與 Memory Limit 強制執行
- [x] 簽名驗證機制
  - [x] SHA-256 套件級驗證
  - [x] Manifest 簽名檢查
  - [x] 驗證失敗的明確錯誤報告
- [x] 82 項新增測試

> **里程碑：v0.4.0-beta**
> *   ✅ Worker 執行緒隔離實作
> *   ✅ SHA-256 簽名驗證
> *   ✅ Memory + CPU timeout 控制

### Plan07.1: Worker Lifecycle + Pool (Cycle 6) ✅
**目標：** 實現 Worker 重啟策略、心跳監控與連接池管理。
- [x] Worker 重啟機制
  - [x] 指數退避重啟策略 (exponential backoff: 100, 200, 400, 800ms → cap)
  - [x] 自動復原故障 Worker
- [x] 心跳監控 (Heartbeat)
  - [x] 定期心跳檢查
  - [x] Stall 偵測與自動隔離
  - [x] Missing heartbeat 計數器
- [x] Worker Pool 管理
  - [x] 預先生成 Worker (pool sizing)
  - [x] 自動回收與重用
  - [x] Plugin reset via protocol message
- [x] 110 項新增測試

> **里程碑：v0.4.1-beta**
> *   ✅ 指數退避重啟策略
> *   ✅ 心跳監控與自動恢復
> *   ✅ Worker Pool 連接池機制

### Plan07.2: Import Analysis + PKI (Cycle 7) ✅
**目標：** 實現靜態分析防護與 Ed25519 PKI 驗證。
- [x] 靜態導入分析器 (AST-based)
  - [x] 遞迴式 require/import 掃描
  - [x] 白名單 (allowed) / 黑名單 (blocked) 模組檢查
  - [x] 違規時拒絕載入 (严格模式)
- [x] Ed25519 PKI 簽名驗證
  - [x] 公鑰配置機制
  - [x] Ed25519 簽名檢驗
  - [x] 多個簽名者支援
- [x] SandboxConfig 介面
  - [x] allowed / blocked modules 清單
  - [x] publicKeys 配置
  - [x] Runtime policy (strict/warn/off)
- [x] Worker Pool interface (initialize/acquire/release/shutdown)
- [x] 135 項新增測試

> **里程碑：v0.4.2-beta**
> *   ✅ 靜態導入分析防護
> *   ✅ Ed25519 PKI 驗證機制
> *   ✅ 模組黑白名單控制

### Plan07.3: Audit Logging + Final Hardening (Cycle 8) ✅
**目標：** 實現審計日誌與 Module._load 攔截，完成沙箱硬化。
- [x] AuditLogger 機制
  - [x] 緩衝式 JSONL 寫入
  - [x] 非同步日誌記錄
  - [x] 密鑰隱匿 (password/token/key/auth/credential)
- [x] Module._load 攔截
  - [x] 運行時模組加載控制
  - [x] Strict / Warn / Off 三種模式
  - [x] 呼叫堆疊追蹤
- [x] RPC 審計整合
  - [x] 工具執行日誌
  - [x] 錯誤追蹤
  - [x] 性能指標
- [x] 日誌輪轉與清理
- [x] 115 項新增測試

> **里程碑：v0.4.3-beta**
> *   ✅ 完整審計日誌機制
> *   ✅ Module._load 運行時控制
> *   ✅ 密鑰隱匿與安全硬化
> *   ✅ 442 tests (Plan07 全線), snapshot saved

---

## 📋 待實作項目 (Deferred Plan05.x)

### Plan05.3: DevTools 調試介面 ⬜
> 延後至 Plan06 (MCP) 之後實作。

### Plan05.4: E2E 測試腳手架 ⬜
- [ ] `createTestAgent()` 工具函數
- [ ] 支援 MockLLM 插件
- [ ] 降低社群貢獻門檻

---

## 🏰 階段五：守護進程 (Phase 5: The Orchestrator Daemon) — Plan07
**目標：** 實現進程級管理與持久化。
**狀態：** 🟢 已完成 (v0.4.3-beta, Cycles 5-8, 2026-02-11)

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
**狀態：** 🟢 已完成 (v0.3.0-beta → v0.10.0-beta, Cycle 3 → Cycle 15, 2026-02-11 → 2026-02-12)
**前置條件：** Plan05.1 Session 隔離完成 ✅ (v0.2.1-beta, 2026-02-10)

> **進入檢查清單：**
> - ✅ Plan05.1 Session 隔離已實作 (Cycle 1)
> - ✅ 多客戶端場景驗證通過 (118 tests)
> - ✅ 訊息不會跨 Session 洩漏 (session-aware routing 驗證)

- [x] **6.1 MCP 深度整合 (Plan06-P1: Tools)**
    - [x] 實作 `@openstarry-plugin/mcp-client`，讓 Agent 具備標準化互聯能力。
    - [x] 實作 `@openstarry-plugin/mcp-server`，支援多外部客戶端同時連接
    - [x] 86 項單元測試驗證完成 (v0.3.0-beta, Cycle 3)
- [x] **6.1a MCP Prompts 整合 (Plan06-P2: Prompts)**
    - [x] MCP Prompts RFC 0005 Section 5.2 實作完成
    - [x] `/mcp-prompt-name` Slash 命令支援
    - [x] 動態提示清單與內容檢索
    - [x] 42 項新測試 (v0.9.0-beta 之前, Cycle 9+)
- [x] **6.1b MCP Resources 整合 (Plan06-P3: Resources)**
    - [x] MCP Resources RFC 0005 Section 5.5 實作完成
    - [x] listResources / readResource RPC 處理
    - [x] OAuth 2.1 token 管理 (PKCE, auto-refresh, TTL)
    - [x] AES-256-GCM 加密 + PBKDF2 (100k iterations)
    - [x] `/mcp-resources` / `/mcp-server-resources` Slash 命令
    - [x] 45 項新測試 (v0.10.0-beta, Cycle 15)
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
**狀態：** 🟡 進行中 (v0.5.0-beta Plan08 完成，Plan09 待實作)

### Plan08: TUI Dashboard MVP (v0.5.0-beta) ✅
**狀態：** 🟢 已完成 (Cycle 9, 2026-02-11) — 524 tests (+82 new), 43 test files, QA PASS, Architect PASS_WITH_NOTES

- [x] **8.1 運行時儀表板 (Runtime Dashboard)**
    - [x] 實作 `@openstarry-plugin/tui-dashboard` (使用 Ink v5 + React 18)
    - [x] 整合事件監控、狀態顯示、訊息串流
    - [x] 支援訊息分類、工具呼叫追蹤、錯誤計數
    - [x] 鍵盤快捷鍵 (q = 離開, Tab = 切換事件日誌)

### Plan12: Daemon Mode MVP (v0.8.0-beta) ✅
**狀態：** 🟢 已完成 (Cycle 13, 2026-02-12) — 714 tests (+44 new), 18 packages, QA PASS, Architect PASS (1 rework)

- [x] **後台進程管理 (Daemon Process Management)**
    - [x] CLI 命令：`daemon start`, `daemon stop`, `daemon ps`
    - [x] 進程生成與分離 (detached process, unref)
    - [x] PID 檔案管理 (`~/.openstarry/agents/{agent-id}.pid`)
    - [x] 優雅關閉 (SIGTERM/SIGINT 級聯)
- [x] **IPC 層 (JSON-RPC over Unix Domain Socket)**
    - [x] Socket 通訊：`~/.openstarry/agents/{agent-id}.sock`
    - [x] JSON-RPC 2.0 協議
    - [x] 健康檢查 RPC：`agent.health` → `{ok, uptime, version}`
- [x] **Daemon 插件** (`@openstarry-plugin/daemon`)
    - [x] IPC 伺服器整合
    - [x] 健康檢查提供者 (IProvider)
    - [x] 進程生命週期管理
- [x] **測試涵蓋**: 44 個新測試 (進程管理、IPC 通訊、健康檢查)

> **🏆 里程碑：v0.8.0-beta**
> *   ✅ Daemon 後台進程實作
> *   ✅ IPC 通訊基礎設施
> *   ✅ 714 tests, 18 packages, snapshot saved

### Plan13: Seamless Attach (v0.9.0-beta) ✅
**狀態：** 🟢 已完成 (Cycle 14, 2026-02-12) — 747 tests (+33 new), 18 packages, QA PASS, Architect PASS (1 rework)

- [x] **IPC 協議擴展**
    - [x] `agent.attach(agentId)` → 連接終端客戶端，返回 sessionId
    - [x] `agent.input(sessionId, message)` → 從附加客戶端發送用戶輸入
    - [x] `agent.detach(sessionId)` → 優雅關閉終端會話（daemon 繼續運行）
    - [x] 事件轉發：Core.bus → IPC 橋接，sessionId 過濾
- [x] **CLI 命令** (`openstarry attach [agent-id]`)
    - [x] 列出運行中的 daemon
    - [x] 自動啟動 daemon（若 agent.json 存在但 daemon 未運行）
    - [x] 終端 I/O 代理：stdin → agent.input RPC，daemon 事件 → stdout/stderr
    - [x] 優雅 Ctrl+C 處理（detach，不 kill daemon）
- [x] **事件轉發器** (`event-forwarder.ts`)
    - [x] 會話過濾的事件遞送
    - [x] 支持 LLM 回應、工具執行、錯誤與指標事件
    - [x] JSON 序列化（含時間戳）
    - [x] 大小限制 (64KB 訊息，1MB 事件)
- [x] **安全性與驗證**
    - [x] sessionId 格式驗證 (UUID v4)
    - [x] inputType 白名單 (user, system)
    - [x] Agent 健康檢查（attach 前驗證 daemon 活力）
    - [x] 資料大小強制執行
- [x] **測試涵蓋**: 33 個新測試 (attach 命令、IPC 處理器、事件轉發、I/O 代理)

> **🏆 里程碑：v0.9.0-beta**
> *   ✅ 無縫連線至運行中的 daemon
> *   ✅ 互動式終端對話 (attach/input/detach)
> *   ✅ 自動啟動與事件轉發
> *   ✅ 747 tests, 18 packages, snapshot saved

### Plan20: Workflow Engine MVP (v0.18.0-beta) ✅
**狀態：** 🟢 已完成 (Cycle 23, 2026-02-12) — 1104 tests (+37 new), QA PASS, Architect PASS

- [x] **工作流引擎插件** (`@openstarry-plugin/workflow-engine`)
  - [x] YAML 宣告式多步驟工作流程編排 (Zod 驗證)
  - [x] 4 個步驟執行器：tool, service, llm, command
  - [x] Mustache 樣板插值 (`{{ }}` 語法)
  - [x] LLM 串流整合 (直接 IProvider AsyncIterable API)
  - [x] EventBus 觀測 (step:start/end, workflow:start/end/error)
  - [x] LRU 快取 (記憶體內，最多 100 個條目)
- [x] **測試涵蓋**: 37 個新測試 (8 檔案)

> **🏆 里程碑：v0.18.0-beta**
> *   ✅ Workflow Engine MVP 完成
> *   ✅ SDK/Core 零變更 (微核心純度保持)
> *   ✅ 1104 tests, snapshot saved

### Plan21: Web-based Remote Attach (v0.19.0-beta) ✅
**狀態：** 🟢 已完成 (Cycle 24, 2026-02-12) — 1132 tests, QA CONDITIONAL PASS

- [x] **3 個新/修改插件**
  - [x] `@openstarry-plugin/http-static` (新) — 靜態 HTTP 檔案伺服器 (路徑跨越防護、MIME 處理)
  - [x] `@openstarry-plugin/web-ui` (新) — 瀏覽器代理介面 (HTML 配置注入、會話恢復)
  - [x] `@openstarry-plugin/transport-websocket` (修改) — 認證/CORS/代理 IP 解析
- [x] **測試涵蓋**: 147 個新測試 (6 檔案)

> **🏆 里程碑：v0.19.0-beta**
> *   ✅ Web-based Remote Attach 完成
> *   ✅ WebSocket Token 認證 + CORS
> *   ✅ 1132 tests, snapshot saved

### Plan22: Plugin Marketplace MVP (v0.20.0-beta) ✅
**狀態：** 🟢 已完成 (Cycle 25, 2026-02-13) — 1330 tests, QA PASS

- [x] **外掛目錄與安裝器**
  - [x] 外掛目錄 (`plugin-catalog.json`)：15 個官方外掛清單
  - [x] 外掛鎖檔 (`~/.openstarry/plugins/lock.json`)：追蹤已安裝外掛
  - [x] 安裝器：workspace 優先解析 + npm 後備
  - [x] 短名稱支援 (e.g. `standard-function-fs` → `@openstarry-plugin/standard-function-fs`)
  - [x] 批量安裝 (`plugin install --all`)
- [x] **5 個新 CLI 命令**: plugin install/uninstall/list/search/info
- [x] **測試涵蓋**: 198 個新測試 (77 marketplace + 基礎設施)

> **🏆 里程碑：v0.20.0-beta**
> *   ✅ Plugin Marketplace MVP 完成
> *   ✅ 1330 tests, snapshot saved

### Hotfix: Windows 跨平台修復 + Attach UX (v0.20.1-beta) ✅
**狀態：** 🟢 已完成 (Cycle 25 hotfix + Cycle 26, 2026-02-13~14) — 1339 tests

- [x] **Windows 跨平台修復** (Cycle 25 hotfix)
  - [x] 路徑處理：`path.sep`、`basename()`、`pathToFileURL()` 取代硬編碼 Unix 路徑
  - [x] Daemon IPC：`platform.ts` — Windows named pipe / Linux Unix socket
  - [x] 平台守衛：SIGHUP、chmod、mkdirSync、unlinkSync 在 Windows 上跳過
  - [x] 外掛安裝器：`cp()` dereference 後備 + `rm()` maxRetries
- [x] **Attach UX 改善** (Cycle 26)
  - [x] Attach 連線後自動顯示 provider 登入狀態 (`/provider status`)
  - [x] 歡迎訊息加 `/help` 提示
- [x] **Token 持久化修復** (Cycle 26)
  - [x] `provider-gemini-oauth` 的 `dispose()` 不再誤刪 token 檔案
  - [x] 新增 `cleanup()` 方法，僅清理 callback server 資源

> **🏆 里程碑：v0.20.1-beta**
> *   ✅ 全平台（Windows/Linux）穩定運行
> *   ✅ Attach 自動顯示 provider 狀態
> *   ✅ OAuth token 跨重啟持久化
> *   ✅ 1339 tests, 117 test files, snapshot saved

---

### Plan09: Interactive Designer (待實作) ⬜
- [ ] **8.2 互動式設計器 (Interactive Designer)**
    - [ ] 實作 `openstarry design` 的上下文感知選單。
    - [ ] 建立「五蘊配置」的引導式 Wizard (Inquirer.js)。

---

## 📊 計畫依賴關係圖

```
                    ┌──────────────────────────────────────────────────┐
                    │           已完成 (v0.20.1-beta)                   │
                    │  Plan01 → Plan02 → Plan03 → Plan04 → Plan05    │
                    │  → Plan06-P1/P2/P3/P4 (MCP Tools/Prompts/Res)  │
                    │  → Plan07-07.3 (Sandbox Hardening)             │
                    │  → Cycle1-8: Sessions, MCP, Sandbox Hardening  │
                    │  → Cycle9: Plan08 TUI Dashboard                │
                    │  → Cycle10-11: Plan09-10 Interactive TUI + CLI │
                    │  → Cycle12: Plan11 DevTools + E2E Framework    │
                    │  → Cycle13: Plan12 Daemon Mode MVP             │
                    │  → Cycle14: Plan13 Seamless Attach             │
                    │  → Cycle15: Plan06-P3 Resources + OAuth 2.1    │
                    │  → Cycle16: Plan14 Multi-client Attach & Mgmt  │
                    │  → Cycle17: Plan06-P4 Sampling & Extensions    │
                    │  → Cycle18: Plan15 SDK Context Extensions      │
                    │  → Cycle19: Plan16 Security Hardening          │
                    │  → Cycle20: Plan17 Plugin Developer Experience │
                    │  → Cycle21: Plan18 Plugin Sync                 │
                    │  → Cycle22: Plan19 Dependency Wiring           │
                    │  → Cycle23: Plan20 Workflow Engine MVP          │
                    │  → Cycle24: Plan21 Web Remote Attach           │
                    │  → Cycle25: Plan22 Plugin Marketplace          │
                    │  → Cycle26: Hotfix (Windows + Attach UX)       │
                    └─────────────────────┬───────────────────────────┘
                                          │
                                          ▼
                                    ┌──────────┐
                                    │ Plan09   │
                                    │ Designer │
                                    │ (待排程)  │
                                    └──────────┘
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
