# Plan08: TUI Dashboard MVP

> **ステータス**: 📋 計画中 (Target: v0.5.0-beta)
> **Cycle**: 20260211_cycle9
> **フレームワーク**: Ink (React for CLI) — NOT OpenTUI (proprietary)

## 概要

**Ink**（React for CLI）を使用して、OpenStarry エージェント向けのフルスクリーンターミナルダッシュボードを実装します。MVP はリアルタイムのイベント可視化、キーボード駆動の制御、ストリーミングメッセージ表示に焦点を当てます。これにより CLI インターフェース層が完成し、ヘッドレスのマルチエージェント監視が可能になります。

### 主要な目標

1. **新規プラグイン**: `@openstarry-plugin/tui-dashboard` — IUI（色蘊）+ IListener（受蘊）インターフェース
2. **インタラクティブダッシュボード**: stdio readline に代わるフルスクリーンターミナル UI
3. **リアルタイムイベント**: エージェントイベントバスにサブスクライブし、ストリーミングアシスタント応答とツール呼び出しを表示
4. **キーボード制御**: Ctrl+C（終了）、/help（コマンド）、Tab（イベントログ切替）
5. **拡張性**: マルチエージェントダッシュボードに対応するアーキテクチャ（Plan09 へ延期）

---

## 問題のコンテキスト

### 現在の状態

| 制限事項 | 影響 |
|-----------|--------|
| stdio 出力のみ | 視覚的な階層構造がなく、イベントの可視性が低い |
| readline 入力のみ | 単一行のプロンプト、コマンド探索ができない |
| イベントログなし | エージェントの実行フローのデバッグが困難 |
| ステータス表示なし | エージェントが処理中かアイドルか不明 |

### 目標の状態

```
┌────────────────────────────────────────────────────────────┐
│  Agent: ai-assistant v0.5.0-beta                  ⚫ Ready │
├────────────────────────────────────────────────────────────┤
│ User: What is the weather in Tokyo?                        │
│                                                            │
│ Assistant: I'll check the weather for you...               │
│ ⧳ Tool: weather_api [in progress...]                      │
│ ⬥ Event: tool:result { temp: 22°C, humidity: 65% }       │
│                                                            │
│ Result: The weather in Tokyo is currently 22°C with...    │
├────────────────────────────────────────────────────────────┤
│ Events [3] ⟦ Toggle: Tab │ Scroll: ↑↓ │ Help: Ctrl+H ⟧   │
│ [E] agent:started                                          │
│ [S] stream:text_delta "I'll check"                        │
│ [T] tool:call weather_api {...}                           │
├────────────────────────────────────────────────────────────┤
│ > (input)                 Commands: /help /quit Ctrl+C:Exit│
└────────────────────────────────────────────────────────────┘
```

---

## 成果物チェックリスト

### Phase 1: プラグインセットアップ ✅ *設計フェーズ*
- [ ] `openstarry_plugin/tui-dashboard/` パッケージ構造の作成
- [ ] Ink + React 依存関係を含む package.json の設定
- [ ] インターフェース仕様の凍結（IUI + IListener）
- [ ] イベントサブスクリプションパターンの文書化

### Phase 2: コアレイアウトコンポーネント ✅ *実装フェーズ*
- [ ] Header: エージェント名 + バージョン + ステータス表示（●○）
- [ ] Chat area: ANSI カラーによるストリーミングメッセージ表示
- [ ] Input area: テキスト入力フィールド（rx 対応）
- [ ] Footer: キーボードショートカット参照
- [ ] Event log sidebar: 切替可能、スクロール可能なイベント履歴

### Phase 3: イベント統合 ✅ *実装フェーズ*
- [ ] `ctx.onEvent()` による EventBus へのサブスクライブ
- [ ] イベントタイプの視覚的表現へのマッピング
- [ ] バッファリングによるストリーミングテキストの処理
- [ ] ツール呼び出しと結果のカラーコード化

### Phase 4: 入力処理 ✅ *実装フェーズ*
- [ ] Ink の useInput() フックによるキーボード入力
- [ ] スラッシュコマンドの解析（/help、/quit、/clear）
- [ ] テキストバッファリングとカーソル制御
- [ ] Ctrl+C または /quit での優雅な終了

### Phase 5: テスト & 統合 ✅ *検証フェーズ*
- [ ] ユニットテスト（Ink コンポーネントレンダリング）
- [ ] 統合テスト（イベントバスサブスクリプション）
- [ ] stdio フォールバックによる手動スモークテスト
- [ ] ランナー設定の更新（agent.json の例）

