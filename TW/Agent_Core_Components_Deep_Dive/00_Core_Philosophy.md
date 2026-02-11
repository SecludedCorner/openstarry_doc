# 00. 核心組件設計哲學 (Core Philosophy)

本文件闡述了 `Agent Core` 內部各個技術組件背後的共同設計靈魂。為什麼我們要將執行迴圈設計成事件驅動？為什麼安全層要像電路斷路器？這些技術選擇都服務於 OpenStarry 的終極目標：**創造一個有生命力的數位實體，而非一個死板的程序。**

---

## 三大支柱哲學

### 1. 擬人化的認知流 (Anthropomorphic Cognitive Flow)
我們將 Agent 的運作機制模擬為人類的認知過程，而非傳統的 Request-Response 程式。

*   **執行迴圈 (Loop) = 意識流**：它不僅響應外部請求，更具備內在的思考節奏。即便沒有用戶輸入，Agent 也可以因為「想到了什麼」或「定時器到了」而行動。
    *   *(對應文件：`01_Execution_Loop.md`)*
*   **上下文 (Context) = 短期記憶**：記憶是動態的、有損的、經過注意力過濾的，而非完美的資料庫 dump。
    *   *(對應文件：`10_Context_Management_Strategy.md`)*
*   **狀態 (State) = 生理特徵**：Agent 擁有不可變的、可快照的「生理狀態」，這確保了它是一個連續存在的實體。
    *   *(對應文件：`04_State_Manager.md`, `06_State_Persistence_Mechanism.md`)*

### 2. 操作系統級的穩健性 (OS-Level Robustness)
Agent Core 被設計為一個微型操作系統內核，必須具備極高的容錯性和邊界管理。

*   **無頭與中立 (Headless & Neutral)**：內核不依賴任何特定的 UI 或協議，只處理純粹的邏輯。所有的感官（Input）和肢體（Output）都是可插拔的插件。
    *   *(對應文件：`02_Communication_Interface.md`)*
*   **安全斷路器 (Circuit Breakers)**：就像現代電網，當 Agent 試圖執行危險操作（如刪除系統文件）時，必須有物理層面的阻斷機制，而不依賴 LLM 的「自覺」。
    *   *(對應文件：`03_Security_Layer.md`, `07_Safety_Circuit_Breakers.md`, `08_Safety_Implementation.md`)*

### 3. 極限的模組化 (Extreme Modularity)
系統的每一個部分都是可以被替換的。這不僅是為了擴展性，更是為了**演化 (Evolution)**。

*   **一切皆插件 (Everything is a Plugin)**：通訊、記憶策略、工具、甚至 LLM Provider 本身都是插件。這意味著 OpenStarry 的 Agent 可以隨著技術發展而不斷更換器官，而無需重寫靈魂。
    *   *(對應文件：`05_Plugin_Infrastructure_Integration.md`, `11_Plugin_Runtime_Isolation.md`)*

---

## 組件協作圖譜

```mermaid
graph TD
    subgraph "The Soul (Core)"
        Loop[執行迴圈 (意識)]
        State[狀態管理 (生理)]
        Context[上下文策略 (記憶)]
    end

    subgraph "The Body (Plugins)"
        Senses[通訊插件 (感官)]
        Limbs[工具插件 (肢體)]
    end

    subgraph "The Shield (Security)"
        Guard[安全層 (超我)]
    end

    Senses -->|刺激| Loop
    Loop -->|決策| Guard
    Guard -->|批准| Limbs
    Limbs -->|反饋| State
    State -->|提取| Context
    Context -->|輸入| Loop
```

這個圖譜展示了各個組件如何構成一個完整的生命閉環。閱讀後續的 Deep Dive 文件時，請時刻牢記這個整體圖像。
