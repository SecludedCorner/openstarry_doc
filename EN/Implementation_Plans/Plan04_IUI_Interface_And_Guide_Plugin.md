# OpenStarry Architectural Refinement Plan: IUI Interface & guide-character-init Plugin

> **Status**: ✅ Completed (2026-02-06)

## Objectives

Refine the Five Aggregates plugin architecture:

1. **Add IUI Interface (Form / 色蘊)** — Separate output rendering from `IListener`, completing the formal definition of the Five Aggregates.
2. **Establish guide-character-init Plugin** — Decouple basic personas from stdio and the Core, making them an independent Guide plugin.
3. **Correct Documentation** — Remove incorrect statements characterizing "UI as a special type of Listener."

---

## Background

### Current Issues

| Issue | Impact |
|------|------|
| `IListener` handles both input and output. | Violates Single Responsibility Principle; confuses Sensation (input) with Form (output). |
| SDK missing `IUI` interface. | Five Aggregates architecture is incomplete (only 4/5 defined). |
| `stdio` plugin bundled with Guide. | Guide should be independent, not coupled with I/O plugins. |
| Core reads `guideFile` directly. | Violates Core purity (Core should not perform file I/O). |
| Incorrect documentation. | Misleads developers into thinking UI is a special case of Listener. |

### Target Architecture

```
Complete Five Aggregates Definition:
├── IUI (Form)         — Output rendering; receives AgentEvents and presents them.
├── IListener (Sensation) — Input reception; pushes InputEvents to the queue.
├── IProvider (Perception) — Cognitive reasoning; LLM adapter.
├── ITool (Volition)     — Action execution; Files/APIs/Code.
└── IGuide (Consciousness) — Identity and persona; system prompt.
```

---

## Implementation Plan

### Phase 1: SDK Layer Modifications (4 files) ✅

**All subsequent modifications depend on the completion of the SDK changes.**

#### 1.1 Add `types/ui.ts`

**Path**: `packages/sdk/src/types/ui.ts`

```typescript
/**
 * UI interface — defines how the agent presents itself (Form / 色蘊).
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

#### 1.2 Modify `types/listener.ts`

**Remove `onEvent`**; Listener is now responsible only for input:

```typescript
/**
 * Listener interface — receives external input (Sensation / 受蘊).
 */
export interface IListener {
  id: string;
  name: string;
  start?(): Promise<void>;
  stop?(): Promise<void>;
}
```

#### 1.3 Modify `types/plugin.ts`

Add `ui` field to `PluginHooks`:

```typescript
import type { IUI } from "./ui.js";

