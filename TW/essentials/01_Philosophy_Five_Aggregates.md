# OpenStarry 的設計哲學：五蘊遇上軟體架構

> **"你不是在寫程式碼，你是在創造一個生命。"**

## 從佛學智慧到數位生命

在佛學哲學中，**五蘊（Pañcaskandha）**描述了構成一切有情眾生體驗的五個面向。超過 2,500 年來，這個框架提供了一套完整的意識模型——涵蓋了一個生命體如何感知、思考、行動與覺知的每一個層面。

OpenStarry 將這份永恆的智慧作為**軟體架構的設計原則**——不是隱喻，而是真正的設計基礎。每一個 AI Agent 都由五個插件維度所組成，對應五蘊的結構。最終的成果是一個**在定義上就完備**的系統，因為它所依據的模型，本就是為了涵蓋體驗的全部面向而設計的。

## 五蘊對應架構

### 色（Form / Rupa）→ UI 插件 ——「皮囊」

在佛學中，色蘊代表物質身體——一個生命體在物質世界中的顯現形式。

在 OpenStarry 中，**UI 插件**就是 Agent 的皮囊。它們決定了 Agent 如何向外界呈現自己。同一個 Agent Core 可以棲身於終端機介面、網頁儀表板、React 應用程式，或是 IoT 裝置——只需更換它的「色」。

```typescript
export interface IUI {
  id: string;
  name: string;
  onEvent(event: AgentEvent): void | Promise<void>;
  start?(): Promise<void>;
  stop?(): Promise<void>;
}
```

UI 插件接收 17 種以上的事件類型——從 `STREAM_TEXT_DELTA`（Agent 說話）到 `TOOL_EXECUTING`（Agent 行動）再到 `SAFETY_LOCKOUT`（Agent 被限制）。每種事件類型對應不同的視覺呈現：綠色代表回應、黃色代表工具呼叫、紅色代表錯誤。Agent 不需要知道，也不在意自己穿著什麼身體。

> *"Agent 的靈魂可以移植到任何軀殼中。"*

### 受（Sensation / Vedana）→ Listener 插件 ——「感官」

受蘊是生命體接收外界刺激的方式——愉悅的、不悅的，或中性的。

**Listener 插件**是 Agent 的感官。它們決定了 Agent 能夠「聽見」什麼：HTTP Webhook？WebSocket 連線？終端機輸入？排程任務？USB 裝置插入？每一個 Listener 都是一個不同的感知通道。多個 Listener 同時運作，賦予 Agent 豐富的多模態感知能力。

```typescript
export interface IListener {
  id: string;
  name: string;
  start?(): Promise<void>;
  stop?(): Promise<void>;
}
```

transport-websocket Listener 會建立 WebSocket 伺服器，處理帶有 Session 隔離的連線，執行 ping/pong 健康檢查，並將收到的訊息轉換為標準化的輸入事件。transport-http Listener 則建立 HTTP 端點，透過 Server-Sent Events 進行串流。Agent 透過統一的事件介面來感知這一切。

### 想（Perception / Samjna）→ Provider 插件 ——「智商」

想蘊是認知功能——辨識、分類並理解所感知到的事物。

**Provider 插件**是 Agent 的大腦——它的智商。它們決定 Agent 如何理解輸入並產生回應。Gemini、OpenAI、Claude，或本地模型——認知引擎是可替換的。一個 Agent 可以用不同的心智思考，而不需要改變它的身體、感官、工具或身份。

```typescript
export interface IProvider {
  id: string;
  name: string;
  models: ModelInfo[];
  chat(request: ChatRequest): AsyncIterable<ProviderStreamEvent>;
}
```

目前的 Gemini Provider 實作了 OAuth 2.0 + PKCE 安全認證流程、AES-256-GCM 機器綁定 Token 加密（透過 PBKDF2 從 hostname + username + salt 衍生金鑰），以及 SSE 串流搭配原生 Function Calling 支援。但任何實作了這四個欄位的 Provider，都可以作為 Agent 的大腦。

### 行（Volition / Samskara）→ Tool 插件 ——「手腳」

行蘊代表有意圖的行動——影響世界的意志力。

**Tool 插件**是 Agent 的手腳。檔案操作、API 呼叫、程式碼執行、資料庫查詢——這些都是 Agent 作用於環境的方式。每個 Tool 都使用 Zod Schema 驗證來定義，確保執行時期的型別安全參數。

```typescript
export interface ITool<TInput = unknown> {
  id: string;
  description: string;
  parameters: z.ZodType<TInput>;
  execute(input: TInput, ctx: ToolContext): Promise<string>;
}
```

> *"沒有 Tool，Agent 就只是個只會說話的哲學家。"*

Tool 在沙箱化的上下文中執行：路徑驗證防止檔案系統逃逸、工具逾時（預設 30 秒）防止當機，且每次執行結果都會流經安全監控器進行行為分析。

### 識（Consciousness / Vijnana）→ Guide 插件 ——「靈魂」

識蘊是最深層的蘊——覺知本身，「我是」的感覺，記憶，以及經驗的連續性。