### Phase 6: ドキュメント ✅ *収束フェーズ*
- [ ] プラグイン API ドキュメント
- [ ] 設定ガイド（プラグイン設定スキーマ）
- [ ] キーボードショートカット参照
- [ ] トラブルシューティングガイド

---

## 依存関係 & 前提条件

### 必須（存在している必要あり）

| 依存関係 | バージョン | ステータス | 場所 |
|-----------|---------|--------|----------|
| EventBus | N/A | ✅ 実装済み (Plan02) | `packages/core/src/transport/` |
| IUI インターフェース | N/A | ✅ 実装済み (Plan04) | `packages/sdk/src/types/ui.ts` |
| IListener インターフェース | N/A | ✅ 実装済み (Plan04) | `packages/sdk/src/types/listener.ts` |
| Agent ライフサイクル | N/A | ✅ 実装済み (Plan01) | `packages/core/src/agents/` |
| プラグインファクトリーパターン | N/A | ✅ 実装済み (Plan03) | `packages/sdk/src/types/plugin.ts` |

### 外部ライブラリ

| ライブラリ | バージョン | 目的 | ライセンス |
|---------|---------|---------|---------|
| ink | ^5.0.1 | ターミナル用 React | MIT |
| react | ^18.3.1 | コンポーネントモデル | MIT |
| ink-text-input | ^6.0.0 | テキスト入力コンポーネント | MIT |
| ink-spinner | ^5.0.0 | ローディングスピナー | MIT |
| @types/react | ^18.2.0 | TypeScript 定義 | MIT |

### インストール

```bash
cd agent_dev/openstarry_plugin/tui-dashboard
pnpm add ink@^5.0.1 react@^18.3.1
pnpm add -D @types/react@^18.2.0 ink-text-input@^6.0.0 ink-spinner@^5.0.0
```

---

## アーキテクチャサマリー

### プラグイン構造

```
openstarry_plugin/tui-dashboard/
├── package.json                    ← マニフェスト + 依存関係
├── tsconfig.json                   ← TypeScript 設定
└── src/
    ├── index.ts                    ← ファクトリーエクスポート
    ├── components/
    │   ├── app.tsx                 ← ルートコンポーネント + プロバイダーツリー
    │   ├── header.tsx              ← エージェント名 + ステータス
    │   ├── chat-area.tsx           ← メッセージ表示
    │   ├── input-area.tsx          ← テキスト入力フィールド
    │   ├── footer.tsx              ← キーボードショートカット
    │   ├── event-log.tsx           ← イベントリストサイドバー
    │   └── status-indicator.tsx    ← (Ready/Processing/Error)
    ├── contexts/
    │   ├── events.tsx              ← イベントキューコンテキスト
    │   └── input.tsx               ← 入力バッファコンテキスト
    ├── hooks/
    │   ├── use-event-bus.ts        ← EventBus サブスクリプション
    │   └── use-keyboard.ts         ← キーボード入力
    └── types.ts                    ← ローカル型定義
```

### コンポーネント階層

```
<TuiDashboard>                      ← IUI.onEvent() サブスクライバー
  └─ <App>                          ← ルートレイアウト
      ├─ <Header/>                  ← エージェント情報 + ステータス
      ├─ <ChatArea/>                ← メッセージストリーミング
      ├─ <EventLog/>                ← オプションサイドバー
      ├─ <InputArea/>               ← テキスト入力
      └─ <Footer/>                  ← キーボードヘルプ
```

### イベントフロー

```
CoreRuntime EventBus
     ↓ (broadcast)
IUI.onEvent(event)
     ↓ (dispatch)
Events Context
     ↓ (update)
<ChatArea/> + <EventLog/>
     ↓ (re-render)
Terminal Output
```

### 入力フロー

```
Keyboard Input
     ↓ (Ink useInput)
<InputArea/>
     ↓ (parse)
Slash command or raw text
     ↓ (pushInput)
ctx.pushInput({ source: "tui", inputType: "user_input", data: "..." })
     ↓
Agent processes input
```

---

## フェーズ詳細

### Phase 1: プラグインセットアップ & インターフェース凍結

**所要期間**: 1日

#### 1.1 プラグインパッケージの作成

**パス**: `openstarry_plugin/tui-dashboard/package.json`

