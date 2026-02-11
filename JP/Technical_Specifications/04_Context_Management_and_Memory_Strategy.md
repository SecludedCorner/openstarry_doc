# 04. コンテキスト管理とメモリ戦略技術仕様 (Context Management & Memory Strategy)

本ドキュメントは、OpenStarry コアが Agent の限られたコンテキストウィンドウ (Context Window) をどのように処理するかを定義します。長期運用時のクラッシュを防ぐため、コアは決定論的なメモリ管理アルゴリズムを備えている必要があります。

## 1. 階層型メモリアーキテクチャ (Hierarchical Memory)

OpenStarry は三層メモリモデルを採用しています：

1.  **Immediate Buffer（即時バッファ）:**
    *   **内容:** 最新の 5〜10 ターンの対話と現在の System Prompt。
    *   **戦略:** 絶対保持。いかなる圧縮や破棄も許可されません。
2.  **Short-Term Memory（短期メモリ）:**
    *   **内容:** 直近の 10〜50 ターンの対話。
    *   **戦略:** スライディングウィンドウ (Sliding Window) を採用。トークン数が上限を超えた場合、最も古いメッセージを長期アーカイブに移動するか破棄します。
3.  **Long-Term Archive（長期アーカイブ/外部ストレージ）:**
    *   **内容:** 過去の対話サマリーおよび重要な事実の抽出結果。
    *   **戦略:** ベクトル化ストレージ (RAG) または構造化サマリー。本仕様書ではインターフェースのみを定義し、具体的な実装は Memory Plugin が担当します。

---

## 2. スライディングウィンドウ切り捨てアルゴリズム (Sliding Window Truncation)

`CurrentTokenCount > MaxTokenBudget` の場合、コアはガベージコレクション (GC) をトリガーする必要があります。

### アルゴリズムの手順：

1.  **ロック:** System Prompt と最新の `K` 件のメッセージをロック（ヘッド＆テイル）。
2.  **評価:** 中間セグメントのメッセージのトークン密度を計算。
3.  **削除:** 最も古いメッセージから順に削除し、バジェットを満たすまで続行。
    *   **重要ルール:** Tool Call と Tool Result のペアリング整合性を維持する必要があります。`Call` だけを削除して `Result` を残すことは厳禁です。これは LLM のロジック破綻を引き起こします。削除が必要な場合は、必ずペアで削除してください。

---

## 3. メモリ圧縮プロトコル (Memory Compression Protocol)

限られたウィンドウ内でより多くの情報を保持するために、OpenStarry は標準的な圧縮フォーマットを定義しています。

### 3.1 サマリーインジェクション (Summary Injection)
対話が切り捨てられた場合、切り捨て地点に System Message を挿入する必要があります：

```json
{
  "role": "system",
  "content": "[Memory Summary] Previous conversation context: User discussed project X. Agent successfully read file Y but failed to write Z due to permission error."
}
```

### 3.2 オブザベーションリダクション (Observation Reduction)
超長のツール出力（例：`cat huge_file.log`）に対して、コアは中間部分を自動的に切り捨て、マーキングを行います：

```text
[Data Truncated]
Start: Line 1-50 content...
... (8000 lines omitted) ...
End: Line 8050-8100 content...
```

---

## 4. ステートスナップショット (State Snapshot)

ホットリスタートをサポートするために、メモリ状態はシリアライズ可能でなければなりません。

```typescript
interface IContextSnapshot {
  version: "1.0";
  timestamp: number;
  tokenCount: number;
  messages: IMessage[]; // 完全なメッセージキュー
  summary?: string;     // 現在の対話サマリー
}
```

コアはこのスナップショットを定期的にディスク (`.openstarry/snapshots/`) に書き込み、Agent が再起動後に以前のコンテキストを「思い出す」ことができるようにします。
