# 實作範例：WebSocket 傳輸插件 (WebSocket Transport Plugin)

本文件詳細說明如何開發一個基於 WebSocket 的傳輸插件。此插件充當 Agent Core 與外部客戶端（如 React 前端、移動 App）之間的即時通訊橋樑。

## 核心概念

在 OpenStarry 架構中，**傳輸層 (Transport Layer)** 被視為插件的一種。Agent Core 本身是「無頭 (Headless)」且「協議無關 (Protocol Agnostic)」的。WebSocket 插件負責將網絡協議的數據流轉換為 Core 能理解的事件。

### 插件職責
1.  **生命週期管理：** 在 Agent 啟動時建立 WebSocket 服務器，在 Agent 停止時優雅關閉連接。
2.  **入向轉換 (Inbound)：** 接收 WebSocket 消息 → 透過 `ctx.pushInput()` 注入事件隊列。
3.  **出向轉換 (Outbound)：** `IUI.onEvent()` 接收 AgentEvent → 發送 WebSocket 消息給客戶端。
4.  **會話管理：** 管理多個客戶端連接，支援定向回覆 (replyTo) 與廣播模式。

---

## 代碼實作範例

以下是基於 OpenStarry 工廠模式 (Factory Pattern) 的簡化實作，對應實際的 `@openstarry-plugin/transport-websocket`。

### 1. 套件結構

```
transport-websocket/
├── package.json
├── tsconfig.json
└── src/
    ├── index.ts        # 插件工廠函數 + Listener + UI
    └── index.test.ts   # 單元測試
```

### 2. 插件實作 (`src/index.ts`)

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

### 關鍵設計重點

1. **工廠模式 (Factory Pattern)：** 插件匯出 `createWebSocketPlugin()` 函數，回傳 `IPlugin` 物件。`factory()` 方法接收 `IPluginContext`，回傳 `PluginHooks`。
2. **受蘊 + 色蘊：** 單一插件同時提供 `IListener`（接收輸入）和 `IUI`（推送輸出），透過 `PluginHooks` 的 `listeners` 和 `ui` 陣列註冊。
3. **ctx.pushInput()：** Listener 不直接呼叫 Core API，而是透過 `IPluginContext.pushInput()` 將輸入注入事件隊列。
4. **replyTo 定向回覆：** 每個客戶端分配唯一 `clientId`，輸入事件攜帶 `replyTo: clientId`。UI 推送時若事件含 `replyTo`，僅發送給對應客戶端。

---

## 通訊協議規範 (Protocol Spec)

為了讓前端開發者容易對接，我們定義以下 JSON 協議：

### Client → Server (Upstream)

1.  **發送消息**
    ```json
    {
      "type": "user_input",
      "payload": { "text": "幫我分析這個文件" }
    }
    ```

2.  **Ping/心跳**
    ```json
    { "type": "ping" }
    ```

### Server → Client (Downstream)

1.  **連線確認**
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
        "payload": { "content": "好的，我正在讀取文件..." }
      }
    }
    ```

3.  **Pong 回應**
    ```json
    { "type": "pong", "timestamp": 1707300000000 }
    ```

4.  **錯誤**
    ```json
    { "type": "error", "error": "Invalid JSON" }
    ```

## agent.json 配置範例

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

在生產環境中，WebSocket Plugin 應注意：
1.  **鑑權 (Authentication)：** 在 `connection` 事件中，應檢查 URL 參數中的 `?token=...` 或 HTTP Header，驗證 JWT。
2.  **TLS：** 建議在 Nginx 反向代理層處理 `wss://` 加密，插件只監聽本地 `ws://`。
