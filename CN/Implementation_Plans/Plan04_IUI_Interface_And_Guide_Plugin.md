# OpenStarry 架构完善计划：IUI 接口 + guide-character-init 插件

> **状态**: ✅ 已完成 (2026-02-06)

## 目标

完善五蕴（Five Aggregates）插件架构：

1. **新增 IUI 接口（色蕴）** — 将输出渲染从 IListener 中分离，完成五蕴的完整定义
2. **建立 guide-character-init 插件** — 将基础人设从 stdio 和 Core 中分离，作为独立的 Guide 插件
3. **修正文档** — 移除「UI 可被视为特殊的 Listener」的错误叙述

---

## 问题背景

### 现状问题

| 问题 | 影响 |
|------|------|
| IListener 同时处理输入和输出 | 违反单一职责，混淆受蕴（输入）与色蕴（输出） |
| SDK 缺少 IUI 接口 | 五蕴架构不完整（只有 4/5） |
| stdio 插件绑定 Guide | Guide 应该独立，不该与 I/O 插件耦合 |
| Core 直接读取 guideFile | 违反 Core 纯净性（Core 不应做文件 I/O） |
| 文档错误叙述 | 误导开发者认为 UI 是 Listener 的特例 |

### 目标架构

```
五蕴完整定义：
├── IUI (色蕴)      — 输出渲染，接收 AgentEvent 并呈现
├── IListener (受蕴) — 输入接收，推送 InputEvent 到队列
├── IProvider (想蕴) — 认知推理，LLM 适配器
├── ITool (行蕴)    — 执行动作，文件/API/代码
└── IGuide (识蕴)   — 身份人设，system prompt
```

---

## 实施计划

### Phase 1: SDK 层修改（4 个文件）✅

**所有后续修改都依赖 SDK 先完成。**

#### 1.1 新增 `types/ui.ts`

**路径**: `packages/sdk/src/types/ui.ts`

```typescript
/**
 * UI interface — defines how the agent presents itself (色蕴).
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

**移除 `onEvent`**，Listener 仅负责输入：

```typescript
/**
 * Listener interface — receives external input (受蕴).
 */
export interface IListener {
  id: string;
  name: string;
  start?(): Promise<void>;
  stop?(): Promise<void>;
}
```

#### 1.3 修改 `types/plugin.ts`

新增 `ui` 字段到 `PluginHooks`：

```typescript
import type { IUI } from "./ui.js";

