# Plan08: TUI Dashboard MVP

> **Status**: 规划中 (Target: v0.5.0-beta)
> **Cycle**: 20260211_cycle9
> **Framework**: Ink (React for CLI) — NOT OpenTUI (proprietary)

## 概述

使用 **Ink** (React for CLI) 为 OpenStarry agents 实现全屏终端仪表板。MVP 聚焦于实时事件可视化、键盘驱动控制和流式消息显示。这将完成 CLI 接口层并实现无头多代理监控。

### 关键目标

1. **新插件**: `@openstarry-plugin/tui-dashboard`，实现 IUI (色蕴) + IListener (受蕴) 接口
2. **交互式仪表板**: 全屏终端 UI 替代 stdio readline
3. **实时事件**: 订阅代理事件总线，显示流式助手响应和工具调用
4. **键盘控制**: Ctrl+C (退出), /help (命令), Tab (事件日志切换)
5. **可扩展性**: 架构支持多代理仪表板 (延后至 Plan09)

---

## 问题背景

### 当前状态

| 限制 | 影响 |
|-----------|--------|
| 仅 stdio 输出 | 无视觉层级，事件可见性差 |
| 仅 readline 输入 | 单行提示符，无命令发现 |
| 无事件日志 | 难以调试代理执行流程 |
| 无状态指示器 | 不清楚代理是在处理中还是空闲 |

### 目标状态

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

## 交付清单

### Phase 1: 插件搭建 ✅ *设计阶段*
- [ ] 创建 `openstarry_plugin/tui-dashboard/` 包结构
- [ ] 配置 package.json，添加 Ink + React 依赖
- [ ] 冻结接口规格 (IUI + IListener)
- [ ] 记录事件订阅模式

### Phase 2: 核心布局组件 ✅ *实现阶段*
- [ ] Header: 代理名称 + 版本 + 状态指示器 (●○)
- [ ] 聊天区域: 带 ANSI 颜色的流式消息显示
- [ ] 输入区域: 文本输入字段 (rx-enabled)
- [ ] Footer: 键盘快捷键参考
- [ ] 事件日志侧边栏: 可切换、可滚动的事件历史

### Phase 3: 事件集成 ✅ *实现阶段*
- [ ] 通过 `ctx.onEvent()` 订阅 EventBus
- [ ] 将事件类型映射为视觉表示
- [ ] 处理带缓冲的流式文本
- [ ] 带颜色编码的工具调用和结果

### Phase 4: 输入处理 ✅ *实现阶段*
- [ ] 通过 Ink 的 useInput() hook 处理键盘输入
- [ ] 斜杠命令解析 (/help, /quit, /clear)
- [ ] 文本缓冲和光标控制
- [ ] 通过 Ctrl+C 或 /quit 优雅退出

### Phase 5: 测试与集成 ✅ *验证阶段*
- [ ] 单元测试 (Ink 组件渲染)
- [ ] 集成测试 (事件总线订阅)
- [ ] 手动冒烟测试（含 stdio 回退）
- [ ] 更新 runner 配置 (agent.json 示例)

### Phase 6: 文档 ✅ *收敛阶段*
- [ ] 插件 API 文档
- [ ] 配置指南 (插件配置 schema)
- [ ] 键盘快捷键参考
- [ ] 故障排除指南

---

## 依赖与前置条件

### 必需 (必须存在)

| 依赖 | 版本 | 状态 | 位置 |
|-----------|---------|--------|----------|
| EventBus | N/A | ✅ 已实现 (Plan02) | `packages/core/src/transport/` |
| IUI 接口 | N/A | ✅ 已实现 (Plan04) | `packages/sdk/src/types/ui.ts` |
| IListener 接口 | N/A | ✅ 已实现 (Plan04) | `packages/sdk/src/types/listener.ts` |
| Agent 生命周期 | N/A | ✅ 已实现 (Plan01) | `packages/core/src/agents/` |
| 插件工厂模式 | N/A | ✅ 已实现 (Plan03) | `packages/sdk/src/types/plugin.ts` |

### 外部库

