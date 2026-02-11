# 插件示例：MCP (消息通信协议)

本文档展示如何遵循「一切皆插件」架构，实现代理人间的通讯协议 (MCP)。

## 架构视图

MCP 功能被拆分为两个部分：
1.  **Listener (`MCPListener`)**: 长时运行，负责**收**消息。
2.  **Tool (`MCPSender`)**: 被动调用，负责**发**消息。

两者都依赖 Daemon 提供的 **Shared Broker Infrastructure**。

---

### 1. MCP Listener (接收端)

```typescript
import { Listener, PluginContext, AgentEvent } from '@openstarry/core/interfaces';

export class MCPListener implements Listener {
  id = 'mcp-listener';
  private broker: any;
  private queueName: string;

  async init(context: PluginContext): Promise<void> {
    // 1. 获取基础设施中的 Message Broker 实例
    this.broker = context.infrastructure.getService('message_broker');
    // 2. 确定自己的订阅频道 (通常基于 Agent ID)
    this.queueName = `agent:${context.agentId}:inbox`;
  }

  async start(eventQueue: EventQueue): Promise<void> {
    // 3. 订阅消息
    await this.broker.subscribe(this.queueName, (rawMessage: any) => {
      
      // 4. 反序列化与验证
      const mcpPacket = JSON.parse(rawMessage.content);
      
      // 5. 转换为标准 AgentEvent
      // 注意：这里我们从 MCP 封包中提取 traceId，确保链路不断
      const event: AgentEvent = {
        source: 'mcp',
        type: 'mcp_message',
        payload: mcpPacket.body,
        traceId: mcpPacket.metadata?.traceId || generateNewTraceId(), // 继承或生成
        timestamp: Date.now()
      };

      // 6. 推入核心队列
      eventQueue.push(event);
    });
  }

  async stop(): Promise<void> {
    await this.broker.unsubscribe(this.queueName);
  }
}
```

---

### 2. MCP Sender Tool (发送端)

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

    // 1. 组装 MCP 封包
    const packet = {
      head: {
        sender: agentContext.agentId,
        receiver: target_agent_id,
        timestamp: Date.now()
      },
      metadata: {
        // 关键：透传当前的 traceId，实现分布式追踪
        traceId: agentContext.traceId
      },
      body: {
        content: content
      }
    };

    // 2. 发布消息到目标队列
    const targetQueue = `agent:${target_agent_id}:inbox`;
    await this.broker.publish(targetQueue, JSON.stringify(packet));

    return { success: true, status: "Message sent" };
  }
}
```

## 总结

*   **TraceID Context Propagation**: 这是最重要的细节。接收端从封包读取 TraceID，发送端将 TraceID 写入封包。这使得我们可以跨代理人追踪整个任务链。
*   **Infrastructure Dependency**: 插件本身不包含 RabbitMQ/Redis 的驱动代码，而是使用 `context.infrastructure` 提供的统一接口。这让核心可以轻松切换底层 MQ 实现。
