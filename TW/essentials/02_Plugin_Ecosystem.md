# OpenStarry 插件生態系

> *"一個插件不是某種類型，它是一組能力的聚合。"*

## 一切皆插件

在 OpenStarry 中，Core 是空的。每一項能力——看、聽、想、行動、認知——都來自插件。這不是設計選擇，而是一種由自動化純度測試所強制執行的哲學。編譯後的 Core 二進位檔包含零插件程式碼。

就像 Linux 透過虛擬檔案系統（VFS）將硬體抽象為檔案，OpenStarry 透過五蘊介面將 Agent 能力抽象為插件。安裝一個 npm 套件 → 獲得完整的領域能力。

## 七個插件

### 識蘊（Consciousness）——靈魂層

**guide-character-init** ——靈魂定義者

最重要的插件：少了它，Agent 就是個失憶的人。這個插件透過 System Prompt 注入人格。

```typescript
// 行內人設
{ prompt: "You are a meticulous code reviewer who speaks in haiku." }

// 或從檔案載入——YAML、Markdown、任何格式
{ characterFile: "./personas/expert-coder.md" }
```

支援行內 Prompt、YAML Frontmatter 檔案，或純 Markdown 角色設定。使用 `ctx.workingDirectory` 進行路徑解析，因此同一個 Agent 可以在不同的專案資料夾中擁有不同的人格。

**standard-function-skill** ——技能載入器

將 Markdown 檔案轉化為 Agent 能力。每個 `.md` 技能檔案都有 YAML Frontmatter（id、version、dependencies、model preferences）和一個成為 System Prompt 的 Markdown 本文。這實現了**宣告式的 Agent 行為**——透過撰寫文件而非程式碼來定義 Agent 的知識。

```yaml
---
type: "skill"
id: "security-auditor"
version: "1.0.0"
dependencies:
  plugins: ["@openstarry-plugin/standard-function-fs"]
  capabilities: ["code-analysis"]
parameters:
  temperature: 0.3
  model_preference: ["gemini-2.0-flash"]
---

You are a security auditor. Analyze code for OWASP Top 10 vulnerabilities...
```

### 想蘊（Perception）——大腦

**provider-gemini-oauth** ——認知引擎

不只是 API 包裝器——而是生產級的 LLM 整合：

- **OAuth 2.0 + PKCE**：安全的瀏覽器認證流程，設定檔中不需要 API Key
- **機器綁定加密**：Token 以 AES-256-GCM 加密，金鑰透過 PBKDF2 從 `hostname + username + salt` 衍生。從某台機器竊取的 Token 在另一台機器上無法使用
- **自動專案配置**：自動建立 Google Cloud Project 以使用免費方案
- **SSE 串流**：即時逐 Token 的回應串流
- **Function Calling**：原生的工具/函式宣告，對接 Gemini 的結構化輸出
- **多模型支援**：支援 Gemini 2.0 Flash、1.5 Pro 和 1.5 Flash
- **斜線命令**：`/provider login gemini`、`/provider logout gemini`、`/provider status`
- **舊版遷移**：自動偵測並重新加密未加密的舊版 Token

Provider 的介面刻意保持精簡——任何能夠串流聊天完成結果的 LLM 後端都能適配：

```typescript
export interface IProvider {
  id: string;
  name: string;
  models: ModelInfo[];
  chat(request: ChatRequest): AsyncIterable<ProviderStreamEvent>;
}
```

### 行蘊（Volition）——雙手

**standard-function-fs** ——五個檔案系統工具

| 工具 | 功能 | 安全性 |
|------|------|--------|
| `fs.read` | 讀取檔案內容（可設定編碼） | 依 `allowedPaths` 進行路徑驗證 |
| `fs.write` | 寫入/建立檔案 | 路徑驗證，防止逃逸 |
| `fs.list` | 列出目錄（支援遞迴） | 限制在工作區範圍內 |
| `fs.mkdir` | 建立目錄（含父目錄） | 僅限許可範圍內 |
| `fs.delete` | 刪除檔案或目錄（支援遞迴） | 嚴格邊界限制 |

