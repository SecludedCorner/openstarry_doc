# 插件範例：MCP (消息通信協議)

本文件展示如何遵循「一切皆插件」架構，實現代理人間的通訊協議 (MCP)。

## 架構視圖

MCP 功能被拆分為兩個部分：
1.  **Listener (`MCPListener`)**: 長時運行，負責**收**消息。
2.  **Tool (`MCPSender`)**: 被動調用，負責**發**消息。

兩者都依賴 Daemon 提供的 **Shared Broker Infrastructure**。

---

### 1. MCP Listener (接收端)

```typescript
import { Listener, PluginContext, AgentEvent } from '@openstarry/core/interfaces';

export class MCPListener implements Listener {
  id = 'mcp-listener';
  private broker: any;
  private queueName: string;

  async init(context: PluginContext): Promise<void> {
    // 1. 獲取基礎設施中的 Message Broker 實例
    this.broker = context.infrastructure.getService('message_broker');
    // 2. 確定自己的訂閱頻道 (通常基於 Agent ID)
    this.queueName = `agent:${context.agentId}:inbox`;
  }

  async start(eventQueue: EventQueue): Promise<void> {
    // 3. 訂閱消息
    await this.broker.subscribe(this.queueName, (rawMessage: any) => {
      
      // 4. 反序列化與驗證
      const mcpPacket = JSON.parse(rawMessage.content);
      
      // 5. 轉換為標準 AgentEvent
      // 注意：這裡我們從 MCP 封包中提取 traceId，確保鏈路不斷
      const event: AgentEvent = {
        source: 'mcp',
        type: 'mcp_message',
        payload: mcpPacket.body,
        traceId: mcpPacket.metadata?.traceId || generateNewTraceId(), // 繼承或生成
        timestamp: Date.now()
      };

      // 6. 推入核心隊列
      eventQueue.push(event);
    });
  }

  async stop(): Promise<void> {
    await this.broker.unsubscribe(this.queueName);
  }
}
```

---

### 2. MCP Sender Tool (發送端)

```typescript
import { Tool, PluginContext, AgentContext } from '@openstarry/core/interfaces';

export class MCPSenderTool implements Tool {
  id = 'mcp-sender';
  name = 'mcp:send';
  description = 'Sends a message to another agent.';
  
  parameters = {
    type: 'object',
    properties: {
      target_agent_id: { type: 'string' },
      content: { type: 'string' }
    },
    required: ['target_agent_id', 'content']
  };

  private broker: any;

  async init(context: PluginContext): Promise<void> {
    this.broker = context.infrastructure.getService('message_broker');
  }

  async execute(agentContext: AgentContext, args: any): Promise<any> {
    const { target_agent_id, content } = args;

    // 1. 組裝 MCP 封包
    const packet = {
      head: {
        sender: agentContext.agentId,
        receiver: target_agent_id,
        timestamp: Date.now()
      },
      metadata: {
        // 關鍵：透傳當前的 traceId，實現分佈式追蹤
        traceId: agentContext.traceId
      },
      body: {
        content: content
      }
    };

    // 2. 發布消息到目標隊列
    const targetQueue = `agent:${target_agent_id}:inbox`;
    await this.broker.publish(targetQueue, JSON.stringify(packet));

    return { success: true, status: "Message sent" };
  }
}
```

## 總結

*   **TraceID Context Propagation**: 這是最重要的細節。接收端從封包讀取 TraceID，發送端將 TraceID 寫入封包。這使得我們可以跨代理人追蹤整個任務鏈。
*   **Infrastructure Dependency**: 插件本身不包含 RabbitMQ/Redis 的驅動代碼，而是使用 `context.infrastructure` 提供的統一介面。這讓核心可以輕鬆切換底層 MQ 實現。