```json
{
  "name": "@openstarry-plugin/tui-dashboard",
  "version": "0.1.0-alpha",
  "description": "TUI Dashboard plugin (Ink-based)",
  "type": "module",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "scripts": {
    "build": "tsc -b",
    "clean": "rimraf --glob dist \"*.tsbuildinfo\"",
    "dev": "tsc -b --watch"
  },
  "dependencies": {
    "@openstarry/sdk": "workspace:*",
    "ink": "^5.0.1",
    "react": "^18.3.1"
  },
  "devDependencies": {
    "@types/node": "^25.2.0",
    "@types/react": "^18.2.0",
    "ink-spinner": "^5.0.0",
    "ink-text-input": "^6.0.0",
    "rimraf": "^5.0.0",
    "typescript": "^5.5.0"
  }
}
```

#### 1.2 tsconfig.json

他のプラグインと一致する標準設定。

#### 1.3 型定義

**パス**: `src/types.ts`

```typescript
/**
 * TUI Dashboard プラグインのローカル型定義。
 * AgentEvent タイプを視覚的表現にマッピングします。
 */

export interface TuiConfig {
  // Optional: showEventLog? boolean;
  // Optional: logLevel? "debug" | "info" | "warn" | "error";
  // Plan09 へ延期: themes?, keyBindings?
}

export interface ChatMessage {
  role: "user" | "assistant";
  content: string;
  timestamp: number;
  streaming?: boolean;  // トークンの受信中は true
}

export interface EventLogEntry {
  id: string;
  type: string;
  timestamp: number;
  summary: string;  // 例: "tool:call weather_api", "stream:text_delta"
}
```

#### 1.4 インターフェース仕様

**凍結インターフェース**（Plan04 から）:

```typescript
// @openstarry/sdk から
interface IUI {
  id: string;
  name: string;
  onEvent(event: AgentEvent): void | Promise<void>;
  start?(): Promise<void>;
  stop?(): Promise<void>;
}

interface IListener {
  id: string;
  name: string;
  start?(): Promise<void>;
  stop?(): Promise<void>;
  // onEvent は削除済み (Plan04)
}

// PluginHooks.ui に TuiDashboard の IUI インスタンスが含まれる
```

---

### Phase 2: コアレイアウトコンポーネント

**所要期間**: 3日

#### 2.1 Header コンポーネント

**パス**: `src/components/header.tsx`

```typescript
import React from "react";
import { Box, Text } from "ink";

interface HeaderProps {
  agentName: string;
  agentVersion: string;
  status: "ready" | "processing" | "error";
}

const statusIndicator: Record<string, string> = {
  ready: "🟢",      // Green circle
  processing: "🟡", // Yellow circle
  error: "🔴",      // Red circle
};

export function Header({ agentName, agentVersion, status }: HeaderProps) {
  return (
    <Box flexDirection="row" borderStyle="round" borderColor="cyan" paddingX={1}>
      <Text bold cyan>
        Agent: {agentName} v{agentVersion}
      </Text>
      <Box flexGrow={1} />
      <Text>{statusIndicator[status]} {status}</Text>
    </Box>
  );
}
```

#### 2.2 Chat Area コンポーネント

**パス**: `src/components/chat-area.tsx`

```typescript
import React from "react";
import { Box, Text } from "ink";

interface Message {
  role: "user" | "assistant" | "tool";
  content: string;
  streaming?: boolean;
}

interface ChatAreaProps {
  messages: Message[];
  height: number;
}

export function ChatArea({ messages, height }: ChatAreaProps) {
  const displayMessages = messages.slice(-Math.max(height - 3, 1));

  return (
    <Box flexDirection="column" borderStyle="round" borderColor="green" paddingX={1} height={height}>
      {displayMessages.map((msg, i) => (
        <Box key={i} flexDirection="column">
          {msg.role === "user" && (
            <Text>
              <Text bold blue>
                User:
              </Text>
              {" "}
              {msg.content}
            </Text>
          )}
          {msg.role === "assistant" && (
            <Text>
              <Text bold yellow>
                Assistant:
              </Text>
              {" "}
              {msg.content}
              {msg.streaming && <Text dim>▊</Text>}
            </Text>
          )}
          {msg.role === "tool" && (
            <Text dim gray>
              ⧳ Tool: {msg.content}
            </Text>
          )}
        </Box>
      ))}
    </Box>
  );
}
```

#### 2.3 Input Area コンポーネント

**パス**: `src/components/input-area.tsx`

