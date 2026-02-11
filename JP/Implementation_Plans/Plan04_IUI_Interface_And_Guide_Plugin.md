# OpenStarry アーキテクチャ完成計画： IUI インターフェース + guide-character-init プラグイン

> **ステータス**: ✅ 完了 (2026-02-06)

## 目標

五蘊（ごうん）プラグインアーキテクチャを完成させる：

1. **IUI インターフェース（色蘊）の新設** — 出力レンダリングを IListener から分離し、五蘊の完全な定義を達成する
2. **guide-character-init プラグインの構築** — 基礎となるキャラクター設定（人設）を stdio とコアから分離し、独立した Guide プラグインとする
3. **ドキュメントの修正** — 「 UI は特殊な Listener と見なすことができる」という誤った記述を削除する

---

## 問題の背景

### 現状の課題

| 問題 | 影響 |
|------|------|
| IListener が同時に入出力の両方を処理している | 単一責任の原則に反し、受蘊（入力）と色蘊（出力）が混同されている |
| SDK に IUI インターフェースが欠けている | 五蘊アーキテクチャが不完全（ 4/5 しかない） |
| stdio プラグインに Guide が紐付いている | Guide は独立しているべきであり、 I/O プラグインと結合すべきではない |
| コアが直接 guideFile を読み取っている | コアの純粋性に反する（コアはファイル I/O を行うべきではない） |
| ドキュメントの誤った記述 | UI が Listener の特例であるという誤解を開発者に与える |

### 目標とするアーキテクチャ

```
五蘊の完全な定義：
├── IUI (色蘊)      — 出力レンダリング。 AgentEvent を受信して提示する
├── IListener (受蘊) — 入力の受信。 InputEvent をキューへプッシュする
├── IProvider (想蘊) — 認知推論。 LLM アダプター
├── ITool (行蘊)    — アクションの実行。ファイル/API/コードなど
└── IGuide (識蘊)   — アイデンティティ/キャラ設定。 system prompt
```

---

## 実装計画

### フェーズ 1： SDK レイヤーの修正（4ファイル） ✅

**以降のすべての修正は、 SDK の完了に依存します。**

#### 1.1 `types/ui.ts` の新設

**パス**: `packages/sdk/src/types/ui.ts`

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

#### 1.2 `types/listener.ts` の修正

** `onEvent` を削除** し、 Listener は入力のみを担当するようにします：

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

#### 1.3 `types/plugin.ts` の修正

`PluginHooks` に `ui` フィールドを追加します：

```typescript
import type { IUI } from "./ui.js";

export interface PluginHooks {
  providers?: IProvider[];
  tools?: ITool[];
  listeners?: IListener[];
  ui?: IUI[];               // 新設 — 色蘊
  guides?: IGuide[];
  commands?: SlashCommand[];
  dispose?: () => Promise<void> | void;
}
```

#### 1.4 `index.ts` の修正

IUI のエクスポートを追加します：

```typescript
export type { IUI } from "./types/ui.js";
```

---

### フェーズ 2：コアレイヤーの修正（6ファイル） ✅

**フェーズ 1 と同時に完了させる必要があります。そうしないとコンパイルに失敗します。**

#### 2.1 `infrastructure/ui-registry.ts` の新設

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

#### 2.2 `infrastructure/index.ts` の修正

UIRegistry のエクスポートを追加。

#### 2.3 `infrastructure/plugin-loader.ts` の修正

- `PluginLoaderDeps` に `uiRegistry: UIRegistry` を追加。
- `hooks.ui` の登録ループを追加。

#### 2.4 `transport/bridge.ts` の修正

**重要な変更点**： ListenerRegistry から UIRegistry に変更。

```typescript
import type { UIRegistry } from "../infrastructure/ui-registry.js";

export function createTransportBridge(
  bus: EventBus,
  uiRegistry: UIRegistry,  // UIRegistry に変更
): TransportBridge {
  function broadcast(event: AgentEvent): void {
    for (const ui of uiRegistry.list()) {  // ui に変更
      try {
        ui.onEvent(event);  // UI の onEvent を呼び出し
      } catch (err) {
        logger.error(`UI ${ui.id} error`, { error: String(err) });
      }
    }
  }
  // ...
}
```

#### 2.5 `agents/agent-core.ts` の修正

- `import { readFile } from "node:fs/promises"` を **削除** （コアの純粋性確保）。
- `import { resolve } from "node:path"` を **削除** 。
- `createUIRegistry` のインポートを **追加** 。
- `AgentCore` インターフェースに `uiRegistry` を **追加** 。
- `createTransportBridge(bus, uiRegistry)` に **修正** 。
- `if (config.guideFile)` ブロックをまるごと **削除** 。
- UI の start/stop 呼び出しを **追加** 。

#### 2.6 `index.ts` (Core barrel) の修正

`createUIRegistry`, `UIRegistry` のエクスポートを追加。

---

### フェーズ 3： standard-function-stdio プラグインの重構 ✅

**パス**: `openstarry_plugin/standard-function-stdio/src/index.ts`

#### 2つのコンポーネントに分離

**StdioListener（入力）**：
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

