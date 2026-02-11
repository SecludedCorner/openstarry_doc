# Plan05: マルチチャネル UI と高度な Listener

> **ステータス**: ✅ 完了 (2026-02-07)
> **ターゲットバージョン**: v0.2 Beta

## 目標

IUI/IListener 分離アーキテクチャの拡張性を検証し、 stdio 以外のチャネルを実装する：

1. **WebSocket Listener** — WebSocket メッセージを入力として受信する
2. **WebSocket UI** — イベントを WebSocket 経由でプッシュする
3. **HTTP Webhook Listener** — HTTP POST をトリガーとして受信する
4. **複数 UI への同時出力** — 同一イベントを stdio + WebSocket へ同時にプッシュする

---

## フェーズ 1： WebSocket トランスポートプラグイン ✅

### 1.1 プラグイン構造

**ディレクトリ**: `openstarry_plugin/transport-websocket/`

```
transport-websocket/
├── package.json
├── tsconfig.json
└── src/
    ├── index.ts
    └── index.test.ts
```

### 1.2 package.json

```json
{
  "name": "@openstarry-plugin/transport-websocket",
  "version": "0.1.0-alpha",
  "type": "module",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "scripts": {
    "build": "tsc -b",
    "clean": "rimraf --glob dist "*.tsbuildinfo"",
    "dev": "tsc -b --watch"
  },
  "dependencies": {
    "@openstarry/sdk": "workspace:*",
    "ws": "^8.16.0"
  },
  "devDependencies": {
    "@types/node": "^25.2.0",
    "@types/ws": "^8.5.10",
    "rimraf": "^5.0.0",
    "typescript": "^5.5.0"
  }
}
```

### 1.3 核心的な実装 (src/index.ts)

```typescript
/**
 * transport-websocket — WebSocket transport plugin.
 *
 * Provides:
 * - WebSocketListener (受蘊) — receives WebSocket messages as input
 * - WebSocketUI (色蘊) — pushes events to WebSocket clients
 *
 * Config:
 *   { port: 8080, host: "0.0.0.0", path: "/ws" }
 */

import { WebSocketServer, WebSocket } from "ws";
import type {
  IPlugin, IPluginContext, PluginHooks,
  IListener, IUI, AgentEvent,
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

          console.log(`[WS] Server listening on ws://${host}:${port}${path}`);
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
        async dispose() { await listener.stop?.(); },
      };
    },
  };
}

export default createWebSocketPlugin;
```

---

## フェーズ 2： HTTP Webhook トランスポートプラグイン ✅

### 2.1 プラグイン構造

**ディレクトリ**: `openstarry_plugin/transport-http/`

```
transport-http/
├── package.json
├── tsconfig.json
└── src/
    ├── index.ts
    └── index.test.ts
```

### 2.2 package.json

```json
{
  "name": "@openstarry-plugin/transport-http",
  "version": "0.1.0-alpha",
  "type": "module",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc -b",
    "clean": "rimraf --glob dist "*.tsbuildinfo"",
    "dev": "tsc -b --watch"
  },
  "dependencies": {
    "@openstarry/sdk": "workspace:*"
  },
  "devDependencies": {
    "@types/node": "^25.2.0",
    "rimraf": "^5.0.0",
    "typescript": "^5.5.0"
  }
}
```

### 2.3 核心的な実装 (src/index.ts)

```typescript
/**
 * transport-http — HTTP webhook transport plugin.
 *
 * Endpoints:
 *   POST /api/input   — Submit user input
 *   GET  /api/status  — Check agent status
 *   GET  /api/response?requestId=xxx — Poll response
 *
 * Config:
 *   { port: 3000, host: "0.0.0.0", basePath: "/api" }
 */

import { createServer, IncomingMessage, ServerResponse } from "node:http";
import type {
  IPlugin, IPluginContext, PluginHooks,
  IListener, IUI, AgentEvent,
} from "@openstarry/sdk";
import { AgentEventType } from "@openstarry/sdk";

interface Config {
  port?: number;
  host?: string;
  basePath?: string;
  responseBufferSize?: number;
  responseTimeout?: number;
}

interface BufferedResponse {
  requestId: string;
  events: AgentEvent[];
  createdAt: number;
  complete: boolean;
}