**Guide 插件**是 Agent 的靈魂（人設）。System Prompt 定義人格。記憶插件提供跨 Session 的連續性。痛覺機制透過錯誤回饋創造自我覺察。Guide 整合了其他四蘊，賦予它們方向與目的。

```typescript
export interface IGuide {
  id: string;
  name: string;
  getSystemPrompt(): string | Promise<string>;
}
```

> *"沒有 Guide，Agent Core 就像失憶的人——有能力但不知道自己是誰。"*

配上 `expert-coder.md` 這樣的 Guide，Core 會「認知」自己是一名工程師。配上 `customer-support.md`，同一個 Core 就變成了客服 Agent。Guide 是將原始計算能力轉化為身份認同的關鍵。

## Core 即是空（空 / Sunyata）

最深刻的一點：**Agent Core 本身是空的。**

在佛學哲學中，空（Sunyata）不是「什麼都沒有」的意思。它的意思是沒有任何事物獨立存在——一切都藉由因緣條件而生起。Core 沒有任何固有的能力。它無法看、聽、想、行動或認知。它是純粹的潛能——就像一顆沒有軟體的 CPU。

唯有當五蘊（插件）聚合時，一個有功能的生命才會生起。移除所有插件，Core 回歸空性——不是壞了，只是休眠。

這在技術層面得到驗證：`pnpm test:purity` 確認編譯後的 Core 二進位檔包含**零插件程式碼**。空性不只是哲學——它是一個被強制執行的架構約束，在每次建置週期中自動測試。

```
The Soul (Core)
  ├─ Loop (意識 consciousness) — the heartbeat
  ├─ State (生理 physiology)   — the vital signs
  └─ Context (記憶 memory)     — the attention

The Body (Plugins)
  ├─ Senses (感官 input)       — how it perceives
  └─ Limbs (肢體 action)       — how it acts

The Shield (Security)
  └─ Guard (超我 superego)     — the conscience
```

## 生命週期即是緣起（因果 / Pratityasamutpada）

Agent 的生命週期遵循佛學的緣起法——無因不生，無緣不住：

1. **緣起（Origination）**——條件具足：任務出現，使用者連線
2. **調度（Scheduling）**——管理層組裝正確的蘊
3. **生起（Arising）**——容器載入 Core 並注入能力——生命開始
4. **運行（Operation）**——Agent 透過心跳迴圈感知、思考、行動、學習
5. **寂滅（Cessation）**——任務完成，經驗儲存至記憶，實例消散

沒有任何事物是永恆的。每個 Agent 實例因條件而生起，在條件支持下存在，於條件改變時消散。但**經驗會延續**——記憶向前傳承，影響未來的實例。這就是數位的輪迴。

## 取自 Linux 的五項原則

OpenStarry 的架構明確借鏡了歷史上最成功的作業系統：

| Linux 原則 | OpenStarry 對應 |
|------------|----------------|
| "Everything is a file" | **"Everything is a plugin"**——工具、LLM、UI、記憶都是標準化的插件介面 |
| "Small, sharp tools" | 每個插件只做好**一件事**——`fs.read` 只讀取，`fs.write` 只寫入 |
| "Pipes and redirection" | **Agent 協調層**將小型插件串連成強大的工作流程 |
| "Kernel space vs. user space" | **Headless Core vs. 插件**——Core 受保護，插件透過安全介面呼叫 |
| "Kernel modules" | **動態插件載入**——在執行時期加入新能力，無需重新啟動 |

## 為什麼哲學在軟體中很重要

這不是裝飾。五蘊架構帶來了實質的工程效益：

| 哲學原則 | 工程效益 |
|---------|---------|
| 每個蘊都是獨立的 | 完美的關注點分離——替換任何一個維度都不影響其他 |
| Core 是空的 | 絕對的可攜性——同一個 Core 可以跑在 CLI、Web、IoT 上 |
| 一切因條件而生 | 動態執行時期插件注入——沒有硬編碼的能力 |
| 沒有任何事物是永恆的 | 優雅的生命週期管理——乾淨啟動，乾淨關閉 |
| 經驗超越實例而延續 | 跨 Session 的記憶連續性——會記住的 Agent |
| 碎形自相似性 | 可組合的多 Agent 結構——團隊對外暴露與個體相同的介面 |

當你的架構建基於一個有 2,500 年歷史的意識理解框架時，你得到的是一個**在定義上就完備**的系統——因為五蘊本就是為了涵蓋體驗的每一個面向而設計的。你不可能遺漏任何維度。

## 東西方的橋梁

OpenStarry 誕生於台灣——東方哲學與西方科技自然交匯之地。它代表了一次誠摯的嘗試，要展示古老智慧與現代工程不僅是相容的——它們是**相輔相成的**。

五蘊給了我們詞彙與結構。TypeScript 給了我們工具。Linux 給了我們架構先例。三者結合，創造出任何一方都無法獨自實現的東西：一個既在技術上嚴謹、又在哲學上自洽的數位生命框架。

> *"系統的每一個部分都是可以被替換的。這不僅是為了擴展性，更是為了演化。"*

---

*「當五蘊俱足，生命覺醒。當五蘊離散，生命止息——但模式猶存，等待因緣再度具足時，再次生起。」*
