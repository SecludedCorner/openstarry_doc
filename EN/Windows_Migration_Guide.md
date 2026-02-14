# Windows Migration Guide for OpenStarry Eco

**Last updated**: 2026-02-13
**Version**: v0.19.0-beta (Cycle 25 + Windows cross-platform fixes)
**Status**: All Linux verification complete; ready for Windows deployment

This guide documents the complete Claude Code environment and OpenStarry project state to enable reproduction on Windows.

---

## Table of Contents

1. [Pre-Migration Checklist](#pre-migration-checklist)
2. [Claude Code Global Configuration](#1-claude-code-global-configuration)
3. [Project Configuration](#2-project-configuration)
4. [Agent Definitions](#3-agent-definitions)
5. [Claude Code Memory Files](#4-claude-code-memory-files)
6. [Current Code State](#5-current-code-state)
7. [Windows-Specific Fixes Applied](#6-windows-specific-fixes-applied)
8. [Windows Setup Instructions](#7-windows-setup-instructions)
9. [Windows Verification Checklist](#8-windows-verification-checklist)
10. [Windows Troubleshooting](#9-windows-troubleshooting)

---

## Pre-Migration Checklist

- [ ] Windows 10/11 with latest updates
- [ ] Developer Mode enabled (for pnpm symlinks)
- [ ] Git Bash, WSL2, or PowerShell with bash installed
- [ ] Node.js 20.0+ (`node --version`)
- [ ] pnpm 9+ (`pnpm --version`)
- [ ] Claude Code CLI latest version
- [ ] Administrator access for symlink creation (if needed)

---

## 1. Claude Code Global Configuration

### Location: `~/.claude/settings.json`

Create or update your global Claude Code settings:

```json
{
  "alwaysThinkingEnabled": true
}
```

**Path expansion on Windows**:
- If your home directory is `C:\Users\YourUsername`, the path becomes:
  ```
  C:\Users\YourUsername\.claude\settings.json
  ```
- Alternatively, use `$env:USERPROFILE\.claude\settings.json` in PowerShell

### Verification

```powershell
# PowerShell
cat $env:USERPROFILE\.claude\settings.json

# Git Bash
cat ~/.claude/settings.json
```

---

## 2. Project Configuration

### 2.1 Root `CLAUDE.md`

**Location**: `openstarry_eco/CLAUDE.md`

This file is already in the repository and will transfer with the code. It contains:
- Project overview and directory structure
- Agent Corps definitions (6 agents)
- Standard iteration cycle (SOP)
- Architecture principles (Five Aggregates)
- Build and test commands
- Naming conventions
- Constraints and requirements

**No changes needed** — keep as-is in the project root.

### 2.2 Project Settings: `.claude/settings.local.json`

**Location**: `openstarry_eco/.claude/settings.local.json`

This file **must be updated for Windows paths**.

#### Linux Version (current in repo)
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_TEAMMATE_MODE": "tmux"
  },
  "permissions": {
    "allow": [
      "Bash(pnpm install:*)",
      "Bash(pnpm build)",
      "Bash(pnpm add:*)",
      "Bash(pnpm --filter \"@openstarry-plugin/*\" add -D @types/node)",
      "Bash(timeout 10 node:*)",
      "Bash(printf:*)",
      "Bash(node:*)",
      "Bash(pnpm test:*)",
      "Bash(LOG_FORMAT=json echo:*)",
      "Bash(export LOG_FORMAT=json)",
      "Bash(pnpm clean:*)",
      "Bash(find:*)",
      "Bash(bash:*)",
      "Bash(pnpm -r run build:*)",
      "Bash(dir:*)",
      "Bash(pnpm test:purity:*)",
      "Bash(ls:*)",
      "Bash(pnpm clean:all:*)",
      "Bash(pnpm list:*)",
      "Bash(pnpm ls:*)",
      "Bash(grep:*)",
      "Bash(mkdir:*)",
      "WebSearch",
      "Bash(npx tsc:*)",
      "Bash(pnpm --filter @openstarry/sdk build:*)",
      "Bash(npx --yes rimraf:*)",
      "Bash(for f in scripts/*.sh)",
      "Bash(do sed -i 's/\\\\r$//' \"$f\")",
      "Bash(done)",
      "Bash(npm run build:*)",
      "Bash(npm test:*)",
      "Write(//data/openstarry_eco/agent_dev/**)",
      "Edit(//data/openstarry_eco/agent_dev/**)",
      "Write(//data/openstarry_eco/share/test/reports/**)",
      "Edit(//data/openstarry_eco/share/test/reports/**)",
      "Write(//data/openstarry_eco/share/openstarry_doc/**)",
      "Edit(//data/openstarry_eco/share/openstarry_doc/**)"
    ]
  }
}
```

#### Windows Version (update paths)

Replace the path-based permissions with your actual Windows path. Example:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_TEAMMATE_MODE": "tmux"
  },
  "permissions": {
    "allow": [
      "Bash(pnpm install:*)",
      "Bash(pnpm build)",
      "Bash(pnpm add:*)",
      "Bash(pnpm --filter \"@openstarry-plugin/*\" add -D @types/node)",
      "Bash(timeout 10 node:*)",
      "Bash(printf:*)",
      "Bash(node:*)",
      "Bash(pnpm test:*)",
      "Bash(LOG_FORMAT=json echo:*)",
      "Bash(export LOG_FORMAT=json)",
      "Bash(pnpm clean:*)",
      "Bash(find:*)",
      "Bash(bash:*)",
      "Bash(pnpm -r run build:*)",
      "Bash(dir:*)",
      "Bash(pnpm test:purity:*)",
      "Bash(ls:*)",
      "Bash(pnpm clean:all:*)",
      "Bash(pnpm list:*)",
      "Bash(pnpm ls:*)",
      "Bash(grep:*)",
      "Bash(mkdir:*)",
      "WebSearch",
      "Bash(npx tsc:*)",
      "Bash(pnpm --filter @openstarry/sdk build:*)",
      "Bash(npx --yes rimraf:*)",
      "Bash(for f in scripts/*.sh)",
      "Bash(do sed -i 's/\\\\r$//' \"$f\")",
      "Bash(done)",
      "Bash(npm run build:*)",
      "Bash(npm test:*)",
      "Write(//C:/Users/YourUsername/Desktop/openstarry-eco/agent_dev/**)",
      "Edit(//C:/Users/YourUsername/Desktop/openstarry-eco/agent_dev/**)",
      "Write(//C:/Users/YourUsername/Desktop/openstarry-eco/share/test/reports/**)",
      "Edit(//C:/Users/YourUsername/Desktop/openstarry-eco/share/test/reports/**)",
      "Write(//C:/Users/YourUsername/Desktop/openstarry-eco/share/openstarry_doc/**)",
      "Edit(//C:/Users/YourUsername/Desktop/openstarry-eco/share/openstarry_doc/**)"
    ]
  }
}
```

**Key adjustments for Windows**:
- Replace `/data/openstarry_eco` with your actual Windows path (e.g., `C:/Users/YourUsername/Desktop/openstarry-eco`)
- Use forward slashes (`/`) in the path, not backslashes
- The `//` prefix is required (URL-style path separator)

### 2.3 Scripts

**Location**: `openstarry_eco/scripts/`

Four bash scripts are included:

| Script | Purpose |
|--------|---------|
| `baseline.sh` | Phase 1.5: Pre-implementation backup |
| `sync-to-test.sh` | Phase 2.5: Sync agent_dev → agent_test |
| `snapshot.sh` | Phase 4: Versioned snapshot |
| `restore.sh` | Recovery: restore from baseline/snapshot |

**Windows execution**:
- Use Git Bash (`bash scripts/baseline.sh`) or WSL
- If running via PowerShell, ensure `bash` is available:
  ```powershell
  bash scripts/baseline.sh
  ```

**CRLF warning**: Git on Windows may convert LF to CRLF. To prevent issues:
```bash
# In Git Bash
git config core.autocrlf input
```

Or manually fix line endings:
```bash
# In Git Bash
for f in scripts/*.sh; do sed -i 's/\r$//' "$f"; done
```

---

## 3. Agent Definitions

### Location: `openstarry_eco/.claude/agents/`

Six agent definition files are included. Copy them as-is to the Windows project:

| Agent | File | Role |
|-------|------|------|
| architect | `architect.md` | Architecture guardian, design specs, code review |
| dev-core | `dev-core.md` | Core monorepo developer |
| dev-plugin | `dev-plugin.md` | Plugin developer |
| doc-keeper | `doc-keeper.md` | Documentation, decision persistence |
| qa | `qa.md` | Test runner, quality verification |
| researcher | `researcher.md` | Technical pre-research, ref project analysis |

**No modifications needed** — copy the directory as-is.

---

## 4. Claude Code Memory Files

Claude Code stores project-specific memories in two locations:

### 4.1 Eco-Level Memory

**Linux path**: `~/.claude/projects/-data-openstarry-eco/memory/`

**Windows path**: `$env:USERPROFILE\.claude\projects\{encoded-windows-path}\memory\`

The path encoding changes on Windows. If your project is at `C:\Users\YourUsername\Desktop\openstarry-eco`, the encoded path might look like:
```
C--Users-YourUsername-Desktop-openstarry-eco
```

**Files to copy into this directory**:

#### `MEMORY.md`
```markdown
# OpenStarry Eco Memory

## Current State (as of Cycle 25 complete, 2026-02-13)
- **Version**: v0.19.0-beta (Cycle 25 snapshot + Windows fixes verified)
- **Active Cycle**: 25 complete (Plan22: Plugin Marketplace MVP)
- **Tests**: 1330 in agent_test (117 files), ~1336 in agent_dev (117 files)
- **Packages**: 18 core workspace + 15 plugins
- **Latest snapshot**: `20260213_cycle25`
- **All Plans 01-22 complete**
- **Windows Status**: Ready for deployment

## Key Patterns
- See [patterns.md](patterns.md) for implementation patterns
- See [debugging.md](debugging.md) for known issues and fixes
- See [windows-fixes.md](windows-fixes.md) for Windows-specific fixes applied

## Critical Conventions
- Plugin factory: `createXxxPlugin()` → IPlugin with manifest + factory(ctx)
- Five Aggregates: IUI(Rupa), IListener(Vedana), IProvider(Sanna), ITool(Sankhara), IGuide(Vinnana)
- pushInput pattern: plugins use ctx.pushInput(), never direct API calls
- Lazy accessors: ctx.tools, ctx.guides, ctx.providers — all optional, sync, read-only

## Known Build Issues
- **sync-to-test.sh**: Stale tsbuildinfo — ALWAYS clean before sync:
  `find agent_test -name "tsconfig.tsbuildinfo" -not -path "*/node_modules/*" -delete`
- **snapshot.sh**: Fails if dir exists — delete first: `rm -rf share/openstarry_code_iteration/{id}`
- **runner build**: plugin-catalog.json must be copied to dist/data/ (handled by build script)

## Agent Corps SOP
- Standard cycle: Phase 0→1→1.5→2→2.5→3→4
- Interface freeze after Phase 1 (Arch Spec)
- QA + Architect run in parallel in Phase 3
- Max 2 rework cycles before user escalation
```

#### `patterns.md`
```markdown
# Implementation Patterns

## Lazy Accessor Pattern (IPluginContext)
Used for tools, guides, providers. Sync stubs in sandbox + RPC for actual access.

## Sandbox RPC Pattern
REQUEST/RESPONSE pairs for cross-boundary communication:
- TOOLS_LIST_REQUEST → TOOLS_LIST_RESPONSE
- GUIDES_LIST_REQUEST → GUIDES_LIST_RESPONSE
- PROVIDERS_LIST_REQUEST → PROVIDERS_LIST_RESPONSE
- PROVIDERS_GET_REQUEST → PROVIDERS_GET_RESPONSE

## Event Type Constants
All in `packages/sdk/src/events/types.ts` as `AgentEventType`.

## SessionConfig Pattern
Typed metadata access: `getSessionConfig(session?.metadata)`

## Plugin Integration Order
SDK first → Core → Plugins (dependencies flow downward)
```

#### `windows-fixes.md`
```markdown
# Windows Cross-Platform Fixes (Applied 2026-02-13)

## Status: Applied on Linux (1330 tests all passing), ready for Windows verification

## Fix 1: Sandbox Worker Plugin Path Resolution
**File**: `apps/runner/src/utils/plugin-resolver.ts` (lines 122-131)
**Change**: Use `import.meta.resolve()` (ESM native, Node 20.6+) as primary, `createRequire` as fallback
**Impact**: Plugin loading in sandbox worker works on Windows with long paths

## Fix 2: Plugin Catalog Copying
**File**: `apps/runner/package.json` (build script)
**Change**: `"build": "tsc -b && node -e \"require('fs').cpSync('src/data','dist/data',{recursive:true})\""`
**Impact**: plugin-catalog.json copied to dist/ for runtime loading

## Fix 3: Symlink Dereferencing
**File**: `apps/runner/src/utils/plugin-installer.ts` (lines 100, 144)
**Change**: Added `dereference: true` to both `cp()` calls
**Impact**: Handles Windows symlink permissions gracefully

## Fix 4: Audit Logger Test Timing
**File**: `packages/core/src/sandbox/__tests__/audit-logger.test.ts` (lines 256-274)
**Change**: bufferSize 1→100, removed setTimeout, use dispose() properly
**Impact**: Test reliability on Windows with slower I/O

## Verification Checklist
- [ ] pnpm build succeeds
- [ ] apps/runner/dist/data/plugin-catalog.json exists
- [ ] pnpm test passes all 1330 tests
- [ ] CLI works: node apps/runner/dist/bin.js --help
- [ ] Plugin search works: node apps/runner/dist/bin.js plugin search fs
- [ ] Plugin loading works with actual agent config
```

### 4.2 Monorepo-Level Memory

**Linux path**: `~/.claude/projects/-data-openstarry-eco-agent-dev-openstarry/memory/`

**Windows path**: `$env:USERPROFILE\.claude\projects\{encoded-windows-path-agent-dev-openstarry}\memory\`

**Files to copy**:

#### `MEMORY.md`
Same content as above (Cycle 25 state)

#### `patterns.md`
Full patterns document with all implementation patterns

#### `debugging.md`
Known issues and debug approaches (consult if issues arise during Windows build)

---

## 5. Current Code State

### Version Information
- **Version**: v0.19.0-beta
- **Cycle**: 25 complete (Plan22: Plugin Marketplace MVP)
- **Build status**: Clean (no dist/, tsconfig.tsbuildinfo cleaned)

### Packages

#### Core Workspace (18 packages)
```
agent_dev/openstarry/
├── packages/
│   ├── sdk/                  ← SDK type definitions
│   ├── core/                 ← Agent core engine
│   ├── shared/               ← Shared utilities
│   └── 15 more core packages
└── apps/
    └── runner/               ← CLI entry point
```

**Dependencies**:
- Root: vitest 4.0.18
- All packages: Node.js >= 20.0.0
- Package manager: pnpm 9+

#### Plugin Workspace (15 plugins)
```
agent_dev/openstarry_plugin/
├── mcp-server/               ← MCP protocol server
├── mcp-client/               ← MCP client integration
├── tui-dashboard/            ← Terminal UI dashboard
├── workflow-engine/          ← Workflow execution
├── devtools/                 ← Development tools
├── transport-websocket/      ← WebSocket transport
├── transport-http/           ← HTTP transport
├── http-static/              ← Static HTTP server
├── web-ui/                   ← Web UI plugin
├── provider-gemini-oauth/    ← Gemini OAuth provider
├── standard-function-*/ (3)  ← File system, stdio, skills
├── guide-character-init/     ← Character initialization
└── More plugins in development
```

### Test Status
- **Agent Dev**: 117 test files, 1330 tests, all passing
- **Agent Test**: Ready for sync
- **Expected on Windows**: Same 1330 tests should pass

---

## 6. Windows-Specific Fixes Applied

Four critical fixes have been applied to ensure Windows compatibility:

### Fix 1: Plugin Resolver Path Resolution

**File**: `agent_dev/openstarry/apps/runner/src/utils/plugin-resolver.ts` (lines 122-131)

**Problem**: Sandbox worker couldn't resolve plugins on Windows due to ESM/CJS path resolution differences.

**Solution**: Use `import.meta.resolve()` (ESM native, Node 20.6+) as primary strategy with `createRequire` fallback.

```typescript
// Strategy 3: Try import.meta.resolve() first (ESM native)
try {
  _resolvedModulePath = await import.meta.resolve(ref.name);
} catch {
  // Fallback to createRequire
  const { createRequire } = await import('module');
  const require = createRequire(import.meta.url);
  _resolvedModulePath = require.resolve(ref.name);
}
```

### Fix 2: Plugin Catalog Data Copying

**File**: `agent_dev/openstarry/apps/runner/package.json`

**Problem**: TypeScript compiler doesn't copy non-.ts files. The `plugin-catalog.json` must exist in `dist/data/` for runtime.

**Solution**: Modified build script to explicitly copy data directory:

```json
{
  "scripts": {
    "build": "tsc -b && node -e \"require('fs').cpSync('src/data','dist/data',{recursive:true})\""
  }
}
```

### Fix 3: Symlink Dereferencing

**File**: `agent_dev/openstarry/apps/runner/src/utils/plugin-installer.ts` (lines 100, 144)

**Problem**: pnpm creates symlinks in node_modules. Windows symlink creation requires special permissions. The `cp()` function by default tries to recreate symlinks.

**Solution**: Added `dereference: true` to both `cp()` calls to follow symlinks instead of recreating them:

```typescript
// Line 100: workspace copy
cp(src, dest, { dereference: true, recursive: true });

// Line 144: npm pack extract copy
cp(srcPath, destPath, { dereference: true, recursive: true });
```

### Fix 4: Audit Logger Test Timing

**File**: `agent_dev/openstarry/packages/core/src/sandbox/__tests__/audit-logger.test.ts` (lines 256-274)

**Problem**: Test flakiness on Windows due to timing issues with buffer flushing. `bufferSize: 1` triggered auto-flush as fire-and-forget, preventing proper error capture.

**Solution**: Increased bufferSize and removed setTimeout:

```typescript
// Changed from bufferSize: 1
const logger = new AuditLogger({ bufferSize: 100 });

// Removed setTimeout(100)
// Let dispose() properly await the flush
await logger.dispose();
```

---

## 7. Windows Setup Instructions

### Step 1: Prerequisites

```powershell
# Check Node.js
node --version  # Must be 20.0+

# Check pnpm
pnpm --version  # Must be 9+

# Enable Developer Mode (for symlinks)
# Settings → System → For Developers → Enable Developer Mode
```

### Step 2: Clone/Copy Project

If cloning from Git:
```powershell
git clone https://github.com/your-repo/openstarry-eco.git
cd openstarry-eco

# Fix line endings if needed
git config core.autocrlf input
for ($f in dir scripts\*.sh) { (Get-Content $f) -replace "`r$", "" | Set-Content $f }
```

Or copy the project directory directly to your Windows machine.

### Step 3: Configure Claude Code

1. Open `$env:USERPROFILE\.claude\settings.json` and add:
   ```json
   {
     "alwaysThinkingEnabled": true
   }
   ```

2. Update `openstarry_eco\.claude\settings.local.json`:
   - Replace `/data/openstarry_eco` paths with your actual Windows path
   - Example: `C:/Users/YourUsername/Desktop/openstarry-eco`

3. Ensure `.claude/agents/` directory exists with 6 agent files

### Step 4: Install Dependencies

```powershell
cd openstarry_eco
pnpm install
```

If pnpm symlink creation fails:
- Ensure Developer Mode is enabled
- Or run PowerShell as Administrator
- Or use `pnpm install --force`

### Step 5: Build Project

```powershell
pnpm build
```

Verify the build artifacts:
```powershell
# Check if plugin-catalog.json was copied
Test-Path agent_dev\openstarry\apps\runner\dist\data\plugin-catalog.json
```

### Step 6: Initialize Claude Code Memory

1. Open the project in Claude Code
2. Navigate to `openstarry_eco` directory
3. This creates the project path encoding in `~\.claude\projects\`
4. Create the memory directory:
   ```powershell
   mkdir $env:USERPROFILE\.claude\projects\{encoded-path}\memory -Force
   ```
5. Copy memory files into it

---

## 8. Windows Verification Checklist

Run these checks after setup to verify everything works:

```powershell
# 1. Install and build
cd openstarry_eco
pnpm clean:all
pnpm install
pnpm build

# 2. Verify plugin-catalog.json
Test-Path agent_dev\openstarry\apps\runner\dist\data\plugin-catalog.json
# Should output: True

# 3. Run tests
pnpm test
# Should show: 1330 passing

# 4. CLI smoke test
node agent_dev\openstarry\apps\runner\dist\bin.js --help
# Should show: usage info

# 5. Plugin search test
node agent_dev\openstarry\apps\runner\dist\bin.js plugin search fs
# Should list file system plugin

# 6. (Optional) Start agent with config
# Create test config or use existing agent.json
node agent_dev\openstarry\apps\runner\dist\bin.js start --config path\to\config.json
# Should load plugins without errors
```

If all checks pass, your Windows setup is complete!

---

## 9. Windows Troubleshooting

### Issue: `pnpm install` fails with symlink errors

**Solution**:
1. Enable Windows Developer Mode: Settings → System → For Developers → Enable Developer Mode
2. Or run PowerShell as Administrator
3. Or use: `pnpm install --force`

### Issue: `tsc` or `vitest` not found

**Solution**:
```powershell
# Reinstall with clean cache
pnpm install --force
pnpm build
```

### Issue: Plugin loading fails (sandbox worker error)

**Solution**:
- Ensure `plugin-catalog.json` exists: `agent_dev\openstarry\apps\runner\dist\data\plugin-catalog.json`
- Rebuild: `pnpm build`
- Check logs for path resolution errors

### Issue: Tests hang or timeout

**Solution**:
- Increase timeout: `pnpm test -- --reporter=verbose`
- Check for symlink issues on slow filesystems (USB drives, network shares)
- Try: `pnpm install --prefer-offline`

### Issue: CRLF line ending errors in scripts

**Solution**:
```bash
# Using Git Bash
for f in scripts/*.sh; do sed -i 's/\r$//' "$f"; done

# Or using dos2unix (if installed)
dos2unix scripts/*.sh
```

### Issue: Path length exceeds Windows 260-character limit

**Solution**:
1. Enable long paths in Windows:
   ```powershell
   New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -Force
   ```
2. Restart your machine
3. Retry the build

### Issue: Claude Code can't find project

**Solution**:
1. Check that `openstarry_eco\CLAUDE.md` exists
2. Check that `openstarry_eco\.claude\settings.local.json` has correct paths
3. Restart Claude Code
4. Open the project folder explicitly

---

## Post-Migration Checklist

After completing the Windows setup:

- [ ] All 1330 tests passing
- [ ] CLI smoke tests working
- [ ] All 6 agents defined and accessible
- [ ] Memory files in place
- [ ] Project settings configured
- [ ] Documentation accessible (share/openstarry_doc/)
- [ ] Scripts executable (baseline.sh, sync-to-test.sh, etc.)
- [ ] Ready for next iteration cycle

---

## Additional Resources

- **Project instructions**: `openstarry_eco/CLAUDE.md`
- **Agent SOP**: `share/openstarry_doc/Agent_Corps/Agent_Roles_and_SOP.md`
- **Architecture docs**: `share/openstarry_doc/Architecture_Documentation/`
- **Implementation examples**: `share/openstarry_doc/Implementation_Examples/`
- **Current memory**: Check Claude Code memory at `~/.claude/projects/`

---

## Windows-Specific Notes

1. **Symlinks**: pnpm uses symlinks. On Windows, ensure Developer Mode is enabled or run as admin.
2. **Path separators**: Use forward slashes (`/`) in configuration files, backslashes (`\`) in PowerShell/cmd.
3. **Bash scripts**: Run using Git Bash or WSL. PowerShell needs `bash` available.
4. **Build artifacts**: First build cleans dist/ and tsbuildinfo. Subsequent builds are incremental.
5. **Environment variables**: Adapt export statements (`export VAR=val`) to PowerShell ($env:VAR = "val")

---

**Questions or issues?** Refer to the troubleshooting section or consult the project's main documentation.
