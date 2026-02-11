# OpenStarry 架構完善計畫：IUI 介面 + guide-character-init 插件

> **狀態**: ✅ 已完成 (2026-02-06)

## 目標

完善五蘊（Five Aggregates）插件架構：

1. **新增 IUI 介面（色蘊）** — 將輸出渲染從 IListener 中分離，完成五蘊的完整定義
2. **建立 guide-character-init 插件** — 將基礎人設從 stdio 和 Core 中分離，作為獨立的 Guide 插件
3. **修正文檔** — 移除「UI 可被視為特殊的 Listener」的錯誤敘述

---

## 問題背景

### 現狀問題

| 問題 | 影響 |
|------|------|
| IListener 同時處理輸入和輸出 | 違反單一職責，混淆受蘊（輸入）與色蘊（輸出） |
| SDK 缺少 IUI 介面 | 五蘊架構不完整（只有 4/5） |
| stdio 插件綁定 Guide | Guide 應該獨立，不該與 I/O 插件耦合 |
| Core 直接讀取 guideFile | 違反 Core 純淨性（Core 不應做 file I/O） |
| 文檔錯誤敘述 | 誤導開發者認為 UI 是 Listener 的特例 |

### 目標架構

```
五蘊完整定義：
├── IUI (色蘊)      — 輸出渲染，接收 AgentEvent 並呈現
├── IListener (受蘊) — 輸入接收，推送 InputEvent 到隊列
├── IProvider (想蘊) — 認知推理，LLM 適配器
├── ITool (行蘊)    — 執行動作，檔案/API/程式碼
└── IGuide (識蘊)   — 身份人設，system prompt
```

---

## 實作計畫

### Phase 1: SDK 層修改（4 個檔案）✅

**所有後續修改都依賴 SDK 先完成。**

#### 1.1 新增 `types/ui.ts`

**路徑**: `packages/sdk/src/types/ui.ts`

```typescript
/**
 * UI interface — defines how the agent presents itself (色蘊).
 */
import type { AgentEvent } from "./events.js";

export interface IUI {
  id: string;
  name: string;
  onEvent(event: AgentEvent): void | Promise<void>;
  start?(): Promise<void>;
  stop?(): Promise<void>;
}
```

#### 1.2 修改 `types/listener.ts`

**移除 `onEvent`**，Listener 只負責輸入：

```typescript
/**
 * Listener interface — receives external input (受蘊).
 */
export interface IListener {
  id: string;
  name: string;
  start?(): Promise<void>;
  stop?(): Promise<void>;
}
```

#### 1.3 修改 `types/plugin.ts`

新增 `ui` 欄位到 `PluginHooks`：

```typescript
import type { IUI } from "./ui.js";

export interface PluginHooks {
  providers?: IProvider[];
  tools?: ITool[];
  listeners?: IListener[];
  ui?: IUI[];               // 新增 — 色蘊
  guides?: IGuide[];
  commands?: SlashCommand[];
  dispose?: () => Promise<void> | void;
}
```

#### 1.4 修改 `index.ts`

新增 IUI export：

```typescript
export type { IUI } from "./types/ui.js";
```

---

### Phase 2: Core 層修改（6 個檔案）✅

**必須與 Phase 1 同時完成，否則編譯失敗。**

#### 2.1 新增 `infrastructure/ui-registry.ts`

```typescript
/**
 * UIRegistry — manages UI renderers (色蘊).
 */
import type { IUI } from "@openstarry/sdk";
import { createLogger } from "@openstarry/shared";

const logger = createLogger("UIRegistry");

export interface UIRegistry {
  register(ui: IUI): void;
  get(id: string): IUI | undefined;
  list(): IUI[];
}

export function createUIRegistry(): UIRegistry {
  const uis = new Map<string, IUI>();
  return {
    register(ui: IUI): void {
      logger.debug(`Registering UI: ${ui.id}`);
      uis.set(ui.id, ui);
    },
    get(id: string): IUI | undefined {
      return uis.get(id);
    },
    list(): IUI[] {
      return [...uis.values()];
    },
  };
}
```

#### 2.2 修改 `infrastructure/index.ts`

新增 UIRegistry export。

#### 2.3 修改 `infrastructure/plugin-loader.ts`

- 新增 `uiRegistry: UIRegistry` 到 `PluginLoaderDeps`
- 新增 `hooks.ui` 的註冊迴圈

#### 2.4 修改 `transport/bridge.ts`

**關鍵改動**：從 ListenerRegistry 改為 UIRegistry

