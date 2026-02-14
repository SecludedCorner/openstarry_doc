# OpenStarry Testing and Release Guide

**Version**: v0.20.1-beta (Cycle 26 — Attach UX + Token Persistence Fix)
**Date**: 2026-02-14
**Status**: Release built, awaiting manual testing

---

## Project Structure and Clone Instructions

OpenStarry is split into two GitHub repos that **must be placed under the same parent folder**:

```
your-workspace/              <- Any parent folder
├── openstarry/              <- Repo 1: Core monorepo (SDK, Core, Shared, Runner)
│   ├── packages/
│   │   ├── sdk/             <- @openstarry/sdk
│   │   ├── core/            <- @openstarry/core
│   │   ├── shared/          <- @openstarry/shared
│   │   └── plugin-signer/   <- @openstarry/plugin-signer
│   ├── apps/
│   │   └── runner/          <- @openstarry/runner (CLI entry point)
│   ├── configs/             <- Example agent configuration files
│   ├── pnpm-workspace.yaml  <- Key: includes ../openstarry_plugin/*
│   └── package.json
└── openstarry_plugin/       <- Repo 2: Plugin ecosystem (15 official plugins)
    ├── standard-function-fs/
    ├── standard-function-stdio/
    ├── transport-websocket/
    ├── web-ui/
    ├── workflow-engine/
    ├── ... (other plugins)
    └── package.json
```

### Clone Steps

```bash
# Create workspace directory
mkdir openstarry-workspace && cd openstarry-workspace

# Clone both repos to the same level
git clone <openstarry-repo-url> openstarry
git clone <openstarry-plugin-repo-url> openstarry_plugin
```

### Why Must They Be at the Same Level?

`openstarry/pnpm-workspace.yaml` is configured as follows:

```yaml
packages:
  - apps/*
  - packages/*
  - ../openstarry_plugin/*     <- References the plugin repo via relative path
```

All plugin `package.json` files also reference core packages via `workspace:*` or `link:../../openstarry/packages/sdk`. If the two repos are not at the same level, pnpm will be unable to resolve cross-repo dependencies.

---

## Prerequisites

- **Node.js** >= 20
- **pnpm** >= 10 (`npm install -g pnpm`)
- Both repos cloned under the same parent folder (as described above)
- For testing Agent conversation features, Gemini OAuth login is required (`/provider login gemini`)

---

## Step 1: Clean (If Re-testing)

If you have previously built the project, perform a full cleanup first:

```bash
cd openstarry        # Enter the core monorepo
pnpm clean:all
```

This clears all `node_modules/`, `dist/`, `tsconfig.tsbuildinfo`, `pnpm-lock.yaml`, as well as `node_modules/` and `dist/` under `../openstarry_plugin/`.

> `clean:all` uses a cross-platform Node script (`scripts/clean-all.mjs`) that works correctly on both Windows and Linux.
> If this is a fresh clone, you can skip this step.

---

## Step 2: Install Dependencies

```bash
cd openstarry        # Enter the core monorepo (if not already there)
pnpm install
```

Expected result: No errors; all 21 workspace packages (including plugins under openstarry_plugin) are installed successfully.

> **Note**: This must be executed from the `openstarry/` directory. pnpm will automatically resolve plugins under `../openstarry_plugin/*` via the workspace configuration.

---

## Step 3: Build

```bash
pnpm build
```

Expected result: All 20 packages build successfully.

---

## Step 4: Run Automated Tests

```bash
pnpm test
```

Expected results:
- **Test Files**: 117 passed (117)
- **Tests**: 1339 passed | 3 skipped (1342)
- Known skips: 1 daemon socket timeout test in attach.test.ts + 2 environment-specific (not bugs)

---

## Step 5: CLI Command Tests

All commands below are executed from the `openstarry/` root directory:

### 5.1 Version Information

```bash
node apps/runner/dist/bin.js version
```

Expected: Displays the version number.

### 5.2 Help Message

```bash
node apps/runner/dist/bin.js --help
```