```typescript
import React, { useState } from "ink";
import { Box, Text } from "ink";
import TextInput from "ink-text-input";

interface InputAreaProps {
  onSubmit: (input: string) => void;
}

export function InputArea({ onSubmit }: InputAreaProps) {
  const [input, setInput] = useState("");

  return (
    <Box flexDirection="row" borderStyle="round" borderColor="magenta" paddingX={1}>
      <Text bold magenta>
        &gt;{" "}
      </Text>
      <TextInput
        value={input}
        onChange={setInput}
        onSubmit={(value) => {
          onSubmit(value);
          setInput("");
        }}
        placeholder="Enter command or message..."
      />
    </Box>
  );
}
```

#### 2.4 Footer コンポーネント

**パス**: `src/components/footer.tsx`

```typescript
import React from "react";
import { Box, Text } from "ink";

export function Footer() {
  return (
    <Box flexDirection="row" borderStyle="round" borderColor="gray">
      <Text dim>
        {" "}
        Ctrl+C: Quit │ Tab: Events │ /help: Commands │ Ctrl+H: Help
      </Text>
    </Box>
  );
}
```

#### 2.5 Event Log コンポーネント

**パス**: `src/components/event-log.tsx`

```typescript
import React from "react";
import { Box, Text } from "ink";

interface EventLogEntry {
  id: string;
  type: string;
  timestamp: number;
  summary: string;
}

interface EventLogProps {
  events: EventLogEntry[];
  height: number;
  visible: boolean;
}

export function EventLog({ events, height, visible }: EventLogProps) {
  if (!visible) return null;

  const displayEvents = events.slice(-height).reverse();

  return (
    <Box
      flexDirection="column"
      borderStyle="round"
      borderColor="yellow"
      width={30}
      height={height}
      paddingX={1}
    >
      <Text bold yellow>
        Events [{events.length}]
      </Text>
      {displayEvents.map((evt) => (
        <Text key={evt.id} dim>
          [{evt.type[0]}] {evt.summary}
        </Text>
      ))}
    </Box>
  );
}
```

#### 2.6 App ルートコンポーネント

**パス**: `src/components/app.tsx`

```typescript
import React, { useState, useEffect } from "react";
import { Box, render } from "ink";
import { Header } from "./header.js";
import { ChatArea } from "./chat-area.js";
import { InputArea } from "./input-area.js";
import { Footer } from "./footer.js";
import { EventLog } from "./event-log.js";

interface AppProps {
  agentName: string;
  agentVersion: string;
  onInput: (text: string) => void;
  onQuit: () => void;
}

export function App({ agentName, agentVersion, onInput, onQuit }: AppProps) {
  const [messages, setMessages] = useState<any[]>([]);
  const [events, setEvents] = useState<any[]>([]);
  const [showEvents, setShowEvents] = useState(false);
  const [status, setStatus] = useState<"ready" | "processing" | "error">("ready");

  // TODO: useEventBus フックで messages と events を更新
  // TODO: useKeyboard フックで Tab/Ctrl+H/Ctrl+C を処理

  return (
    <Box flexDirection="column" width={100}>
      <Header agentName={agentName} agentVersion={agentVersion} status={status} />
      <Box flexDirection="row" height={15}>
        <Box flexGrow={1} flexDirection="column">
          <ChatArea messages={messages} height={13} />
        </Box>
        <EventLog events={events} height={13} visible={showEvents} />
      </Box>
      <InputArea onSubmit={onInput} />
      <Footer />
    </Box>
  );
}
```

---

### Phase 3: イベントバス統合

**所要期間**: 2日

#### 3.1 フック: useEventBus

**パス**: `src/hooks/use-event-bus.ts`

