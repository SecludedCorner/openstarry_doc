# Implementation Example: Basic Sliding Window Strategy

This is the earliest and most fundamental implementation of context management in OpenStarry. It follows a simple FIFO (First-In-First-Out) principle, suitable for simple tasks that do not require long memory retention.

## Implementation Logic

This strategy does not perform any text compression; it simply truncates based on Token limits.

### 1. Core Code (Node.js Example)

```javascript
class SlidingWindowStrategy {
  constructor(config) {
    this.maxTokens = config.maxTokens || 4096;
    this.reservedTokens = config.reservedTokens || 1000; // Reserved for System Prompt and Tools
  }

  async composePayload(systemPrompt, history, tools) {
    let currentTokens = this.estimateTokens(systemPrompt + JSON.stringify(tools));
    const availableForHistory = this.maxTokens - currentTokens - this.reservedTokens;

    let selectedHistory = [];
    let historyTokens = 0;

    // Select messages starting from the most recent
    for (let i = history.length - 1; i >= 0; i--) {
      const msg = history[i];
      const msgTokens = this.estimateTokens(msg.content);

      if (historyTokens + msgTokens <= availableForHistory) {
        selectedHistory.unshift(msg); // Insert maintaining original order
        historyTokens += msgTokens;
      } else {
        break; // Stop selection once limit is exceeded
      }
    }

    return {
      messages: [
        { role: 'system', content: systemPrompt },
        ...selectedHistory
      ],
      tools: tools
    };
  }

  // Simple Token estimation (use libraries like tiktoken in real applications)
  estimateTokens(text) {
    return Math.ceil(text.length / 4);
  }

  async onTurnComplete() {
    // Basic strategy requires no background processing
    return;
  }
}
```

## Pros and Cons Analysis

*   **Pros:**
    *   Extremely simple to implement with no additional computational overhead.
    *   Guarantees 100% precision for recent dialogues.
*   **Cons:**
    *   **"Fragmented Forgetting"**: Once the dialogue becomes too long, the earliest instructions or critical information vanish completely.
    *   Lacks depth in understanding long contexts.

## Applicable Scenarios
*   Single-task execution.
*   Models with massive Token windows (e.g., Gemini 1.5 Pro, Claude 3), where the side effects of a sliding window are minimized.
