# 05. 插件接口定義 (Plugin Interface Definitions)

> ⚠️ **[漂移更正 — v0.59.6 / 2026-06-16] 本文件原始 5 個接口草案與已出貨 SDK 不符（早期設計快照），現按真實 SDK 型別逐節改寫。**
>
> **接口最高權威＝SDK 型別檔（非本文件）：**
> - `packages/sdk/src/types/provider.ts` — `IProvider.chat(req): AsyncIterable<ProviderStreamEvent>`（provider.ts:40-49；非舊草案的 `generate(history, tools): Promise<{is_final,...}>`）
> - `packages/sdk/src/types/tool.ts` — `ITool { id; description; parameters: z.ZodType; execute(input, ctx): Promise<string> }`（tool.ts:53-64；非 `name`/`args: JSON Schema`/`Promise<any>`）
> - `packages/sdk/src/types/listener.ts` — `IListener { id; name; start?(); stop?() }`（listener.ts:8-13；輸入經 `ctx.pushInput`，**無** `emitInput`/`onStart`/`onStop`）
> - `packages/sdk/src/types/guide.ts` — `IGuide { id; name; getSystemPrompt(): string | Promise<string> }`（guide.ts:8-12；**無** `systemPrompt`/`temperature`/`memoryPolicy`/`initialMemories` 這些幻影欄位）
> - `packages/sdk/src/types/ui.ts` — `IUI { id; name; onEvent(event: AgentEvent) }`（ui.ts:9-15；事件聯集見 `types/events.ts`，非舊草案的 `onNewMessage`/`onToolCallRequest`/... 命名回呼）
>
> 本文件以下各節已對齊上述 SDK。原始草案敘述（含舊命名）保留於各節末「歷史草案」區塊供考古，但**以 SDK 為準**。任何衝突一律 SDK 贏。

本文件定義了各種插件需要實現的抽象接口，以確保它們能與「無頭代理人核心」正確交互。具體的程式碼實現範例，請參考 `Implementation_Examples/` 資料夾下的對應文件。

---

## 1. UI 插件接口（色蘊·輸出）

> ✅ **[實作狀態 — v0.59.6] 真實接口＝單一事件入口 `onEvent`，不是多個命名回呼。** 權威：`packages/sdk/src/types/ui.ts:9-15`。

UI 插件實現 `IUI`。Core 把所有對外狀態變更都包成 `AgentEvent` 聯集，透過**單一** `onEvent` 推給 UI；UI 不對外暴露 `onNewMessage`/`onToolCallRequest`/`onReadyForInput`/`onAgentStateChange` 這類分立回呼。事件種類見 `packages/sdk/src/types/events.ts`（含訊息輸出、工具調用請求/結果、確認請求、狀態變更等 discriminated union）。

```typescript
import type { IUI, AgentEvent } from "@openstarry/sdk";

export const myUI: IUI = {
  id: "ui.console",
  name: "Console UI",
  // 核心 → UI：所有事件走這一個入口，依 event.type 分流
  async onEvent(event: AgentEvent): Promise<void> {
    switch (event.type) {
      // e.g. "message", "tool_call", "confirmation_request", "state_change" ...
      // 完整聯集見 packages/sdk/src/types/events.ts
      default:
        // render(event)
    }
  },
  start: async () => { /* 掛載輸出裝置 */ },
  stop: async () => { /* 釋放資源 */ },
};
```

UI → 核心的回送（用戶輸入、確認回應）**不是** UI 接口上的方法，而是經由發送 `InputEvent`（`source`/`inputType`/`data`/`replyTo`，見 `types/events.ts:27`）回到 Core 的處理佇列——與 Listener 用的 `ctx.pushInput` 同一條輸入管道。

### 具體實現範例

*   參考實作：`@openstarry-plugin/web-ui`、`@openstarry-plugin/tui-dashboard`
*   舊範例文件：`Implementation_Examples/UI_Plugin_Example.md`（早於現行 `onEvent` 模型，僅供考古）

<details><summary>歷史草案（已被 SDK 取代，僅供考古）</summary>

> 早期草案把 UI 設計成多個命名回呼（`onNewMessage(message)` / `onToolCallRequest(request)` / `onReadyForInput` / `onAgentStateChange(state)`）+ 反向命令方法（`submitUserInput(input)` / `provideConfirmation(confirmationId, approved)`）。出貨 SDK 改為單一 `onEvent(event: AgentEvent)` 入口 + `InputEvent` 回送，故此草案不再成立。

