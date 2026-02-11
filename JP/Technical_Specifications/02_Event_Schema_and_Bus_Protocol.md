# 02. イベントバスプロトコル仕様 (Event Bus & Schema Protocol)

本ドキュメントは、OpenStarry 内部およびエッジコンポーネント間の通信に使用される標準イベントフォーマットを定義します。トランスポートプラグイン (Transport) からコア (Core) に流れるすべてのデータ、およびコアが生成するフィードバックは、本プロトコルに準拠する必要があります。

## 1. コアイベント構造 (Core Event Envelope)

すべてのイベントは、トレーサビリティとルーティングの正確性を確保するために、標準のエンベロープオブジェクトにカプセル化される必要があります。

```typescript
interface IOpenStarryEvent {
  // 基本メタデータ
  readonly id: string;            // UUID v4
  readonly timestamp: number;     // Unix Timestamp (ms)
  readonly traceId: string;       // リクエストチェーン全体を追跡するためのユニーク ID

  // ルーティング情報
  readonly source: string;        // ソースコンポーネント ID (例: "transport-websocket-01")
  readonly sessionId: string;     // セッション ID、マルチユーザー分離に使用
  readonly priority: number;      // 0 (最高、システム緊急) から 5 (最低、バックグラウンドタスク)

  // ペイロード
  readonly type: EventType;       // イベントタイプ
  readonly payload: any;          // 具体的なデータ内容
}
```

## 2. イベントタイプ列挙 (EventType)

### 2.1 入力イベント (Input Events)
*   `INPUT:USER_MESSAGE`: ユーザーが入力したプレーンテキスト。
*   `INPUT:SYSTEM_COMMAND`: システムレベルのコマンド（例：`/stop`, `/reset`）。
*   `INPUT:SIGNAL`: センサーまたは外部 Webhook からのトリガーシグナル。

### 2.2 カーネルイベント (Kernel Events)
*   `KERNEL:TICK_START`: 実行ループが新しいイテレーションを開始。
*   `KERNEL:STATE_CHANGE`: Agent のステート遷移（例：`IDLE` -> `EXECUTING`）。
*   `KERNEL:ERROR`: コアが標準化したエラー出力。

### 2.3 実行イベント (Execution Events)
*   `EXEC:TOOL_CALL`: Agent がツールの実行をリクエスト。
*   `EXEC:TOOL_RESULT`: ツール実行後の結果フィードバック。

---

## 3. ペイロード仕様例

### 3.1 `INPUT:USER_MESSAGE`
```json
{
  "type": "INPUT:USER_MESSAGE",
  "payload": {
    "text": "最近のシステムログを読み取ってください",
    "mimeType": "text/plain",
    "metadata": { "platform": "discord" }
  }
}
```

### 3.2 `EXEC:TOOL_RESULT` (ペインメカニズムに関連するデータソース)
```json
{
  "type": "EXEC:TOOL_RESULT",
  "payload": {
    "toolCallId": "call_abc123",
    "success": false,
    "error": {
      "code": "EPERM",
      "message": "Operation not permitted",
      "source": "fs-plugin"
    }
  }
}
```

## 4. ルーティングと伝播ルール

1.  **優先度処理**: コアキューは `priority: 0` のイベントを優先的に処理する必要があります。
2.  **冪等性**: コアはスライディングウィンドウ内で重複して受信したイベントを `id` に基づいて破棄する必要があります。
3.  **TraceId の伝播**: イベントによって引き起こされた新しいイベント（例：Input によって引き起こされた ToolCall）は、元のイベントの `traceId` を継承する必要があります。