export function createHttpPlugin(): IPlugin {
  return {
    manifest: {
      name: "transport-http",
      version: "0.1.0-alpha",
      description: "HTTP webhook transport plugin (Listener + UI)",
    },

    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      const config = ctx.config as Config;
      const port = config.port ?? 3000;
      const host = config.host ?? "0.0.0.0";
      const basePath = config.basePath ?? "/api";

      const responseBuffer = new Map<string, BufferedResponse>();
      let server: ReturnType<typeof createServer> | null = null;

      // ─── HTTP Listener ───
      const listener: IListener = {
        id: "http-webhook-listener",
        name: "HTTP Webhook Listener",

        async start(): Promise<void> {
          server = createServer(async (req, res) => {
            res.setHeader("Access-Control-Allow-Origin", "*");
            res.setHeader("Access-Control-Allow-Methods", "GET, POST, OPTIONS");
            res.setHeader("Access-Control-Allow-Headers", "Content-Type");

            if (req.method === "OPTIONS") {
              res.writeHead(204);
              res.end();
              return;
            }

            const url = new URL(req.url ?? "/", `http://${req.headers.host}`);

            if (url.pathname === `${basePath}/input` && req.method === "POST") {
              const body = await readBody(req);
              const { text, requestId } = JSON.parse(body);
              const id = requestId ?? `http-${Date.now()}-${Math.random().toString(36).slice(2)}`;

              responseBuffer.set(id, {
                requestId: id,
                events: [],
                createdAt: Date.now(),
                complete: false,
              });

              ctx.pushInput({
                source: "http",
                inputType: "user_input",
                data: text,
                replyTo: id,
              });

              res.writeHead(202, { "Content-Type": "application/json" });
              res.end(JSON.stringify({ status: "accepted", requestId: id }));
            } else if (url.pathname === `${basePath}/status` && req.method === "GET") {
              res.writeHead(200, { "Content-Type": "application/json" });
              res.end(JSON.stringify({ status: "running" }));
            } else if (url.pathname === `${basePath}/response` && req.method === "GET") {
              const reqId = url.searchParams.get("requestId");
              const resp = reqId ? responseBuffer.get(reqId) : null;
              if (resp) {
                res.writeHead(200, { "Content-Type": "application/json" });
                res.end(JSON.stringify(resp));
              } else {
                res.writeHead(404, { "Content-Type": "application/json" });
                res.end(JSON.stringify({ error: "Not found" }));
              }
            } else {
              res.writeHead(404, { "Content-Type": "application/json" });
              res.end(JSON.stringify({ error: "Not found" }));
            }
          });

          server.listen(port, host, () => {
            console.log(`[HTTP] Server listening on http://${host}:${port}${basePath}`);
          });
        },

        async stop(): Promise<void> {
          if (server) {
            await new Promise<void>((resolve) => server!.close(() => resolve()));
            server = null;
          }
        },
      };

      // ─── HTTP UI (ポーリング用応答バッファ) ───
      const ui: IUI = {
        id: "http-webhook-ui",
        name: "HTTP Webhook UI",

        onEvent(event: AgentEvent): void {
          const payload = event.payload as Record<string, unknown> | undefined;
          const replyTo = payload?.replyTo as string | undefined;

          if (replyTo && responseBuffer.has(replyTo)) {
            const resp = responseBuffer.get(replyTo)!;
            resp.events.push(event);
            if (event.type === AgentEventType.LOOP_FINISHED ||
                event.type === AgentEventType.LOOP_ERROR) {
              resp.complete = true;
            }
          }
        },
      };

      return {
        listeners: [listener],
        ui: [ui],
        async dispose() {
          await listener.stop?.();
          responseBuffer.clear();
        },
      };
    },
  };
}

function readBody(req: IncomingMessage): Promise<string> {
  return new Promise((resolve, reject) => {
    let body = "";
    req.on("data", (chunk) => (body += chunk.toString()));
    req.on("end", () => resolve(body));
    req.on("error", reject);
  });
}

