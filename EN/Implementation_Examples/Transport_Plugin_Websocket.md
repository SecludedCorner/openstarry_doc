# Implementation Example: WebSocket Transport Plugin

This document provides a detailed explanation of how to develop a WebSocket-based transport plugin. This plugin serves as a real-time communication bridge between the Agent Core and external clients, such as React frontends or mobile apps.

## Core Concepts

In the OpenStarry architecture, the **Transport Layer** is treated as a type of plugin. The Agent Core itself is "Headless" and "Protocol Agnostic." The WebSocket plugin is responsible for converting network protocol data streams into events that the Core can understand.

### Plugin Responsibilities
1.  **Lifecycle Management**: Establishes the WebSocket server when the Agent starts and gracefully closes connections when the Agent stops.
2.  **Inbound Transformation**: Receives WebSocket messages and injects them into the event queue via `ctx.pushInput()`.
3.  **Outbound Transformation**: Receives `AgentEvent`s through `IUI.onEvent()` and sends WebSocket messages to the client.
4.  **Session Management**: Manages multiple client connections, supporting targeted replies (`replyTo`) and broadcast modes.

---

## Implementation Code Example

The following is a simplified implementation based on OpenStarry's Factory Pattern, corresponding to the actual `@openstarry-plugin/transport-websocket`.

### 1. Package Structure

```
transport-websocket/
├── package.json
├── tsconfig.json
└── src/
    ├── index.ts        # Plugin factory function + Listener + UI
    └── index.test.ts   # Unit tests
```

### 2. Plugin Implementation (`src/index.ts`)

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

// ─── WebSocket UI (Form / Rupa) ───
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

      // Check if it's a targeted reply
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

### Key Design Points

1. **Factory Pattern**: The plugin exports a `createWebSocketPlugin()` function that returns an `IPlugin` object. The `factory()` method receives `IPluginContext` and returns `PluginHooks`.
2. **Sensation + Form**: A single plugin provides both `IListener` (input) and `IUI` (output) simultaneously, registered via the `listeners` and `ui` arrays in `PluginHooks`.
3. **ctx.pushInput()**: The Listener does not call Core APIs directly but injects input into the event queue via `IPluginContext.pushInput()`.
4. **replyTo Targeted Replies**: Each client is assigned a unique `clientId`. Input events carry `replyTo: clientId`. During UI push, if the event contains `replyTo`, it is sent only to the corresponding client.

---

## Communication Protocol Specification

To facilitate integration for frontend developers, we define the following JSON protocol:

### Client → Server (Upstream)

1.  **Send Message**
    ```json
    {
      "type": "user_input",
      "payload": { "text": "Help me analyze this document" }
    }
    ```

2.  **Ping/Heartbeat**
    ```json
    { "type": "ping" }
    ```

### Server → Client (Downstream)

1.  **Connection Confirmation**
    ```json
    {
      "type": "connected",
      "clientId": "ws-client-1-1707300000000",
      "message": "Connected to OpenStarry Agent"
    }
    ```

2.  **Agent Event**
    ```json
    {
      "type": "agent_event",
      "event": {
        "type": "RESPONSE",
        "timestamp": 1707300000000,
        "payload": { "content": "Sure, I am reading the document..." }
      }
    }
    ```

3.  **Pong Response**
    ```json
    { "type": "pong", "timestamp": 1707300000000 }
    ```

4.  **Error**
    ```json
    { "type": "error", "error": "Invalid JSON" }
    ```

## agent.json Configuration Example

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

## Security Considerations

In production environments, the WebSocket Plugin should consider:
1.  **Authentication**: Validate JWTs by checking the `?token=...` parameter in the URL or HTTP headers during the `connection` event.
2.  **TLS**: It is recommended to handle `wss://` encryption at the Nginx reverse proxy layer, with the plugin listening only on local `ws://`.