```typescript
import { useEffect, useState } from "react";
import type { IPluginContext } from "@openstarry/sdk";
import type { AgentEvent } from "@openstarry/sdk";

export function useEventBus(ctx: IPluginContext) {
  const [messages, setMessages] = useState<any[]>([]);
  const [events, setEvents] = useState<any[]>([]);
  const [status, setStatus] = useState<"ready" | "processing" | "error">("ready");

  useEffect(() => {
    const handleEvent = (event: AgentEvent) => {
      // イベントログに追加
      setEvents((prev) => [
        ...prev,
        {
          id: event.id,
          type: event.type,
          timestamp: event.timestamp,
          summary: summarizeEvent(event),
        },
      ]);

      // メッセージ表示を更新
      const payload = event.payload as Record<string, unknown> | undefined;
      switch (event.type) {
        case "stream:text_delta":
          setMessages((prev) => {
            const last = prev[prev.length - 1];
            if (last?.role === "assistant") {
              return [
                ...prev.slice(0, -1),
                {
                  ...last,
                  content: last.content + (payload?.delta ?? ""),
                  streaming: true,
                },
              ];
            }
            return prev;
          });
          break;
        case "stream:end":
          setMessages((prev) => {
            const last = prev[prev.length - 1];
            if (last?.role === "assistant") {
              return [
                ...prev.slice(0, -1),
                { ...last, streaming: false },
              ];
            }
            return prev;
          });
          setStatus("ready");
          break;
        case "tool:call":
          setMessages((prev) => [
            ...prev,
            {
              role: "tool",
              content: `${payload?.tool} [in progress...]`,
            },
          ]);
          setStatus("processing");
          break;
        case "tool:result":
          // 最後のツールメッセージに結果を追加
          break;
        case "agent:error":
          setStatus("error");
          break;
      }
    };

    // すべてのイベントにサブスクライブ
    // 注: ctx.bus または同様のイベントサブスクリプションメカニズムが必要
    // Phase 3 の最終化で実装
    // ctx.onEvent(handleEvent);

    return () => {
      // クリーンアップ: サブスクライブ解除
    };
  }, [ctx]);

  return { messages, events, status };
}

function summarizeEvent(event: AgentEvent): string {
  const payload = event.payload as Record<string, unknown> | undefined;
  switch (event.type) {
    case "stream:text_delta":
      return `"${(payload?.delta as string)?.slice(0, 20) ?? "..."}"`;
    case "tool:call":
      return `tool:call ${payload?.tool ?? "unknown"}`;
    case "tool:result":
      return "tool:result";
    case "agent:started":
      return "agent:started";
    case "agent:stopped":
      return "agent:stopped";
    default:
      return event.type;
  }
}
```

#### 3.2 フック: useKeyboard

**パス**: `src/hooks/use-keyboard.ts`

```typescript
import { useEffect } from "react";
import type { UnknownObject } from "ink";

interface KeyboardHandlers {
  onQuit: () => void;
  onToggleEvents: () => void;
  onHelp: () => void;
}

export function useKeyboard(handlers: KeyboardHandlers) {
  useEffect(() => {
    const handleKeyInput = (
      ch: string,
      key: { ctrl: boolean; shift: boolean; name: string }
    ) => {
      if (key.ctrl && key.name === "c") {
        handlers.onQuit();
      } else if (ch === "\t") {
        // Tab: イベントログの切替
        handlers.onToggleEvents();
      } else if ((key.ctrl && key.name === "h") || ch === "?") {
        // Ctrl+H または ?: ヘルプを表示
        handlers.onHelp();
      }
    };

    // 注: Ink の useInput フックが代わりに使用される
    // キーボード処理パターンのプレースホルダー

    return () => {
      // クリーンアップ: stdin リスナーの解除
    };
  }, [handlers]);
}
```

---

### Phase 4: 入力処理 & テキスト入力

**所要期間**: 1日

#### 4.1 TuiListener コンポーネント

**パス**: `src/components/tui-listener.tsx`

```typescript
import React from "react";
import type { IListener } from "@openstarry/sdk";
import type { IPluginContext } from "@openstarry/sdk";

/**
 * TuiListener は IListener インターフェース（受蘊）を実装します。
 * Ink コンポーネントからユーザー入力を受け取り、コアにプッシュします。
 */
export function createTuiListener(ctx: IPluginContext): IListener {
  return {
    id: "tui-listener",
    name: "TUI Dashboard Listener",

    async start(): Promise<void> {
      // リスナーは直接 stdin をバインドしない; 入力は Ink コンポーネントから来る
      // プラグイン起動時に呼び出される
    },

    async stop(): Promise<void> {
      // クリーンアップ
    },
  };
}
```

#### 4.2 TuiUI コンポーネント

**パス**: `src/components/tui-ui.tsx`

```typescript
import React from "react";
import type { IUI, AgentEvent, IPluginContext } from "@openstarry/sdk";
import { App } from "./app.js";

/**
 * TuiUI は IUI インターフェース（色蘊）を実装します。
 * コアからイベントを受け取り、ターミナルにレンダリングします。
 */
export function createTuiUI(ctx: IPluginContext): IUI {
  return {
    id: "tui-dashboard",
    name: "TUI Dashboard UI",

    onEvent(event: AgentEvent): void {
      // イベントハンドラーが React の状態を更新
      // Events コンテキスト経由でディスパッチ
      // useEventBus フックを参照
    },

    async start(): Promise<void> {
      // Ink アプリのレンダリング
    },

    async stop(): Promise<void> {
      // クリーンアップ
    },
  };
}
```