export default createHttpPlugin;
```

---

## フェーズ 3：設定の統合 ✅

### 3.1 マルチチャネル設定例 (stdio + WebSocket)

```json
{
  "identity": {
    "id": "multi-channel-agent",
    "name": "Multi-Channel Agent"
  },
  "cognition": {
    "provider": "gemini-oauth",
    "model": "gemini-2.0-flash"
  },
  "plugins": [
    { "name": "@openstarry-plugin/provider-gemini-oauth" },
    { "name": "@openstarry-plugin/standard-function-fs" },
    { "name": "@openstarry-plugin/standard-function-stdio" },
    {
      "name": "@openstarry-plugin/transport-websocket",
      "config": { "port": 8080, "host": "0.0.0.0", "path": "/ws" }
    },
    { "name": "@openstarry-plugin/guide-character-init" }
  ],
  "guide": "default-guide"
}
```

### 3.2 apps/runner/package.json の更新

プラグインをワークスペースの依存関係として追加：

```json
{
  "dependencies": {
    "@openstarry-plugin/transport-websocket": "workspace:*",
    "@openstarry-plugin/transport-http": "workspace:*"
  }
}
```

---

## フェーズ 4： JSON プロトコル仕様

### 4.1 WebSocket プロトコル

**クライアント → サーバー**
```json
{ "type": "user_input", "payload": { "text": "Hello" } }
{ "type": "ping" }
```

**サーバー → クライアント**
```json
{ "type": "connected", "clientId": "ws-client-1-xxx", "message": "Connected" }
{ "type": "agent_event", "event": { "type": "stream:text_delta", "payload": {...} } }
{ "type": "pong", "timestamp": 1234567890 }
{ "type": "error", "error": "Invalid JSON" }
```

### 4.2 HTTP プロトコル

**POST /api/input**
```json
// Request
{ "text": "Hello", "requestId": "optional-id" }
// Response
{ "status": "accepted", "requestId": "http-xxx" }
```

**GET /api/response?requestId=xxx**
```json
{ "requestId": "xxx", "events": [...], "complete": true }
```

---

## フェーズ 5：検証結果 ✅

### 5.1 コンパイル検証

| 検証項目 | 結果 |
|---------|------|
| `pnpm install` | ✅ 2つの新規パッケージ (ws, @types/ws) |
| `pnpm build` | ✅ 全11パッケージのコンパイル成功 |
| `pnpm test` | ✅ 82テストすべて通過 (新規26テスト追加) |

### 5.2 機能検証

**WebSocket テスト**
```javascript
const ws = new WebSocket("ws://localhost:8080/ws");
ws.onmessage = (e) => console.log(JSON.parse(e.data));
ws.onopen = () => ws.send(JSON.stringify({
  type: "user_input",
  payload: { text: "Hello!" }
}));
```

**HTTP テスト**
```bash
# 入力の送信
curl -X POST http://localhost:3000/api/input 
  -H "Content-Type: application/json" 
  -d '{"text": "Hello"}'

# 応答のポーリング
curl "http://localhost:3000/api/response?requestId=<id>"
```

### 5.3 複数 UI 同時出力の検証

1. エージェントを起動 (stdio + WebSocket)
2. WebSocket クライアントを接続
3. stdio を通じてメッセージを入力
4. **両方の UI が同じイベントを受信する**ことを確認

---

## ファイルリスト

### 新規ファイル (8個)

| パス | 説明 |
|------|------|
| `openstarry_plugin/transport-websocket/package.json` | WebSocket プラグインパッケージ設定 |
| `openstarry_plugin/transport-websocket/tsconfig.json` | TS 設定 |
| `openstarry_plugin/transport-websocket/src/index.ts` | 核心的な実装 |
| `openstarry_plugin/transport-websocket/src/index.test.ts` | ユニットテスト (10 tests) |
| `openstarry_plugin/transport-http/package.json` | HTTP プラグインパッケージ設定 |
| `openstarry_plugin/transport-http/tsconfig.json` | TS 設定 |
| `openstarry_plugin/transport-http/src/index.ts` | 核心的な実装 |
| `openstarry_plugin/transport-http/src/index.test.ts` | ユニットテスト (16 tests) |

### 修正ファイル (2個)

| パス | 修正 |
|------|------|
| `apps/runner/package.json` | プラグインの依存関係を追加 |
| `openstarry/tsconfig.json` | references を追加 |

---

## 実行順序

```
フェーズ 1 (WebSocket) ──→ フェーズ 2 (HTTP) ──→ フェーズ 3 (Config) ──→ フェーズ 4 (Protocol) ──→ フェーズ 5 (Test)
      ↓                      ↓                   ↓                    ↓
    ビルド                  ビルド              pnpm install         検証
```

---

## リスク評価

| リスク | 緩和策 |
|------|----------|
| WebSocket 接続のリーク | close/error イベント処理の徹底 |
| HTTP 応答バッファの肥大化 | bufferSize の上限設定と自動クリーンアップ |
| 複数 UI 間でのイベント順序の不一致 | TransportBridge の同期ブロードキャストにより順序を保証 |
| プラグインのロード順序問題 | Listener/UI の start() はすべてのプラグインがロードされた後に実行される |

---

## アーキテクチャの検証

この実装により、 Plan04 で導入された IUI/IListener 分離アーキテクチャの拡張性が検証されました：

1. **コアを修正せずにトランスポートチャネルを追加可能** — プラグインを追加するだけ
2. **複数 UI への同時出力** — TransportBridge のブロードキャストメカニズムが正常に動作
3. **ターゲット指定返信メカニズム** — replyTo フィールドにより精密なルーティングが可能
4. **プラグインの隔離性が良好** — WebSocket と HTTP プラグインがそれぞれ独立
