# 実装例：基礎的なスライディングウィンドウ戦略 (Sliding Window Context Strategy)

これは OpenStarry の最初期における、最も基礎的なコンテキスト管理の実装方式です。単純な FIFO （先入れ先出し）原則に従っており、記憶の長さをそれほど必要としない単純なタスクに適しています。

## 実装ロジック

この戦略ではテキストの圧縮は行わず、 Token 制限に基づいて切り詰めのみを行います。

### 1. コアコード (Node.js の例)

```javascript
class SlidingWindowStrategy {
  constructor(config) {
    this.maxTokens = config.maxTokens || 4096;
    this.reservedTokens = config.reservedTokens || 1000; // System Prompt と Tools 用に予約
  }

  async composePayload(systemPrompt, history, tools) {
    let currentTokens = this.estimateTokens(systemPrompt + JSON.stringify(tools));
    const availableForHistory = this.maxTokens - currentTokens - this.reservedTokens;

    let selectedHistory = [];
    let historyTokens = 0;

    // 最新のメッセージから遡って選択
    for (let i = history.length - 1; i >= 0; i--) {
      const msg = history[i];
      const msgTokens = this.estimateTokens(msg.content);

      if (historyTokens + msgTokens <= availableForHistory) {
        selectedHistory.unshift(msg); // 順序を維持して挿入
        historyTokens += msgTokens;
      } else {
        break; // 制限を超えたため選択を停止
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

  // 簡易 Token 試算 (実際のアプリケーションでは tiktoken などのライブラリを使用すべき)
  estimateTokens(text) {
    return Math.ceil(text.length / 4);
  }

  async onTurnComplete() {
    // 基礎的な戦略ではバックグラウンド処理は不要
    return;
  }
}
```

## 優缺点分析

*   **利点：**
    *   実装が極めてシンプルで、追加の計算オーバーヘッドがない。
    *   直近の対話については 100% の正確性を保証する。
*   **欠点：**
    *   **「断片的な忘却」**：対話が長くなると、初期の指示や重要な情報が完全に消失する。
    *   長いテキストに対する理解の深さに欠ける。

## 適用シナリオ
*   単発のタスク実行。
*   Token ウィンドウが非常に大きいモデル ( Gemini 1.5 Pro, Claude 3 など) 。この場合、スライディングウィンドウの副作用は少なくなります。
