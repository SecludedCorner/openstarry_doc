# 04. Context Management & Memory Strategy Technical Specification

This document defines how the OpenStarry core handles the limited Context Window of an Agent. To ensure stable long-running operation, the core must implement deterministic memory management algorithms.

## 1. Hierarchical Memory Architecture

OpenStarry adopts a three-tier memory model:

1.  **Immediate Buffer:**
    *   **Contents:** The most recent 5-10 conversation turns and the current System Prompt.
    *   **Strategy:** Absolute retention — no compression or discarding is permitted.
2.  **Short-Term Memory:**
    *   **Contents:** The most recent 10-50 conversation turns.
    *   **Strategy:** Uses a Sliding Window approach. When the token count exceeds the limit, the oldest messages are moved to the Long-Term Archive or discarded.
3.  **Long-Term Archive (External Storage):**
    *   **Contents:** Historical conversation summaries and extracted key facts.
    *   **Strategy:** Vector-based storage (RAG) or structured summaries. This specification only defines the interface; the concrete implementation is the responsibility of the Memory Plugin.

---

## 2. Sliding Window Truncation Algorithm

When `CurrentTokenCount > MaxTokenBudget`, the core must trigger garbage collection (GC).

### Algorithm Steps:

1.  **Lock:** Lock the System Prompt and the most recent `K` messages (Head & Tail).
2.  **Evaluate:** Calculate the token density of the intermediate message segment.
3.  **Remove:** Remove messages starting from the oldest until the budget is satisfied.
    *   **Critical Rule:** The pairing integrity of Tool Calls and Tool Results must be preserved. It is strictly forbidden to delete a `Call` while retaining its `Result`, as this would cause LLM logic corruption. If deletion is necessary, both must be deleted as a pair.

---

## 3. Memory Compression Protocol

To retain more information within a limited window, OpenStarry defines a standard compression format.

### 3.1 Summary Injection
When a conversation is truncated, a System Message must be inserted at the truncation point:

```json
{
  "role": "system",
  "content": "[Memory Summary] Previous conversation context: User discussed project X. Agent successfully read file Y but failed to write Z due to permission error."
}
```

### 3.2 Observation Reduction
For excessively long tool outputs (e.g., `cat huge_file.log`), the core should automatically truncate the middle portion and mark it accordingly:

```text
[Data Truncated]
Start: Line 1-50 content...
... (8000 lines omitted) ...
End: Line 8050-8100 content...
```

---

## 4. State Snapshot

To support hot restarts, the memory state must be serializable.

```typescript
interface IContextSnapshot {
  version: "1.0";
  timestamp: number;
  tokenCount: number;
  messages: IMessage[]; // Complete message queue
  summary?: string;     // Current conversation summary
}
```

The core is responsible for periodically writing this snapshot to disk (`.openstarry/snapshots/`), ensuring that the Agent can "recall" previous context after a restart.
