# 開發者指南：插件載入規範與移植 (Plugin Loading & Migration Guide)

本文件詳細說明 OpenStarry 的插件載入規範，並提供將現有 Node.js 模組或第三方插件移植到 OpenStarry 生態系的具體步驟。

## 1. 核心載入規範 (Loading Specification)

OpenStarry 採用 **「工廠模式 (Factory Pattern)」** 與 **「依賴注入 (Dependency Injection)」** 來載入插件。

### 1.1 物理結構
插件必須是一個標準的 NPM Package，並且其 `package.json` **必須** 包含 `openstarry` 元數據欄位。

```json
{
  "name": "@openstarry-plugin/my-plugin",
  "main": "dist/index.js",
  "dependencies": {
    "@openstarry/sdk": "^1.0.0"
  },
  "openstarry": {
    "type": "aggregate", // 或 'tool', 'provider'
    "components": {
      "providers": ["MyProvider"],
      "tools": ["LoginTool"]
    }
  }
}
```

### 1.2 入口點 (Entry Point)
`main` 指向的文件必須導出一個 `initialize` 函數或實現了 `IPlugin` 介面的類別。

```typescript
import { IPlugin, IPluginContext } from '@openstarry/sdk';

export default class MyPlugin implements IPlugin {
  async initialize(context: IPluginContext) {
    // 在這裡進行初始化與註冊
  }
}
```

---

## 2. 移植指南 (Migration Guide)

如果您有一個現成的功能模組（例如來自其他 AI 框架的 Provider），請遵循以下步驟進行移植。

### 步驟 1：依賴替換
將原本依賴的框架（如 `@ai-core/core`, `langchain`）移除，替換為 `@openstarry/sdk`。
*   **Context:** 使用 `IAgentContext` 替代原本的上下文對象。
*   **Logger:** 使用 `context.logger` 替代 `console.log`。

### 步驟 2：介面適配 (Adapter Pattern)
OpenStarry 的介面可能與原框架不同。您需要編寫一個 Adapter 類別。

*   **Provider:** 實作 `IProvider`。將 OpenStarry 的 `generate(prompt)` 轉換為原模組需要的格式（如 `chat(messages)`）。
*   **Tool:** 實作 `ITool`。將 `execute(args)` 映射到原函數。

### 步驟 3：指令轉工具
如果原模組註冊了 CLI 指令（如 `/login`），請將其轉換為 **`ITool`**。
*   這樣不僅 CLI 可以用，Agent 也可以在思考過程中決定「我需要登入」。

### 步驟 4：輸入推送 (pushInput 取代建構式回調)
在舊版本中，部分插件透過建構式回調 (constructor callback) 將使用者輸入注入 Agent。現在請改用 `IPluginContext.pushInput` 方法，由插件主動向 Agent 推送輸入訊息：
```typescript
// ❌ 舊方式：透過建構式回調
constructor(onInput: (msg: string) => void) { ... }

// ✅ 新方式：透過 context.pushInput
context.pushInput({ role: 'user', content: userMessage });
```

### 步驟 5：配置注入
不要從 `process.env` 讀取配置。請從 `context.config` 讀取。
```typescript
// ❌ 舊代碼
const key = process.env.API_KEY;

// ✅ OpenStarry 代碼
const key = context.config.apiKey;
```

---

## 3. 範例：將 OAuth Provider 移植

假設您有一個 `GeminiOAuthManager` 類別。

1.  **封裝為 Plugin:** 建立 `GeminiOAuthPlugin` 實作 `IPlugin`。
2.  **初始化:** 在 `initialize()` 中實例化 `GeminiOAuthManager`。
3.  **註冊 Provider:** 建立 `GeminiProvider` 類別，內部呼叫 Manager 的 API，並透過 `context.registerProvider()` 註冊。
4.  **註冊 Tool:** 建立 `LoginTool`，內部呼叫 `manager.startOAuthFlow()`，並透過 `context.registerTool()` 註冊。

這樣，一個外部模組就完美融入 OpenStarry 的五蘊架構了。
