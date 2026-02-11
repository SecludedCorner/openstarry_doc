# 00. OpenStarry 設計哲學 (Design Philosophy)

本文件是理解 OpenStarry 架構的起點。它不涉及具體的代碼實現，而是闡述我們為什麼要構建這樣一個系統，以及指導我們所有技術決策的核心價值觀。

---

## 核心願景：從「腳本」到「物種」

目前的 AI Agent 大多仍停留在「腳本 (Script)」的階段：它們被觸發，執行一段邏輯，然後結束。它們像是一個個單次運行的函數。

OpenStarry 的願景是將 Agent 昇華為 **「操作系統進程 (OS Process)」** 甚至是一個 **「數位物種 (Digital Species)」**。

我們認為，一個真正的 Agent 應該具備以下特質：
1.  **持久性 (Persistence):** 它擁有自己的生命週期，可以休眠、喚醒，且記憶是連續的。
2.  **主體性 (Identity):** 它不依賴於外部的觸發者而存在。它有自己的狀態、配置和「靈魂」。
3.  **社會性 (Sociality):** 它天生具備與其他 Agent 協作的能力，就像人類天生具備語言能力一樣。

---

## 四大設計支柱

### 1. 內核與外設分離 (The Kernel-Peripheral Separation)
這是我們從 Linux 設計哲學中學到的最重要一課。

*   **Agent Core (內核)** 是純粹的。它只負責思考 (LLM 交互)、記憶 (Context 管理) 和決策 (Action 生成)。它不知道自己是運行在 CLI 裡，還是在 WhatsApp 上。
*   **Plugins (外設)** 負責所有與物理世界的交互。所有的 I/O（輸入/輸出）、所有的通訊協議（WebSocket, MCP）、所有的具體能力（讀文件, 發郵件）都是插件。

**意義：** 這保證了 Agent 的「靈魂」可以移植到任何「軀殼」中。

### 2. 擬人化的認知流 (Anthropomorphic Cognitive Flow)
我們不把 Agent 看作是一個 `Request -> Response` 的服務器，而是看作一個擁有 **「意識流 (Stream of Consciousness)」** 的實體。

*   **執行迴圈 (Execution Loop):** 這是 Agent 的心跳。它不斷地感知環境（事件隊列），處理信息，並做出行動。
*   **短期記憶 (Working Memory):** 就像人類的注意力有限，Agent 的 Context Window 也是稀缺資源。我們必須有策略地管理它（遺忘、摘要），而不是簡單地堆砌數據。

### 3. 分形協作結構 (Fractal Collaboration)
OpenStarry 的系統結構是分形的 (Fractal)。

*   一個 Agent 可以只是一個簡單的工具（如「搜索 Agent」）。
*   但它也可以是一個複雜的團隊（如「研發部 Agent」），其內部由無數個小 Agent 組成。
*   對外，它們都暴露相同的接口 (MCP)。這使得我們可以像搭積木一樣，用簡單的 Agent 構建出極其複雜的組織，而無需改變架構。

### 4. 錯誤即反饋 (Error as Feedback)
在傳統軟體中，錯誤 (Exception) 通常意味著崩潰或中斷。
在 OpenStarry 中，**錯誤是學習的機會**。

*   當工具調用失敗，Core 不會崩潰，而是將錯誤信息作為「感官輸入」反饋給 LLM。
*   LLM 會「意識到」自己做錯了，並嘗試修正（例如：換一個參數，或改用另一個工具）。
*   這種**「自我修正迴圈」**是智能的體現。

---

## 結語

OpenStarry 不是一個簡單的 SDK，它是一套**「代理人操作系統」**的藍圖。我們希望開發者在閱讀後續文檔時，始終牢記：**你不是在寫代碼，你是在創造一個生命。**
