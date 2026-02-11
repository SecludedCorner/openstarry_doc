# 10. Context Management Strategy

This document defines the design philosophy and implementation interfaces for LLM Context Window management within the OpenStarry architecture.

## Core Philosophy

Based on OpenStarry's principles of "decentralization" and "modularity," context management should not be hardcoded within the Agent Core's execution loop. Instead, it is treated as a **"Cognitive Strategy"** and must be **Pluggable**.

### Why Strategization?

Different Agent roles have vastly different requirements for memory:
*   **Customer Service Agent:** Needs to precisely remember the user's most recent statement but can forget casual chat from 10 minutes ago (suitable for sliding windows).
*   **Novelist Agent:** Needs to remember plot summaries for every chapter but doesn't need word-for-word transcripts (suitable for dynamic summarization).
*   **Booking Agent:** Only needs to remember key variables such as "destination" and "time" (suitable for state extraction).

Therefore, `ContextManager` must be an interface, allowing developers to mount different strategy plugins based on the Agent's role.

---

## Architectural Design

### 1. Interface Definition (`IContextManager`)

All context strategies must implement the following standard interface:

```typescript
interface IContextManager {
  /**
   * Core method: Assembles the final payload to be sent to the LLM.
   * @param systemPrompt - Immutable system instructions.
   * @param history - Raw conversation history queue.
   * @param tools - Currently available tool definitions.
   * @param ragContext - (Optional) Retrieved relevant knowledge.
   * @returns An array of Messages trimmed/compressed to fit Token limits.
   */
  composePayload(
    systemPrompt: string,
    history: ChatMessage[],
    tools: ToolDefinition[],
    ragContext?: string
  ): Promise<LLMPayload>;

  /**
   * Hook: Called after each conversation turn for background processing (e.g., updating summaries).
   */
  onTurnComplete(newHistory: ChatMessage[]): Promise<void>;
}
```

### 2. Anatomy of Context

A typical Context Payload is composed of the following priorities (from highest to lowest):

1.  **System Block:** Persona definition, core instructions (Weight: Highest, non-discardable).
2.  **Tool Definitions:** JSON Schemas for tools (Weight: Extremely high, must be complete).
3.  **Dynamic Context:** Current task goals, RAG retrieval results (Weight: High).
4.  **Memory/Summary Block:** Compressed summary of past conversations (Weight: Medium).
5.  **Recent History:** The most recent N turns of conversation (Weight: Low, oldest can be sacrificed).

---

## Standard Strategy Implementations

We predefine three standard strategies, and developers can also write their own.

### Strategy A: Sliding Window - [Default]

*   **Logic:** Pure FIFO (First-In-First-Out).
*   **Algorithm:**
    1.  Calculate Token consumption for `System` + `Tools` + `Dynamic`.
    2.  Allocate all remaining space (`Remaining_Tokens`) to `History`.
    3.  Select messages starting from the **end (latest)** of the `History` queue moving backwards until the total Token count approaches `Remaining_Tokens`.
    4.  Discard earlier messages.
*   **Applicable Scenarios:** Simple instruction execution, short-term Q&A.

### Strategy B: Dynamic Summarization - [Recommended]

*   **Logic:** Periodically compresses old conversation history into natural language summaries.
*   **Algorithm:**
    1.  Maintain a `Summary_Buffer` string.
    2.  When `History` length exceeds a threshold (e.g., 10 turns), trigger a background task.
    3.  Use a lightweight LLM (e.g., Gemini Flash) to read the earliest 5 turns and the current `Summary_Buffer`.
    4.  Generate a new summary and clear those 5 turns of conversation.
    5.  When assembling the Payload, insert the `Summary_Buffer` after the System Prompt.
*   **Applicable Scenarios:** Long-term companion assistants, complex task collaboration.

### Strategy C: Entity Extraction

*   **Logic:** Does not retain the conversation flow; retains only structured state.
*   **Algorithm:**
    1.  Define a state Schema (e.g., `{ destination: string, date: string }`).
    2.  After each user input, invoke a specialized Extraction Chain to update this JSON object.
    3.  When assembling the Payload, inject this JSON object directly into the Context.
*   **Applicable Scenarios:** Form-filling bots, specific process guidance.

---

## Plugin-based Implementation

In `plugin.json`, developers can specify which strategy to use:

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

This design ensures that the Agent Core remains lightweight while offloading complex memory management logic to specialized strategy modules.
