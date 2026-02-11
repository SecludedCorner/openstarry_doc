# Plan05: 多通道 UI 與進階 Listener

> **狀態**: ✅ 已完成 (2026-02-07)
> **目標版本**: v0.2 Beta

## 目標

驗證 IUI/IListener 分離架構的可擴展性，實作非 stdio 的通道：

1. **WebSocket Listener** — 接收 WebSocket 訊息作為輸入
2. **WebSocket UI** — 將事件透過 WebSocket 推送
3. **HTTP Webhook Listener** — 接收 HTTP POST 作為觸發
4. **多 UI 同時輸出** — 同一事件可同時推送到 stdio + WebSocket

---

## Phase 1: WebSocket 傳輸插件 ✅

### 1.1 插件結構

**目錄**: `openstarry_plugin/transport-websocket/`

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
    "clean": "rimraf --glob dist \"*.tsbuildinfo\"",
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

### 1.3 核心實作 (src/index.ts)

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

      // 檢查是否為定向回覆
      const payload = event.payload as Record<string, unknown> | undefined;
      const replyTo = payload?.replyTo as string | undefined;

      if (replyTo && connections.has(replyTo)) {
        // 定向回覆給特定客戶端
        const conn = connections.get(replyTo)!;
        if (conn.ws.readyState === WebSocket.OPEN) {
          conn.ws.send(message);
        }
      } else {
        // 廣播給所有連線的客戶端
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

            // 發送歡迎訊息
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

## Phase 2: HTTP Webhook 傳輸插件 ✅

### 2.1 插件結構

**目錄**: `openstarry_plugin/transport-http/`

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
    "clean": "rimraf --glob dist \"*.tsbuildinfo\"",
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

### 2.3 核心實作 (src/index.ts)

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

      // ─── HTTP UI (緩衝回應供輪詢) ───
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

## Phase 3: 配置整合 ✅

### 3.1 多通道配置範例 (stdio + WebSocket)

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

### 3.2 更新 apps/runner/package.json

新增插件為 workspace 依賴：

```json
{
  "dependencies": {
    "@openstarry-plugin/transport-websocket": "workspace:*",
    "@openstarry-plugin/transport-http": "workspace:*"
  }
}
```

---

## Phase 4: JSON 協議規範

### 4.1 WebSocket 協議

**客戶端 → 伺服器**
```json
{ "type": "user_input", "payload": { "text": "Hello" } }
{ "type": "ping" }
```

**伺服器 → 客戶端**
```json
{ "type": "connected", "clientId": "ws-client-1-xxx", "message": "Connected" }
{ "type": "agent_event", "event": { "type": "stream:text_delta", "payload": {...} } }
{ "type": "pong", "timestamp": 1234567890 }
{ "type": "error", "error": "Invalid JSON" }
```

### 4.2 HTTP 協議

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

## Phase 5: 驗證結果 ✅

### 5.1 編譯驗證

| 驗證項目 | 結果 |
|---------|------|
| `pnpm install` | ✅ 新增 2 個套件 (ws, @types/ws) |
| `pnpm build` | ✅ 全部 11 個套件編譯成功 |
| `pnpm test` | ✅ 82 個測試全數通過 (新增 26 個) |

### 5.2 功能驗證

**WebSocket 測試**
```javascript
const ws = new WebSocket("ws://localhost:8080/ws");
ws.onmessage = (e) => console.log(JSON.parse(e.data));
ws.onopen = () => ws.send(JSON.stringify({
  type: "user_input",
  payload: { text: "Hello!" }
}));
```

**HTTP 測試**
```bash
# 提交輸入
curl -X POST http://localhost:3000/api/input \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello"}'

# 輪詢回應
curl "http://localhost:3000/api/response?requestId=<id>"
```

### 5.3 多 UI 同時輸出驗證

1. 啟動 Agent (stdio + WebSocket)
2. 連接 WebSocket 客戶端
3. 透過 stdio 輸入訊息
4. 確認 **兩個 UI 都收到相同事件**

---

## 檔案清單

### 新增檔案 (8 個)

| 路徑 | 說明 |
|------|------|
| `openstarry_plugin/transport-websocket/package.json` | WebSocket 插件 package |
| `openstarry_plugin/transport-websocket/tsconfig.json` | TS 設定 |
| `openstarry_plugin/transport-websocket/src/index.ts` | 核心實作 |
| `openstarry_plugin/transport-websocket/src/index.test.ts` | 單元測試 (10 tests) |
| `openstarry_plugin/transport-http/package.json` | HTTP 插件 package |
| `openstarry_plugin/transport-http/tsconfig.json` | TS 設定 |
| `openstarry_plugin/transport-http/src/index.ts` | 核心實作 |
| `openstarry_plugin/transport-http/src/index.test.ts` | 單元測試 (16 tests) |

### 修改檔案 (2 個)

| 路徑 | 修改 |
|------|------|
| `apps/runner/package.json` | 新增插件依賴 |
| `openstarry/tsconfig.json` | 新增 references |

---

## 執行順序

```
Phase 1 (WebSocket) ──→ Phase 2 (HTTP) ──→ Phase 3 (Config) ──→ Phase 4 (Protocol) ──→ Phase 5 (Test)
      ↓                      ↓                   ↓                    ↓
    build                  build              pnpm install         驗證
```

---

## 風險評估

| 風險 | 緩解措施 |
|------|----------|
| WebSocket 連線洩漏 | 完善 close/error 事件處理 |
| HTTP 回應緩衝過大 | 設定 bufferSize 上限與自動清理 |
| 多 UI 事件順序不一致 | TransportBridge 同步廣播已保證順序 |
| 插件載入順序問題 | Listener/UI 的 start() 在所有插件載入後才執行 |

---

## 架構驗證

此實作驗證了 Plan04 中 IUI/IListener 分離架構的可擴展性：

1. **新增傳輸通道無需修改 Core** — 只需新增插件
2. **多 UI 同時輸出** — TransportBridge 廣播機制正常運作
3. **定向回覆機制** — replyTo 欄位支援精確路由
4. **插件隔離性良好** — WebSocket 和 HTTP 插件各自獨立