</details>

---

## 2. Tool 插件接口（行蘊·意志行動）

> ✅ **[實作狀態 — v0.59.6] 真實 `ITool` 用 `id` + Zod `parameters` + `execute(input, ctx): Promise<string>`。** 權威：`packages/sdk/src/types/tool.ts:53-64`。

Tool 插件導出一個 `ITool` 物件。關鍵差異 vs 早期草案：欄位是 `id`（非 `name`）、參數用 **Zod schema** `parameters: z.ZodType<TInput>`（非裸 JSON Schema 物件；Core 會自動把它轉成 provider 需要的 `ToolJsonSchema`，見 tool.ts:67）、`execute` 收 `(input, ctx: ToolContext)` 並回傳 `Promise<string>`（非 `Promise<any>`）。`input` 已是經 Zod 解析的 typed 值，無需自行驗證。可選 `metadata`（`hasSideEffects`/`riskCategory`/`requiresConfirmation`）供確認閘與 IVolition 決策。

```typescript
import { z } from "zod";
import type { ITool, ToolContext } from "@openstarry/sdk";

export const readFileTool: ITool<{ path: string }> = {
  id: "fs.read",                       // 唯一標識符，供 LLM 調用（不是 name）
  description: "Read a UTF-8 text file within the allowed sandbox paths.",
  parameters: z.object({               // Zod schema，不是裸 JSON Schema
    path: z.string().describe("File path inside allowedPaths."),
  }),
  metadata: { hasSideEffects: false, riskCategory: "safe" },
  // input 已被 Zod 解析為 { path: string }；ctx 提供 workingDirectory/allowedPaths/signal/bus
  async execute(input, ctx: ToolContext): Promise<string> {
    // ...讀檔（受 ctx.allowedPaths 約束）...
    return "<file contents>";          // 回傳字串，不是任意物件
  },
};
```

`ToolContext`（tool.ts:11-16）提供 `workingDirectory`、`allowedPaths`、`signal?`、`bus`。沙盒/隔離工具（如代碼解釋器）的安全前提見 §6。

### 具體實現範例

*   參考實作：`@openstarry-plugin/standard-function-fs`（`fs.read`/`fs.write`/`fs.list`/`fs.mkdir`/`fs.delete`，皆為上述 Zod `ITool` 形狀）
*   舊範例文件：`Implementation_Examples/Tool_ReadFile_Example.md`、`Implementation_Examples/Tool_CodeInterpreter_Example.md`（早於 Zod `parameters` 模型，僅供考古）

<details><summary>歷史草案（已被 SDK 取代，僅供考古）</summary>

> 早期草案：`name: string` / `description: string` / `args: object`（JSON Schema）/ `execute(args): Promise<any>`。出貨 SDK 改為 `id` / `parameters: z.ZodType` / `execute(input, ctx): Promise<string>`。

</details>

---

## 3. Provider 插件接口（想蘊·認知處理）

> ✅ **[實作狀態 — v0.59.6] 真實 `IProvider` 是串流的：`chat(request): AsyncIterable<ProviderStreamEvent>`，不是 `generate(...) : Promise<{is_final,...}>`。** 權威：`packages/sdk/src/types/provider.ts:40-49`。

Provider 插件實現 `IProvider`。它**不是** request-response（`generate → {is_final, tool_calls, text_content}`），而是 **async 串流**：傳入一個 `ChatRequest`（`model`/`messages`/`systemPrompt?`/`tools?`/`maxTokens?`/`temperature?`/`signal?`，provider.ts:19-27），回傳一個 `AsyncIterable<ProviderStreamEvent>`，逐段 yield 事件。事件聯集見 `packages/sdk/src/types/message.ts:38-45`：

```typescript
type ProviderStreamEvent =
  | { type: "text_delta"; text: string }
  | { type: "reasoning_delta"; text: string }
  | { type: "tool_call_start"; toolCallId: string; name: string }
  | { type: "tool_call_delta"; toolCallId: string; input: string }
  | { type: "tool_call_end"; toolCallId: string; name: string; input: string }
  | { type: "finish"; stopReason: "end_turn" | "tool_use" | "max_tokens" | "error"; usage?: TokenUsage }
  | { type: "error"; error: Error };
```

最小骨架：