---

### Phase 5: プラグインファクトリー & 統合

**所要期間**: 1日

#### 5.1 メインインデックスエクスポート

**パス**: `src/index.ts`

```typescript
/**
 * tui-dashboard — フルスクリーン TUI ダッシュボードプラグイン。
 *
 * 提供するもの:
 * - TuiUI（色蘊）— エージェントイベントをターミナルにレンダリング
 * - TuiListener（受蘊）— ユーザー入力を受信
 *
 * Config: {}（MVP では設定不要）
 */

import type { IPlugin, IPluginContext, PluginHooks } from "@openstarry/sdk";

export interface TuiDashboardConfig {
  // MVP: 設定なし
  // 将来（Plan09）: theme, keyBindings, logLevel
}

export function createTuiDashboardPlugin(): IPlugin {
  return {
    manifest: {
      name: "@openstarry-plugin/tui-dashboard",
      version: "0.1.0-alpha",
      description: "Full-screen TUI Dashboard (Ink-based, MVP)",
    },

    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      const config = ctx.config as TuiDashboardConfig;

      // UI と Listener インスタンスの作成
      const ui = createTuiUI(ctx);
      const listener = createTuiListener(ctx);

      return {
        ui: [ui],
        listeners: [listener],
        async dispose() {
          await listener.stop?.();
          await ui.stop?.();
        },
      };
    },
  };
}

export default createTuiDashboardPlugin;

// 内部ヘルパー
function createTuiUI(ctx: IPluginContext) {
  // Phase 4.2 の実装
}

function createTuiListener(ctx: IPluginContext) {
  // Phase 4.1 の実装
}
```

#### 5.2 ランナー設定の更新

**パス**: `apps/runner/src/bin.ts` (defaultConfig)

```typescript
function defaultConfig(): AgentConfig {
  return {
    identity: {
      id: "openstarry-agent",
      name: "OpenStarry AI Agent",
    },
    cognition: {
      provider: "gemini-oauth",
      model: "gemini-2.0-flash",
    },
    plugins: [
      { name: "@openstarry-plugin/provider-gemini-oauth" },
      { name: "@openstarry-plugin/standard-function-fs" },
      { name: "@openstarry-plugin/standard-function-stdio" },
      { name: "@openstarry-plugin/guide-character-init" },
      { name: "@openstarry-plugin/tui-dashboard" },  // NEW
    ],
    guide: "default-guide",
  };
}
```

---

### Phase 6: テスト & 検証

**所要期間**: 2日

#### 6.1 ユニットテスト

**パス**: `src/components/header.test.ts`

```typescript
import { describe, it, expect } from "vitest";
import { Header } from "./header.js";

describe("Header component", () => {
  it("renders agent name and version", () => {
    const props = {
      agentName: "Test Agent",
      agentVersion: "0.1.0",
      status: "ready" as const,
    };
    // コンポーネントレンダリングのテスト（Ink スナップショット）
    // expect(...).toMatchSnapshot();
  });

  it("displays status indicator", () => {
    // ステータスシンボル表示のテスト
  });
});
```

#### 6.2 統合テスト

**パス**: `src/index.test.ts`

```typescript
import { describe, it, expect } from "vitest";
import { createTuiDashboardPlugin } from "./index.js";
import type { IPluginContext } from "@openstarry/sdk";

describe("TUI Dashboard Plugin", () => {
  it("exports IUI and IListener hooks", async () => {
    const mockCtx: Partial<IPluginContext> = {
      config: {},
      pushInput: vi.fn(),
    };

    const plugin = createTuiDashboardPlugin();
    const hooks = await plugin.factory(mockCtx as IPluginContext);

    expect(hooks.ui).toHaveLength(1);
    expect(hooks.listeners).toHaveLength(1);
    expect(hooks.ui![0].id).toBe("tui-dashboard");
    expect(hooks.listeners![0].id).toBe("tui-listener");
  });

  it("calls onEvent without error", () => {
    const hooks = await plugin.factory(mockCtx as IPluginContext);
    const ui = hooks.ui![0];

    const event: AgentEvent = {
      id: "test-1",
      type: "stream:text_delta",
      timestamp: Date.now(),
      payload: { delta: "Hello" },
    };

    expect(() => ui.onEvent(event)).not.toThrow();
  });

  it("pushes input on listener submission", async () => {
    const pushInputSpy = vi.fn();
    const mockCtx: Partial<IPluginContext> = {
      config: {},
      pushInput: pushInputSpy,
    };

    const hooks = await plugin.factory(mockCtx as IPluginContext);
    const listener = hooks.listeners![0];

    // 入力送信のシミュレーション
    // listener.onInput("Hello");  // 入力コールバックが存在する場合

    // expect(pushInputSpy).toHaveBeenCalledWith(
    //   expect.objectContaining({ data: "Hello" })
    // );
  });
});
```

