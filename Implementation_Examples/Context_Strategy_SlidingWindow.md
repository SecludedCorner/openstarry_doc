# 實作範例：基礎滑動窗口策略 (Sliding Window Context Strategy)

這是 OpenStarry 最初期、最基礎的上下文管理實作方式。它遵循簡單的 FIFO（先進先出）原則，適合對記憶長度要求不高的簡單任務。

## 實作邏輯

此策略不進行任何文本壓縮，僅根據 Token 限制進行截斷。

### 1. 核心代碼 (Node.js 範例)

```javascript
class SlidingWindowStrategy {
  constructor(config) {
    this.maxTokens = config.maxTokens || 4096;
    this.reservedTokens = config.reservedTokens || 1000; // 預留給 System Prompt 和 Tools
  }

  async composePayload(systemPrompt, history, tools) {
    let currentTokens = this.estimateTokens(systemPrompt + JSON.stringify(tools));
    const availableForHistory = this.maxTokens - currentTokens - this.reservedTokens;

    let selectedHistory = [];
    let historyTokens = 0;

    // 從最新的消息開始往前選取
    for (let i = history.length - 1; i >= 0; i--) {
      const msg = history[i];
      const msgTokens = this.estimateTokens(msg.content);

      if (historyTokens + msgTokens <= availableForHistory) {
        selectedHistory.unshift(msg); // 保持順序插入
        historyTokens += msgTokens;
      } else {
        break; // 超過限制，停止選取
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

  // 簡易 Token 估算 (實際應用中應使用 tiktoken 等庫)
  estimateTokens(text) {
    return Math.ceil(text.length / 4);
  }

  async onTurnComplete() {
    // 基礎策略不需要後台處理
    return;
  }
}
```

## 優缺點分析

*   **優點：**
    *   實作極其簡單，無額外運算開銷。
    *   保證了近期對話的 100% 精確度。
*   **缺點：**
    *   **「斷片式遺忘」**：一旦對話過長，最早的指令或重要資訊會徹底消失。
    *   缺乏對長文本的理解深度。

## 適用場景
*   單次任務執行。
*   Token 窗口極大的模型 (如 Gemini 1.5 Pro, Claude 3)，此時滑動窗口的副作用較小。
