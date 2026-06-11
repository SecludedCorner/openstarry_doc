# 04. 上下文管理與記憶策略技術規範 (Context Management & Memory Strategy)

本文件定義了 OpenStarry 核心如何處理 Agent 的有限上下文窗口 (Context Window)。為了確保長期運行不崩潰，核心必須具備確定性的記憶管理算法。

## 1. 記憶分層架構 (Hierarchical Memory)

OpenStarry 採用三級記憶模型：

1.  **Immediate Buffer (即時區):** 
    *   **內容:** 最新 5-10 輪對話與當前 System Prompt。
    *   **策略:** 絕對保留，不允許任何壓縮或丟棄。
2.  **Short-Term Memory (短期區):** 
    *   **內容:** 最近 10-50 輪對話。
    *   **策略:** 採用滑動窗口 (Sliding Window)。當 Token 超限時，最舊的訊息移入長期區或丟棄。
3.  **Long-Term Archive (長期區/外存):**
    *   **內容:** 歷史對話摘要、重要事實提取。
    *   **策略:** 向量化存儲 (RAG) 或結構化摘要。本規格書僅定義介面，具體實作由 Memory Plugin 負責。

---

## 2. 滑動窗口截斷算法 (Sliding Window Truncation)

當 `CurrentTokenCount > MaxTokenBudget` 時，核心必須觸發垃圾回收 (GC)。

### 算法步驟：

1.  **鎖定:** 鎖定 System Prompt 與最新的 `K` 條消息 (Head & Tail)。
2.  **評估:** 計算中間段消息的 Token 密度。
3.  **移除:** 從最舊的消息開始移除，直到滿足預算。
    *   **重要規則:** 必須保持 Tool Call 與 Tool Result 的配對完整性。嚴禁只刪除 `Call` 而保留 `Result`，這會導致 LLM 邏輯錯亂。若需刪除，必須成對刪除。

---

## 3. 記憶壓縮協議 (Memory Compression Protocol)

為了在有限窗口內保留更多信息，OpenStarry 定義了標準壓縮格式。

### 3.1 摘要注入 (Summary Injection)
當對話被截斷時，必須在截斷點插入一條 System Message：

```json
{
  "role": "system",
  "content": "[Memory Summary] Previous conversation context: User discussed project X. Agent successfully read file Y but failed to write Z due to permission error."
}
```

### 3.2 觀察結果縮減 (Observation Reduction)
對於超長的工具輸出（如 `cat huge_file.log`），核心應自動截斷中間部分並標記：

```text
[Data Truncated]
Start: Line 1-50 content...
... (8000 lines omitted) ...
End: Line 8050-8100 content...
```

---

## 4. 狀態快照 (State Snapshot)

為了支持熱重啟，記憶狀態必須可序列化。

```typescript
interface IContextSnapshot {
  version: "1.0";
  timestamp: number;
  tokenCount: number;
  messages: IMessage[]; // 完整的消息隊列
  summary?: string;     // 當前的對話摘要
}
```

核心負責定期將此快照寫入磁盤 (`.openstarry/snapshots/`), 以確保 Agent 重啟後能「想起」之前的事情。
