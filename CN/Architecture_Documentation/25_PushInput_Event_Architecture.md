# 25. pushInput 输入事件架构 (PushInput Event Architecture)

本文档说明 `IPluginContext.pushInput` 的设计决策与运作机制。

---

## 1. 问题背景

在 Plan02 之前，stdio 插件通过宿主注入的 `onInput` 回调来推送用户输入：

```typescript
// 旧模式（Plan01-02）
export function createStdioPlugin(opts: { onInput: (text: string) => void }): IPlugin {
  // opts.onInput 由宿主（CLI）在建构时注入
}
```

这带来三个问题：

| 问题 | 影响 |
|------|------|
| **宿主必须认识插件** | CLI 需要侦测 `ref.name === "standard-function-stdio"` 并注入回调 |
| **特殊处理** | stdio 的载入路径和其他插件不同，破坏了统一性 |
| **无法完全动态化** | 每新增一个需要输入能力的 Listener，宿主就要加特殊处理 |

---

## 2. 解决方案：IPluginContext.pushInput

在 SDK 的 `IPluginContext` 新增标准化的输入推送方法：

```typescript
export interface IPluginContext {
  bus: EventBus;
  workingDirectory: string;
  agentId: string;
  config: Record<string, unknown>;
  /** Push an input event into the agent's processing queue. */
  pushInput: (event: InputEvent) => void;  // ← 新增
}
```

Core 在建立 plugin context 时自动注入：

```typescript
function getPluginContext(pluginConfig?: Record<string, unknown>): IPluginContext {
  return {
    bus,
    workingDirectory: process.cwd(),
    agentId: config.identity.id,
    config: pluginConfig ?? {},
    pushInput: (event) => core.pushInput(event),  // ← 绑定到 core
  };
}
```

---

## 3. 输入事件流

```
用户在终端输入 "hello"
    ↓
stdio listener (readline "line" event)
    ↓
ctx.pushInput({ source: "cli", inputType: "user_input", data: "hello" })
    ↓
core.pushInput(inputEvent)
    ↓
┌─ 是斜杠指令？ ──→ handleSlashCommand() ──→ 快速路径（不进 LLM）
│
└─ 一般输入 ──→ queue.push(INPUT_RECEIVED) ──→ ExecutionLoop.processEvent()
```

### InputEvent 结构

```typescript
export interface InputEvent {
  source: string;                          // 来源标识（"cli", "web", "api"）
  inputType: string;                       // 类型（"user_input", "system_event"）
  data: string | Record<string, unknown>;  // 内容
  replyTo?: string;                        // 回复目标（可选）
}
```

---

## 4. 对插件开发者的影响

### 之前（需要宿主配合）

```typescript
// 插件需要在构造函数接收回调
export function createMyListener(opts: { onInput: Function }): IPlugin { ... }

// 宿主需要特殊处理
const isMyPlugin = ref.name === "my-listener";
const opts = isMyPlugin ? { onInput: handleInput } : undefined;
```

### 之后（标准化）

```typescript
// 插件在 factory 中直接使用 ctx.pushInput
export function createMyListener(): IPlugin {
  return {
    manifest: { name: "my-listener", version: "1.0.0" },
    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      const listener: IListener = {
        id: "my-listener",
        name: "My Custom Listener",
        onEvent(event) { /* handle output events */ },
        async start() {
          // 任何输入来源都可以推送
          someInputSource.on("data", (text) => {
            ctx.pushInput({
              source: "my-source",
              inputType: "user_input",
              data: text,
            });
          });
        },
      };
      return { listeners: [listener] };
    },
  };
}

// 宿主不需要做任何特殊处理
```

---

## 5. 适用场景

`pushInput` 不仅限于 CLI。任何需要向 Agent 推送输入的插件都可以使用：

| 插件 | 输入来源 | source 值 |
|------|---------|-----------|
| `standard-function-stdio` | 终端 readline | `"cli"` |
| 未来：Web Listener | HTTP 请求 | `"web"` |
| 未来：Discord Bot | Discord 消息 | `"discord"` |
| 未来：File Watcher | 文件变更事件 | `"file-watcher"` |
| 未来：Cron Scheduler | 定时触发 | `"scheduler"` |

---

## 6. 设计决策

### 为什么不用 EventBus 直接推送？

插件已经有 `ctx.bus`，为什么不直接 `bus.emit(INPUT_RECEIVED, payload)` ？

因为 `pushInput` 封装了额外逻辑：
1. **斜杠指令快速路径**：`/help`、`/quit` 等不进 EventQueue，直接处理
2. **事件格式标准化**：自动包装为 `{ type: INPUT_RECEIVED, payload: inputEvent }`
3. **未来可扩展**：可以在 pushInput 中加入限流、过滤、认证等逻辑

直接用 EventBus 会绕过这些机制。
