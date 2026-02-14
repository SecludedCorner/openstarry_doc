# Plan08: TUI Dashboard MVP

> **Status**: 📋 Planned (Target: v0.5.0-beta)
> **Cycle**: 20260211_cycle9
> **Framework**: Ink (React for CLI) — NOT OpenTUI (proprietary)

## Overview

Implement a full-screen terminal dashboard for OpenStarry agents using **Ink** (React for CLI). The MVP focuses on real-time event visualization, keyboard-driven controls, and streaming message display. This completes the CLI interface layer and enables headless multi-agent monitoring.

### Key Objectives

1. **New Plugin**: `@openstarry-plugin/tui-dashboard` with IUI (色蘊) + IListener (受蘊) interfaces
2. **Interactive Dashboard**: Full-screen terminal UI replacing stdio readline
3. **Real-Time Events**: Subscribe to agent event bus, display streaming assistant responses and tool calls
4. **Keyboard Controls**: Ctrl+C (quit), /help (commands), Tab (event log toggle)
5. **Extensibility**: Architecture supports multi-agent dashboard (deferred to Plan09)

---

## Problem Context

### Current State

| Limitation | Impact |
|-----------|--------|
| stdio output only | No visual hierarchy, poor event visibility |
| readline input only | Single-line prompt, no command discovery |
| No event logging | Hard to debug agent execution flow |
| No status indicator | Unclear if agent is processing or idle |

### Target State

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

## Deliverables Checklist

### Phase 1: Plugin Setup ✅ *Design Phase*
- [ ] Create `openstarry_plugin/tui-dashboard/` package structure
- [ ] Configure package.json with Ink + React dependencies
- [ ] Freeze interface specification (IUI + IListener)
- [ ] Document event subscription pattern

### Phase 2: Core Layout Components ✅ *Implementation Phase*
- [ ] Header: Agent name + version + status indicator (●○)
- [ ] Chat area: Streaming message display with ANSI colors
- [ ] Input area: Text input field (rx-enabled)
- [ ] Footer: Keyboard shortcuts reference
- [ ] Event log sidebar: Toggleable, scrollable event history

### Phase 3: Event Integration ✅ *Implementation Phase*
- [ ] Subscribe to EventBus via `ctx.onEvent()`
- [ ] Map event types to visual representations
- [ ] Handle streaming text with buffering
- [ ] Color-coded tool calls and results

### Phase 4: Input Handling ✅ *Implementation Phase*
- [ ] Keyboard input via Ink's useInput() hook
- [ ] Slash command parsing (/help, /quit, /clear)
- [ ] Text buffering and cursor control
- [ ] Graceful exit on Ctrl+C or /quit

### Phase 5: Testing & Integration ✅ *Verification Phase*
- [ ] Unit tests (Ink component rendering)
- [ ] Integration tests (event bus subscription)
- [ ] Manual smoke test with stdio fallback
- [ ] Update runner config (agent.json example)

### Phase 6: Documentation ✅ *Convergence Phase*
- [ ] Plugin API documentation
- [ ] Configuration guide (plugin config schema)
- [ ] Keyboard shortcuts reference
- [ ] Troubleshooting guide

---

## Dependencies & Prerequisites

### Required (Must Exist)

| Dependency | Version | Status | Location |
|-----------|---------|--------|----------|
| EventBus | N/A | ✅ Implemented (Plan02) | `packages/core/src/transport/` |
| IUI interface | N/A | ✅ Implemented (Plan04) | `packages/sdk/src/types/ui.ts` |
| IListener interface | N/A | ✅ Implemented (Plan04) | `packages/sdk/src/types/listener.ts` |
| Agent lifecycle | N/A | ✅ Implemented (Plan01) | `packages/core/src/agents/` |
| Plugin factory pattern | N/A | ✅ Implemented (Plan03) | `packages/sdk/src/types/plugin.ts` |

### External Libraries

| Library | Version | Purpose | License |
|---------|---------|---------|---------|
| ink | ^5.0.1 | React for terminal | MIT |
| react | ^18.3.1 | Component model | MIT |
| ink-text-input | ^6.0.0 | Text input component | MIT |
| ink-spinner | ^5.0.0 | Loading spinner | MIT |
| @types/react | ^18.2.0 | TypeScript definitions | MIT |

### Installation

```bash
cd agent_dev/openstarry_plugin/tui-dashboard
pnpm add ink@^5.0.1 react@^18.3.1
pnpm add -D @types/react@^18.2.0 ink-text-input@^6.0.0 ink-spinner@^5.0.0
```

---

## Architecture Summary

### Plugin Structure

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

### Component Hierarchy

