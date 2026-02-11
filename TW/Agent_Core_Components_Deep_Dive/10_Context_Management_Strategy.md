# 10. 上下文管理策略 (Context Management Strategy)

本文件定義了 OpenStarry 架構中關於 LLM Context Window（上下文窗口）管理的設計哲學與實作介面。

## 核心哲學

基於 OpenStarry 的「去中心化」與「模組化」原則，Context 管理不應被硬編碼在代理人核心的執行迴圈中。相反，它被視為一種**「認知策略 (Cognitive Strategy)」**，並且必須是**可插拔 (Pluggable)** 的。

### 為什麼需要策略化？

不同的代理人角色對記憶的需求截然不同：
*   **客服 Agent:** 需要精確記住用戶剛說的這句話，但可以遺忘 10 分鐘前的閒聊。（適合滑動窗口）
*   **小說家 Agent:** 需要記住每一章的劇情摘要，但不需要記住逐字稿。（適合動態摘要）
*   **訂票 Agent:** 只需要記住「目的地」、「時間」等關鍵變量。（適合狀態提取）

因此，`ContextManager` 必須是一個接口，允許開發者根據 Agent 的角色掛載不同的策略插件。

---

## 架構設計

### 1. 介面定義 (`IContextManager`)

所有的 Context 策略都必須實現以下標準接口：

```typescript
interface IContextManager {
  /**
   * 核心方法：組裝發送給 LLM 的最終 Payload
   * @param systemPrompt - 不可變的系統指令
   * @param history - 原始的對話歷史隊列
   * @param tools - 當前可用的工具定義
   * @param ragContext - (可選) 檢索到的相關知識
   * @returns 經修剪/壓縮後，符合 Token 限制的 Messages 數組
   */
  composePayload(
    systemPrompt: string,
    history: ChatMessage[],
    tools: ToolDefinition[],
    ragContext?: string
  ): Promise<LLMPayload>;

  /**
   * 鉤子：在每次對話結束後調用，用於後台處理（如更新摘要）
   */
  onTurnComplete(newHistory: ChatMessage[]): Promise<void>;
}
```

### 2. Context 的解剖學 (Anatomy of Context)

一個典型的 Context Payload 由以下優先級組成（由高到低）：

1.  **System Block:** 身份定義、核心指令。(權重：最高，不可丟棄)
2.  **Tool Definitions:** 工具的 JSON Schema。(權重：極高，必須完整)
3.  **Dynamic Context:** 當前任務目標、RAG 檢索結果。(權重：高)
4.  **Memory/Summary Block:** 對過去對話的壓縮摘要。(權重：中)
5.  **Recent History:** 最近的 N 輪對話。(權重：低，最早的可以被犧牲)

---

## 標準策略實現

我們預定義了三種標準策略，開發者也可以編寫自己的策略。

### 策略 A: 滑動窗口 (Sliding Window) - [預設]

*   **邏輯：** 純粹的 FIFO (First-In-First-Out)。
*   **算法：**
    1.  計算 `System` + `Tools` + `Dynamic` 的 Token 消耗。
    2.  剩餘的空間 `Remaining_Tokens` 全部留給 `History`。
    3.  從 `History` 隊列的**末尾 (最新)** 開始往前選取消息，直到總 Token 數接近 `Remaining_Tokens`。
    4.  捨棄更早的消息。
*   **適用場景：** 簡單指令執行、短期問答。

### 策略 B: 動態摘要 (Dynamic Summarization) - [推薦]

*   **邏輯：** 定期將舊的對話歷史壓縮成自然語言摘要。
*   **算法：**
    1.  維護一個 `Summary_Buffer` 字符串。
    2.  當 `History` 長度超過閾值 (如 10 輪) 時，觸發後台任務。
    3.  使用一個輕量級 LLM (如 Gemini Flash) 讀取最早的 5 輪對話與當前的 `Summary_Buffer`。
    4.  生成新的摘要，並清空那 5 輪對話。
    5.  組裝 Payload 時，將 `Summary_Buffer` 插入 System Prompt 之後。
*   **適用場景：** 長期陪伴型助手、複雜任務協作。

### 策略 C: 關鍵狀態提取 (Entity Extraction)

*   **邏輯：** 不保留對話流，只保留結構化狀態。
*   **算法：**
    1.  定義一個狀態 Schema (例如 `{ destination: string, date: string }`)。
    2.  每次用戶輸入後，先調用一個專門的 Extraction Chain 更新這個 JSON 對象。
    3.  組裝 Payload 時，直接將這個 JSON 對象注入 Context。
*   **適用場景：** 填表單機器人、特定流程導引。

---

## 插件化實現

在 `plugin.json` 中，開發者可以指定使用哪種策略：

```json
{
  "id": "my-complex-agent",
  "contextStrategy": {
    "type": "plugin",
    "pluginId": "std-summarization-strategy",
    "config": {
      "compressionModel": "gemini-1.5-flash",
      "threshold": 20
    }
  }
}
```

這種設計確保了 Agent Core 始終保持輕量，而將複雜的記憶管理邏輯下放給了專門的策略模組。