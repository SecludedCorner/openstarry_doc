# OpenStarry：系統概覽與架構演進

> *"我們不只構建 Chatbot，我們構建的是數位物種的作業系統。"*

## OpenStarry 包含什麼

一個完整的、Headless 的 AI Agent 作業系統：

| 層級 | 組件 |
|------|------|
| **五蘊 SDK** | TypeScript 介面：IUI、IListener、IProvider、ITool、IGuide |
| **Agent Core** | 6 狀態執行迴圈、EventBus、上下文管理、Session 隔離 |
| **安全系統** | 3 層斷路器（資源級 → 行為級 → 人為覆寫） |
| **插件生態系** | Transport（stdio、WebSocket、HTTP+SSE）、Provider（Gemini OAuth）、Tools（檔案系統）、Guides（角色、技能） |
| **Session 管理** | 每連線隔離、Session 恢復、統一 TraceId |
| **Orchestrator Daemon** | `openstarryd`——生命週期管理、程序隔離（Docker、WASM）、狀態持久化 |
| **MCP 協定** | 跨 Agent 協作，基於 JSON-RPC 2.0、工具暴露、遞迴防護 |
| **TUI 儀表板** | 即時 Agent 監控、互動式設計器、插件編排 |
| **可觀測性** | 結構化 JSON 日誌、Trace 上下文傳播、健康檢查 |

## 如何建造的：八個演進階段

OpenStarry 不是一次建成的。它經歷了八個精心規劃的架構階段逐步演進，每個階段都增加一層新的能力——就像一個數位生物體從胚胎發育到成熟。

### 第一階段：創世紀——骨架

Monorepo 腳手架、TypeScript strict 設定、pnpm workspace protocol，以及五蘊 SDK 介面。在這個階段，Agent 只是一組型別定義——純粹的潛能，就像生物體成形前的 DNA。

**關鍵產出：** 11 個 workspace 套件，具有清晰的依賴邊界。

### 第二階段：意識核心——心跳

執行迴圈狀態機、EventBus、上下文管理，以及帶有斷路器的安全監控器。Agent 獲得了「心跳」——一個持續的感知 → 思考 → 行動 → 學習循環。

**關鍵產出：**
- 6 狀態執行迴圈（WAITING → ASSEMBLING → AWAITING_LLM → PROCESSING → EXECUTING_TOOLS → SAFETY_LOCKOUT）
- Token 預算（100k）、迴圈上限（50 次迭代）、重複失敗偵測（SHA-256 指紋辨識）
- 非同步 FIFO 事件佇列，解耦生產者與消費者
- 滑動視窗上下文管理，搭配 System Prompt 錨定

### 第三階段：身體與感官——第一批器官

插件基礎建設：Plugin Loader 自動 Hook 註冊、Tool Registry 搭配 Zod→JSON Schema 轉換、Provider Registry、UI Registry、Listener Registry、Guide Registry。空的 Core 現在可以接收器官了。

**關鍵產出：** 一個通用的插件載入系統，「安裝一個 npm 套件 = 獲得完整的領域能力」。

### 第四階段：第一口呼吸——活了

CLI Runner（`apps/runner`）、stdio 插件（終端機 I/O）、Gemini OAuth Provider（大腦）、檔案系統工具（雙手）、以及角色初始化 Guide（靈魂）。一個 OpenStarry Agent 第一次能夠感知、思考、行動並說話。

**關鍵產出：**
- 端到端 Agent 生命週期：啟動 → 載入插件 → 監聽 → 回應 → 關閉
- OAuth 2.0 + PKCE 認證搭配 AES-256-GCM 機器綁定 Token 加密
- 路徑沙箱化的檔案系統工具，搭配 Zod 驗證
- 痛覺機制：錯誤作為回饋，而非崩潰

### 第五階段：多通道——多副身體

WebSocket 和 HTTP 的 Transport 插件。同一個 Agent 現在可以同時透過終端機、WebSocket 或 HTTP API 被存取。Session 隔離為每個連線提供了獨立的對話狀態。

**關鍵產出：**
- WebSocket：雙向通訊、ping/pong 健康檢查、Session 恢復
- HTTP：REST 端點 + Server-Sent Events 用於即時串流
- Transport Bridge：將事件從 Core 路由至所有已註冊的 UI，具備錯誤隔離
- 每連線 Session 管理，向下相容預設 Session

### 第六階段：碎形社會——Agent 協作

MCP（Model Context Protocol）整合。Agent 可以將工具暴露給其他 Agent、呼叫其他 Agent 的工具，並形成動態團隊。

**關鍵產出：**
- MCP Server：任何 Agent 都可以透過 JSON-RPC 2.0 暴露其工具
- MCP Client：任何 Agent 都可以呼叫其他 Agent 的工具
- 工具白名單：`expose_tools` / `private_tools` 設定
- 遞迴防護：TraceId + 深度計數器（最大 5 層）防止無限迴圈
- DevTools：用於檢視 Agent 內部狀態的除錯介面
- 碎形組合：Agent 團隊對外暴露與個別 Agent 相同的介面

