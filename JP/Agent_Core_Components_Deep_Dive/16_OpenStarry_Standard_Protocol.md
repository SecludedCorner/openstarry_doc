# 16. OpenStarry 標準プロトコル (OpenStarry Standard Protocol)

このドキュメントでは、 OpenStarry エコシステムにおいて、エージェントコアと外部プラグイン（特にプロバイダーとツール）が通信するための技術標準を定義します。コンポーネント間の相互運用性を確保するため、すべての公式およびサードパーティ製プラグインはこのプロトコルに従う必要があります。

---

## 1. コマンドパスの比較：スラッシュコマンド vs. 自律的行動

OpenStarry において、同じツール（ `fs.read` など）は、2つの全く異なるパスを通じてトリガーされます。

| 特性 | スラッシュコマンド (Slash Commands) | 自律的行動 (Autonomous Actions) |
| :--- | :--- | :--- |
| **トリガー源** | ユーザー (User) | エージェントの脳 (LLM/Provider) |
| **トリガー方法** | `/read filename.txt` を入力 | LLM がツール呼び出しを決定し、 `tool_call` を返却 |
| **思考のバイパス** | はい (直接実行) | いいえ (OODA ループの決定を経由) |
| **権限モデル** | ユーザー認可 (User Authorized) | エージェントの自律性 (Agent Autonomy) |
| **処理ロジック** | `ExecutionLoop` による前置解析 | `ExecutionLoop` による `ProviderResponse` の解析 |

---

## 2. 翻訳ガイド：異質な API から標準形式へ

プロバイダーの核心的な価値は「差異の解消」にあります。以下は、 Google Gemini API を例とした翻訳の疑似コードです。

### A. アップストリーム翻訳 (コアのツールを API 形式に変換)
```typescript
// コアのツール定義
const coreTool: ITool = { name: "read", description: "...", parameters: { ... } };

// Gemini 形式に翻訳
const geminiFunction = {
  name: coreTool.name,
  description: coreTool.description,
  parameters: coreTool.parameters // Gemini は JSON Schema を受け入れる
};
```

### B. ダウンストリーム翻訳 (API の応答をコアプロトコルに変換)
```typescript
// LLM API からの生の返却値 (Raw Content)
const geminiPart = { functionCall: { name: "read", args: { path: "a.txt" } } };

// OpenStarry 標準プロトコルに翻訳
const response: ProviderResponse = {
  segments: [{
    type: 'tool_call',
    toolCall: {
      name: geminiPart.functionCall.name,
      args: geminiPart.functionCall.args
    }
  }]
};
```

---

## 3. 型定義 (Type Definitions)

これらの定義は、 `@openstarry/sdk` の `interfaces.ts` にあります。

### A. ツール呼び出し構造 (ToolCall)
これはエージェントがあるアクションを実行しようとする意図を運ぶための標準的な担体です。

```typescript
export interface ToolCall {
  /**
   * 呼び出しの一意の識別コード (追跡とコールバックのマッチングに使用)
   * 一部の LLM ( OpenAI など) はこの ID を提供します。提供されない場合、プロバイダーは UUID を自動生成する必要があります。
   */
  id?: string;

  /**
   * 呼び出すツール名 (登録されている tool.name と完全に一致する必要があります)
   */
  name: string;

  /**
   * ツールに渡されるパラメータオブジェクト
   */
  args: any;
}
```

### B. プロバイダー応答構造 (ProviderResponse)
これは `IProvider.generate()` メソッドが必ず返さなければならない標準的な結果です。マルチモーダル (VLLM, Audio, UI) をサポートするため、 **セグメント化されたコンテンツ (Segmented Content)** 設計を採用しています。

```typescript
export type ContentType = 'text' | 'image' | 'audio' | 'video' | 'ui' | 'tool_call';

export interface ContentSegment {
  type: ContentType;
  
  /**
   * type='text' の場合に使用
   */
  text?: string;
  
  /**
   * type='image' | 'audio' | 'video' の場合に使用
   * 通常は Base64 エンコードされたデータまたは URL
   */
  data?: string;
  
  /**
   * メディアタイプ (MIME Type), e.g., "image/png", "audio/mp3"
   */
  mimeType?: string;

  /**
   * type='tool_call' の場合に使用
   */
  toolCall?: ToolCall;

  /**
   * type='ui' の場合に使用
   * フロントエンドがレンダリングすべきコンポーネント (JSON) を記述
   */
  ui?: any;
}

export interface ProviderResponse {
  /**
   * 順序付けられたコンテンツセグメントのリスト
   * エージェントは同時に話し、画像を表示し、ツールを実行することができます。
   */
  segments: ContentSegment[];

  /**
   * 追加のメタデータ ( Token 使用量、モデルの遅延など)
   */
  metadata?: any;
}
```

