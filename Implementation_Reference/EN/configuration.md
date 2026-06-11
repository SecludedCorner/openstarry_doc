# Configuration Format

## agent.json Structure

```json
{
  "identity": {
    "id": "my-agent",
    "name": "My Agent",
    "description": "Agent description",
    "version": "0.1.0"
  },
  "cognition": {
    "provider": "gemini-oauth",
    "model": "gemini-2.0-flash",
    "temperature": 0.7,
    "maxTokens": 8192,
    "maxToolRounds": 10
  },
  "capabilities": {
    "tools": ["fs.read", "fs.write", "fs.list"],
    "allowedPaths": ["."]
  },
  "policy": {
    "maxConcurrentTools": 1,
    "toolTimeout": 30000
  },
  "memory": {
    "slidingWindowSize": 5
  },
  "plugins": [
    { "name": "@openstarry-plugin/provider-gemini-oauth" },
    { "name": "@openstarry-plugin/standard-function-fs" },
    { "name": "@openstarry-plugin/standard-function-stdio" },
    {
      "name": "@openstarry-plugin/transport-websocket",
      "config": { "port": 8080, "path": "/ws" }
    }
  ],
  "guide": "default-guide"
}
```

### Section Reference

| Section | Description |
|---------|-------------|
| `identity` | Agent identity (ID, name, description, version) |
| `cognition` | LLM settings (provider, model, temperature, token limit) |
| `capabilities` | Allowed tools list and accessible paths |
| `policy` | Execution policy (concurrent tool limit, timeout) |
| `memory` | Memory settings (sliding window size) |
| `plugins` | Plugins to load, with optional `config` |
| `guide` | Guide name to use |

## Plugin Resolution Order

1. File path specified by `path` (dynamic `import()`)
2. System directory search: `~/.openstarry/plugins/installed/`
3. Full package name (resolved from workspace / node_modules)

## Preset Configurations

Preset configurations are in the `configs/` directory:

| Config | Description | Use Case |
|--------|-------------|----------|
| `basic-agent.json` | Minimal CLI agent | Basic chat + file operations |
| `web-agent.json` | Browser Web UI | Open `http://localhost:8081` |
| `websocket-agent.json` | WebSocket only (no CLI) | Programmatic API access |
| `tui-agent.json` | Terminal fullscreen dashboard | Visual monitoring |
| `mcp-agent.json` | MCP protocol agent | Integration with Claude Code and other MCP clients |
| `full-agent.json` | All features | Development and demos |

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `LOG_LEVEL` | Log level: `debug`, `info`, `warn`, `error` | `info` |
| `LOG_FORMAT` | Log format: `text`, `json` | `text` |
| `OPENSTARRY_WS_TOKEN` | WebSocket auth token (alternative to config) | (none) |

```bash
# Debug mode
LOG_LEVEL=debug node apps/runner/dist/bin.js --config ./configs/basic-agent.json

# JSON log output
LOG_FORMAT=json node apps/runner/dist/bin.js
```
