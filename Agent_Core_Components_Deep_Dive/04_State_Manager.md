# 深度解析：狀態管理器

本文件深入探討「無頭代理人核心」的記憶中樞——狀態管理器的數據結構與職責。

## 核心職責

狀態管理器負責維護**單次對話會話**的完整上下文，確保對話的連貫性。它主要管理「短期記憶」（當前正在處理的思考鏈）和「中期記憶」（本次會話的完整歷史）。它**不負責**「長期記憶」。

---

## 數據結構深度解析

狀態管理器的核心是一個線性的消息列表 (Message List)。為了準確地追蹤工具調用與其結果之間的關係，數據結構需要更為精細。

```typescript
// 定義消息的類型
type MessageRole = 'user' | 'assistant' | 'tool';

// 助理發起的工具調用請求
interface ToolCallRequest {
  id: string;      // 每次工具調用的唯一 ID，由核心生成
  name: string;    // 工具名稱
  args: object;    // 工具參數
}

// 工具執行的結果
interface ToolCallResult {
  tool_call_id: string; // 對應 ToolCallRequest 的 id
  result: any;          // 工具執行的返回結果
  is_error: boolean;    // 標記結果是否為一個錯誤
}

// 統一的消息對象
interface Message {
  id: string;           // 消息的唯一 ID
  timestamp: string;    // 消息創建時間
  role: MessageRole;

  // 當 role 為 'user' 或 'assistant' (最終答覆) 時
  content?: string | null;

  // 當 role 為 'assistant' 且發起工具調用時
  tool_calls?: ToolCallRequest[];

  // 當 role 為 'tool' 時
  tool_results?: ToolCallResult[]; 
}
```
**關鍵設計：** 通過 `tool_calls` 列表中的 `id` 和 `tool_results` 列表中的 `tool_call_id`，我們可以清晰地將一個或多個工具調用請求與其對應的結果關聯起來，即使在多工具並行調用的情況下也能保持上下文的準確性。

---

## 關鍵實現細節

### 1. 記憶截斷策略 (Truncation Strategy)
為了防止上下文超過 LLM 的 Token 限制，狀態管理器必須實現記憶截斷策略。
*   **Token 計數器:** 在添加任何新消息之前，實時計算當前消息列表的總 Token 數。
*   **滑動窗口 (Sliding Window):** 當 Token 超限時，從列表的**開頭**（即最舊的消息，但通常會跳過第一條系統提示）移除一條或多條消息。
*   **對話總結 (Summarization):** 一種更高級的策略。當歷史記錄過長時，可以異步地調用一個工具或專門的 LLM，將較早的幾輪對話進行總結。然後，用一條包含 `role: 'system'` 和 `content: '前文摘要：...'` 的總結性消息，替換掉被總結的多條原始消息。

### 2. 與長期記憶引擎的交互
狀態管理器不存儲長期記憶，但它是長期記憶的**來源**。
*   **數據上報/卸載 (Data Offloading):**
    *   在一次對話會話結束時 (`/reset` 或超時)，狀態管理器可以將本次會話的完整歷史記錄發送給外部的「記憶與 RAG 引擎」。
    *   「記憶與 RAG 引擎」接收到這些原始對話後，會在後台進行異步處理，提取關鍵實體、用戶偏好、生成新的知識，並存儲到其知識庫中。
*   **引導上下文 (Bootstrapping Context):** 在一次新對話會話開始時，「代理人核心」可以先向「記憶與 RAG 引擎」查詢關於當前用戶的關鍵信息摘要，並將其作為初始的系統提示的一部分，預先填入狀態管理器。

### 3. 會話管理 (Session Management)
在實際應用中，系統需要同時處理多個用戶的多個會話。
*   狀態管理器本身應該是**無狀態**的，它的所有數據（消息列表）都來自於傳入的參數。
*   在「代理人核心」之上，應該有一個**「會話管理器」 (Session Manager)**，負責：
    1.  創建和銷毀會話。
    2.  維護一個從 `session_id` 到 `Message[]` (消息列表) 的映射。
    3.  每次調用「執行循環」時，將對應 `session_id` 的消息列表傳遞給狀態管理器進行操作。
    4.  可以將這個映射存儲在內存 (如 Redis) 或數據庫中，以支持無狀態的水平擴展。