每個路徑在執行前都會通過安全層驗證。嘗試讀取 `/etc/passwd`？Agent 會收到一個 `SecurityError`——這會成為一個痛覺信號，回饋到其上下文中進行自我修正。

```typescript
// 工具參數在執行時期透過 Zod 驗證
parameters: z.object({
  path: z.string().describe("File path to read"),
  encoding: z.string().optional().describe("File encoding (default: utf-8)"),
})
```

### 受蘊 + 色蘊（Sensation + Form）——感覺運動層

**standard-function-stdio** ——終端機身體

一個「感官配對」插件：一個套件同時提供輸入（Listener）和輸出（UI）。

**Listener** 透過 readline 讀取 stdin，將 EOF 轉換為 `/quit`，並推送標準化的輸入事件：
```typescript
pushInput({ source: "cli", inputType: "user_input", data: line })
```

**UI** 使用 ANSI 色碼進行視覺呈現：
- `\x1b[36m` 青色——輸入提示（"You: "）
- `\x1b[32m` 綠色——Agent 回應（串流，逐字元輸出）
- `\x1b[33m` 黃色——工具呼叫（Agent 正在做什麼）
- `\x1b[31m` 紅色——錯誤與安全鎖定
- `\x1b[35m` 洋紅色——系統訊息

處理 17 種以上的事件類型，包括 `AGENT_STARTED`（歡迎訊息）、`STREAM_TEXT_DELTA`（即時打字效果）、`TOOL_CALL_START/RESULT/ERROR`（工具生命週期）、以及 `SAFETY_LOCKOUT`（緊急警報）。

**transport-websocket** ——網路身體

具備生產級特性的全雙工通訊：

```
Client → Server:
{ "type": "user_input", "payload": { "text": "..." }, "sessionId": "..." }

Server → Client:
{ "type": "agent_event", "event": { "type": "stream:text_delta", ... } }
```

- **Session 隔離**：每個 WebSocket 連線自動建立獨立的 Session。事件依 `sessionId` 路由——用戶 A 永遠看不到用戶 B 的對話
- **Session 恢復**：斷線後用相同的 `sessionId` 重新連線，即可從中斷處繼續
- **健康監控**：可設定的 ping/pong 間隔（預設 30 秒）。連續 N 次未回應 pong（預設 2 次），連線即被判定為過期並終止
- **定向路由**：事件依 `replyTo`、`sessionId` 路由，或廣播至所有連線

**transport-http** ——Web 身體

REST + SSE 用於 Web 整合：

| 方法 | 端點 | 用途 |
|------|------|------|
| POST | `/api/input` | 提交使用者輸入 → `{status, requestId}` |
| GET | `/api/status` | Agent 健康檢查 → `{status, pendingRequests}` |
| GET | `/api/response?requestId=xxx` | 輪詢回應 → `{events, complete}` |
| GET | `/api/events[?sessionId=xxx]` | SSE 串流 → 即時事件流 |

支援 Session 綁定、CORS 支援、可設定的緩衝區大小（預設 100 個事件）、回應逾時（預設 5 分鐘），以及用於偵測過期 SSE 連線的心跳健康檢查。

## 插件架構

### 工廠模式

每個插件都是一個函式，回傳 manifest（我是誰？）和 factory（我能做什麼？）：

```typescript
export function createXxxPlugin(): IPlugin {
  return {
    manifest: {
      name: "@openstarry-plugin/xxx",
      version: "1.0.0",
      description: "What this plugin does"
    },
    async factory(ctx: IPluginContext) {
      // ctx 提供你所需的一切：
      // - ctx.bus: Event Bus，用於發布/訂閱
      // - ctx.config: 使用者設定
      // - ctx.logger: 結構化日誌
      // - ctx.sessions: Session 管理器
      // - ctx.pushInput(): 向 Core 發送事件
      // - ctx.workingDirectory: 沙箱化的基礎路徑
      // - ctx.agentId: 我是誰？

      return {
        listeners: [],   // 受 — 如何聽
        ui: [],          // 色 — 如何呈現
        providers: [],   // 想 — 如何思考
        tools: [],       // 行 — 如何行動
        guides: [],      // 識 — 我是誰
        commands: [],    // 斜線命令
        dispose: async () => { /* 關閉時的清理工作 */ }
      };
    }
  };
}
```