Expected: Displays the list of all available commands, including:
- `start`, `daemon start`, `daemon stop`, `attach`, `ps`
- `init`, `create-plugin`
- `plugin install`, `plugin uninstall`, `plugin list`, `plugin search`, `plugin info`, `plugin sync`
- `version`

### 5.3 Initialize Configuration

```bash
node apps/runner/dist/bin.js init
```

Expected: Generates an `agent.json` configuration file in the current directory (prompts if it already exists).

---

## Step 6: Plugin Marketplace Command Tests

### 6.1 Search Plugins

```bash
node apps/runner/dist/bin.js plugin search fs
```

Expected: Displays a list of plugins containing "fs" (e.g., `standard-function-fs`).

```bash
node apps/runner/dist/bin.js plugin search websocket
```

Expected: Displays `transport-websocket` related plugins.

### 6.2 View Plugin Info

```bash
node apps/runner/dist/bin.js plugin info standard-function-fs
```

Expected: Displays the plugin's name, version, description, aggregates, and tags.

```bash
node apps/runner/dist/bin.js plugin info @openstarry-plugin/web-ui
```

Expected: Same as above; full names can also be resolved.

### 6.3 List All Plugins (Catalog)

```bash
node apps/runner/dist/bin.js plugin list --all
```

Expected: Lists all 15 official plugins, marked as `[installed]` or `[available]`.

### 6.4 List Installed Plugins

```bash
node apps/runner/dist/bin.js plugin list
```

Expected: If no plugins have ever been installed, displays "No plugins installed".

### 6.5 Install a Single Plugin

```bash
node apps/runner/dist/bin.js plugin install standard-function-fs
```

Expected: Resolves from workspace and installs, updating the lock file (`~/.openstarry/plugins/lock.json`).

### 6.6 Confirm Installation Success

```bash
node apps/runner/dist/bin.js plugin list
```

Expected: Displays the newly installed `@openstarry-plugin/standard-function-fs`.

### 6.7 Install All Plugins

```bash
node apps/runner/dist/bin.js plugin install --all
```

Expected: Installs all 15 official plugins, skipping already installed ones.

### 6.8 List Again

```bash
node apps/runner/dist/bin.js plugin list
```

Expected: All 15 plugins shown as installed.

### 6.9 Uninstall a Plugin

```bash
node apps/runner/dist/bin.js plugin uninstall standard-function-fs
```

Expected: Removes the plugin and updates the lock file.

### 6.10 Force Reinstall

```bash
node apps/runner/dist/bin.js plugin install standard-function-fs --force
```

Expected: Reinstalls even if already installed.

---

## Step 7: Agent Startup Tests

### 7.1 Basic Agent (CLI Mode)

```bash
node apps/runner/dist/bin.js start --config configs/basic-agent.json
```

Expected: Agent starts, CLI waits for user input. Enter any text to test conversation, `Ctrl+C` to exit.

> **Note**: On first startup, `[gemini-oauth] Not logged in` will be displayed. After logging in with `/provider login gemini`, the token is persisted and subsequent logins are not required.

### 7.2 TUI Agent (Terminal Interface Mode)

```bash
node apps/runner/dist/bin.js start --config configs/tui-agent.json
```

Expected: Ink-based TUI interface launches, displaying the dashboard. `Ctrl+C` to exit.

### 7.3 WebSocket Agent (Headless Mode)

```bash
node apps/runner/dist/bin.js start --config configs/websocket-agent.json
```

Expected: Agent starts and listens on `ws://0.0.0.0:8080/ws`. Can be tested with a WebSocket client.

### 7.4 Web Agent (Browser Interface)

```bash
node apps/runner/dist/bin.js start --config configs/web-agent.json
```

Expected:
- WebSocket server on `ws://0.0.0.0:8080/ws`
- Web UI on `http://0.0.0.0:8081`
- Opening `http://localhost:8081` in a browser shows the chat interface

---

## Step 8: Daemon Mode Tests

### 8.1 Start Daemon

```bash
node apps/runner/dist/bin.js daemon start --config configs/basic-agent.json
```

Expected: Agent starts as a background daemon.

### 8.2 View Running Agents

```bash
node apps/runner/dist/bin.js ps
```

Expected: Lists `basic-agent` (or the agent ID from the configuration).