---

## 成功基準

### 機能要件

| 要件 | 受入基準 | 優先度 |
|-------------|-------------------|----------|
| フルスクリーンターミナル UI | アプリがクラッシュせずにレンダリングされ、ターミナルを占有する | ✅ MUST |
| リアルタイムイベント表示 | イベント発行後100ms以内に表示される | ✅ MUST |
| ユーザー入力処理 | テキスト入力とスラッシュコマンドが動作する | ✅ MUST |
| 優雅な終了 | Ctrl+C または /quit でクリーンに終了する | ✅ MUST |
| ステータス表示 | ready/processing/error の状態を表示する | 🟡 SHOULD |
| イベントログサイドバー | Tab キーで表示/非表示を切り替え | 🟡 SHOULD |
| ストリーミングメッセージ | アシスタントの応答が段階的に表示される | ✅ MUST |
| キーボードショートカット | フッターに少なくとも5つのショートカットが表示される | 🟡 SHOULD |

### 非機能要件

| 要件 | 受入基準 | 目標値 |
|-------------|-------------------|--------|
| テストカバレッジ | >80% カバレッジ | 85%+ |
| パフォーマンス | 再レンダリング時間 <50ms | <30ms |
| ターミナル互換性 | macOS + Linux + Windows で動作 | 100% |
| コード品質 | TypeScript strict、エラーなし | 0 errors |
| ドキュメント | README + API ドキュメント完備 | 100% |

### ビルド & QA ゲート

| ゲート | ステータス | 備考 |
|------|--------|-------|
| `pnpm build` | 通過必須 | 全11パッケージがコンパイル |
| `pnpm test` | 通過必須 | >470テスト（Plan08 から新規 +20） |
| `pnpm test:purity` | 通過必須 | コアがプラグインをインポートしない |
| 手動スモークテスト | 通過必須 | tui-dashboard プラグインでエージェントを実行 |
| コードレビュー | 通過必須 | architect が承認 |

---

## スコープ外（Plan09 へ延期）

- **マルチエージェントダッシュボード**: まだ daemon やプロセス管理がない
- **`openstarry design`**: インタラクティブデザイナー CLI
- **`openstarry attach`**: 実行中のエージェントへのシームレスな接続
- **高度なテーマ**: カラースキーム、カスタムスタイリング
- **コマンドパレット**: コマンドのファジー検索
- **シンタックスハイライト**: チャット内のコードブロックレンダリング
- **クリップボードコピー**: ターミナルからテキストを選択してコピー
- **マウスサポート**: クリックフォーカス、ドラッグ選択
- **ログ永続化**: チャット履歴のディスクへの保存

---

## ファイルチェックリスト

### 新規ファイル（合計11）

| パス | タイプ | 行数 | 備考 |
|------|------|-------|-------|
| `openstarry_plugin/tui-dashboard/package.json` | 設定 | 30 | 依存関係: ink, react |
| `openstarry_plugin/tui-dashboard/tsconfig.json` | 設定 | 20 | 標準 TS 設定 |
| `openstarry_plugin/tui-dashboard/src/index.ts` | コード | 80 | プラグインファクトリー |
| `openstarry_plugin/tui-dashboard/src/types.ts` | コード | 30 | ローカル型 |
| `openstarry_plugin/tui-dashboard/src/components/app.tsx` | コード | 60 | ルートコンポーネント |
| `openstarry_plugin/tui-dashboard/src/components/header.tsx` | コード | 25 | ヘッダーバー |
| `openstarry_plugin/tui-dashboard/src/components/chat-area.tsx` | コード | 45 | チャット表示 |
| `openstarry_plugin/tui-dashboard/src/components/input-area.tsx` | コード | 35 | 入力フィールド |
| `openstarry_plugin/tui-dashboard/src/components/footer.tsx` | コード | 15 | フッターショートカット |
| `openstarry_plugin/tui-dashboard/src/components/event-log.tsx` | コード | 40 | イベントサイドバー |
| `openstarry_plugin/tui-dashboard/src/hooks/use-event-bus.ts` | コード | 80 | EventBus フック |

