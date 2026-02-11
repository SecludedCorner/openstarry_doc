# 03. Plugin Interface Technical Specifications

This document precisely defines the technical contracts of the OpenStarry plugin system. Plugin developers must implement the following interfaces to ensure seamless integration with the microkernel.

## 1. Base Plugin Lifecycle

All plugins must inherit or implement the base lifecycle methods.

```typescript
interface IOpenStarryPlugin {
  id: string;
  version: string;
  author?: string;

  // Core API proxy is injected during initialization
  initialize(host: ICoreHost): Promise<void>;

  // Called when the Core shuts down or the plugin is uninstalled
  shutdown(): Promise<void>;
}
```

---

## 2. The Five Aggregates Interfaces

### 2.1 UI (Rupa - Form)
Defines user interface components.
```typescript
interface IUIComponent {
  id: string;
  type: 'terminal' | 'web_view' | 'widget';
  render(data: any): void;
  onInput(callback: (val: string) => void): void;
}
```

### 2.2 Listener (Vedana - Sensation)
Defines sensors for external events.
```typescript
interface IEventListener {
  id: string;
  start(): Promise<void>;
  stop(): Promise<void>;
  // Encapsulates external signals as standard Events and pushes them to the Core queue
  emit(event: IOpenStarryEvent): void;
}
```

### 2.3 Provider (Samjna - Perception)
Defines the cognitive backend (LLM).
```typescript
interface ILLMProvider {
  id: string;
  generateResponse(messages: IMessage[], tools?: ITool[]): Promise<ILLMResult>;
  streamResponse(messages: IMessage[]): AsyncIterable<string>;
}
```

### 2.4 Tool (Samskara - Formation)
Defines concrete execution capabilities.
```typescript
interface IAgentTool {
  name: string;
  description: string;
  parameters: JSONSchema7; // Uses standard JSON Schema to define parameters
  execute(args: any, context: IExecutionContext): Promise<IToolResult>;
}
```

### 2.5 Guide (Vijnana - Consciousness)
This is the critical interface that endows an Agent with consciousness and logic. It is the only place permitted to handle "pain interpretation."

```typescript
interface IAgentGuide {
  id: string;

  // Injects persona and core logic
  getSystemInstructions(): string;

  // Pain Interpretation Hooks:
  // When the Core returns a standardized error, the Guide is responsible for
  // transforming it into a consciousness signal that the LLM can understand
  interpretPain?(error: IStandardError): string;

  // Determines whether reflection mode should be entered
  shouldReflect(lastHistory: IMessage[]): boolean;
}
```

---

## 3. Security and Isolation Specifications

1.  **Sandbox Restrictions**: Plugin code is not permitted to directly access `process.env` or execute `fs.rm`. All sensitive operations must be performed through the proxy API provided by `ICoreHost`.
2.  **Resource Limits**: The Core will monitor each plugin's memory and CPU consumption. Plugins that exceed the threshold will be forcibly `shutdown` and transitioned to an `ERROR` state.
3.  **Stateless Principle**: Plugins should remain stateless as much as possible. All persistence requirements must be fulfilled through the Core's `StateManager` service.
