# 03. 插件介面技術規範 (Plugin Interface Specifications)

本文件精確定義了 OpenStarry 插件系統的技術契約。插件開發者必須實現以下介面，以確保與微內核的無縫對接。

## 1. 基礎插件生命週期 (Base Plugin)

所有插件必須繼承或實現基礎生命週期方法。

```typescript
interface IOpenStarryPlugin {
  id: string;
  version: string;
  author?: string;
  
  // 初始化時注入 Core API 代理
  initialize(host: ICoreHost): Promise<void>;
  
  // 核心關閉或插件解除安裝時調用
  shutdown(): Promise<void>;
}
```

---

## 2. 五蘊插件介面 (The Five Aggregates Interfaces)

### 2.1 UI (色蘊 - Rupa)
定義用戶界面組件。
```typescript
interface IUIComponent {
  id: string;
  type: 'terminal' | 'web_view' | 'widget';
  render(data: any): void;
  onInput(callback: (val: string) => void): void;
}
```

### 2.2 Listener (受蘊 - Vedana)
定義外部事件的感應器。
```typescript
interface IEventListener {
  id: string;
  start(): Promise<void>;
  stop(): Promise<void>;
  // 將外部訊號封裝為標準 Event 並推入核心隊列
  emit(event: IOpenStarryEvent): void;
}
```

### 2.3 Provider (想蘊 - Samjna)
定義認知的後端（LLM）。
```typescript
interface ILLMProvider {
  id: string;
  generateResponse(messages: IMessage[], tools?: ITool[]): Promise<ILLMResult>;
  streamResponse(messages: IMessage[]): AsyncIterable<string>;
}
```

### 2.4 Tool (行蘊 - Samskara)
定義具體的執行能力。
```typescript
interface IAgentTool {
  name: string;
  description: string;
  parameters: JSONSchema7; // 使用標準 JSON Schema 定義參數
  execute(args: any, context: IExecutionContext): Promise<IToolResult>;
}
```

### 2.5 Guide (識蘊 - Vijnana)
這是賦予 Agent 意識與邏輯的關鍵介面。它是唯一被允許處理「痛覺詮釋」的地方。

```typescript
interface IAgentGuide {
  id: string;
  
  // 注入人設與核心邏輯
  getSystemInstructions(): string;
  
  // 痛覺詮釋 Hooks:
  // 當核心回傳標準化錯誤時，Guide 負責將其轉化為 LLM 可理解的意識訊號
  interpretPain?(error: IStandardError): string;
  
  // 決定是否需要進入反思模式
  shouldReflect(lastHistory: IMessage[]): boolean;
}
```

---

## 3. 安全與隔離規範

1.  **沙盒限制**: 插件代碼不允許直接訪問 `process.env` 或執行 `fs.rm`。所有敏感操作必須透過 `ICoreHost` 提供的代理 API 進行。
2.  **資源限制**: 核心將監控插件的內存與 CPU 消耗。超過閾值的插件將被強制 `shutdown` 並轉換為 `ERROR` 狀態。
3.  **無狀態原則**: 插件應盡量保持無狀態 (Stateless)。所有持久化需求必須透過核心的 `StateManager` 服務完成。