### 8.3 Attach to Daemon

```bash
node apps/runner/dist/bin.js attach basic-agent
```

Expected:
- Connects to the background agent's IPC channel
- Displays a welcome message (with `/help` hint)
- **Automatically displays provider login status** (e.g., "Gemini OAuth: Logged in" or "Not logged in")

### 8.4 Stop Daemon

```bash
node apps/runner/dist/bin.js daemon stop basic-agent
```

Expected: Background agent stops.

---

## Step 9: Plugin Scaffolding Test

```bash
cd /tmp
node <path-to-openstarry>/apps/runner/dist/bin.js create-plugin my-test-plugin
```

Expected: Generates a `my-test-plugin/` directory in `/tmp`, containing:
- `package.json`
- `tsconfig.json`
- `src/index.ts` (with factory template)

---

## Step 10: Plugin Sync Test

```bash
node apps/runner/dist/bin.js plugin sync --source ../openstarry_plugin
```

Expected: Syncs plugins from the source directory to the system plugin directory (`~/.openstarry/plugins/system/`).

---

## Step 11: Workflow Engine Test (Cycle 23 Feature)

```bash
# Create a test workflow YAML
cat > /tmp/test-workflow.yaml << 'EOF'
name: hello-workflow
version: "1.0"
steps:
  - id: greet
    type: tool
    tool: fs.list
    input:
      path: "."
EOF
```

Using workflow commands requires the agent to be started with the workflow-engine plugin loaded.

---

## Complete Plugin List (15 Plugins)

| # | Plugin Name | Five Aggregates Category | Description |
|---|------------|------------------------|-------------|
| 1 | standard-function-fs | ITool | File read/write operations |
| 2 | standard-function-stdio | ITool | Standard input/output |
| 3 | standard-function-skill | ITool | Skill execution |
| 4 | guide-character-init | IGuide | Character initialization guide |
| 5 | provider-gemini-oauth | IProvider | Gemini OAuth provider |
| 6 | tui-dashboard | IUI | Terminal interface (Ink) |
| 7 | transport-websocket | IListener | WebSocket transport |
| 8 | transport-http | IListener | HTTP transport |
| 9 | http-static | IListener | Static file serving |
| 10 | web-ui | IUI | Browser chat interface |
| 11 | mcp-client | IListener/ITool | MCP client |
| 12 | mcp-server | IListener | MCP server |
| 13 | mcp-common | (shared) | MCP shared types |
| 14 | devtools | ITool | Developer tools |
| 15 | workflow-engine | ITool | Workflow engine |

---

## Configuration File Reference (configs/)

| Config File | Description | Plugins |
|-------------|-------------|---------|
| `basic-agent.json` | Minimal CLI agent | fs + stdio + guide + gemini |
| `example-agent.json` | Example agent | (same as basic) |
| `full-agent.json` | Full-featured agent | All plugins |
| `mcp-agent.json` | MCP integration agent | mcp-client + mcp-server |
| `tui-agent.json` | Terminal interface agent | tui-dashboard |
| `web-agent.json` | Browser interface agent | transport-websocket + web-ui |
| `websocket-agent.json` | WebSocket headless agent | transport-websocket |

---

## Test Results Record

| Step | Item | Result | Notes |
|------|------|--------|-------|
| 2 | pnpm install | | |
| 3 | pnpm build | | |
| 4 | pnpm test (1339) | | |
| 5.1 | version | | |
| 5.2 | --help | | |
| 5.3 | init | | |
| 6.1 | plugin search | | |
| 6.2 | plugin info | | |
| 6.3 | plugin list --all | | |
| 6.5 | plugin install (single) | | |
| 6.7 | plugin install --all | | |
| 6.9 | plugin uninstall | | |
| 7.1 | basic-agent start | | |
| 7.2 | tui-agent start | | |
| 7.3 | websocket-agent start | | |
| 7.4 | web-agent start | | |
| 8.1 | daemon start | | |
| 8.3 | attach (auto provider status) | | |
| 8.4 | daemon stop | | |
| 9 | create-plugin | | |
| 10 | plugin sync | | |