export interface PluginHooks {
  providers?: IProvider[];
  tools?: ITool[];
  listeners?: IListener[];
  ui?: IUI[];               // 新增 — 色蕴
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

### Phase 2: Core 层修改（6 个文件）✅

**必须与 Phase 1 同时完成，否则编译失败。**

#### 2.1 新增 `infrastructure/ui-registry.ts`

```typescript
/**
 * UIRegistry — manages UI renderers (色蕴).
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
- 新增 `hooks.ui` 的注册循环

#### 2.4 修改 `transport/bridge.ts`

**关键改动**：从 ListenerRegistry 改为 UIRegistry

```typescript
import type { UIRegistry } from "../infrastructure/ui-registry.js";

export function createTransportBridge(
  bus: EventBus,
  uiRegistry: UIRegistry,  // 改为 UIRegistry
): TransportBridge {
  function broadcast(event: AgentEvent): void {
    for (const ui of uiRegistry.list()) {  // 改为 ui
      try {
        ui.onEvent(event);  // 调用 UI 的 onEvent
      } catch (err) {
        logger.error(`UI ${ui.id} error`, { error: String(err) });
      }
    }
  }
  // ...
}
```

#### 2.5 修改 `agents/agent-core.ts`

- **移除** `import { readFile } from "node:fs/promises"`（Core 纯净性）
- **移除** `import { resolve } from "node:path"`
- **新增** `createUIRegistry` import
- **新增** `uiRegistry` 到 AgentCore 接口
- **修改** `createTransportBridge(bus, uiRegistry)`
- **移除** 整个 `if (config.guideFile)` 区块
- **新增** UI 的 start/stop 调用

#### 2.6 修改 `index.ts` (Core barrel)

新增 `createUIRegistry`, `UIRegistry` export。

---

### Phase 3: 重构 standard-function-stdio 插件 ✅

**路径**: `openstarry_plugin/standard-function-stdio/src/index.ts`

#### 拆分为两个组件

**StdioListener（输入）**：
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

**StdioUI（输出）**：
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
      // 原本 onEvent 的 switch 逻辑完全保留
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
        // 不再回传 guides
      };
    },
  };
}
```

---

### Phase 4: 新增 guide-character-init 插件（3 个文件）✅

**路径**: `openstarry_plugin/guide-character-init/`

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

与其他插件相同配置。

#### 4.3 src/index.ts

```typescript
/**
 * guide-character-init — Base persona / system prompt provider (识蕴).
 *
 * Config:
 *   { prompt: "..." }           — 内联 system prompt
 *   { characterFile: "./x.md" } — 从文件加载
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

### Phase 5: 配置文件更新（3 个文件）✅

#### 5.1 SDK `types/agent.ts`

**移除** `guideFile` 字段。

#### 5.2 Shared `utils/config-schema.ts`

**移除** `guideFile` 从 Zod schema。

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

### Phase 6: 文档更新（4 个文件）✅

#### 6.1 `02_Headless_Agent_Core.md`

**移除**：
> UI 插件现在可以被视为一种特殊的 Listener

**改为**：
> UI 插件 (色蕴) 有独立的 IUI 接口，负责接收事件并呈现输出。
> Listener 插件 (受蕴) 专注于接收外部输入。两者各司其职。

#### 6.2 `16_Plugin_Types_Philosophical_Mapping.md`

新增 IUI 接口说明到「色蕴」段落。

#### 6.3 `00_Plugin_Philosophy_Five_Aggregates.md`

更新 JSON 示例，加入 `ui` 字段。

#### 6.4 `21_Plugin_Interface_Deep_Dive.md`

更新 USB 类比，将 UI 和 Listener 分开描述。

---

## 文件清单

### 新增文件（5 个）

| 路径 | 说明 |
|------|------|
| `packages/sdk/src/types/ui.ts` | IUI 接口定义 |
| `packages/core/src/infrastructure/ui-registry.ts` | UIRegistry |
| `openstarry_plugin/guide-character-init/package.json` | 插件 package |
| `openstarry_plugin/guide-character-init/tsconfig.json` | TS 设置 |
| `openstarry_plugin/guide-character-init/src/index.ts` | 插件实作 |

### 修改文件（13 个）

| 路径 | 修改内容 |
|------|----------|
| `packages/sdk/src/types/listener.ts` | 移除 onEvent |
| `packages/sdk/src/types/plugin.ts` | 新增 ui 字段 |
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

### 文档更新（4 个）

| 路径 |
|------|
| `openstarry_doc/Architecture_Documentation/02_Headless_Agent_Core.md` |
| `openstarry_doc/Architecture_Documentation/16_Plugin_Types_Philosophical_Mapping.md` |
| `openstarry_doc/Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md` |
| `openstarry_doc/Architecture_Documentation/21_Plugin_Interface_Deep_Dive.md` |

---

## 验证结果

| 验证项目 | 结果 |
|---------|------|
| `pnpm build` | ✅ 9 个包全部编译成功 |
| `pnpm test` | ✅ 56 个测试全数通过 |
| `pnpm test:purity` | ✅ Core 不 import 插件 |

---

## 风险评估

| 风险 | 缓解措施 |
|------|----------|
| IListener.onEvent 是 breaking change | transport/bridge.ts 在同一 Phase 更新 |
| 旧 agent.json 有 guideFile | Zod 会静默忽略未知字段，不会报错 |
| standard-function-skill 受影响？ | 不受影响，它只提供 Guide，不用 onEvent |
