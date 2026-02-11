# Developer Guide: Plugin Loading Specification & Migration

This document provides a detailed explanation of the OpenStarry plugin loading specification and outlines the specific steps for migrating existing Node.js modules or third-party plugins into the OpenStarry ecosystem.

## 1. Core Loading Specification

OpenStarry utilizes the **"Factory Pattern"** and **"Dependency Injection"** to load plugins.

### 1.1 Physical Structure
A plugin must be a standard NPM package, and its `package.json` **must** include an `openstarry` metadata field.

```json
{
  "name": "@openstarry-plugin/my-plugin",
  "main": "dist/index.js",
  "dependencies": {
    "@openstarry/sdk": "^1.0.0"
  },
  "openstarry": {
    "type": "aggregate", // or 'tool', 'provider'
    "components": {
      "providers": ["MyProvider"],
      "tools": ["LoginTool"]
    }
  }
}
```

### 1.2 Entry Point
The file pointed to by `main` must export an `initialize` function or a class that implements the `IPlugin` interface.

```typescript
import { IPlugin, IPluginContext } from '@openstarry/sdk';

export default class MyPlugin implements IPlugin {
  async initialize(context: IPluginContext) {
    // Perform initialization and registration here
  }
}
```

---

## 2. Migration Guide

If you have an existing functional module (e.g., a Provider from another AI framework), follow these steps to migrate it.

### Step 1: Replace Dependencies
Remove dependencies on the original framework (e.g., `@ai-core/core`, `langchain`) and replace them with `@openstarry/sdk`.
*   **Context:** Use `IAgentContext` instead of the original context object.
*   **Logger:** Use `context.logger` instead of `console.log`.

### Step 2: Interface Adaptation (Adapter Pattern)
OpenStarry's interfaces may differ from the original framework. You will need to write an Adapter class.

*   **Provider:** Implement `IProvider`. Convert OpenStarry's `generate(prompt)` into the format required by the original module (e.g., `chat(messages)`).
*   **Tool:** Implement `ITool`. Map `execute(args)` to the original function.

### Step 3: Transform Commands into Tools
If the original module registered CLI commands (e.g., `/login`), convert them into **`ITool`** objects.
*   This makes them available not only to the CLI but also allows the Agent to decide "I need to log in" during its reasoning process.

### Step 4: Input Pushing (Replace Constructor Callbacks with pushInput)
In older versions, some plugins injected user input into the Agent via constructor callbacks. Now, please use the `IPluginContext.pushInput` method, allowing the plugin to actively push input messages to the Agent:
```typescript
// ❌ Old way: via constructor callback
constructor(onInput: (msg: string) => void) { ... }

// ✅ New way: via context.pushInput
context.pushInput({ role: 'user', content: userMessage });
```

### Step 5: Configuration Injection
Do not read configurations from `process.env`. Use `context.config` instead.
```typescript
// ❌ Old code
const key = process.env.API_KEY;

// ✅ OpenStarry code
const key = context.config.apiKey;
```

---

## 3. Example: Migrating an OAuth Provider

Assume you have a `GeminiOAuthManager` class.

1.  **Encapsulate as a Plugin:** Create `GeminiOAuthPlugin` implementing `IPlugin`.
2.  **Initialize:** Instantiate `GeminiOAuthManager` within `initialize()`.
3.  **Register Provider:** Create a `GeminiProvider` class that internally calls the Manager's API and register it via `context.registerProvider()`.
4.  **Register Tool:** Create a `LoginTool` that calls `manager.startOAuthFlow()` and register it via `context.registerTool()`.

Thus, an external module integrates perfectly into OpenStarry's Five Aggregates architecture.