| 库 | 版本 | 用途 | 许可证 |
|---------|---------|---------|---------|
| ink | ^5.0.1 | React for terminal | MIT |
| react | ^18.3.1 | 组件模型 | MIT |
| ink-text-input | ^6.0.0 | 文本输入组件 | MIT |
| ink-spinner | ^5.0.0 | 加载旋转器 | MIT |
| @types/react | ^18.2.0 | TypeScript 定义 | MIT |

### 安装

```bash
cd agent_dev/openstarry_plugin/tui-dashboard
pnpm add ink@^5.0.1 react@^18.3.1
pnpm add -D @types/react@^18.2.0 ink-text-input@^6.0.0 ink-spinner@^5.0.0
```

---

## 架构概要

### 插件结构

```
openstarry_plugin/tui-dashboard/
├── package.json                    ← Manifest + dependencies
├── tsconfig.json                   ← TypeScript config
└── src/
    ├── index.ts                    ← Factory export
    ├── components/
    │   ├── app.tsx                 ← Root component + provider tree
    │   ├── header.tsx              ← Agent name + status
    │   ├── chat-area.tsx           ← Message display
    │   ├── input-area.tsx          ← Text input field
    │   ├── footer.tsx              ← Keyboard shortcuts
    │   ├── event-log.tsx           ← Event list sidebar
    │   └── status-indicator.tsx    ← (Ready/Processing/Error)
    ├── contexts/
    │   ├── events.tsx              ← Event queue context
    │   └── input.tsx               ← Input buffer context
    ├── hooks/
    │   ├── use-event-bus.ts        ← EventBus subscription
    │   └── use-keyboard.ts         ← Keyboard input
    └── types.ts                    ← Local type definitions
```

### 组件层级

```
<TuiDashboard>                      ← IUI.onEvent() subscriber
  └─ <App>                          ← Root layout
      ├─ <Header/>                  ← Agent info + status
      ├─ <ChatArea/>                ← Message streaming
      ├─ <EventLog/>                ← Optional sidebar
      ├─ <InputArea/>               ← Text input
      └─ <Footer/>                  ← Keyboard help
```

### 事件流

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

### 输入流

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

## 阶段详细分解

### Phase 1: 插件搭建与接口冻结

**时长**: 1 天

#### 1.1 创建插件包

**路径**: `openstarry_plugin/tui-dashboard/package.json`

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

与其他插件一致的标准配置。

#### 1.3 类型定义

**路径**: `src/types.ts`

```typescript
/**
 * TUI Dashboard 插件的本地类型定义。
 * 将 AgentEvent 类型映射为视觉表示。
 */

export interface TuiConfig {
  // 可选: showEventLog? boolean;
  // 可选: logLevel? "debug" | "info" | "warn" | "error";
  // 延后至 Plan09: themes?, keyBindings?
}

export interface ChatMessage {
  role: "user" | "assistant";
  content: string;
  timestamp: number;
  streaming?: boolean;  // true 表示仍在接收 token
}

export interface EventLogEntry {
  id: string;
  type: string;
  timestamp: number;
  summary: string;  // 如 "tool:call weather_api", "stream:text_delta"
}
```

#### 1.4 接口规格

**冻结接口** (来自 Plan04):

```typescript
// 来自 @openstarry/sdk
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
  // onEvent 已移除 (Plan04)
}

// PluginHooks.ui 将包含 TuiDashboard IUI 实例
```

---

### Phase 2: 核心布局组件

**时长**: 3 天

#### 2.1 Header 组件

**路径**: `src/components/header.tsx`