### 第七階段：Daemon——持久的生命

Orchestrator Daemon（`openstarryd`）。Agent 成為真正的 OS 層級程序，具備持久的生命週期管理。

**關鍵產出：**
- Agent 生命週期管理：生成、監控、重啟、終止
- 狀態持久化與復原：Agent 在重啟後存活，記憶向前傳承
- 程序隔離：Docker 容器、WASM 沙箱
- 硬體抽象層（HAL）：攝影機、感測器、執行器——Agent 進入物理世界
- 自動啟動註冊：Agent 在系統開機時啟動

### 第八階段：OS 演進——作業系統

TUI 儀表板與互動式 Agent 設計器。完整願景的實現：數位生命的作業系統。

**關鍵產出：**
- **TUI 儀表板**（`openstarry`）：即時監控所有運行中的 Agent——CPU、記憶體、thoughts/sec、狀態
- **互動式設計器**（`openstarry design`）：視覺化地組裝 Agent 的五蘊——選擇大腦、感官、工具、靈魂
- **工作流程引擎**（`openstarry run workflow.yaml`）：將 Agent 串連成多步驟工作流程，支援動態交接
- **插件同步**（`openstarry plugin sync`）：管理與更新能力庫
- **Agent 註冊**（`openstarry register`）：將 Agent 註冊至 Daemon 進行管理

## 技術規格

七份技術規格定義了系統的契約：

| 規格 | 範疇 | 關鍵細節 |
|------|------|---------|
| **01: Command Registry** | 插件 CLI 命令註冊 | 動態發現，`registry.json` 為唯一真實來源 |
| **02: Event Bus Protocol** | 標準事件封包 | UUID、timestamp、traceId、source、sessionId、priority |
| **03: Plugin Interfaces** | 五蘊 SDK | IUI、IListener、IProvider、ITool、IGuide + IPluginContext |
| **04: Context Management** | 層級式記憶 | 即時（5-10 輪，鎖定）→ 短期（滑動視窗）→ 長期（RAG） |
| **05: Security Protocol** | 多層防禦 | 檔案系統沙箱、命令白名單、資源配額（50 ticks、100k tokens、30s 逾時） |
| **06: MCP Protocol** | 跨 Agent 通訊 | JSON-RPC 2.0、工具暴露白名單、遞迴防護（最大 5 層） |
| **07: Management Zone** | Daemon 架構 | Process/Docker/WASM 隔離、HAL 用於 IoT、YAML 編排規則 |

## 可插拔的記憶策略

不同的 Agent 需要不同的記憶方式。OpenStarry 的上下文管理是插件，不是寫死的：

| 策略 | 運作方式 | 最適用場景 |
|------|---------|-----------|
| **滑動視窗**（預設） | FIFO——保留最近 N 輪，丟棄最舊的 | 簡單問答、短期任務 |
| **動態摘要** | 使用輕量 LLM 將舊對話壓縮為自然語言摘要 | 長期陪伴、複雜專案 |
| **關鍵狀態提取** | 從對話中提取結構化 JSON 狀態，丟棄敘述文字 | 表單填寫機器人、訂票 Agent |

```json
{
  "contextStrategy": {
    "type": "plugin",
    "pluginId": "std-summarization-strategy",
    "config": { "compressionModel": "gemini-1.5-flash", "threshold": 20 }
  }
}
```

> *"Core 始終保持輕量，複雜的記憶管理邏輯下放給了專門的策略模組。"*

## Agent 設定

一個完整的 Agent 透過其五蘊以宣告式方式定義：

```jsonc
{
  "identity": { "id": "dev-bot-01", "name": "Resilient Developer" },
  "plugins": [
    // [想] 大腦：認知引擎
    { "name": "@openstarry-plugin/provider-gemini" },
    // [行] 雙手：檔案系統操作
    { "name": "@openstarry-plugin/standard-function-fs" },
    // [受] 感官：終端機輸入
    { "name": "@openstarry-plugin/standard-function-stdio" },
    // [受+色] 網路身體：WebSocket 傳輸
    { "name": "@openstarry-plugin/transport-websocket" },
    // [識] 靈魂：人設與痛覺機制
    { "name": "@openstarry-plugin/guide-character-init",
      "config": { "characterFile": "./personas/developer.md" } }
  ],
  "policy": {
    "safety": { "max_consecutive_errors": 3 }
  }
}
```

## 數據一覽

| 指標 | 數值 |
|------|------|
| Workspace 套件 | 11 |
| 插件套件 | 7+ |
| 架構文件 | 27 |
| 深度剖析文章 | 14 |
| 技術規格 | 7 |
| 實作計畫 | 9+ |
| 執行迴圈狀態 | 6 |
| 事件類型 | 25+ |
| 文件行數 | 10,000+ |