---

## 4. インターフェース動作規範 (Interface Behavior Specifications)

### IProvider (想蘊)

プロバイダープラグインは `generate` メソッドを実装し、以下の動作に従う必要があります。

1.  **能力の感知:** プロバイダーは `context.tools` から現在利用可能なツールのリストを読み取る必要があります。
2.  **プロトコルの変換:** `context.tools` をターゲットの LLM API が必要とする Schema 形式に変換します。
3.  **結果の解析:** LLM API の応答を受信し、それを `ProviderResponse` として標準化します。
    *   **プレーンテキスト:** `[{ type: 'text', text: '...' }]` を返却
    *   **ツール呼び出し:** `[{ type: 'tool_call', toolCall: { ... } }]` を返却
    *   **混合モード:** 複数の Segment を返却。例えば Gemini は、まず説明 (text) を返し、次にツール呼び出し (tool_call) を返すことがあります。

```typescript
export interface IProvider {
  name: string;
  generate(prompt: string, context: IAgentContext): Promise<ProviderResponse>;
}
```

### IAgentContext (環境コンテキスト)

コアは、自身の状態と能力を Context を通じてプロバイダーに公開しなければなりません。

```typescript
export interface IAgentContext {
  // ... その他の属性
  
  /**
   * 能力の反映 (Capability Reflection)
   * エージェントが現在どのようなツール (行蘊) を持っているかをプロバイダーに知らせます。
   * Map のキーはツール名です。
   */
  tools: Map<string, ITool>;
}
```

---

## 5. 対話シーケンスの例 (Sequence Example)

以下は、標準的な「思考-行動」ループのプロトコルフローです。

1.  **Input:** ユーザーが「台北の天気を調べて」と入力。
2.  **Core -> Provider:** `generate("台北の天気を調べて", context)` を呼び出し。
    *   *Context には `get_weather` ツールの定義が含まれています。*
3.  **Provider -> LLM API:** プロンプト + 関数定義を送信。
4.  **LLM API -> Provider:** 生の JSON を返却 (例： Gemini の `functionCall` 構造)。
5.  **Provider (適応):** 生の JSON を `ProviderResponse` に変換：
    ```json
    {
      "segments": [
        { "type": "text", "text": "わかりました。お調べします。" },
        { "type": "tool_call", "toolCall": { "name": "get_weather", "args": { "location": "Taipei" } } }
      ]
    }
    ```
6.  **Provider -> Core:** 上記のオブジェクトを返却。
7.  **Core (実行):** `tool_call` を検知し、 `name` に基づいて `context.tools` を検索して実行。
8.  **Core (フィードバック):** ツールの実行結果（例：「25°C, 晴れ」）を記憶に保存し、再度プロバイダーを呼び出し（次のターンへ）。

---

## 6. 高度なパターン：大規模ツールの管理 (Advanced Pattern)

現在の標準プロトコルは、中小規模のエージェントに適した「ジャストインタイム注入 (Just-in-Time Injection)」モードを採用しています。数百のツールを持つ大規模システムについては、以下の最適化戦略の採用を推奨します。

### A. 静的登録とキャッシュ (Registry & Caching)
Context Caching をサポートする LLM （ OpenAI Assistants や Gemini 1.5 など）に対して、プロバイダーは初期化段階でツールの Schema を一度にアップロードし、 `tools_id` を取得しておくべきです。以降の対話では ID を渡すだけで済み、 Token 消費量と遅延を大幅に削減できます。

### B. 動的なコンテキストフィルタリング (Dynamic Context Filtering)
Context Window が大量のツール定義で埋め尽くされるのを避けるため、コアに「ツール検索レイヤー (Tool Retrieval Layer)」を導入できます。
1.  **意図の識別:** ユーザーの現在の入力意図 (Intent) を分析。
2.  **ベクトル検索:** ツールライブラリから Top-N の最も関連性の高いツールを検索。
3.  **精密な注入:** すべてではなく、この N 個のツールのみをプロバイダーに注入。

これにより、エージェントが数千ものスキルを持っていても、軽量かつ集中した状態を維持できます。

---

## 7. エラー処理規範

*   **解析失敗:** プロバイダーが LLM の出力を解析できない場合（無効な JSON など）、エラーをキャッチして `text` セグメントでエラー内容を返すか、標準の `AgentPainException` をスローする必要があります。
*   **ツールが存在しない:** プロバイダーが存在しない `toolCall.name` を返した場合、コアはこのエラーをキャッチし、エラーメッセージを「システムの観察」として次のターンの対話にフィードバックし、 LLM に自己修正の機会を与える必要があります。
