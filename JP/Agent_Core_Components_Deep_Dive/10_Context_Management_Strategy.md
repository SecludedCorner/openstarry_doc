# 10. コンテキスト管理戦略 (Context Management Strategy)

このドキュメントでは、 OpenStarry アーキテクチャにおける LLM Context Window（コンテキストウィンドウ）管理の設計哲学と実装インターフェースを定義します。

## 核心的な哲学

OpenStarry の「非中央集権化」と「モジュール化」の原則に基づき、コンテキスト管理はエージェントコアの実行ループ内にハードコードされるべきではありません。代わりに、それは一種の **「認知戦略 (Cognitive Strategy)」** と見なされ、 **プラグイン可能 (Pluggable)** でなければなりません。

### なぜ戦略化が必要なのか？

エージェントの役割によって、記憶に対するニーズは全く異なります。
*   **カスタマーサポート Agent:** ユーザーが今言ったことを正確に覚えている必要がありますが、10分前の世間話は忘れても構いません。（スライディングウィンドウが適しています）
*   **小説家 Agent:** 各章のあらすじを覚えている必要がありますが、逐一の書き起こしを覚えている必要はありません。（動的な要約が適しています）
*   **チケット予約 Agent:** 「目的地」「時間」などの主要な変数だけを覚えていれば十分です。（状態抽出が適しています）

したがって、 `ContextManager` はインターフェースである必要があり、開発者がエージェントの役割に応じて異なる戦略プラグインをマウントできるようにします。

---

## アーキテクチャ設計

### 1. インターフェース定義 ( `IContextManager` )

すべてのコンテキスト戦略は、以下の標準インターフェースを実装する必要があります。

```typescript
interface IContextManager {
  /**
   * コアメソッド： LLM に送信する最終的な Payload を組み立てる
   * @param systemPrompt - 不変のシステム指示
   * @param history - 生の対話履歴キュー
   * @param tools - 現在利用可能なツールの定義
   * @param ragContext - (任意) 取得された関連知識
   * @returns Token 制限に適合するように修復/圧縮された Messages 配列
   */
  composePayload(
    systemPrompt: string,
    history: ChatMessage[],
    tools: ToolDefinition[],
    ragContext?: string
  ): Promise<LLMPayload>;

  /**
   * フック：各対話の終了後に呼び出され、バックグラウンド処理（要約の更新など）に使用される
   */
  onTurnComplete(newHistory: ChatMessage[]): Promise<void>;
}
```

### 2. コンテキストの解剖学 (Anatomy of Context)

典型的な Context Payload は、以下の優先順位で構成されます（高から低へ）：

1.  **System Block:** 人格の定義、核心的な指示（重み：最高、破棄不可）。
2.  **Tool Definitions:** ツールの JSON Schema（重み：極めて高い、完全である必要がある）。
3.  **Dynamic Context:** 現在のタスク目標、 RAG 検索結果（重み：高）。
4.  **Memory/Summary Block:** 過去の対話の圧縮された要約（重み：中）。
5.  **Recent History:** 直近の N ターンの対話（重み：低、最も古いものは犠牲にできる）。

---

## 標準戦略の実装

3つの標準戦略があらかじめ定義されています。開発者は独自の戦略を作成することもできます。

### 戦略 A: スライディングウィンドウ (Sliding Window) - [デフォルト]

*   **ロジック：** 純粋な FIFO (First-In-First-Out)。
*   **アルゴリズム：**
    1.  `System` + `Tools` + `Dynamic` の Token 消費量を計算します。
    2.  残りのスペース `Remaining_Tokens` をすべて `History` に割り当てます。
    3.  `History` キューの **最後（最新）** から遡ってメッセージを選択し、総 Token 数が `Remaining_Tokens` に近づくまで含めます。
    4.  それより古いメッセージは破棄します。
*   **適用シナリオ：** シンプルな指示の実行、短期的な質疑応答。

### 戦略 B: 動的な要約 (Dynamic Summarization) - [推奨]

*   **ロジック：** 古い対話履歴を定期的に自然言語の要約に圧縮します。
*   **アルゴリズム：**
    1.  `Summary_Buffer` 文字列を維持します。
    2.  `History` の長さが閾値（10ターンなど）を超えたとき、バックグラウンドタスクをトリガーします。
    3.  軽量な LLM （ Gemini Flash など）を使用して、最も古い5ターンの対話と現在の `Summary_Buffer` を読み取ります。
    4.  新しい要約を生成し、その5ターンの対話をクリアします。
    5.  Payload を組み立てる際、 `Summary_Buffer` を System Prompt の後に挿入します。
*   **適用シナリオ：** 長期的なパートナー型アシスタント、複雑なタスクの協調。

### 戦略 C: 主要状態の抽出 (Entity Extraction)

*   **ロジック：** 対話フローを保持せず、構造化された状態のみを保持します。
*   **アルゴリズム：**
    1.  状態 Schema （例： `{ destination: string, date: string }` ）を定義します。
    2.  ユーザーの入力があるたびに、まず専用の Extraction Chain を呼び出して、この JSON オブジェクトを更新します。
    3.  Payload を組み立てる際、この JSON オブジェクトを直接 Context に注入します。
*   **適用シナリオ：** フォーム入力ボット、特定のプロセスガイド。

---

## プラグイン化による実装

`plugin.json` 内で、開発者はどの戦略を使用するかを指定できます。

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

この設計により、エージェントコアは常に軽量な状態を維持でき、複雑な記憶管理ロジックは専用の戦略モジュールに委ねられます。
