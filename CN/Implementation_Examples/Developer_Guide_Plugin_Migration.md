# 开发者指南：插件载入规范与移植 (Plugin Loading & Migration Guide)

本文档详细说明 OpenStarry 的插件载入规范，并提供将现有 Node.js 模块或第三方插件移植到 OpenStarry 生态系统的具体步骤。

## 1. 核心载入规范 (Loading Specification)

OpenStarry 采用 **「工厂模式 (Factory Pattern)」** 与 **「依赖注入 (Dependency Injection)」** 来载入插件。

### 1.1 物理结构
插件必须是一个标准的 NPM Package，并且其 `package.json` **必须** 包含 `openstarry` 元数据字段。

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

### 1.2 入口点 (Entry Point)
`main` 指向的文件必须导出一个 `initialize` 函数或实现了 `IPlugin` 接口的类。

```typescript
import { IPlugin, IPluginContext } from '@openstarry/sdk';

export default class MyPlugin implements IPlugin {
  async initialize(context: IPluginContext) {
    // 在这里进行初始化与注册
  }
}
```

---

## 2. 移植指南 (Migration Guide)

如果您有一个现成的功能模块（例如来自其他 AI 框架的 Provider），请遵循以下步骤进行移植。

### 步骤 1：依赖替换
将原本依赖的框架（如 `@ai-core/core`, `langchain`）移除，替换为 `@openstarry/sdk`。
*   **Context:** 使用 `IAgentContext` 替代原本的上下文对象。
*   **Logger:** 使用 `context.logger` 替代 `console.log`。

### 步骤 2：接口适配 (Adapter Pattern)
OpenStarry 的接口可能与原框架不同。您需要编写一个 Adapter 类。

*   **Provider:** 实现 `IProvider`。将 OpenStarry 的 `generate(prompt)` 转换为原模块需要的格式（如 `chat(messages)`）。
*   **Tool:** 实现 `ITool`。将 `execute(args)` 映射到原函数。

### 步骤 3：指令转工具
如果原模块注册了 CLI 指令（如 `/login`），请将其转换为 **`ITool`**。
*   这样不仅 CLI 可以用，Agent 也可以在思考过程中决定「我需要登录」。

### 步骤 4：输入推送 (pushInput 取代构造式回调)
在旧版本中，部分插件通过构造式回调 (constructor callback) 将用户输入注入 Agent。现在请改用 `IPluginContext.pushInput` 方法，由插件主动向 Agent 推送输入消息：
```typescript
// ❌ 旧方式：通过构造式回调
constructor(onInput: (msg: string) => void) { ... }

// ✅ 新方式：通过 context.pushInput
context.pushInput({ role: 'user', content: userMessage });
```

### 步骤 5：配置注入
不要从 `process.env` 读取配置。请从 `context.config` 读取。
```typescript
// ❌ 旧代码
const key = process.env.API_KEY;

// ✅ OpenStarry 代码
const key = context.config.apiKey;
```

---

## 3. 示例：将 OAuth Provider 移植

假设您有一个 `GeminiOAuthManager` 类。

1.  **封装为 Plugin:** 建立 `GeminiOAuthPlugin` 实现 `IPlugin`。
2.  **初始化:** 在 `initialize()` 中实例化 `GeminiOAuthManager`。
3.  **注册 Provider:** 建立 `GeminiProvider` 类，内部调用 Manager 的 API，并通过 `context.registerProvider()` 注册。
4.  **注册 Tool:** 建立 `LoginTool`，内部调用 `manager.startOAuthFlow()`，并通过 `context.registerTool()` 注册。

这样，一个外部模块就完美融入 OpenStarry 的五蕴架构了。
