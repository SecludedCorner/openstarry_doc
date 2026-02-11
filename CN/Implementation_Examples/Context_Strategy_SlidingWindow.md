# 实作示例：基础滑动窗口策略 (Sliding Window Context Strategy)

这是 OpenStarry 最初期、最基础的上下文管理实作方式。它遵循简单的 FIFO（先进先出）原则，适合对记忆长度要求不高的简单任务。

## 实作逻辑

此策略不进行任何文本压缩，仅根据 Token 限制进行截断。

### 1. 核心代码 (Node.js 示例)

```javascript
class SlidingWindowStrategy {
  constructor(config) {
    this.maxTokens = config.maxTokens || 4096;
    this.reservedTokens = config.reservedTokens || 1000; // 预留给 System Prompt 和 Tools
  }

  async composePayload(systemPrompt, history, tools) {
    let currentTokens = this.estimateTokens(systemPrompt + JSON.stringify(tools));
    const availableForHistory = this.maxTokens - currentTokens - this.reservedTokens;

    let selectedHistory = [];
    let historyTokens = 0;

    // 从最新的消息开始往前选取
    for (let i = history.length - 1; i >= 0; i--) {
      const msg = history[i];
      const msgTokens = this.estimateTokens(msg.content);

      if (historyTokens + msgTokens <= availableForHistory) {
        selectedHistory.unshift(msg); // 保持顺序插入
        historyTokens += msgTokens;
      } else {
        break; // 超过限制，停止选取
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

  // 简易 Token 估算 (实际应用中应使用 tiktoken 等库)
  estimateTokens(text) {
    return Math.ceil(text.length / 4);
  }

  async onTurnComplete() {
    // 基础策略不需要后台处理
    return;
  }
}
```

## 优缺点分析

*   **优点：**
    *   实作极其简单，无额外运算开销。
    *   保证了近期对话的 100% 精确度。
*   **缺点：**
    *   **「断片式遗忘」**：一旦对话过长，最早的指令或重要信息会彻底消失。
    *   缺乏对长文本的理解深度。

## 适用场景
*   单次任务执行。
*   Token 窗口极大的模型 (如 Gemini 1.5 Pro, Claude 3)，此时滑动窗口的副作用较小。
