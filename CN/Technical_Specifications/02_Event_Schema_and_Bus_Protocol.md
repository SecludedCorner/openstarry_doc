# 02. 事件总线协议规格 (Event Bus & Schema Protocol)

本文件定义了 OpenStarry 内部与边缘组件通信的标准事件格式。所有从传输插件 (Transport) 流向核心 (Core) 的数据，以及核心产生的反馈，都必须遵守此协议。

## 1. 核心事件结构 (Core Event Envelope)

每个事件都必须被封装在一个标准的信封对象中，以确保可追溯性与路由正确。

```typescript
interface IOpenStarryEvent {
  // 基础元数据
  readonly id: string;            // UUID v4
  readonly timestamp: number;     // Unix Timestamp (ms)
  readonly traceId: string;       // 用于追踪整个请求链条的唯一 ID

  // 路由信息
  readonly source: string;        // 来源组件 ID (例如 "transport-websocket-01")
  readonly sessionId: string;     // 会话 ID，用于多用户隔离
  readonly priority: number;      // 0 (最高, 系统紧急) 到 5 (最低, 后台任务)

  // 载荷
  readonly type: EventType;       // 事件类型
  readonly payload: any;          // 具体数据内容
}
```

## 2. 事件类型枚举 (EventType)

### 2.1 输入类 (Input Events)
*   `INPUT:USER_MESSAGE`: 用户输入的纯文本。
*   `INPUT:SYSTEM_COMMAND`: 系统级指令（如 `/stop`, `/reset`）。
*   `INPUT:SIGNAL`: 传感器或外部 Webhook 的触发信号。

### 2.2 核心状态类 (Kernel Events)
*   `KERNEL:TICK_START`: 执行循环开始一次迭代。
*   `KERNEL:STATE_CHANGE`: Agent 状态切换（如 `IDLE` -> `EXECUTING`）。
*   `KERNEL:ERROR`: 核心标准化后的错误输出。

### 2.3 执行类 (Execution Events)
*   `EXEC:TOOL_CALL`: Agent 请求执行工具。
*   `EXEC:TOOL_RESULT`: 工具执行后的结果反馈。

---

## 3. Payload 规范范例

### 3.1 `INPUT:USER_MESSAGE`
```json
{
  "type": "INPUT:USER_MESSAGE",
  "payload": {
    "text": "读取最近的系统日志",
    "mimeType": "text/plain",
    "metadata": { "platform": "discord" }
  }
}
```

### 3.2 `EXEC:TOOL_RESULT` (与痛觉机制相关的数据源)
```json
{
  "type": "EXEC:TOOL_RESULT",
  "payload": {
    "toolCallId": "call_abc123",
    "success": false,
    "error": {
      "code": "EPERM",
      "message": "Operation not permitted",
      "source": "fs-plugin"
    }
  }
}
```

## 4. 路由与传播规则

1.  **优先级处理**: 核心队列必须优先处理 `priority: 0` 的事件。
2.  **幂等性**: 核心应根据 `id` 丢弃在滑动窗口内重复收到的事件。
3.  **TraceId 透传**: 任何由事件引发的新事件（例如 Input 引发的 ToolCall），必须继承原始事件的 `traceId`。