```
<TuiDashboard>                      ← IUI.onEvent() subscriber
  └─ <App>                          ← Root layout
      ├─ <Header/>                  ← Agent info + status
      ├─ <ChatArea/>                ← Message streaming
      ├─ <EventLog/>                ← Optional sidebar
      ├─ <InputArea/>               ← Text input
      └─ <Footer/>                  ← Keyboard help
```

### Event Flow

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

### Input Flow

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

## Phase Breakdown

### Phase 1: Plugin Setup & Interface Freezing

**Duration**: 1 day

#### 1.1 Create Plugin Package

**Path**: `openstarry_plugin/tui-dashboard/package.json`

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

Standard configuration matching other plugins.

#### 1.3 Type Definitions

**Path**: `src/types.ts`

```typescript
/**
 * Local type definitions for TUI Dashboard plugin.
 * Maps AgentEvent types to visual representations.
 */

export interface TuiConfig {
  // Optional: showEventLog? boolean;
  // Optional: logLevel? "debug" | "info" | "warn" | "error";
  // Deferred to Plan09: themes?, keyBindings?
}

export interface ChatMessage {
  role: "user" | "assistant";
  content: string;
  timestamp: number;
  streaming?: boolean;  // true if still receiving tokens
}

export interface EventLogEntry {
  id: string;
  type: string;
  timestamp: number;
  summary: string;  // e.g., "tool:call weather_api", "stream:text_delta"
}
```

#### 1.4 Interface Specification

**Frozen Interfaces** (from Plan04):

```typescript
// From @openstarry/sdk
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
  // onEvent REMOVED (Plan04)
}

// PluginHooks.ui will contain TuiDashboard IUI instance
```

---

### Phase 2: Core Layout Components

**Duration**: 3 days

#### 2.1 Header Component

**Path**: `src/components/header.tsx`

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

#### 2.2 Chat Area Component

**Path**: `src/components/chat-area.tsx`

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

#### 2.3 Input Area Component

**Path**: `src/components/input-area.tsx`

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

#### 2.4 Footer Component

**Path**: `src/components/footer.tsx`

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

#### 2.5 Event Log Component

**Path**: `src/components/event-log.tsx`

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

#### 2.6 App Root Component

**Path**: `src/components/app.tsx`

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

### Phase 3: Event Bus Integration

**Duration**: 2 days

#### 3.1 Hook: useEventBus

**Path**: `src/hooks/use-event-bus.ts`

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
      // Add to event log
      setEvents((prev) => [
        ...prev,
        {
          id: event.id,
          type: event.type,
          timestamp: event.timestamp,
          summary: summarizeEvent(event),
        },
      ]);

      // Update message display
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
          // Append result to last tool message
          break;
        case "agent:error":
          setStatus("error");
          break;
      }
    };

    // Subscribe to all events
    // Note: Requires ctx.bus or similar event subscription mechanism
    // This will be implemented in Phase 3 finalization
    // ctx.onEvent(handleEvent);

    return () => {
      // Cleanup: unsubscribe
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

**Path**: `src/hooks/use-keyboard.ts`

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
        // Tab: toggle event log
        handlers.onToggleEvents();
      } else if ((key.ctrl && key.name === "h") || ch === "?") {
        // Ctrl+H or ?: show help
        handlers.onHelp();
      }
    };

    // Note: Ink's useInput hook will be used instead
    // This is a placeholder for keyboard handling pattern

    return () => {
      // Cleanup stdin listeners
    };
  }, [handlers]);
}
```

---

### Phase 4: Input Handling & Text Input

**Duration**: 1 day

#### 4.1 TuiListener Component

**Path**: `src/components/tui-listener.tsx`

```typescript
import React from "react";
import type { IListener } from "@openstarry/sdk";
import type { IPluginContext } from "@openstarry/sdk";

/**
 * TuiListener implements IListener interface (受蘊).
 * Receives user input from Ink components and pushes to core.
 */
export function createTuiListener(ctx: IPluginContext): IListener {
  return {
    id: "tui-listener",
    name: "TUI Dashboard Listener",

    async start(): Promise<void> {
      // Listeners don't bind stdin directly; input comes from Ink components
      // This is called when plugin starts
    },

    async stop(): Promise<void> {
      // Cleanup
    },
  };
}
```

#### 4.2 TuiUI Component

**Path**: `src/components/tui-ui.tsx`

```typescript
import React from "react";
import type { IUI, AgentEvent, IPluginContext } from "@openstarry/sdk";
import { App } from "./app.js";

/**
 * TuiUI implements IUI interface (色蘊).
 * Receives events from core and renders to terminal.
 */