```typescript
import type { UIRegistry } from "../infrastructure/ui-registry.js";

export function createTransportBridge(
  bus: EventBus,
  uiRegistry: UIRegistry,  // 改為 UIRegistry
): TransportBridge {
  function broadcast(event: AgentEvent): void {
    for (const ui of uiRegistry.list()) {  // 改為 ui
      try {
        ui.onEvent(event);  // 呼叫 UI 的 onEvent
      } catch (err) {
        logger.error(`UI ${ui.id} error`, { error: String(err) });
      }
    }
  }
  // ...
}
```

#### 2.5 修改 `agents/agent-core.ts`

- **移除** `import { readFile } from "node:fs/promises"`（Core 純淨性）
- **移除** `import { resolve } from "node:path"`
- **新增** `createUIRegistry` import
- **新增** `uiRegistry` 到 AgentCore 介面
- **修改** `createTransportBridge(bus, uiRegistry)`
- **移除** 整個 `if (config.guideFile)` 區塊
- **新增** UI 的 start/stop 呼叫

#### 2.6 修改 `index.ts` (Core barrel)

新增 `createUIRegistry`, `UIRegistry` export。

---

### Phase 3: 重構 standard-function-stdio 插件 ✅

**路徑**: `openstarry_plugin/standard-function-stdio/src/index.ts`

#### 拆分為兩個元件

**StdioListener（輸入）**：
```typescript
function createStdioListener(ctx: IPluginContext): IListener {
  let rl: ReturnType<typeof createInterface> | null = null;

  return {
    id: "stdio-listener",
    name: "Standard I/O Listener",

    async start(): Promise<void> {
      rl = createInterface({ input: process.stdin, output: process.stdout, terminal: false });
      rl.on("line", (line) => {
        const trimmed = line.trim();
        if (trimmed) ctx.pushInput({ source: "cli", inputType: "user_input", data: trimmed });
      });
      rl.on("close", () => ctx.pushInput({ source: "cli", inputType: "user_input", data: "/quit" }));
    },

    async stop(): Promise<void> {
      rl?.close();
      rl = null;
    },
  };
}
```

**StdioUI（輸出）**：
```typescript
function createStdioUI(): IUI {
  let isStreaming = false;

  function promptUser(): void {
    process.stdout.write(`${BOLD}${CYAN}> ${RESET}`);
  }

  return {
    id: "stdio-ui",
    name: "Standard I/O UI",

    onEvent(event: AgentEvent): void {
      // 原本 onEvent 的 switch 邏輯完全保留
      // AGENT_STARTED, STREAM_TEXT_DELTA, TOOL_RESULT, etc.
    },
  };
}
```

**Factory 更新**：
```typescript
export function createStdioPlugin(): IPlugin {
  return {
    manifest: { name: "standard-function-stdio", version: "0.1.0-alpha", ... },
    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      return {
        listeners: [createStdioListener(ctx)],
        ui: [createStdioUI()],
        // 不再回傳 guides
      };
    },
  };
}
```

---

### Phase 4: 新增 guide-character-init 插件（3 個檔案）✅

**路徑**: `openstarry_plugin/guide-character-init/`

#### 4.1 package.json

```json
{
  "name": "@openstarry-plugin/guide-character-init",
  "version": "0.1.0-alpha",
  "type": "module",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "dependencies": {
    "@openstarry/sdk": "workspace:*"
  }
}
```

#### 4.2 tsconfig.json

與其他插件相同配置。

#### 4.3 src/index.ts

```typescript
/**
 * guide-character-init — Base persona / system prompt provider (識蘊).
 *
 * Config:
 *   { prompt: "..." }           — 內聯 system prompt
 *   { characterFile: "./x.md" } — 從檔案載入
 */
import { readFile } from "node:fs/promises";
import { resolve } from "node:path";
import type { IPlugin, IPluginContext, PluginHooks, IGuide } from "@openstarry/sdk";

const DEFAULT_PROMPT = `You are a helpful AI assistant powered by OpenStarry.
You can read, write, and manage files on the local filesystem using the available tools.
Always explain what you are doing before and after using tools.
If a tool call fails, analyze the error and try a different approach.
Be concise and helpful.`;

interface Config {
  prompt?: string;
  characterFile?: string;
  guideId?: string;
}

export function createGuideCharacterInitPlugin(): IPlugin {
  return {
    manifest: { name: "guide-character-init", version: "0.1.0-alpha", ... },

    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      const config = ctx.config as Config;
      const guideId = config.guideId ?? "default-guide";

      let systemPrompt: string;
      if (config.characterFile) {
        const path = resolve(ctx.workingDirectory, config.characterFile);
        systemPrompt = await readFile(path, "utf-8");
      } else {
        systemPrompt = config.prompt ?? DEFAULT_PROMPT;
      }

      const guide: IGuide = {
        id: guideId,
        name: `Character Guide (${guideId})`,
        getSystemPrompt: () => systemPrompt,
      };

      return { guides: [guide] };
    },
  };
}

export default createGuideCharacterInitPlugin;
```