```typescript
import type { IProvider, ChatRequest, ProviderStreamEvent, ModelInfo } from "@openstarry/sdk";

export const myProvider: IProvider = {
  id: "provider.acme",
  name: "Acme LLM",
  models: [{ id: "acme-1", name: "Acme 1" } satisfies ModelInfo],
  // 串流產生器：逐段 yield，不是一次回傳 { is_final, ... }
  async *chat(request: ChatRequest): AsyncIterable<ProviderStreamEvent> {
    // ...呼叫底層 API（可用 request.signal 取消）...
    yield { type: "text_delta", text: "Hello" };
    yield { type: "finish", stopReason: "end_turn" };
  },
  isConfigured: () => true,            // 可選：是否有有效憑證
};
```

「是否最終答覆」由 `finish.stopReason`（`end_turn` vs `tool_use`）表達，而非舊草案的 `is_final` 布林；工具調用以 `tool_call_*` 事件串流呈現，而非 `tool_calls` 陣列。

### 具體實現範例

*   參考實作：`@openstarry-plugin/provider-claude` / `provider-gemini` / `provider-chatgpt` / `provider-claude-cli` / `provider-lmstudio` / `provider-local-llama`（皆為上述串流 `chat` 形狀）
*   舊範例文件：`Implementation_Examples/Provider_Gemini_Example.md`（早於串流模型，僅供考古）

<details><summary>歷史草案（已被 SDK 取代，僅供考古）</summary>

> 早期草案：`constructor(config)` + `generate(history, tools): Promise<{ is_final, tool_calls?, text_content? }>`。出貨 SDK 改為宣告式物件 + `chat(request): AsyncIterable<ProviderStreamEvent>` 串流。

</details>

---

## 4. Listener 插件接口 (色蘊·輸入)

> ✅ **[實作狀態 — v0.59.6] 真實 `IListener` 是 `{ id; name; start?(); stop?() }`，輸入經 `ctx.pushInput`——無 `onStart`/`onStop`/`emitInput`。** 權威：`packages/sdk/src/types/listener.ts:8-13`；`ctx.pushInput` 簽名見 `packages/sdk/src/types/plugin.ts:240`，`InputEvent` 形狀見 `packages/sdk/src/types/events.ts:27`。

Listener 監聽外部訊號並轉成 Core 可理解的輸入。關鍵差異 vs 早期草案：欄位是 `id` + `name`；生命週期是**可選**的 `start?()` / `stop?()`（無參數，非 `onStart(context)`/`onStop()`）；把訊號送進 Core 用的是**插件 context** 上的 `ctx.pushInput(event: InputEvent)`（在 `factory(ctx)` 閉包內取得），**不是** listener 上的 `context.emitInput(...)`。`InputEvent = { source; inputType; data; replyTo? }`。

```typescript
import type { IPlugin, IPluginContext, IListener } from "@openstarry/sdk";

export function createCliListenerPlugin(): IPlugin {
  return {
    manifest: { /* ... */ },
    factory(ctx: IPluginContext) {
      const listener: IListener = {
        id: "listener.cli",
        name: "CLI stdin Listener",
        async start() {                 // 可選；無參數
          process.stdin.on("data", (chunk) => {
            ctx.pushInput({             // 經插件 context 推入，不是 emitInput
              source: "cli",
              inputType: "user_input",
              data: chunk.toString().trim(),
            });
          });
        },
        async stop() { /* 關閉連接、釋放資源 */ },
      };
      return { listeners: [listener] };
    },
  };
}
```

### 具體實現範例

*   參考實作：`@openstarry-plugin/transport-local-cli`（stdin → `ctx.pushInput`，含 UID/PID source-binding）、`transport-http`、`transport-websocket`

<details><summary>歷史草案（已被 SDK 取代，僅供考古）</summary>

> 早期草案：`name` + `onStart(context: IAgentContext): Promise<void>` + `onStop(): Promise<void>`，並用 `context.emitInput(data)` 回傳訊號。出貨 SDK 改為 `id`/`name` + 可選 `start?()`/`stop?()`，輸入走 `ctx.pushInput(event)`。

</details>

## 5. Guide 插件接口 (識蘊)

> ✅ **[實作狀態 — v0.59.6] 真實 `IGuide` 是 `{ id; name; getSystemPrompt(): string | Promise<string> }`。`temperature`/`memoryPolicy`/`initialMemories` 是從未建造的幻影欄位。** 權威：`packages/sdk/src/types/guide.ts:8-12`。