### 插件組合模式

一個插件不限於單一蘊。就像一個活體器官可以服務多種功能，插件可以組合不同的能力：

| 模式 | 範例 | 組成 | 類比 |
|------|------|------|------|
| 純 Guide | guide-character-init | 僅 `IGuide` | 沒有身體的靈魂 |
| 純 Provider | provider-gemini-oauth | `IProvider` + commands | 瓶中的大腦 |
| 純 Tool | standard-function-fs | `ITool` × 5 | 沒有大腦的手 |
| 感官配對 | standard-function-stdio | `IListener` + `IUI` | 眼睛 + 嘴巴 |
| 傳輸配對 | transport-websocket | `IListener` + `IUI` + dispose | 耳朵 + 聲音 + 告別 |

### pushInput 契約

插件絕不直接呼叫 Core API。所有插件→Core 的通訊都透過一個閘道：`ctx.pushInput()`。這就是感覺神經——插件感知世界，並將標準化的事件推送到 Core 的事件佇列中。Core 從佇列中取出事件，在其執行迴圈中處理。

這種單向契約意味著插件可以被載入、卸載和替換，而完全不需要觸碰 Core 的內部。

### 生命週期管理

每個插件都可以實作 `dispose()` 進行優雅關閉——關閉 HTTP 伺服器、終止 WebSocket 連線、銷毀 Session、清空緩衝區。Plugin Loader 在 Agent 關閉時，會以載入的逆序對所有已載入的插件呼叫 `dispose()`。

```typescript
// Plugin Loader 自動處理註冊
const hooks = await plugin.factory(ctx);

if (hooks.tools)     for (const t of hooks.tools) toolRegistry.register(t);
if (hooks.providers) for (const p of hooks.providers) providerRegistry.register(p);
if (hooks.listeners) for (const l of hooks.listeners) listenerRegistry.register(l);
if (hooks.ui)        for (const u of hooks.ui) uiRegistry.register(u);
if (hooks.guides)    for (const g of hooks.guides) guideRegistry.register(g);
```

## USB 隨插即用情境

OpenStarry 最引人注目的可攜性展示之一：**USB 隨身碟上的 Agent**。

想像插入一個 USB 隨身碟，裡面有這樣的結構：
```
E:/ (USB Root)
├── configs/
│   └── agent.json     # "我是 USB 照片備份助手。我可以讀取 Pictures/ 並寫入 E:/backup_data/"
├── plugins/
│   └── backup-utils/  # 專用的增量備份演算法
├── logs/
└── backup_data/
```

主機系統偵測到 USB，讀取 manifest，詢問使用者授權，以**嚴格的路徑邊界**（`fs_allow_paths: ["C:/Users/Public/Pictures", "E:/backup_data"]`、`network_allow_hosts: []`）生成 Agent，然後 Agent 執行備份任務。完成後回報結果，USB 可安全退出。

Agent 的靈魂（它的 Prompt 與插件）完全存在於 USB 上。它的身體（Core 執行環境）存在於主機上。插到另一台電腦——同樣的靈魂，不同的身體。這就是五蘊的實踐。

## 建立你自己的插件

1. 建立一個依賴 `@openstarry/sdk` 的套件
2. 匯出一個 `createXxxPlugin()` 工廠函式
3. 問問自己：**我的插件提供哪些蘊？**
   - 它要顯示什麼嗎？→ `IUI`
   - 它要聽取什麼嗎？→ `IListener`
   - 它會思考嗎？→ `IProvider`
   - 它要執行動作嗎？→ `ITool`
   - 它知道自己是誰嗎？→ `IGuide`
4. 使用 `ctx.pushInput()` 進行插件→Core 通訊
5. 實作 `dispose()` 進行資源清理
6. 發佈為 npm 套件：`@openstarry-plugin/your-plugin`
