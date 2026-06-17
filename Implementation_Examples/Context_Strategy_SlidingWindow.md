# 實作範例：基礎滑動窗口策略 (Sliding Window Context Strategy)

> ⚠️ **[漂移更正 — v0.59.6 stale-API]** 本文下方原始範例（`SlidingWindowStrategy` 類別、`composePayload(systemPrompt, history, tools)` / `estimateTokens` / `onTurnComplete`、以「Token 上限」截斷）**與實際出貨代碼不符，僅保留為早期構想的歷史記錄**。
>
> **真實 API 是「回合制（turn-based）」而非「Token 制」：**
> - 介面權威：`packages/sdk/src/interfaces/context.ts:8-11` — `IContextManager.assembleContext(messages: Message[], maxTurns: number): Message[]`（單一方法；無 `composePayload`/`estimateTokens`/`onTurnComplete`）。
> - 滑動窗口實作：`context-sliding-window` 插件 `src/context.ts:11-41`（`createContextManager()`）。它**保留最後 N 個 user 回合 + 所有 system 訊息**：先過濾出 system 訊息恆保留，再從尾端往前數 user 回合，超過 `maxTurns` 即裁切；`maxTurns <= 0` 時回傳全部。**此插件完全不做任何 Token 估算或文本截斷。**
> - `Math.ceil(text.length / 4)` 這個 Token 估算邏輯**不在滑動窗口插件裡**，而是位於另一個獨立插件 `context-summary` 的 `src/token-estimator.ts:16-21`（`estimateTokens`）與 `:29-40`（`estimateMessagesTokens`），供 summary 策略判斷是否觸發壓縮門檻用，非計費等級精度。

## 真實實作（v0.59.6 出貨版）

`@openstarry-plugin/context-sliding-window` 透過 `createContextManager()` 回傳一個 `IContextManager`，其唯一方法 `assembleContext` 接收完整 `Message[]` 與 `maxTurns`，回傳裁切後的訊息陣列：

```typescript
// context-sliding-window/src/context.ts （與 packages/sdk/src/interfaces/context.ts 介面對齊）
export function createContextManager(): IContextManager {
  return {
    assembleContext(messages: Message[], maxTurns: number): Message[] {
      if (messages.length === 0) return [];

      const systemMessages = messages.filter((m) => m.role === "system");
      const conversationMessages = messages.filter((m) => m.role !== "system");

      if (maxTurns <= 0) {
        return [...systemMessages, ...conversationMessages];
      }

      let userTurnCount = 0;
      let cutIndex = conversationMessages.length;

      // 從最新訊息往前數 user 回合，超過 maxTurns 即裁切
      for (let i = conversationMessages.length - 1; i >= 0; i--) {
        if (conversationMessages[i].role === "user") {
          userTurnCount++;
          if (userTurnCount > maxTurns) {
            cutIndex = i + 1;
            break;
          }
          cutIndex = i;
        }
      }

      const windowedMessages = conversationMessages.slice(cutIndex);
      return [...systemMessages, ...windowedMessages]; // system 恆保留
    },
  };
}
```

要點：以**回合數（maxTurns）**為單位，而非 Token 上限；system 訊息恆保留；無壓縮、無 Token 估算。Token 估算（`Math.ceil(len/4)`）由 `context-summary` 插件另行提供。

---

## 以下為早期構想（歷史記錄，非出貨代碼）

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
