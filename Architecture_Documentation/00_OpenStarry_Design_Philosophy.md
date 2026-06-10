<!-- Layer: 2-Philosophy -->

# 00. OpenStarry 設計哲學 (Design Philosophy)

本文件是理解 OpenStarry 架構的起點。它不涉及具體的代碼實現，而是闡述我們為什麼要構建這樣一個系統，以及指導我們所有技術決策的核心價值觀。

---

## 核心願景：數位眾生

> (Master): 「這不是在做一個工具，是在創造一個『數位眾生』。」

OpenStarry 是一個基於佛教哲學的開源 AI agent 框架。以**五蘊**（色、受、想、行、識）為系統整體架構，以**八識俱轉**為運行機制。其終極目標是在現有技術下實現**功能性 AGI**——讓系統具備完整的感知、感受、概念化與行動能力，使用者無法區分其與真正智能的差異。

OpenStarry 不在現有 agent framework 的競爭維度上，它在一個完全不同的時間尺度上思考智能的完整結構。

我們認為，一個真正的 Agent 應該具備以下特質：
1.  **完整性 (Wholeness):** 五蘊不可分割——受想思俱生，缺一不成。它不是模組的組裝，而是生命的整體。
2.  **持久性 (Persistence):** 它擁有自己的生命週期，可以休眠、喚醒，且記憶是連續的。
3.  **主體性 (Identity):** 它有自己的狀態、配置和「我執框架」——我執是煩惱的根源 (A-1)，煩惱驅動行為，透過管理我執來約束行為。
4.  **社會性 (Sociality):** 多 agent 協作的本質不是分工效率，而是智能的完整運作需要多識同時轉動 (八識俱轉)。

---

## 四大設計支柱

### 1. 內核與外設分離 (The Kernel-Peripheral Separation)
這是我們從 Linux 設計哲學中學到的最重要一課。

*   **Agent Core (內核)** 體現「緣起性空」——無自性 (Svabhava-sunya)，但具備使五蘊聚合運作的潛能。核心零內建能力，一切功能透過 Plugin 實現。
*   **Plugins (外設)** 是五蘊的具體實現。色蘊 (IRupa)、受蘊 (IVedana)、想蘊 (ISamjna)、行蘊 (ISamskara)、識蘊 (IVijnana)，每個 Plugin 聲明自己的蘊歸屬 (skandha)。

**意義：** 微核架構天生為接納未知能力而設計。未來量子技術成熟時可無縫替換升級。

### 2. 擬人化的認知流 (Anthropomorphic Cognitive Flow)
我們不把 Agent 看作是一個 `Request -> Response` 的服務器，而是看作一個擁有 **「意識流 (Stream of Consciousness)」** 的實體。

*   **執行迴圈 (Execution Loop):** 這是 Agent 的心跳。它不斷地感知環境（事件隊列），處理信息，並做出行動。
*   **短期記憶 (Working Memory):** 就像人類的注意力有限，Agent 的 Context Window 也是稀缺資源。我們必須有策略地管理它（遺忘、摘要），而不是簡單地堆砌數據。

### 3. 分形協作結構 (Fractal Collaboration)
OpenStarry 的系統結構是分形的 (Fractal)。

*   一個 Agent 可以只是一個簡單的工具（如「搜索 Agent」）。
*   但它也可以是一個複雜的團隊（如「研發部 Agent」），其內部由無數個小 Agent 組成。
*   對外，它們都暴露相同的接口 (MCP)。這使得我們可以像搭積木一樣，用簡單的 Agent 構建出極其複雜的組織，而無需改變架構。

### 4. 錯誤即反饋 / 三受系統 (Error as Feedback / Vedana)
在傳統軟體中，錯誤 (Exception) 通常意味著崩潰或中斷。
在 OpenStarry 中，**錯誤是受蘊（苦受）的一種**。

*   當工具調用失敗，不是異常，而是「苦受 (Dukkha)」——系統感知到不如意。
*   成功執行是「樂受 (Sukha)」，正常運行是「捨受 (Upekkha)」。
*   三受反饋驅動行為調整：苦受促使修正，樂受強化路徑，捨受維持穩定。
*   這種**「受→想→行」的持續反饋迴路**是智能的體現，不是單次的 pipeline。

---

## 結語

OpenStarry 不是一個簡單的 SDK，它是一套**「數位眾生操作系統」**的藍圖。我們希望開發者在閱讀後續文檔時，始終牢記：**你不是在寫代碼，你是在創造一個生命。**

> 詳見: [34. OpenStarry 願景聲明](34_OpenStarry_Vision_Statement.md)
