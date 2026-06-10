# 25. pushInput 輸入事件架構 (PushInput Event Architecture)

本文件說明 `IPluginContext.pushInput` 的設計決策與運作機制。

---

## 1. 問題背景

在 Plan02 之前，stdio 插件透過宿主注入的 `onInput` 回呼來推送用戶輸入：

```typescript
// 舊模式（Plan01-02）
export function createStdioPlugin(opts: { onInput: (text: string) => void }): IPlugin {
  // opts.onInput 由宿主（CLI）在建構時注入
}
```

這帶來三個問題：

| 問題 | 影響 |
|------|------|
| **宿主必須認識插件** | CLI 需要偵測 `ref.name === "standard-function-stdio"` 並注入回呼 |
| **特殊處理** | stdio 的載入路徑和其他插件不同，破壞了統一性 |
| **無法完全動態化** | 每新增一個需要輸入能力的 Listener，宿主就要加特殊處理 |

---

## 2. 解決方案：IPluginContext.pushInput

在 SDK 的 `IPluginContext` 新增標準化的輸入推送方法：

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

Core 在建立 plugin context 時自動注入：

```typescript
function getPluginContext(pluginConfig?: Record<string, unknown>): IPluginContext {
  return {
    bus,
    workingDirectory: process.cwd(),
    agentId: config.identity.id,
    config: pluginConfig ?? {},
    pushInput: (event) => core.pushInput(event),  // ← 綁定到 core
  };
}
```

---

## 3. 輸入事件流

```
用戶在終端輸入 "hello"
    ↓
stdio listener (readline "line" event)
    ↓
ctx.pushInput({ source: "cli", inputType: "user_input", data: "hello" })
    ↓
core.pushInput(inputEvent)
    ↓
┌─ 是斜線指令？ ──→ handleSlashCommand() ──→ 快速路徑（不進 LLM）
│
└─ 一般輸入 ──→ queue.push(INPUT_RECEIVED) ──→ ExecutionLoop.processEvent()
```

### InputEvent 結構

```typescript
export interface InputEvent {
  source: string;                          // 來源標識（"cli", "web", "api"）
  inputType: string;                       // 類型（"user_input", "system_event"）
  data: string | Record<string, unknown>;  // 內容
  replyTo?: string;                        // 回覆目標（可選）
}
```

---

## 4. 對插件開發者的影響

### 之前（需要宿主配合）

```typescript
// 插件需要在建構函數接收回呼
export function createMyListener(opts: { onInput: Function }): IPlugin { ... }

// 宿主需要特殊處理
const isMyPlugin = ref.name === "my-listener";
const opts = isMyPlugin ? { onInput: handleInput } : undefined;
```

### 之後（標準化）

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
          // 任何輸入來源都可以推送
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

// 宿主不需要做任何特殊處理
```

---

## 5. 適用場景

`pushInput` 不僅限於 CLI。任何需要向 Agent 推送輸入的插件都可以使用：

| 插件 | 輸入來源 | source 值 |
|------|---------|-----------|
| `standard-function-stdio` | 終端 readline | `"cli"` |
| 未來：Web Listener | HTTP 請求 | `"web"` |
| 未來：Discord Bot | Discord 訊息 | `"discord"` |
| 未來：File Watcher | 檔案變更事件 | `"file-watcher"` |
| 未來：Cron Scheduler | 定時觸發 | `"scheduler"` |

---

## 6. 設計決策

### 為什麼不用 EventBus 直接推送？

插件已經有 `ctx.bus`，為什麼不直接 `bus.emit(INPUT_RECEIVED, payload)` ？

因為 `pushInput` 封裝了額外邏輯：
1. **斜線指令快速路徑**：`/help`、`/quit` 等不進 EventQueue，直接處理
2. **事件格式標準化**：自動包裝為 `{ type: INPUT_RECEIVED, payload: inputEvent }`
3. **未來可擴展**：可以在 pushInput 中加入限流、過濾、認證等邏輯

直接用 EventBus 會繞過這些機制。