```typescript
import React from "react";
import { Box, Text } from "ink";

interface HeaderProps {
  agentName: string;
  agentVersion: string;
  status: "ready" | "processing" | "error";
}

const statusIndicator: Record<string, string> = {
  ready: "🟢",      // 绿色圆圈
  processing: "🟡", // 黄色圆圈
  error: "🔴",      // 红色圆圈
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

#### 2.2 聊天区域组件

**路径**: `src/components/chat-area.tsx`

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

#### 2.3 输入区域组件

**路径**: `src/components/input-area.tsx`

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

#### 2.4 Footer 组件

**路径**: `src/components/footer.tsx`

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

#### 2.5 事件日志组件

**路径**: `src/components/event-log.tsx`

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

#### 2.6 App 根组件

**路径**: `src/components/app.tsx`

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

  // TODO: useEventBus hook will update messages and events
  // TODO: useKeyboard hook will handle Tab/Ctrl+H/Ctrl+C

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

### Phase 3: 事件总线集成

**时长**: 2 天

#### 3.1 Hook: useEventBus

**路径**: `src/hooks/use-event-bus.ts`

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
      // 添加到事件日志
      setEvents((prev) => [
        ...prev,
        {
          id: event.id,
          type: event.type,
          timestamp: event.timestamp,
          summary: summarizeEvent(event),
        },
      ]);

      // 更新消息显示
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
          // 将结果追加到最后一条工具消息
          break;
        case "agent:error":
          setStatus("error");
          break;
      }
    };

    // 订阅所有事件
    // 注意: 需要 ctx.bus 或类似的事件订阅机制
    // 将在 Phase 3 收尾时实现
    // ctx.onEvent(handleEvent);

    return () => {
      // 清理: 取消订阅
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

#### 3.2 Hook: useKeyboard

**路径**: `src/hooks/use-keyboard.ts`

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
        // Tab: 切换事件日志
        handlers.onToggleEvents();
      } else if ((key.ctrl && key.name === "h") || ch === "?") {
        // Ctrl+H 或 ?: 显示帮助
        handlers.onHelp();
      }
    };

    // 注意: 将使用 Ink 的 useInput hook 替代
    // 这是键盘处理模式的占位符

    return () => {
      // 清理 stdin 监听器
    };
  }, [handlers]);
}
```

---

### Phase 4: 输入处理与文本输入

**时长**: 1 天

#### 4.1 TuiListener 组件

**路径**: `src/components/tui-listener.tsx`

```typescript
import React from "react";
import type { IListener } from "@openstarry/sdk";
import type { IPluginContext } from "@openstarry/sdk";

/**
 * TuiListener 实现 IListener 接口 (受蕴)。
 * 接收来自 Ink 组件的用户输入并推送到核心。
 */
export function createTuiListener(ctx: IPluginContext): IListener {
  return {
    id: "tui-listener",
    name: "TUI Dashboard Listener",

    async start(): Promise<void> {
      // Listener 不直接绑定 stdin；输入来自 Ink 组件
      // 插件启动时调用
    },

    async stop(): Promise<void> {
      // 清理
    },
  };
}
```

#### 4.2 TuiUI 组件

**路径**: `src/components/tui-ui.tsx`

```typescript
import React from "react";
import type { IUI, AgentEvent, IPluginContext } from "@openstarry/sdk";
import { App } from "./app.js";

/**
 * TuiUI 实现 IUI 接口 (色蕴)。
 * 接收来自核心的事件并渲染到终端。
 */
export function createTuiUI(ctx: IPluginContext): IUI {
  return {
    id: "tui-dashboard",
    name: "TUI Dashboard UI",

    onEvent(event: AgentEvent): void {
      // 事件处理器将更新 React 状态
      // 通过 Events context 分发
      // 参见 useEventBus hook
    },

    async start(): Promise<void> {
      // 渲染 Ink 应用
    },

    async stop(): Promise<void> {
      // 清理
    },
  };
}
```

---

### Phase 5: 插件工厂与集成

**时长**: 1 天

#### 5.1 主入口导出

**路径**: `src/index.ts`

```typescript
/**
 * tui-dashboard — 全屏 TUI 仪表板插件。
 *
 * 提供:
 * - TuiUI (色蕴) — 在终端渲染代理事件
 * - TuiListener (受蕴) — 接收用户输入
 *
 * Config: {} (MVP 无需配置)
 */

import type { IPlugin, IPluginContext, PluginHooks } from "@openstarry/sdk";

export interface TuiDashboardConfig {
  // MVP: 无配置
  // 未来 (Plan09): theme, keyBindings, logLevel
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

      // 创建 UI 和 Listener 实例
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

// 内部辅助函数
function createTuiUI(ctx: IPluginContext) {
  // Phase 4.2 的实现
}

function createTuiListener(ctx: IPluginContext) {
  // Phase 4.1 的实现
}
```

#### 5.2 更新 Runner 配置

