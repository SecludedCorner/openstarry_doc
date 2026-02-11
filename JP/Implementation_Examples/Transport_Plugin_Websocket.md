# 実装例： WebSocket トランスポートプラグイン (WebSocket Transport Plugin)

このドキュメントでは、 WebSocket ベースのトランスポートプラグインの開発方法について詳細に説明します。このプラグインは、エージェントコアと外部クライアント（ React フロントエンド、モバイルアプリなど）の間のリアルタイム通信の架け橋として機能します。

## 核心的な概念

OpenStarry アーキテクチャにおいて、 **トランスポート層 (Transport Layer)** はプラグインの一種と見なされます。エージェントコア自体は「ヘッドレス (Headless)」かつ「プロトコルにとらわれない (Protocol Agnostic)」存在です。 WebSocket プラグインは、ネットワークプロトコルのデータストリームをコアが理解できるイベントに変換する役割を担います。

### プラグインの責務
1.  **ライフサイクル管理：** エージェントの起動時に WebSocket サーバーを確立し、エージェントの停止時に接続を正常に終了させます。
2.  **インバウンド変換 (Inbound)：** WebSocket メッセージを受信し、 `ctx.pushInput()` を介してイベントキューに注入します。
3.  **アウトバウンド変換 (Outbound)：** `IUI.onEvent()` で AgentEvent を受信し、クライアントへ WebSocket メッセージを送信します。
4.  **セッション管理：** 複数のクライアント接続を管理し、ターゲット指定の返信 (replyTo) やブロードキャストモードをサポートします。

---

## コード実装例

以下は、 OpenStarry のファクトリパターン (Factory Pattern) に基づく簡略化された実装で、実際の `@openstarry-plugin/transport-websocket` に対応しています。

### 1. パッケージ構造

```
transport-websocket/
├── package.json
├── tsconfig.json
└── src/
    ├── index.ts        # プラグインファクトリ関数 + Listener + UI
    └── index.test.ts   # ユニットテスト
```

### 2. プラグインの実装 ( `src/index.ts` )

```typescript
import { WebSocketServer, WebSocket } from "ws";
import type {
  IPlugin,
  IPluginContext,
  PluginHooks,
  IListener,
  IUI,
  AgentEvent,
} from "@openstarry/sdk";

interface Config {
  port?: number;
  host?: string;
  path?: string;
}

interface ClientConnection {
  id: string;
  ws: WebSocket;
  connectedAt: number;
}

// ─── WebSocket UI (色蘊) ───
function createWebSocketUI(connections: Map<string, ClientConnection>): IUI {
  return {
    id: "websocket-ui",
    name: "WebSocket UI",

    onEvent(event: AgentEvent): void {
      const message = JSON.stringify({
        type: "agent_event",
        event: {
          type: event.type,
          timestamp: event.timestamp,
          payload: event.payload,
        },
      });

      // ターゲット指定の返信かどうかをチェック
      const payload = event.payload as Record<string, unknown> | undefined;
      const replyTo = payload?.replyTo as string | undefined;

      if (replyTo && connections.has(replyTo)) {
        // 特定のクライアントへのターゲット返信
        const conn = connections.get(replyTo)!;
        if (conn.ws.readyState === WebSocket.OPEN) {
          conn.ws.send(message);
        }
      } else {
        // 接続されているすべてのクライアントへのブロードキャスト
        for (const conn of connections.values()) {
          if (conn.ws.readyState === WebSocket.OPEN) {
            conn.ws.send(message);
          }
        }
      }
    },
  };
}

// ─── Plugin Export ───
export function createWebSocketPlugin(): IPlugin {
  return {
    manifest: {
      name: "transport-websocket",
      version: "0.1.0-alpha",
      description: "WebSocket transport plugin (Listener + UI)",
    },

    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      const config = ctx.config as Config;
      const port = config.port ?? 8080;
      const host = config.host ?? "0.0.0.0";
      const path = config.path ?? "/ws";

      const connections = new Map<string, ClientConnection>();
      let wss: WebSocketServer | null = null;
      let clientIdCounter = 0;

      const listener: IListener = {
        id: "websocket-listener",
        name: "WebSocket Listener",

        async start(): Promise<void> {
          wss = new WebSocketServer({ port, host, path });

          wss.on("connection", (ws: WebSocket) => {
            const clientId = `ws-client-${++clientIdCounter}-${Date.now()}`;
            connections.set(clientId, { id: clientId, ws, connectedAt: Date.now() });

            // ウェルカムメッセージの送信
            ws.send(JSON.stringify({
              type: "connected",
              clientId,
              message: "Connected to OpenStarry Agent",
            }));

            ws.on("message", (data: Buffer | string) => {
              try {
                const msg = JSON.parse(data.toString());
                if (msg.type === "user_input") {
                  ctx.pushInput({
                    source: "websocket",
                    inputType: "user_input",
                    data: msg.payload?.text ?? "",
                    replyTo: clientId,
                  });
                } else if (msg.type === "ping") {
                  ws.send(JSON.stringify({ type: "pong", timestamp: Date.now() }));
                }
              } catch {
                ws.send(JSON.stringify({ type: "error", error: "Invalid JSON" }));
              }
            });

            ws.on("close", () => connections.delete(clientId));
            ws.on("error", () => connections.delete(clientId));
          });
        },

        async stop(): Promise<void> {
          if (wss) {
            for (const conn of connections.values()) {
              conn.ws.close(1000, "Server shutting down");
            }
            connections.clear();
            await new Promise<void>((resolve) => wss!.close(() => resolve()));
            wss = null;
          }
        },
      };

      return {
        listeners: [listener],
        ui: [createWebSocketUI(connections)],
        async dispose() {
          await listener.stop?.();
        },
      };
    },
  };
}

export default createWebSocketPlugin;
```

