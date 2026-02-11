# 03. 插件接口技术规范 (Plugin Interface Specifications)

本文件精确定义了 OpenStarry 插件系统的技术契约。插件开发者必须实现以下接口，以确保与微内核的无缝对接。

## 1. 基础插件生命周期 (Base Plugin)

所有插件必须继承或实现基础生命周期方法。

```typescript
interface IOpenStarryPlugin {
  id: string;
  version: string;
  author?: string;

  // 初始化时注入 Core API 代理
  initialize(host: ICoreHost): Promise<void>;

  // 核心关闭或插件卸载时调用
  shutdown(): Promise<void>;
}
```

---

## 2. 五蕴插件接口 (The Five Aggregates Interfaces)

### 2.1 UI (色蕴 - Rupa)
定义用户界面组件。
```typescript
interface IUIComponent {
  id: string;
  type: 'terminal' | 'web_view' | 'widget';
  render(data: any): void;
  onInput(callback: (val: string) => void): void;
}
```

### 2.2 Listener (受蕴 - Vedana)
定义外部事件的感应器。
```typescript
interface IEventListener {
  id: string;
  start(): Promise<void>;
  stop(): Promise<void>;
  // 将外部信号封装为标准 Event 并推入核心队列
  emit(event: IOpenStarryEvent): void;
}
```

### 2.3 Provider (想蕴 - Samjna)
定义认知的后端（LLM）。
```typescript
interface ILLMProvider {
  id: string;
  generateResponse(messages: IMessage[], tools?: ITool[]): Promise<ILLMResult>;
  streamResponse(messages: IMessage[]): AsyncIterable<string>;
}
```

### 2.4 Tool (行蕴 - Samskara)
定义具体的执行能力。
```typescript
interface IAgentTool {
  name: string;
  description: string;
  parameters: JSONSchema7; // 使用标准 JSON Schema 定义参数
  execute(args: any, context: IExecutionContext): Promise<IToolResult>;
}
```

### 2.5 Guide (识蕴 - Vijnana)
这是赋予 Agent 意识与逻辑的关键接口。它是唯一被允许处理「痛觉诠释」的地方。

```typescript
interface IAgentGuide {
  id: string;

  // 注入人设与核心逻辑
  getSystemInstructions(): string;

  // 痛觉诠释 Hooks:
  // 当核心返回标准化错误时，Guide 负责将其转化为 LLM 可理解的意识信号
  interpretPain?(error: IStandardError): string;

  // 决定是否需要进入反思模式
  shouldReflect(lastHistory: IMessage[]): boolean;
}
```

---

## 3. 安全与隔离规范

1.  **沙盒限制**: 插件代码不允许直接访问 `process.env` 或执行 `fs.rm`。所有敏感操作必须通过 `ICoreHost` 提供的代理 API 进行。
2.  **资源限制**: 核心将监控插件的内存与 CPU 消耗。超过阈值的插件将被强制 `shutdown` 并转换为 `ERROR` 状态。
3.  **无状态原则**: 插件应尽量保持无状态 (Stateless)。所有持久化需求必须通过核心的 `StateManager` 服务完成。