**StdioUI（出力）**：
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
      // 元の onEvent の switch ロジックを完全に保持
      // AGENT_STARTED, STREAM_TEXT_DELTA, TOOL_RESULT, etc.
    },
  };
}
```

**ファクトリの更新**：
```typescript
export function createStdioPlugin(): IPlugin {
  return {
    manifest: { name: "standard-function-stdio", version: "0.1.0-alpha", ... },
    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      return {
        listeners: [createStdioListener(ctx)],
        ui: [createStdioUI()],
        // guide は返さないように変更
      };
    },
  };
}
```

---

### フェーズ 4： guide-character-init プラグインの新設（3ファイル） ✅

**パス**: `openstarry_plugin/guide-character-init/`

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

他のプラグインと同様の設定。

#### 4.3 src/index.ts

```typescript
/**
 * guide-character-init — Base persona / system prompt provider (識蘊).
 *
 * Config:
 *   { prompt: "..." }           — インライン system prompt
 *   { characterFile: "./x.md" } — ファイルからロード
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

### フェーズ 5：設定ファイルの更新（3ファイル） ✅

#### 5.1 SDK `types/agent.ts`

`guideFile` フィールドを **削除** 。

#### 5.2 Shared `utils/config-schema.ts`

Zod schema から `guideFile` を **削除** 。

#### 5.3 Runner `apps/runner/src/bin.ts`

`defaultConfig()` を更新：

```typescript
plugins: [
  { name: "@openstarry-plugin/provider-gemini-oauth" },
  { name: "@openstarry-plugin/standard-function-fs" },
  { name: "@openstarry-plugin/standard-function-stdio" },
  { name: "@openstarry-plugin/guide-character-init" },  // 新設
],
guide: "default-guide",
```

---

### フェーズ 6：ドキュメントの更新（4ファイル） ✅

#### 6.1 `02_Headless_Agent_Core.md`

**削除**：
> UI プラグインは、特殊な Listener と見なすことができるようになりました。

**修正後**：
> UI プラグイン (色蘊) は独立した IUI インターフェースを持ち、イベントを受信して出力をレンダリングする役割を担います。
> リスナープラグイン (受蘊) は外部入力の受信に専念します。両者はそれぞれの役割を果たします。

#### 6.2 `16_Plugin_Types_Philosophical_Mapping.md`

「色蘊」のセクションに IUI インターフェースの説明を追加。

#### 6.3 `00_Plugin_Philosophy_Five_Aggregates.md`

JSON の例を更新し、 `ui` フィールドを追加。

#### 6.4 `21_Plugin_Interface_Deep_Dive.md`

USB の比喩を更新し、 UI と Listener を分けて記述。

---

## ファイルリスト

### 新設ファイル（5個）

| パス | 説明 |
|------|------|
| `packages/sdk/src/types/ui.ts` | IUI インターフェース定義 |
| `packages/core/src/infrastructure/ui-registry.ts` | UIRegistry 実装 |
| `openstarry_plugin/guide-character-init/package.json` | プラグインパッケージ |
| `openstarry_plugin/guide-character-init/tsconfig.json` | TS 設定 |
| `openstarry_plugin/guide-character-init/src/index.ts` | プラグイン実装 |

### 修正ファイル（13個）

| パス | 修正内容 |
|------|----------|
| `packages/sdk/src/types/listener.ts` | onEvent の削除 |
| `packages/sdk/src/types/plugin.ts` | ui フィールドの追加 |
| `packages/sdk/src/types/agent.ts` | guideFile の削除 |
| `packages/sdk/src/index.ts` | IUI のエクスポート |
| `packages/core/src/infrastructure/index.ts` | UIRegistry のエクスポート |
| `packages/core/src/infrastructure/plugin-loader.ts` | uiRegistry の追加 |
| `packages/core/src/transport/bridge.ts` | UIRegistry の使用に変更 |
| `packages/core/src/agents/agent-core.ts` | UIRegistry の追加、 guideFile I/O の削除 |
| `packages/core/src/index.ts` | UIRegistry のエクスポート |
| `packages/shared/src/utils/config-schema.ts` | guideFile の削除 |
| `openstarry_plugin/standard-function-stdio/src/index.ts` | Listener と UI を分離、 Guide を削除 |
| `apps/runner/src/bin.ts` | defaultConfig に guide-character-init を追加 |
| `openstarry/tsconfig.json` | guide-character-init の参照を追加 |

### ドキュメントの更新（4個）

| パス |
|------|
| `openstarry_doc/Architecture_Documentation/02_Headless_Agent_Core.md` |
| `openstarry_doc/Architecture_Documentation/16_Plugin_Types_Philosophical_Mapping.md` |
| `openstarry_doc/Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md` |
| `openstarry_doc/Architecture_Documentation/21_Plugin_Interface_Deep_Dive.md` |

---

## 検証結果

| 検証項目 | 結果 |
|---------|------|
| `pnpm build` | ✅ 9個のパッケージすべてコンパイル成功 |
| `pnpm test` | ✅ 56個のテストすべて通過 |
| `pnpm test:purity` | ✅ コアがプラグインをインポートしていない |

---

## リスク評価

| リスク | 緩和策 |
|------|----------|
| `IListener.onEvent` の削除は破壊的変更 | `transport/bridge.ts` を同じフェーズで更新済み |
| 古い `agent.json` に `guideFile` が残っている | Zod が未知のフィールドを無視するため、エラーは発生しない |
| `standard-function-skill` への影響は？ | 影響なし。 Guide のみを提供しており、 `onEvent` は未使用 |