export interface PluginHooks {
  providers?: IProvider[];
  tools?: ITool[];
  listeners?: IListener[];
  ui?: IUI[];               // New — Form / 色蘊
  guides?: IGuide[];
  commands?: SlashCommand[];
  dispose?: () => Promise<void> | void;
}
```

#### 1.4 Modify `index.ts`

Export `IUI`:

```typescript
export type { IUI } from "./types/ui.js";
```

---

### Phase 2: Core Layer Modifications (6 files) ✅

**Must be completed concurrently with Phase 1 to avoid build failures.**

#### 2.1 Add `infrastructure/ui-registry.ts`

```typescript
/**
 * UIRegistry — manages UI renderers (Form / 色蘊).
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

#### 2.2 Modify `infrastructure/index.ts`

Export `UIRegistry`.

#### 2.3 Modify `infrastructure/plugin-loader.ts`

- Add `uiRegistry: UIRegistry` to `PluginLoaderDeps`.
- Add registration loop for `hooks.ui`.

#### 2.4 Modify `transport/bridge.ts`

**Key Change**: Switch from `ListenerRegistry` to `UIRegistry`.

```typescript
import type { UIRegistry } from "../infrastructure/ui-registry.js";

export function createTransportBridge(
  bus: EventBus,
  uiRegistry: UIRegistry,  // Changed to UIRegistry
): TransportBridge {
  function broadcast(event: AgentEvent): void {
    for (const ui of uiRegistry.list()) {  // Iterating through UIs
      try {
        ui.onEvent(event);  // Invoke UI's onEvent
      } catch (err) {
        logger.error(`UI ${ui.id} error`, { error: String(err) });
      }
    }
  }
  // ...
}
```

#### 2.5 Modify `agents/agent-core.ts`

- **Remove** `import { readFile } from "node:fs/promises"` (Ensures Core purity).
- **Remove** `import { resolve } from "node:path"`.
- **Add** `createUIRegistry` import.
- **Add** `uiRegistry` to the `AgentCore` interface.
- **Modify** `createTransportBridge(bus, uiRegistry)`.
- **Remove** the entire `if (config.guideFile)` block.
- **Add** start/stop calls for UIs.

#### 2.6 Modify `index.ts` (Core barrel)

Export `createUIRegistry` and `UIRegistry`.

---

### Phase 3: Refactor standard-function-stdio Plugin ✅

**Path**: `openstarry_plugin/standard-function-stdio/src/index.ts`

#### Split into Two Components

**StdioListener (Input)**:
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

**StdioUI (Output)**:
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
      // The original switch logic for onEvent is fully retained
      // AGENT_STARTED, STREAM_TEXT_DELTA, TOOL_RESULT, etc.
    },
  };
}
```

**Factory Update**:
```typescript
export function createStdioPlugin(): IPlugin {
  return {
    manifest: { name: "standard-function-stdio", version: "0.1.0-alpha", ... },
    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      return {
        listeners: [createStdioListener(ctx)],
        ui: [createStdioUI()],
        // No longer returns guides
      };
    },
  };
}
```

---

### Phase 4: Add guide-character-init Plugin (3 files) ✅

**Path**: `openstarry_plugin/guide-character-init/`

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

Standard configuration consistent with other plugins.

#### 4.3 src/index.ts

```typescript
/**
 * guide-character-init — Base persona / system prompt provider (Consciousness / 識蘊).
 *
 * Config:
 *   { prompt: "..." }           — Inline system prompt
 *   { characterFile: "./x.md" } — Load from file
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

### Phase 5: Configuration Updates (3 files) ✅

#### 5.1 SDK `types/agent.ts`

**Remove** `guideFile` field.

#### 5.2 Shared `utils/config-schema.ts`

**Remove** `guideFile` from the Zod schema.

#### 5.3 Runner `apps/runner/src/bin.ts`

Update `defaultConfig()`:

```typescript
plugins: [
  { name: "@openstarry-plugin/provider-gemini-oauth" },
  { name: "@openstarry-plugin/standard-function-fs" },
  { name: "@openstarry-plugin/standard-function-stdio" },
  { name: "@openstarry-plugin/guide-character-init" },  // New
],
guide: "default-guide",
```

---

### Phase 6: Documentation Updates (4 files) ✅

#### 6.1 `02_Headless_Agent_Core.md`

**Remove**:
> UI plugins can now be seen as a special type of Listener.

**Replace with**:
> UI plugins (Form / 色蘊) have an independent IUI interface responsible for receiving events and rendering output.
> Listener plugins (Sensation / 受蘊) focus on receiving external input. Each serves its own distinct purpose.

#### 6.2 `16_Plugin_Types_Philosophical_Mapping.md`

Add description of the `IUI` interface to the "Form (色蘊)" section.

#### 6.3 `00_Plugin_Philosophy_Five_Aggregates.md`

Update JSON examples to include the `ui` field.

#### 6.4 `21_Plugin_Interface_Deep_Dive.md`

Update the USB analogy to describe UI and Listener separately.

---

## File Manifest

### New Files (5)

| Path | Description |
|------|------|
| `packages/sdk/src/types/ui.ts` | IUI interface definition |
| `packages/core/src/infrastructure/ui-registry.ts` | UIRegistry implementation |
| `openstarry_plugin/guide-character-init/package.json` | Plugin package configuration |
| `openstarry_plugin/guide-character-init/tsconfig.json` | TS configuration |
| `openstarry_plugin/guide-character-init/src/index.ts` | Plugin implementation |

### Modified Files (13)

| Path | Description of Modification |
|------|----------|
| `packages/sdk/src/types/listener.ts` | Removed `onEvent`. |
| `packages/sdk/src/types/plugin.ts` | Added `ui` field to `PluginHooks`. |
| `packages/sdk/src/types/agent.ts` | Removed `guideFile`. |
| `packages/sdk/src/index.ts` | Exported `IUI`. |
| `packages/core/src/infrastructure/index.ts` | Exported `UIRegistry`. |
| `packages/core/src/infrastructure/plugin-loader.ts` | Added `uiRegistry` support. |
| `packages/core/src/transport/bridge.ts` | Switched to using `UIRegistry`. |
| `packages/core/src/agents/agent-core.ts` | Added `UIRegistry`; removed `guideFile` I/O. |
| `packages/core/src/index.ts` | Exported `UIRegistry`. |
| `packages/shared/src/utils/config-schema.ts` | Removed `guideFile` from schema. |
| `openstarry_plugin/standard-function-stdio/src/index.ts` | Split into Listener and UI; removed Guide. |
| `apps/runner/src/bin.ts` | Added `guide-character-init` to `defaultConfig`. |
| `openstarry/tsconfig.json` | Added reference to `guide-character-init`. |

---

## Verification Results

| Item | Result |
|---------|------|
| `pnpm build` | ✅ All 9 packages compiled successfully. |
| `pnpm test` | ✅ All 56 tests passed. |
| `pnpm test:purity` | ✅ Core does not import plugins. |

---

## Risk Assessment

| Risk | Mitigation |
|------|----------|
| `IListener.onEvent` removal is a breaking change. | `transport/bridge.ts` updated in the same phase. |
| Old `agent.json` contains `guideFile`. | Zod silently ignores unknown fields; no crash. |
| Impact on `standard-function-skill`? | None; it provides only a Guide and does not use `onEvent`. |
