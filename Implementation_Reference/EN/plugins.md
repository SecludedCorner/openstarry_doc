# Plugin Overview

> ⚠️ **[漂移更正 — v0.59.6；v0.59.8 +agent-comm +comm-channel-p2p] 套件數：49 package directories = 48 loadable plugins + 1 shared library (`mcp-common`)。** 原文「43 plugin directories as of v0.58.0-alpha」已陳舊。實測依據：`openstarry_plugin/` 下 `package.json` 共 49 個（`find openstarry_plugin -maxdepth 2 -name package.json -not -path "*/node_modules/*"` = 47；pnpm workspace glob `../openstarry_plugin/*` 見 `openstarry/pnpm-workspace.yaml:4` 全數納管）。其中 `mcp-common`（`@openstarry-plugin/mcp-common` v0.4.0，`mcp-common/package.json:1-22`）無 `test` script、不依賴 `@openstarry/sdk`、僅匯出協定 types/constants ＝ 共享函式庫，非可載入 plugin（對照可載入者如 `mcp-server/package.json:20-23` 依賴 `@openstarry/sdk` 且有 `test`）。下方表格為策展精選（非窮舉），其數字與標籤仍正確（`mcp-common` 已標 shared library）。

OpenStarry's plugin workspace contains **49 package directories** as of v0.59.x-alpha — **48 loadable plugins + 1 shared library (`mcp-common`)** (count updated 2026-06-27 for `agent-comm` + `comm-channel-p2p`; the tables below are a curated selection, not exhaustive — see `openstarry_plugin/` for the full set).

> ⚠️ **[honest note — v0.59.7 (2026-06-17)] "loadable" ≠ "provides agent capability once mounted".** All 46 load (factory runs, returns a valid `PluginHooks`), but **3 of them — `api-runtime` (Plan59), `mesh` (Plan58), `vasana-engine` (Plan57 Track1) — have a plugin factory that only does boot-time validation and returns `{dispose}`, registering NO agent-facing hook/tool/service** (the loader emits a sigma-4/5/14 "Declares skandha but no hook registered" warning at load; visible in the cold-start smoke). Their real value is the **imported library** (consumed by other plugins/specs; each has full `__tests__/` coverage), not a mounted capability. I.e. these 3 are "a library wearing a plugin wrapper" — bounded by design (see each src comment + the MEMORY exclusion list).

> The plugin repository is at `openstarry_plugin/`, a sibling directory to the core framework, managed together via `pnpm-workspace.yaml`.

## Standard Functions

| Plugin | Aggregate | Description |
|--------|-----------|-------------|
| `standard-function-fs` | Formation (ITool) | File read/write and directory operations |
| `standard-function-stdio` | Form+Sensation (IUI+IListener) | CLI text input/output |
| `standard-function-skill` | Formation+Consciousness (ITool+IGuide) | Markdown skill file loading |
| `guide-character-init` | Consciousness (IGuide) | Default system prompt / character |

## LLM Provider

| Plugin | Aggregate | Description |
|--------|-----------|-------------|
| `provider-gemini-oauth` | Perception (IProvider) | Google Gemini + PKCE OAuth |

## Transport Layer

| Plugin | Aggregate | Description |
|--------|-----------|-------------|
| `transport-websocket` | Form+Sensation (IUI+IListener) | WebSocket bidirectional communication (with token auth, CORS) |
| `transport-http` | Form+Sensation (IUI+IListener) | HTTP REST + SSE streaming |

## User Interface

| Plugin | Aggregate | Description |
|--------|-----------|-------------|
| `web-ui` | Form (IUI) | Browser chat interface (pairs with transport-websocket) |
| `tui-dashboard` | Form (IUI) | Terminal fullscreen monitoring dashboard |
| `http-static` | Form (IUI) | General-purpose static file server |

## Developer Tools

| Plugin | Aggregate | Description |
|--------|-----------|-------------|
| `devtools` | Form+Sensation (IUI+IListener) | Runtime metrics, event logs |
| `workflow-engine` | Formation (ITool) | YAML declarative workflow engine |

## MCP Protocol

| Plugin | Aggregate | Description |
|--------|-----------|-------------|
| `mcp-server` | Sensation (IListener) | Expose Agent tools to MCP clients |
| `mcp-client` | Perception (IProvider) | Connect to external MCP servers |
| `mcp-common` | (shared library) | MCP protocol types and constants |

## Naming Convention

All official plugins use the npm scope `@openstarry-plugin/`, for example:

```
@openstarry-plugin/standard-function-fs
@openstarry-plugin/transport-websocket
@openstarry-plugin/web-ui
```
