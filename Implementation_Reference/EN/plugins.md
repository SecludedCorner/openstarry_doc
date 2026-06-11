# Plugin Overview

OpenStarry's plugin workspace contains **43 plugin directories** as of v0.58.0-alpha (count corrected 2026-06-11; the tables below are a curated selection, not exhaustive — see `openstarry_plugin/` for the full set).

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
