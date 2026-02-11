# プラグイン例： MCP (メッセージ通信プロトコル)

このドキュメントでは、「すべてはプラグインである」アーキテクチャに従って、エージェント間の通信プロトコル (MCP) をどのように実装するかを示します。

## アーキテクチャの視点

MCP 機能は、以下の2つの部分に分割されます：
1.  **Listener ( `MCPListener` )**: 長時間実行され、メッセージの **受信** を担当します。
2.  **Tool ( `MCPSender` )**: 受動的に呼び出され、メッセージの **送信** を担当します。

両者は、デーモンが提供する **共有ブローカー・インフラストラクチャ (Shared Broker Infrastructure)** に依存します。

---

### 1. MCP リスナー (受信側)

```typescript
import { Listener, PluginContext, AgentEvent } from '@openstarry/core/interfaces';

export class MCPListener implements Listener {
  id = 'mcp-listener';
  private broker: any;
  private queueName: string;

  async init(context: PluginContext): Promise<void> {
    // 1. インフラ内の Message Broker インスタンスを取得
    this.broker = context.infrastructure.getService('message_broker');
    // 2. 自身の購読チャネルを決定（通常はエージェント ID に基づく）
    this.queueName = `agent:${context.agentId}:inbox`;
  }

  async start(eventQueue: EventQueue): Promise<void> {
    // 3. メッセージを購読
    await this.broker.subscribe(this.queueName, (rawMessage: any) => {
      
      // 4. デシリアライズと検証
      const mcpPacket = JSON.parse(rawMessage.content);
      
      // 5. 標準の AgentEvent に変換
      // 注意：ここでは MCP パケットから traceId を抽出し、リンクが途切れないようにします
      const event: AgentEvent = {
        source: 'mcp',
        type: 'mcp_message',
        payload: mcpPacket.body,
        traceId: mcpPacket.metadata?.traceId || generateNewTraceId(), // 継承または生成
        timestamp: Date.now()
      };

      // 6. コアのキューにプッシュ
      eventQueue.push(event);
    });
  }

  async stop(): Promise<void> {
    await this.broker.unsubscribe(this.queueName);
  }
}
```

---

### 2. MCP 送信ツール (送信側)

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

    // 1. MCP パケットを組み立て
    const packet = {
      head: {
        sender: agentContext.agentId,
        receiver: target_agent_id,
        timestamp: Date.now()
      },
      metadata: {
        // 重要：現在の traceId を透過させ、分散トレーシングを実現
        traceId: agentContext.traceId
      },
      body: {
        content: content
      }
    };

    // 2. ターゲットのキューにメッセージをパブリッシュ
    const targetQueue = `agent:${target_agent_id}:inbox`;
    await this.broker.publish(targetQueue, JSON.stringify(packet));

    return { success: true, status: "Message sent" };
  }
}
```

## まとめ

*   **TraceID Context Propagation**: これは最も重要な詳細です。受信側はパケットから TraceID を読み取り、送信側はパケットに TraceID を書き込みます。これにより、エージェントをまたいでタスクチェーン全体を追跡できるようになります。
*   **Infrastructure Dependency**: プラグイン自体には RabbitMQ や Redis のドライバコードは含まれず、 `context.infrastructure` が提供する統一インターフェースを使用します。これにより、コアは下層の MQ 実装を簡単に切り替えることができます。