### 変更ファイル（合計2）

| パス | 変更内容 | 備考 |
|------|--------|-------|
| `apps/runner/src/bin.ts` | defaultConfig に tui-dashboard を追加 | プラグイン設定 |
| `openstarry/tsconfig.json` | tui-dashboard 参照を追加 | ワークスペース参照 |

### テストファイル（合計3）

| パス | テスト数 | 備考 |
|------|-------|-------|
| `openstarry_plugin/tui-dashboard/src/index.test.ts` | 4 | プラグインファクトリーテスト |
| `openstarry_plugin/tui-dashboard/src/components/header.test.ts` | 2 | コンポーネントレンダリング |
| `openstarry_plugin/tui-dashboard/src/hooks/use-event-bus.test.ts` | 6 | イベントフック統合 |

---

## リスク評価

| リスク | 確率 | 影響 | 緩和策 |
|------|-------------|--------|-----------|
| Ink API の不安定性 | 低 | 中 | v5 LTS を使用、バージョンを固定、複数 Node バージョンでテスト |
| EventBus サブスクリプションの遅延 | 低 | 低 | イベントをバッチ処理（16ms）、更新をデバウンス |
| ターミナルリサイズ処理 | 中 | 低 | Ink が自動処理、一般的なターミナルでテスト |
| 長時間セッションでのメモリリーク | 低 | 中 | イベントリスナーのクリーンアップ、メッセージ履歴を1000に制限 |
| React フックの学習曲線 | 低 | 低 | Plan04 参照、Ink のドキュメントは優秀 |
| StdIO プラグインとの競合 | 中 | 高 | tui-dashboard 有効時に stdio を無効にする（設定で相互排他） |

---

## 統合チェックリスト

- [ ] `@openstarry-plugin/tui-dashboard` をワークスペース `pnpm-workspace.yaml` に追加
- [ ] ランナーの package.json に tui-dashboard 依存関係を追加
- [ ] ルート tsconfig.json に tui-dashboard 参照を追加
- [ ] `pnpm install && pnpm build` でテスト
- [ ] 既存のテストスイートを実行してリグレッションがないことを確認
- [ ] agent.json のデフォルト設定に tui-dashboard を追加
- [ ] User_Scenario_and_Workflow_Guide.md にプラグインを記載

---

## 参照

### 事前調査
- `/data/openstarry_eco/share/test/reports/research/Plan08_TUI_PreResearch.md`
  - フレームワーク分析: OpenTUI vs Ink vs Blessed
  - 推奨: Ink（React for CLI）
  - 状態管理パターン: 二層構成（sync + local）

### アーキテクチャ基盤
- `/data/openstarry_eco/share/openstarry_doc/Architecture_Documentation/02_Headless_Agent_Core.md`
  - EventBus ブロードキャスト
  - Transport bridge パターン

### プラグインシステム
- `/data/openstarry_eco/share/openstarry_doc/Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md`
  - IUI（色蘊）インターフェース
  - IListener（受蘊）インターフェース
  - 五蘊アーキテクチャ

### 外部参照
- **Ink Documentation**: https://github.com/vadimdemedes/ink
- **React Hooks**: https://react.dev/reference/react
- **TypeScript Strict Mode**: https://www.typescriptlang.org/tsconfig#strict

---

## 次のステップ（収束後）

### 直近（Plan08+1）
1. architect によるコードレビュー
2. 複数ターミナルでの QA テスト
3. ドキュメントレビュー
4. v0.5.0-beta としてリリース

### 短期（Plan09）
1. マルチエージェントダッシュボードサポートの追加
2. `openstarry design` CLI の実装
3. テーマカスタマイズの追加
4. React.memo() によるパフォーマンス改善

### 中期（Plan10+）
1. ターミナルアタッチメント（`openstarry attach`）
2. シームレスなエージェント生成
3. 高度なキーボードショートカット（コマンドパレット）
4. LSP/MCP ステータス統合

---

## ドキュメント管理

| フィールド | 値 |
|-------|-------|
| Plan ID | Plan08 |
| バージョン | 1.0-DRAFT |
| 作成日 | 2026-02-12 |
| ステータス | 📋 計画フェーズの準備完了 |
| Cycle | 20260211_cycle9 |
| オーナー | coordinator |
| フレームワーク | Ink ^5.0.1 |
| ターゲットバージョン | v0.5.0-beta |
