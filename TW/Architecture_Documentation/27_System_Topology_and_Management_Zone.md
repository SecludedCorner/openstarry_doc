# 27. 系統拓樸與管理層架構 (System Topology & Management Zone)

本文件定義了 OpenStarry 的宏觀運作邏輯：它是一個由「管理層」提供土壤與養分，並讓「核心層」透過「插件」實現物種多樣性的現代 AI 作業系統。

---

## 1. Agent 協調層 (The Management Zone)

**定位：系統的宿主環境（Host）與行政中樞。**
這一層不負責「思考」，它負責「確保思考環境的穩定與安全」。

### 1.1 容器層 (Plumbing)
*   **機制**：採用動態連結或容器化技術（如 WebAssembly 沙盒或特定進程隔離）。
*   **職責**：實作 `AgentLoader`。當系統需要一個「開發者」時，容器層負責從倉庫抓取 `AgentCore` 鏡像，並將指定的 `Plugins` 透過 **依賴注入 (DI)** 掛載進去。

### 1.2 規劃與調度層 (Orchestration)
*   **機制**：基於 **因果鏈 (Causality Chain)** 的事件驅動架構。
*   **職責**：它不監控 Agent 的私有隱私，它監控 **「邊界事件」**。當 A Agent 拋出一個 `TaskCompleted` 事件，調度層負責判斷這是否為喚醒 B Agent 的「緣」。

### 1.3 策略與安全層 (Policy)
*   **機制**：攔截器 (Interceptors) 與 資源配額管理 (Quota Management)。
*   **職責**：執行「戒律」。例如限制測試 Agent 的網路存取權限，或強制限制開發 Agent 的 Token 使用預算。

### 1.4 資源與環境層 (Environment)
*   **機制**：硬體抽象層 (HAL)。
*   **職責**：將物理實體（如 PiKVM 的畫面、NVIDIA Jetson 的溫度）轉換為標準的 **感官數據流** 供應給 Agent。

### 1.5 多元交互層 (Interface)
*   **機制**：狀態投影 (State Projection)。
*   **職責**：將看不見的 Agent 思考過程（來自協議插件的導出數據）轉化為人類可讀的儀表板。

---

## 2. Agent Core (The Autonomous Life Zone)

**定位：純粹的「五蘊」計算循環。**
它是空的，它唯一的職責是**維持循環的運行**。

### 五蘊能力層 (Aggregate)
*   **受 (Input)**：接收來自環境層的訊號。
*   **想 (Reasoning)**：呼叫大模型進行語義解析。
*   **行 (Action)**：產生操作工具的意圖。
*   **識 (Integrator)**：整合所有插件的資訊，維持自我的連貫性。

**核心特性**：它是 **Stateless (無狀態)** 的。所有的狀態都寄存在「記憶插件」中，所有的通訊規範都依賴「協議插件」。這意味著 Core 可以在不同的容器間無損遷移。

---

## 3. 能力插件 (The Plugins)

**定位：賦予 Agent 個性、專業與靈魂的功能組件。**

### 3.1 數據與協議插件 (Internal Protocol Plugin)
*   **工程細節**：定義內部的訊息傳遞總線（Internal Bus）。
*   **場景應用**：
    *   **高速模式**：開發人員 Agent 使用 Protobuf 協議，追求極速的代碼生成與傳輸。
    *   **透明模式**：測試人員 Agent 使用「日誌追蹤協議」，記錄下每一秒的神經傳導，以便在出錯時進行因果回溯。

### 3.2 評估與進化插件 (Reflection Plugin)
*   **工程細節**：實作「雙層推理」機制。在 Action 輸出前，由評估插件進行攔截（Self-check）。
*   **場景應用**：
    *   **輕量化**：簡單的文檔整理 Agent 可以不掛載此插件，降低延遲。
    *   **深度決策**：涉及工廠設備操作的 Agent，必須掛載具備「物理規則驗證」的反思插件，避免執行危險動作。

### 3.3 狀態與記憶插件 (Memory Plugin)
*   **工程細節**：抽象化儲存介面 `IMemoryStore`。
*   **場景應用**：
    *   **短期戰場**：使用 `In-Memory` 插件，讓開發人員在撰寫當前 Function 時擁有極快的 Context 切換。
    *   **長期經驗**：掛載 `Vector-RAG` 插件，讓 Agent 記得三個月前解決類似 Bug 的經驗。

---

## 4. OpenStarry 架構的終極運作流程 (The Lifecycle)

這是一個從「緣起」到「寂滅」的完整生命週期：

1.  **緣起 (Origination)**：環境層偵測到硬體錯誤。
2.  **調度 (Scheduling)**：調度層判斷需要「維修專家」，通知容器層。
3.  **生起 (Arising)**：容器層載入 **Agent Core**，並插上 **診斷協議**、**高階反思** 與 **設備知識庫記憶** 三個插件。
4.  **運行 (Operation)**：Agent Core 在插件的加持下，開始處理「痛覺」，產生修復指令。
5.  **寂滅 (Cessation)**：任務完成，容器層銷毀 Core 實例，記憶插件將經驗存回資料庫。
