# 实作示例：WebSocket 传输插件 (WebSocket Transport Plugin)

本文档详细说明如何开发一个基于 WebSocket 的传输插件。此插件充当 Agent Core 与外部客户端（如 React 前端、移动 App）之间的即时通讯桥梁。

## 核心概念

在 OpenStarry 架构中，**传输层 (Transport Layer)** 被视为插件的一种。Agent Core 本身是「无头 (Headless)」且「协议无关 (Protocol Agnostic)」的。WebSocket 插件负责将网络协议的数据流转换为 Core 能理解的事件。

### 插件职责
1.  **生命周期管理：** 在 Agent 启动时建立 WebSocket 服务器，在 Agent 停止时优雅关闭连接。
2.  **入向转换 (Inbound)：** 接收 WebSocket 消息 → 通过 `ctx.pushInput()` 注入事件队列。
3.  **出向转换 (Outbound)：** `IUI.onEvent()` 接收 AgentEvent → 发送 WebSocket 消息给客户端。
4.  **会话管理：** 管理多个客户端连接，支持定向回复 (replyTo) 与广播模式。

---

## 代码实作示例

以下是基于 OpenStarry 工厂模式 (Factory Pattern) 的简化实作，对应实际的 `@openstarry-plugin/transport-websocket`。

### 1. 套件结构

```
transport-websocket/
├── package.json
├── tsconfig.json
└── src/
    ├── index.ts        # 插件工厂函数 + Listener + UI
    └── index.test.ts   # 单元测试
```

### 2. 插件实作 (`src/index.ts`)

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

// ─── WebSocket UI (色蕴) ───
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

      // 检查是否为定向回复
      const payload = event.payload as Record<string, unknown> | undefined;
      const replyTo = payload?.replyTo as string | undefined;

      if (replyTo && connections.has(replyTo)) {
        // 定向回复给特定客户端
        const conn = connections.get(replyTo)!;
        if (conn.ws.readyState === WebSocket.OPEN) {
          conn.ws.send(message);
        }
      } else {
        // 广播给所有连线的客户端
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

            // 发送欢迎消息
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

### 关键设计重点

1. **工厂模式 (Factory Pattern)：** 插件导出 `createWebSocketPlugin()` 函数，回传 `IPlugin` 对象。`factory()` 方法接收 `IPluginContext`，回传 `PluginHooks`。
2. **受蕴 + 色蕴：** 单一插件同时提供 `IListener`（接收输入）和 `IUI`（推送输出），通过 `PluginHooks` 的 `listeners` 和 `ui` 数组注册。
3. **ctx.pushInput()：** Listener 不直接调用 Core API，而是通过 `IPluginContext.pushInput()` 将输入注入事件队列。
4. **replyTo 定向回复：** 每个客户端分配唯一 `clientId`，输入事件携带 `replyTo: clientId`。UI 推送时若事件含 `replyTo`，仅发送给对应客户端。

---

## 通讯协议规范 (Protocol Spec)

为了让前端开发者容易对接，我们定义以下 JSON 协议：

### Client → Server (Upstream)

1.  **发送消息**
    ```json
    {
      "type": "user_input",
      "payload": { "text": "帮我分析这个文件" }
    }
    ```

2.  **Ping/心跳**
    ```json
    { "type": "ping" }
    ```

### Server → Client (Downstream)

1.  **连线确认**
    ```json
    {
      "type": "connected",
      "clientId": "ws-client-1-1707300000000",
      "message": "Connected to OpenStarry Agent"
    }
    ```

2.  **Agent 事件**
    ```json
    {
      "type": "agent_event",
      "event": {
        "type": "RESPONSE",
        "timestamp": 1707300000000,
        "payload": { "content": "好的，我正在读取文件..." }
      }
    }
    ```

3.  **Pong 回应**
    ```json
    { "type": "pong", "timestamp": 1707300000000 }
    ```

4.  **错误**
    ```json
    { "type": "error", "error": "Invalid JSON" }
    ```

## agent.json 配置范例

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

## 安全考量

在生产环境中，WebSocket Plugin 应注意：
1.  **鉴权 (Authentication)：** 在 `connection` 事件中，应检查 URL 参数中的 `?token=...` 或 HTTP Header，验证 JWT。
2.  **TLS：** 建议在 Nginx 反向代理层处理 `wss://` 加密，插件只监听本地 `ws://`。
