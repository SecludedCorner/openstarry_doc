# Plugin Example: MCP (Message Communication Protocol)

This document demonstrates how to implement an inter-agent communication protocol (MCP) following the "Everything is a Plugin" architecture.

## Architectural View

MCP functionality is split into two components:
1.  **Listener (`MCPListener`)**: Long-running, responsible for **receiving** messages.
2.  **Tool (`MCPSender`)**: Passively invoked, responsible for **sending** messages.

Both rely on the **Shared Broker Infrastructure** provided by the Daemon.

---

### 1. MCP Listener (Receiver)

```typescript
import { Listener, PluginContext, AgentEvent } from '@openstarry/core/interfaces';

export class MCPListener implements Listener {
  id = 'mcp-listener';
  private broker: any;
  private queueName: string;

  async init(context: PluginContext): Promise<void> {
    // 1. Obtain the Message Broker instance from the infrastructure
    this.broker = context.infrastructure.getService('message_broker');
    // 2. Determine the subscription channel (usually based on Agent ID)
    this.queueName = `agent:${context.agentId}:inbox`;
  }

  async start(eventQueue: EventQueue): Promise<void> {
    // 3. Subscribe to messages
    await this.broker.subscribe(this.queueName, (rawMessage: any) => {
      
      // 4. Deserialization and validation
      const mcpPacket = JSON.parse(rawMessage.content);
      
      // 5. Convert to standard AgentEvent
      // Note: We extract the traceId from the MCP packet to ensure trace continuity
      const event: AgentEvent = {
        source: 'mcp',
        type: 'mcp_message',
        payload: mcpPacket.body,
        traceId: mcpPacket.metadata?.traceId || generateNewTraceId(), // Inherit or generate
        timestamp: Date.now()
      };

      // 6. Push to Core event queue
      eventQueue.push(event);
    });
  }

  async stop(): Promise<void> {
    await this.broker.unsubscribe(this.queueName);
  }
}
```

---

### 2. MCP Sender Tool (Sender)

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

    // 1. Assemble MCP packet
    const packet = {
      head: {
        sender: agentContext.agentId,
        receiver: target_agent_id,
        timestamp: Date.now()
      },
      metadata: {
        // Critical: Propagate the current traceId for distributed tracing
        traceId: agentContext.traceId
      },
      body: {
        content: content
      }
    };

    // 2. Publish message to target queue
    const targetQueue = `agent:${target_agent_id}:inbox`;
    await this.broker.publish(targetQueue, JSON.stringify(packet));

    return { success: true, status: "Message sent" };
  }
}
```

## Summary

*   **TraceID Context Propagation**: This is the most crucial detail. The receiver reads the TraceID from the packet, and the sender writes the TraceID into the packet. This allows us to trace the entire task chain across multiple agents.
*   **Infrastructure Dependency**: The plugin itself does not contain RabbitMQ/Redis driver code; instead, it uses the unified interface provided by `context.infrastructure`. This allows the core to easily switch the underlying MQ implementation.
