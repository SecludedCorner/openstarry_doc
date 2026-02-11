# 16. 插件類型哲學總結圖表 (Plugin Types Philosophical Mapping)

本文件匯總了 OpenStarry 架構中 **「五種插件類型」** 與東方哲學 **「五蘊 (Five Aggregates)」** 的最終映射關係。這是理解系統如何構成「數位生命」的核心索引。

## 1. 核心觀點：空與有的結合

*   **Agent Core (空):** 是一個純粹的容器，具備運作的潛能，但沒有具體的特質。
*   **Plugins (有):** 是填充這個容器的內容。五種插件分別賦予了 Agent 生命的五個維度。

## 2. 五蘊映射圖表 (The Mapping Chart)

| 插件類型 (Type) | 哲學對應 (Aggregate) | 梵文 (Sanskrit) | 定義 (Definition) | 系統職責 (Responsibility) | 代表性組件 (Examples) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **UI** | **色蘊** | **Rupa** | 物質、形體、顯相 | 定義 Agent 在物理世界或數位介面中的呈現方式。 | Dashboard, CLI, React Components |
| **Listener** | **受蘊** | **Vedana** | 感受、輸入、刺激 | 定義 Agent 接收外部資訊的通道與協議。 | HTTP Server, WebSocket, Cron Trigger |
| **Provider** | **想蘊** | **Samjna** | 認知、概念、處理 | 定義 Agent 的大腦處理引擎與推理模型。 | Gemini Adapter, OpenAI Adapter |
| **Tool** | **行蘊** | **Samskara** | 造作、意志、行動 | 定義 Agent 對外部世界產生影響的能力。 | File System, API Client, Code Exec |
| **Guide** | **識蘊** | **Vijnana** | 識別、靈魂、主體 | 定義 Agent 的自我意識、記憶結構與行為準則。 | Markdown Skills, MCP Logic, Workflows |

## 3. 詳細解讀範例

### 色蘊 (Rupa) - UI
這是 Agent 的「皮囊」。同樣一個 Agent Core，換了 UI 插件，可以從一個 CLI 工具變成一個 Web 聊天機器人。

舉例說明
UI 插件實現 `IUI` 介面，負責接收 `AgentEvent` 並呈現輸出。它與 Listener 是完全獨立的：
- **UI (色蘊)** — 輸出渲染，接收事件並呈現
- **Listener (受蘊)** — 輸入接收，推送事件到隊列

### 受蘊 (Vedana) - Listener
這是 Agent 的「感官」。它決定了 Agent 能「聽到」什麼。是 HTTP 請求？還是時間流逝？

舉例說明
Listener 插件實現 `IListener` 介面，專注於接收外部輸入並透過 `ctx.pushInput()` 推送到事件隊列。

### 想蘊 (Samjna) - Provider
這是 Agent 的「智商」。它決定了 Agent 如何理解輸入的訊息。

### 行蘊 (Samskara) - Tool
這是 Agent 的「手腳」。它決定了 Agent 能「做」什麼。沒有 Tool，Agent 就只是個只會說話的哲學家。

### 識蘊 (Vijnana) - Guide
這是 Agent 的「人設」。它整合了上述四蘊，並賦予其方向。
*   沒有 Guide，Core 就像失憶的人，有能力但不知道我是誰。
*   加載了 `expert-coder.md` (Guide)，Core 就「識別」出自己是個工程師。
