# 04. 上下文管理与记忆策略技术规范 (Context Management & Memory Strategy)

本文件定义了 OpenStarry 核心如何处理 Agent 的有限上下文窗口 (Context Window)。为了确保长期运行不崩溃，核心必须具备确定性的记忆管理算法。

## 1. 记忆分层架构 (Hierarchical Memory)

OpenStarry 采用三级记忆模型：

1.  **Immediate Buffer (即时区):**
    *   **内容:** 最新 5-10 轮对话与当前 System Prompt。
    *   **策略:** 绝对保留，不允许任何压缩或丢弃。
2.  **Short-Term Memory (短期区):**
    *   **内容:** 最近 10-50 轮对话。
    *   **策略:** 采用滑动窗口 (Sliding Window)。当 Token 超限时，最旧的消息移入长期区或丢弃。
3.  **Long-Term Archive (长期区/外存):**
    *   **内容:** 历史对话摘要、重要事实提取。
    *   **策略:** 向量化存储 (RAG) 或结构化摘要。本规格书仅定义接口，具体实现由 Memory Plugin 负责。

---

## 2. 滑动窗口截断算法 (Sliding Window Truncation)

当 `CurrentTokenCount > MaxTokenBudget` 时，核心必须触发垃圾回收 (GC)。

### 算法步骤：

1.  **锁定:** 锁定 System Prompt 与最新的 `K` 条消息 (Head & Tail)。
2.  **评估:** 计算中间段消息的 Token 密度。
3.  **移除:** 从最旧的消息开始移除，直到满足预算。
    *   **重要规则:** 必须保持 Tool Call 与 Tool Result 的配对完整性。严禁只删除 `Call` 而保留 `Result`，这会导致 LLM 逻辑错乱。若需删除，必须成对删除。

---

## 3. 记忆压缩协议 (Memory Compression Protocol)

为了在有限窗口内保留更多信息，OpenStarry 定义了标准压缩格式。

### 3.1 摘要注入 (Summary Injection)
当对话被截断时，必须在截断点插入一条 System Message：

```json
{
  "role": "system",
  "content": "[Memory Summary] Previous conversation context: User discussed project X. Agent successfully read file Y but failed to write Z due to permission error."
}
```

### 3.2 观察结果缩减 (Observation Reduction)
对于超长的工具输出（如 `cat huge_file.log`），核心应自动截断中间部分并标记：

```text
[Data Truncated]
Start: Line 1-50 content...
... (8000 lines omitted) ...
End: Line 8050-8100 content...
```

---

## 4. 状态快照 (State Snapshot)

为了支持热重启，记忆状态必须可序列化。

```typescript
interface IContextSnapshot {
  version: "1.0";
  timestamp: number;
  tokenCount: number;
  messages: IMessage[]; // 完整的消息队列
  summary?: string;     // 当前的对话摘要
}
```

核心负责定期将此快照写入磁盘 (`.openstarry/snapshots/`), 以确保 Agent 重启后能「想起」之前的事情。