---

### Phase 5: 設定檔更新（3 個檔案）✅

#### 5.1 SDK `types/agent.ts`

**移除** `guideFile` 欄位。

#### 5.2 Shared `utils/config-schema.ts`

**移除** `guideFile` 從 Zod schema。

#### 5.3 Runner `apps/runner/src/bin.ts`

更新 `defaultConfig()`：

```typescript
plugins: [
  { name: "@openstarry-plugin/provider-gemini-oauth" },
  { name: "@openstarry-plugin/standard-function-fs" },
  { name: "@openstarry-plugin/standard-function-stdio" },
  { name: "@openstarry-plugin/guide-character-init" },  // 新增
],
guide: "default-guide",
```

---

### Phase 6: 文檔更新（4 個檔案）✅

#### 6.1 `02_Headless_Agent_Core.md`

**移除**：
> UI 插件現在可以被視為一種特殊的 Listener

**改為**：
> UI 插件 (色蘊) 有獨立的 IUI 介面，負責接收事件並呈現輸出。
> Listener 插件 (受蘊) 專注於接收外部輸入。兩者各司其職。

#### 6.2 `16_Plugin_Types_Philosophical_Mapping.md`

新增 IUI 介面說明到「色蘊」段落。

#### 6.3 `00_Plugin_Philosophy_Five_Aggregates.md`

更新 JSON 範例，加入 `ui` 欄位。

#### 6.4 `21_Plugin_Interface_Deep_Dive.md`

更新 USB 類比，將 UI 和 Listener 分開描述。

---

## 檔案清單

### 新增檔案（5 個）

| 路徑 | 說明 |
|------|------|
| `packages/sdk/src/types/ui.ts` | IUI 介面定義 |
| `packages/core/src/infrastructure/ui-registry.ts` | UIRegistry |
| `openstarry_plugin/guide-character-init/package.json` | 插件 package |
| `openstarry_plugin/guide-character-init/tsconfig.json` | TS 設定 |
| `openstarry_plugin/guide-character-init/src/index.ts` | 插件實作 |

### 修改檔案（13 個）

| 路徑 | 修改內容 |
|------|----------|
| `packages/sdk/src/types/listener.ts` | 移除 onEvent |
| `packages/sdk/src/types/plugin.ts` | 新增 ui 欄位 |
| `packages/sdk/src/types/agent.ts` | 移除 guideFile |
| `packages/sdk/src/index.ts` | export IUI |
| `packages/core/src/infrastructure/index.ts` | export UIRegistry |
| `packages/core/src/infrastructure/plugin-loader.ts` | 新增 uiRegistry |
| `packages/core/src/transport/bridge.ts` | 改用 UIRegistry |
| `packages/core/src/agents/agent-core.ts` | 新增 UIRegistry，移除 guideFile I/O |
| `packages/core/src/index.ts` | export UIRegistry |
| `packages/shared/src/utils/config-schema.ts` | 移除 guideFile |
| `openstarry_plugin/standard-function-stdio/src/index.ts` | 拆分 Listener + UI，移除 Guide |
| `apps/runner/src/bin.ts` | 新增 guide-character-init 到 defaultConfig |
| `openstarry/tsconfig.json` | 新增 guide-character-init reference |

### 文檔更新（4 個）

| 路徑 |
|------|
| `openstarry_doc/Architecture_Documentation/02_Headless_Agent_Core.md` |
| `openstarry_doc/Architecture_Documentation/16_Plugin_Types_Philosophical_Mapping.md` |
| `openstarry_doc/Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md` |
| `openstarry_doc/Architecture_Documentation/21_Plugin_Interface_Deep_Dive.md` |

---

## 驗證結果

| 驗證項目 | 結果 |
|---------|------|
| `pnpm build` | ✅ 9 個套件全部編譯成功 |
| `pnpm test` | ✅ 56 個測試全數通過 |
| `pnpm test:purity` | ✅ Core 不 import 插件 |

---

## 風險評估

| 風險 | 緩解措施 |
|------|----------|
| IListener.onEvent 是 breaking change | transport/bridge.ts 在同一 Phase 更新 |
| 舊 agent.json 有 guideFile | Zod 會靜默忽略未知欄位，不會報錯 |
| standard-function-skill 受影響？ | 不受影響，它只提供 Guide，不用 onEvent |
