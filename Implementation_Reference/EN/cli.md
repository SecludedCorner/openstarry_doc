# CLI Commands

## Command Reference

```bash
node apps/runner/dist/bin.js --help
```

| Command | Description |
|---------|-------------|
| `start` | Start an agent (default) |
| `daemon start` | Start an agent in the background |
| `daemon stop` | Stop a background agent |
| `attach` | Attach to a running daemon |
| `ps` | List running agents |
| `init` | Interactively create a new configuration |
| `create-plugin` | Scaffold a new plugin |
| `plugin install` | Install a plugin |
| `plugin uninstall` | Uninstall a plugin |
| `plugin list` | List plugins |
| `plugin search` | Search for plugins |
| `plugin info` | Show plugin details |
| `plugin sync` | Sync plugins to system directory |
| `version` | Show version info |

## Slash Commands (Inside Agent)

Use these in the CLI after starting an agent:

| Command | Description |
|---------|-------------|
| `/help` | Show all available commands |
| `/reset` | Reset conversation history |
| `/quit` | Exit the agent |
| `/provider login gemini` | Log in to Gemini OAuth |
| `/provider logout gemini` | Log out |
| `/provider status` | Check login status |
| `/devtools` | Toggle developer panel |
| `/metrics` | Show performance metrics |
| `/workflow <file>` | Execute a workflow |
| `/mcp-server-status` | MCP server status |

## Usage Examples

### Basic Startup

```bash
# Use default configuration
node apps/runner/dist/bin.js

# Use a specific configuration
node apps/runner/dist/bin.js start --config ./configs/basic-agent.json
```

### Daemon Mode

```bash
# Start in background
node apps/runner/dist/bin.js daemon start --config ./configs/basic-agent.json

# List running agents
node apps/runner/dist/bin.js ps

# Attach to a background agent
node apps/runner/dist/bin.js attach

# Stop
node apps/runner/dist/bin.js daemon stop
```

### Debug Mode

```bash
LOG_LEVEL=debug node apps/runner/dist/bin.js start --config ./configs/basic-agent.json
```