### 主要な設計ポイント

1. **ファクトリパターン (Factory Pattern)：** プラグインは `createWebSocketPlugin()` 関数をエクスポートし、 `IPlugin` オブジェクトを返します。 `factory()` メソッドは `IPluginContext` を受け取り、 `PluginHooks` を返します。
2. **受蘊 + 色蘊：** 1つのプラグインで `IListener` （入力の受信）と `IUI` （出力のプッシュ）の両方を同時に提供し、 `PluginHooks` の `listeners` と `ui` 配列を介して登録します。
3. **ctx.pushInput()：** リスナーはコアの API を直接呼び出すのではなく、 `IPluginContext.pushInput()` を通じて入力をイベントキューに注入します。
4. **replyTo によるターゲット返信：** 各クライアントに一意の `clientId` を割り当て、入力イベントに `replyTo: clientId` を付帯させます。 UI プッシュ時にイベントに `replyTo` が含まれている場合、対応するクライアントにのみ送信されます。

---

## 通信プロトコル仕様 (Protocol Spec)

フロントエンド開発者が容易に接続できるように、以下の JSON プロトコルを定義します：

### クライアント → サーバー (アップストリーム)

1.  **メッセージの送信**
    ```json
    {
      "type": "user_input",
      "payload": { "text": "このファイルを分析して" }
    }
    ```

2.  **Ping/ハートビート**
    ```json
    { "type": "ping" }
    ```

### サーバー → クライアント (ダウンストリーム)

1.  **接続確認**
    ```json
    {
      "type": "connected",
      "clientId": "ws-client-1-1707300000000",
      "message": "Connected to OpenStarry Agent"
    }
    ```

2.  **エージェントイベント**
    ```json
    {
      "type": "agent_event",
      "event": {
        "type": "RESPONSE",
        "timestamp": 1707300000000,
        "payload": { "content": "わかりました。ファイルを読み取っています..." }
      }
    }
    ```

3.  **Pong 応答**
    ```json
    { "type": "pong", "timestamp": 1707300000000 }
    ```

4.  **エラー**
    ```json
    { "type": "error", "error": "Invalid JSON" }
    ```

## agent.json 設定例

```json
{
  "plugins": [
    {
      "name": "@openstarry-plugin/transport-websocket",
      "config": { "port": 8080, "host": "0.0.0.0", "path": "/ws" }
    }
  ]
}
```

## セキュリティ上の考慮事項

本番環境では、 WebSocket プラグインにおいて以下の点に注意してください：
1.  **認証 (Authentication)：** `connection` イベントにおいて、 URL パラメータの `?token=...` または HTTP ヘッダーを確認し、 JWT を検証する必要があります。
2.  **TLS：** Nginx リバースプロキシ層で `wss://` 暗号化を処理し、プラグインはローカルの `ws://` のみを監視することを推奨します。