Guide 注入 Agent 的我執框架（身份認同與行為準則）。真實接口只有三個成員：`id`、`name`，以及 `getSystemPrompt()`（回傳 `string` 或 `Promise<string>`，故可動態組裝 prompt）。**沒有** `systemPrompt` 欄位，也**沒有** `temperature`/`memoryPolicy`/`initialMemories`——溫度屬於 `ChatRequest`（provider 層，§3），記憶策略屬於獨立的 context 插件（如 `@openstarry-plugin/context-sliding-window`），二者皆非 Guide 的職責。需要跨 session 持久指令時用 `IPersistentGuide`（guide.ts:44-49，`addDirective`/`removeDirective`/`clearDirectives`/`listDirectives`）。

```typescript
import type { IGuide } from "@openstarry/sdk";

export const myGuide: IGuide = {
  id: "guide.assistant",
  name: "Helpful Assistant",
  // 同步或非同步皆可；在此組裝人設、價值觀、行為準則
  getSystemPrompt(): string {
    return "You are a careful, honest assistant. ...";
  },
};
```

### 具體實現範例

*   參考實作：`@openstarry-plugin/guide-character-init`、`@openstarry-plugin/guide-persistent`（後者實現 `IPersistentGuide`）

<details><summary>歷史草案（已被 SDK 取代，僅供考古）</summary>

> 早期草案：`systemPrompt: string` + `temperature?: number` + `memoryPolicy?: string` + `initialMemories?: string[]`。出貨 SDK 僅保留 `id`/`name`/`getSystemPrompt()`；其餘三個欄位從未建造（temperature → provider `ChatRequest`；記憶策略 → context 插件）。

</details>

---

## 6. Tool 插件套件範例：代碼解釋器

> ⚠️ **[漂移更正 — v0.59.6] 以下 §4.1/§4.2 程式碼仍用早期草案的 `name`/`args`(JSON Schema)/`execute(args)` 形狀，僅示意「沙盒隔離」概念，非現行 `ITool` 寫法。** 真實工具形狀請以 §2（`id` + Zod `parameters` + `execute(input, ctx): Promise<string>`，權威 `packages/sdk/src/types/tool.ts:53-64`）為準。

為了實現類似 `opencode` 的功能，代理人需要一套功能強大的代碼執行和文件管理工具。**此套件的所有工具在實現時，都必須在一個安全的、與主機隔離的沙盒環境中（如 Docker 容器）運行，這是保障安全的最高前提。**

### 4.1 代碼執行工具

```javascript
// 示例：一個用於執行 Python 代碼的 Tool 插件
class Python_Interpreter_Tool {
  name = "python:execute";
  description = "Executes a block of Python code in a sandboxed environment. The environment has a temporary file system and can install packages if specified. Returns the stdout, stderr, and the final result.";
  args = {
    code: {
      type: "string",
      description: "The Python code to execute."
    },
    dependencies: {
      type: "array",
      description: "An optional list of pip packages to install before execution.",
      items: { type: "string" }
    }
  };

  /**
   * @returns {Promise<{stdout: string, stderr: string, result: any}>}
   */
  async execute(args) {
    // 實現細節：
    // 1. 啟動一個新的、乾淨的 Docker 容器。
    // 2. 如果提供了 dependencies，在容器中運行 pip install。
    // 3. 將 args.code 寫入容器中的一個 .py 文件。
    // 4. 執行該 .py 文件。
    // 5. 捕獲 stdout, stderr, 和可能的返回值。
    // 6. 銷毀容器。
    // 7. 返回捕獲的輸出。
    const result = await this.runInSandbox(args.code, args.dependencies);
    return result;
  }

  async runInSandbox(code, deps) { /* ... sandboxing logic ... */ }
}
```

### 4.2 沙盒文件系統工具

這些工具操作的是代碼解釋器所在**沙盒內部**的文件系統。

```javascript
// 示例：一個用於列出沙盒內目錄的 Tool 插件
class Sandbox_FS_List_Tool {
  name = "sandbox_fs:list";
  description = "Lists the files and directories inside the sandbox at a given path.";
  args = {
    path: { type: "string", description: "The path inside the sandbox." }
  };

  async execute(args) {
    // 實現細節：
    // 通過 'docker exec' 或類似命令在對應的沙盒容器中執行 'ls -l' 或 'dir'
    const result = await this.executeCommandInSandbox(`ls -l ${args.path}`);
    return result.stdout;
  }

  async executeCommandInSandbox(cmd) { /* ... sandboxing logic ... */ }
}
```
*其他文件系統工具 (`read`, `write`, `mkdir`, `remove`) 的實現與此類似。*