export function createTuiUI(ctx: IPluginContext): IUI {
  return {
    id: "tui-dashboard",
    name: "TUI Dashboard UI",

    onEvent(event: AgentEvent): void {
      // Event handler will update React state
      // Dispatched via Events context
      // See useEventBus hook
    },

    async start(): Promise<void> {
      // Render Ink app
    },

    async stop(): Promise<void> {
      // Cleanup
    },
  };
}
```

---

### Phase 5: Plugin Factory & Integration

**Duration**: 1 day

#### 5.1 Main Index Export

**Path**: `src/index.ts`

```typescript
/**
 * tui-dashboard — Full-screen TUI dashboard plugin.
 *
 * Provides:
 * - TuiUI (色蘊) — renders agent events in terminal
 * - TuiListener (受蘊) — receives user input
 *
 * Config: {} (no config needed for MVP)
 */

import type { IPlugin, IPluginContext, PluginHooks } from "@openstarry/sdk";

export interface TuiDashboardConfig {
  // MVP: no configuration
  // Future (Plan09): theme, keyBindings, logLevel
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

      // Create UI and Listener instances
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

// Internal helpers
function createTuiUI(ctx: IPluginContext) {
  // Implementation from Phase 4.2
}

function createTuiListener(ctx: IPluginContext) {
  // Implementation from Phase 4.1
}
```

#### 5.2 Update Runner Configuration

**Path**: `apps/runner/src/bin.ts` (defaultConfig)

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

### Phase 6: Testing & Verification

**Duration**: 2 days

#### 6.1 Unit Tests

**Path**: `src/components/header.test.ts`

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
    // Test component rendering (Ink snapshot)
    // expect(...).toMatchSnapshot();
  });

  it("displays status indicator", () => {
    // Test status symbol display
  });
});
```

#### 6.2 Integration Tests

**Path**: `src/index.test.ts`

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

    // Simulate input submission
    // listener.onInput("Hello");  // if input callback exists