**路径**: `apps/runner/src/bin.ts` (defaultConfig)

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
      { name: "@openstarry-plugin/tui-dashboard" },  // 新增
    ],
    guide: "default-guide",
  };
}
```

---

### Phase 6: 测试与验证

**时长**: 2 天

#### 6.1 单元测试

**路径**: `src/components/header.test.ts`

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
    // 测试组件渲染 (Ink snapshot)
    // expect(...).toMatchSnapshot();
  });

  it("displays status indicator", () => {
    // 测试状态符号显示
  });
});
```

#### 6.2 集成测试

**路径**: `src/index.test.ts`

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

    // 模拟输入提交
    // listener.onInput("Hello");  // 如果存在输入回调

    // expect(pushInputSpy).toHaveBeenCalledWith(
    //   expect.objectContaining({ data: "Hello" })
    // );
  });
});
```

---

## 成功标准

### 功能需求

| 需求 | 验收标准 | 优先级 |
|-------------|-------------------|----------|
| 全屏终端 UI | 应用渲染不崩溃，覆盖终端 | ✅ 必须 |
| 实时事件显示 | 事件在触发后 <100ms 内出现 | ✅ 必须 |
| 用户输入处理 | 文本输入和斜杠命令正常工作 | ✅ 必须 |
| 优雅退出 | Ctrl+C 或 /quit 干净退出 | ✅ 必须 |
| 状态指示器 | 显示 ready/processing/error 状态 | 应该 |
| 事件日志侧边栏 | Tab 键切换可见性 | 应该 |
| 流式消息 | 助手响应逐步出现 | ✅ 必须 |
| 键盘快捷键 | 至少 5 个快捷键列在 Footer | 应该 |

### 非功能需求

| 需求 | 验收标准 | 目标 |
|-------------|-------------------|--------|
| 测试覆盖率 | >80% 覆盖率 | 85%+ |
| 性能 | <50ms 重新渲染时间 | <30ms |
| 终端兼容性 | 在 macOS + Linux + Windows 上工作 | 100% |
| 代码质量 | TypeScript strict，无错误 | 0 errors |
| 文档 | README + API 文档完整 | 100% |

### 构建与 QA 门禁

| 门禁 | 状态 | 备注 |
|------|--------|-------|
| `pnpm build` | 必须通过 | 全部 11 个包编译 |
| `pnpm test` | 必须通过 | >470 tests (Plan08 新增 +20) |
| `pnpm test:purity` | 必须通过 | Core 不导入 plugin |
| 手动冒烟测试 | 必须通过 | 使用 tui-dashboard 插件运行代理 |
| 代码审查 | 必须通过 | architect 审批 |

---

## 不在范围内 (延后至 Plan09)

- **多代理仪表板**: 尚无 daemon 或进程管理
- **`openstarry design`**: 交互式设计器 CLI
- **`openstarry attach`**: 无缝连接运行中的代理
- **高级主题**: 配色方案，自定义样式
- **命令面板**: 命令模糊搜索
- **语法高亮**: 聊天中的代码块渲染
- **复制到剪贴板**: 从终端选择和复制文本
- **鼠标支持**: 点击聚焦，拖拽选择
- **日志持久化**: 将聊天历史保存到磁盘

---

## 文件清单

### 新文件 (11 个)

| 路径 | 类型 | 行数 | 备注 |
|------|------|-------|-------|
| `openstarry_plugin/tui-dashboard/package.json` | Config | 30 | 依赖: ink, react |
| `openstarry_plugin/tui-dashboard/tsconfig.json` | Config | 20 | 标准 TS 配置 |
| `openstarry_plugin/tui-dashboard/src/index.ts` | Code | 80 | 插件工厂 |
| `openstarry_plugin/tui-dashboard/src/types.ts` | Code | 30 | 本地类型 |
| `openstarry_plugin/tui-dashboard/src/components/app.tsx` | Code | 60 | 根组件 |
| `openstarry_plugin/tui-dashboard/src/components/header.tsx` | Code | 25 | Header 栏 |
| `openstarry_plugin/tui-dashboard/src/components/chat-area.tsx` | Code | 45 | 聊天显示 |
| `openstarry_plugin/tui-dashboard/src/components/input-area.tsx` | Code | 35 | 输入字段 |
| `openstarry_plugin/tui-dashboard/src/components/footer.tsx` | Code | 15 | Footer 快捷键 |
| `openstarry_plugin/tui-dashboard/src/components/event-log.tsx` | Code | 40 | 事件侧边栏 |
| `openstarry_plugin/tui-dashboard/src/hooks/use-event-bus.ts` | Code | 80 | EventBus hook |

### 修改文件 (2 个)

| 路径 | 变更 | 备注 |
|------|--------|-------|
| `apps/runner/src/bin.ts` | 在 defaultConfig 中添加 tui-dashboard | 插件配置 |
| `openstarry/tsconfig.json` | 添加 tui-dashboard 引用 | Workspace 引用 |

### 测试文件 (3 个)

| 路径 | 测试数 | 备注 |
|------|-------|-------|
| `openstarry_plugin/tui-dashboard/src/index.test.ts` | 4 | 插件工厂测试 |
| `openstarry_plugin/tui-dashboard/src/components/header.test.ts` | 2 | 组件渲染 |
| `openstarry_plugin/tui-dashboard/src/hooks/use-event-bus.test.ts` | 6 | 事件 hook 集成 |

---

## 风险评估

| 风险 | 概率 | 影响 | 缓解措施 |
|------|-------------|--------|-----------|
| Ink API 不稳定 | 低 | 中 | 使用 v5 LTS，锁定版本，多 Node 版本测试 |
| EventBus 订阅延迟 | 低 | 低 | 批量事件 (16ms)，防抖更新 |
| 终端调整大小处理 | 中 | 低 | Ink 自动处理，在常见终端上测试 |
| 长会话内存泄漏 | 低 | 中 | 清理事件监听器，消息历史限制 1000 条 |
| React hooks 学习曲线 | 低 | 低 | Plan04 参考，Ink 文档优秀 |
| StdIO 插件冲突 | 中 | 高 | 启用 tui-dashboard 时禁用 stdio (配置互斥) |

---

## 集成清单

- [ ] 将 `@openstarry-plugin/tui-dashboard` 添加到 workspace `pnpm-workspace.yaml`
- [ ] 更新 runner package.json 以包含 tui-dashboard 依赖
- [ ] 更新根 tsconfig.json 以包含 tui-dashboard 引用
- [ ] 使用 `pnpm install && pnpm build` 测试
- [ ] 运行现有测试套件确保无回归
- [ ] 将 tui-dashboard 添加到 agent.json 默认配置
- [ ] 在 User_Scenario_and_Workflow_Guide.md 中记录插件

---

## 参考资料

### 预研
- `/data/openstarry_eco/share/test/reports/research/Plan08_TUI_PreResearch.md`
  - 框架分析: OpenTUI vs Ink vs Blessed
  - 建议: Ink (React for CLI)
  - 状态管理模式: 双层 (sync + local)

### 架构基础
- `/data/openstarry_eco/share/openstarry_doc/Architecture_Documentation/02_Headless_Agent_Core.md`
  - EventBus 广播
  - Transport bridge 模式

### 插件系统
- `/data/openstarry_eco/share/openstarry_doc/Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md`
  - IUI (色蕴) 接口
  - IListener (受蕴) 接口
  - 五蕴架构

### 外部参考
- **Ink 文档**: https://github.com/vadimdemedes/ink
- **React Hooks**: https://react.dev/reference/react
- **TypeScript Strict Mode**: https://www.typescriptlang.org/tsconfig#strict

---

## 后续步骤 (收敛后)

### 即时 (Plan08+1)
1. 架构师代码审查
2. QA 在多终端测试
3. 文档审查
4. 发布为 v0.5.0-beta

### 短期 (Plan09)
1. 添加多代理仪表板支持
2. 实现 `openstarry design` CLI
3. 添加主题自定义
4. 使用 React.memo() 提升性能

### 中期 (Plan10+)
1. 终端连接 (`openstarry attach`)
2. 无缝代理生成
3. 高级键盘快捷键 (命令面板)
4. LSP/MCP 状态集成

---

## 文档控制

| 字段 | 值 |
|-------|-------|
| Plan ID | Plan08 |
| Version | 1.0-DRAFT |
| Created | 2026-02-12 |
| Status | 规划中 |
| Cycle | 20260211_cycle9 |
| Owner | coordinator |
| Framework | Ink ^5.0.1 |
| Target Version | v0.5.0-beta |
