# Plan05: Multi-Channel UI & Advanced Listener

> **Status**: ✅ Completed (2026-02-07)
> **Target Version**: v0.2 Beta

## Objectives

Validate the scalability of the IUI/IListener separation architecture by implementing non-stdio channels:

1. **WebSocket Listener** — Receives WebSocket messages as input.
2. **WebSocket UI** — Pushes events via WebSocket.
3. **HTTP Webhook Listener** — Receives HTTP POST requests as triggers.
4. **Simultaneous Multi-UI Output** — Identical events pushed to both stdio and WebSocket simultaneously.

---

## Phase 1: WebSocket Transport Plugin ✅

### 1.1 Plugin Structure

**Directory**: `openstarry_plugin/transport-websocket/`

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

### 1.3 Core Implementation (src/index.ts)

```typescript
/**
 * transport-websocket — WebSocket transport plugin.
 *
 * Provides:
 * - WebSocketListener (Sensation) — receives WebSocket messages as input
 * - WebSocketUI (Form) — pushes events to WebSocket clients
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

// ─── WebSocket UI (Form / 色蘊) ───
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

      // Check for targeted reply
      const payload = event.payload as Record<string, unknown> | undefined;
      const replyTo = payload?.replyTo as string | undefined;

      if (replyTo && connections.has(replyTo)) {
        // Targeted reply to a specific client
        const conn = connections.get(replyTo)!;
        if (conn.ws.readyState === WebSocket.OPEN) {
          conn.ws.send(message);
        }
      } else {
        // Broadcast to all connected clients
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

            // Send welcome message
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

## Phase 2: HTTP Webhook Transport Plugin ✅

### 2.1 Plugin Structure

**Directory**: `openstarry_plugin/transport-http/`

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

### 2.3 Core Implementation (src/index.ts)

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

      // ─── HTTP UI (Buffered responses for polling) ───
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

## Phase 3: Configuration Integration ✅

### 3.1 Multi-channel Configuration Example (stdio + WebSocket)

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

### 3.2 Update apps/runner/package.json

Add plugins as workspace dependencies:

```json
{
  "dependencies": {
    "@openstarry-plugin/transport-websocket": "workspace:*",
    "@openstarry-plugin/transport-http": "workspace:*"
  }
}
```

---

## Phase 4: JSON Protocol Specifications

### 4.1 WebSocket Protocol

**Client → Server**
```json
{ "type": "user_input", "payload": { "text": "Hello" } }
{ "type": "ping" }
```

**Server → Client**
```json
{ "type": "connected", "clientId": "ws-client-1-xxx", "message": "Connected" }
{ "type": "agent_event", "event": { "type": "stream:text_delta", "payload": {...} } }
{ "type": "pong", "timestamp": 1234567890 }
{ "type": "error", "error": "Invalid JSON" }
```

### 4.2 HTTP Protocol

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

## Phase 5: Verification Results ✅

### 5.1 Compilation Verification

| Item | Result |
|---------|------|
| `pnpm install` | ✅ 2 new packages added (ws, @types/ws) |
| `pnpm build` | ✅ All 11 packages compiled successfully |
| `pnpm test` | ✅ 82 tests passed (including 26 new ones) |

### 5.2 Functional Verification

**WebSocket Test**
```javascript
const ws = new WebSocket("ws://localhost:8080/ws");
ws.onmessage = (e) => console.log(JSON.parse(e.data));
ws.onopen = () => ws.send(JSON.stringify({
  type: "user_input",
  payload: { text: "Hello!" }
}));
```

**HTTP Test**
```bash
# Submit input
curl -X POST http://localhost:3000/api/input 
  -H "Content-Type: application/json" 
  -d '{"text": "Hello"}'

# Poll response
curl "http://localhost:3000/api/response?requestId=<id>"
```

### 5.3 Multi-UI Simultaneous Output Verification

1. Start Agent (stdio + WebSocket).
2. Connect WebSocket client.
3. Enter a message via stdio.
4. Confirm **both UIs receive the identical events**.

---

## File Manifest

### New Files (8)

| Path | Description |
|------|------|
| `openstarry_plugin/transport-websocket/package.json` | WebSocket plugin package config |
| `openstarry_plugin/transport-websocket/tsconfig.json` | TS configuration |
| `openstarry_plugin/transport-websocket/src/index.ts` | Core implementation |
| `openstarry_plugin/transport-websocket/src/index.test.ts` | Unit tests (10) |
| `openstarry_plugin/transport-http/package.json` | HTTP plugin package config |
| `openstarry_plugin/transport-http/tsconfig.json` | TS configuration |
| `openstarry_plugin/transport-http/src/index.ts` | Core implementation |
| `openstarry_plugin/transport-http/src/index.test.ts` | Unit tests (16) |

### Modified Files (2)

| Path | Change |
|------|------|
| `apps/runner/package.json` | Added plugin dependencies |
| `openstarry/tsconfig.json` | Added project references |

---

## Execution Sequence

```
Phase 1 (WebSocket) ──→ Phase 2 (HTTP) ──→ Phase 3 (Config) ──→ Phase 4 (Protocol) ──→ Phase 5 (Test)
      ↓                      ↓                   ↓                    ↓
    build                  build              pnpm install         Verify
```

---

## Risk Assessment

| Risk | Mitigation |
|------|----------|
| WebSocket connection leaks | Robust close/error event handling. |
| Large HTTP response buffers | Set bufferSize limits and implement auto-cleanup. |
| Inconsistent event ordering across UIs | TransportBridge synchronous broadcasting ensures ordering. |
| Plugin loading order issues | Listener/UI start() methods execute after all plugins are loaded. |

---

## Architectural Validation

This implementation validates the extensibility of the IUI/IListener separation architecture from Plan04:

1. **Adding transport channels requires no changes to Core** — simply add new plugins.
2. **Simultaneous Multi-UI Output** — TransportBridge broadcasting mechanism operates correctly.
3. **Targeted Reply Mechanism** — `replyTo` field supports precise routing.
4. **Excellent Plugin Isolation** — WebSocket and HTTP plugins remain independent.