    // expect(pushInputSpy).toHaveBeenCalledWith(
    //   expect.objectContaining({ data: "Hello" })
    // );
  });
});
```

---

## Success Criteria

### Functional Requirements

| Requirement | Acceptance Criteria | Priority |
|-------------|-------------------|----------|
| Full-screen terminal UI | App renders without crashing, covers terminal | ✅ MUST |
| Real-time event display | Events appear <100ms after emission | ✅ MUST |
| User input handling | Text input and slash commands work | ✅ MUST |
| Graceful exit | Ctrl+C or /quit exits cleanly | ✅ MUST |
| Status indicator | Shows ready/processing/error states | 🟡 SHOULD |
| Event log sidebar | Tab key toggles visibility | 🟡 SHOULD |
| Streaming messages | Assistant responses appear incrementally | ✅ MUST |
| Keyboard shortcuts | At least 5 shortcuts listed in footer | 🟡 SHOULD |

### Non-Functional Requirements

| Requirement | Acceptance Criteria | Target |
|-------------|-------------------|--------|
| Test coverage | >80% coverage | 85%+ |
| Performance | <50ms re-render time | <30ms |
| Terminal compatibility | Works on macOS + Linux + Windows | 100% |
| Code quality | TypeScript strict, no errors | 0 errors |
| Documentation | README + API docs complete | 100% |

### Build & QA Gates

| Gate | Status | Notes |
|------|--------|-------|
| `pnpm build` | Must pass | All 11 packages compile |
| `pnpm test` | Must pass | >470 tests (new +20 from Plan08) |
| `pnpm test:purity` | Must pass | Core doesn't import plugin |
| Manual smoke test | Must pass | Run agent with tui-dashboard plugin |
| Code review | Must pass | architect approves |

---

## NOT In Scope (Deferred to Plan09)

- **Multi-Agent Dashboard**: No daemon or process management yet
- **`openstarry design`**: Interactive designer CLI
- **`openstarry attach`**: Seamless connection to running agents
- **Advanced Themes**: Color schemes, custom styling
- **Command Palette**: Fuzzy search for commands
- **Syntax Highlighting**: Code block rendering in chat
- **Copy-to-Clipboard**: Selecting and copying text from terminal
- **Mouse Support**: Click-to-focus, drag-to-select
- **Log Persistence**: Saving chat history to disk

---

## File Checklist

### New Files (11 total)

| Path | Type | Lines | Notes |
|------|------|-------|-------|
| `openstarry_plugin/tui-dashboard/package.json` | Config | 30 | Dependencies: ink, react |
| `openstarry_plugin/tui-dashboard/tsconfig.json` | Config | 20 | Standard TS config |
| `openstarry_plugin/tui-dashboard/src/index.ts` | Code | 80 | Plugin factory |
| `openstarry_plugin/tui-dashboard/src/types.ts` | Code | 30 | Local types |
| `openstarry_plugin/tui-dashboard/src/components/app.tsx` | Code | 60 | Root component |
| `openstarry_plugin/tui-dashboard/src/components/header.tsx` | Code | 25 | Header bar |
| `openstarry_plugin/tui-dashboard/src/components/chat-area.tsx` | Code | 45 | Chat display |
| `openstarry_plugin/tui-dashboard/src/components/input-area.tsx` | Code | 35 | Input field |
| `openstarry_plugin/tui-dashboard/src/components/footer.tsx` | Code | 15 | Footer shortcuts |
| `openstarry_plugin/tui-dashboard/src/components/event-log.tsx` | Code | 40 | Event sidebar |
| `openstarry_plugin/tui-dashboard/src/hooks/use-event-bus.ts` | Code | 80 | EventBus hook |

### Modified Files (2 total)

| Path | Change | Notes |
|------|--------|-------|
| `apps/runner/src/bin.ts` | Add tui-dashboard to defaultConfig | Plugin config |
| `openstarry/tsconfig.json` | Add tui-dashboard reference | Workspace reference |

### Test Files (3 total)

| Path | Tests | Notes |
|------|-------|-------|
| `openstarry_plugin/tui-dashboard/src/index.test.ts` | 4 | Plugin factory tests |
| `openstarry_plugin/tui-dashboard/src/components/header.test.ts` | 2 | Component rendering |
| `openstarry_plugin/tui-dashboard/src/hooks/use-event-bus.test.ts` | 6 | Event hook integration |

---

## Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|-----------|
| Ink API instability | Low | Medium | Use v5 LTS, pin version, test on multiple Node versions |
| EventBus subscription latency | Low | Low | Batch events (16ms), debounce updates |
| Terminal resize handling | Medium | Low | Ink handles automatically, test with common terminals |
| Memory leaks on long sessions | Low | Medium | Cleanup event listeners, limit message history to 1000 |
| React hooks learning curve | Low | Low | Plan04 reference, Ink docs are excellent |
| StdIO plugin conflict | Medium | High | Disable stdio when tui-dashboard enabled (config mutually exclusive) |

---

## Integration Checklist

- [ ] Add `@openstarry-plugin/tui-dashboard` to workspace `pnpm-workspace.yaml`
- [ ] Update runner package.json to include tui-dashboard dependency
- [ ] Update root tsconfig.json to include tui-dashboard reference
- [ ] Test with `pnpm install && pnpm build`
- [ ] Run existing test suite to ensure no regressions
- [ ] Add tui-dashboard to agent.json default config
- [ ] Document plugin in User_Scenario_and_Workflow_Guide.md

---

## References

### Pre-Research
- `/data/openstarry_eco/share/test/reports/research/Plan08_TUI_PreResearch.md`
  - Framework analysis: OpenTUI vs Ink vs Blessed
  - Recommendation: Ink (React for CLI)
  - State management pattern: Two-tier (sync + local)

### Architecture Foundation
- `/data/openstarry_eco/share/openstarry_doc/Architecture_Documentation/02_Headless_Agent_Core.md`
  - EventBus broadcasting
  - Transport bridge pattern

### Plugin System
- `/data/openstarry_eco/share/openstarry_doc/Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md`
  - IUI (色蘊) interface
  - IListener (受蘊) interface
  - Five Aggregates architecture

### External References
- **Ink Documentation**: https://github.com/vadimdemedes/ink
- **React Hooks**: https://react.dev/reference/react
- **TypeScript Strict Mode**: https://www.typescriptlang.org/tsconfig#strict

---

## Next Steps (Post-Convergence)

### Immediate (Plan08+1)
1. Code review by architect
2. QA testing on multiple terminals
3. Documentation review
4. Release as v0.5.0-beta

### Short-term (Plan09)
1. Add multi-agent dashboard support
2. Implement `openstarry design` CLI
3. Add theme customization
4. Improve performance with React.memo()

### Medium-term (Plan10+)
1. Terminal attachment (`openstarry attach`)
2. Seamless agent spawning
3. Advanced keyboard shortcuts (command palette)
4. LSP/MCP status integration

---

## Document Control

| Field | Value |
|-------|-------|
| Plan ID | Plan08 |
| Version | 1.0-DRAFT |
| Created | 2026-02-12 |
| Status | 📋 Ready for Planning Phase |
| Cycle | 20260211_cycle9 |
| Owner | coordinator |
| Framework | Ink ^5.0.1 |
| Target Version | v0.5.0-beta |
