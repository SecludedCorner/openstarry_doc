# Windows Migration Guide for OpenStarry Eco

**Last updated**: 2026-02-13
**Version**: v0.19.0-beta (Cycle 25 + Windows 跨平台修复)
**Status**: All Linux verification complete; ready for Windows deployment

本指南记录了完整的 Claude Code 环境和 OpenStarry 项目状态，以便在 Windows 上重现。

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

创建或更新你的全局 Claude Code 设置：

```json
{
  "alwaysThinkingEnabled": true
}
```

**Windows 上的路径展开**：
- 如果你的主目录是 `C:\Users\YourUsername`，路径变为：
  ```
  C:\Users\YourUsername\.claude\settings.json
  ```
- 或者在 PowerShell 中使用 `$env:USERPROFILE\.claude\settings.json`

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

此文件已在仓库中，会随代码一起迁移。它包含：
- 项目概述和目录结构
- Agent Corps 定义（6 个 agent）
- 标准迭代周期（SOP）
- 架构原则（五蕴）
- 构建和测试命令
- 命名约定
- 约束和要求

**无需修改** — 保持项目根目录中的原样。

### 2.2 Project Settings: `.claude/settings.local.json`

**Location**: `openstarry_eco/.claude/settings.local.json`

此文件**必须更新为 Windows 路径**。

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

将基于路径的权限替换为你实际的 Windows 路径。示例：

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

**Windows 的关键调整**：
- 将 `/data/openstarry_eco` 替换为你实际的 Windows 路径（例如 `C:/Users/YourUsername/Desktop/openstarry-eco`）
- 在路径中使用正斜杠（`/`），而不是反斜杠
- 需要 `//` 前缀（URL 风格路径分隔符）

### 2.3 Scripts

**Location**: `openstarry_eco/scripts/`

包含四个 bash 脚本：

| Script | Purpose |
|--------|---------|
| `baseline.sh` | Phase 1.5: 实现前备份 |
| `sync-to-test.sh` | Phase 2.5: 同步 agent_dev → agent_test |
| `snapshot.sh` | Phase 4: 版本快照 |
| `restore.sh` | Recovery: 从 baseline/snapshot 恢复 |

**Windows 执行**：
- 使用 Git Bash (`bash scripts/baseline.sh`) 或 WSL
- 如果通过 PowerShell 运行，确保 `bash` 可用：
  ```powershell
  bash scripts/baseline.sh
  ```

**CRLF 警告**：Windows 上的 Git 可能将 LF 转换为 CRLF。为防止问题：
```bash
# In Git Bash
git config core.autocrlf input
```

或手动修复行尾：
```bash
# In Git Bash
for f in scripts/*.sh; do sed -i 's/\r$//' "$f"; done
```

---

## 3. Agent Definitions

### Location: `openstarry_eco/.claude/agents/`

包含六个 agent 定义文件。将它们原样复制到 Windows 项目：

| Agent | File | Role |
|-------|------|------|
| architect | `architect.md` | 架构守护者、设计规格、代码审查 |
| dev-core | `dev-core.md` | 核心 monorepo 开发者 |
| dev-plugin | `dev-plugin.md` | 插件开发者 |
| doc-keeper | `doc-keeper.md` | 文档管理、决策持久化 |
| qa | `qa.md` | 测试运行、质量验证 |
| researcher | `researcher.md` | 技术预研、参考项目分析 |

**无需修改** — 直接复制目录即可。

---

## 4. Claude Code Memory Files

Claude Code 在两个位置存储项目特定的记忆：

### 4.1 Eco-Level Memory

**Linux path**: `~/.claude/projects/-data-openstarry-eco/memory/`

**Windows path**: `$env:USERPROFILE\.claude\projects\{encoded-windows-path}\memory\`

路径编码在 Windows 上会发生变化。如果你的项目位于 `C:\Users\YourUsername\Desktop\openstarry-eco`，编码路径可能如下：
```
C--Users-YourUsername-Desktop-openstarry-eco
```

**需要复制到此目录的文件**：

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
- Five Aggregates: IUI(色), IListener(受), IProvider(想), ITool(行), IGuide(识)
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

**需要复制的文件**：

#### `MEMORY.md`
与上面相同的内容（Cycle 25 状态）

#### `patterns.md`
包含所有实现模式的完整 patterns 文档

#### `debugging.md`
已知问题和调试方法（如果在 Windows 构建期间遇到问题请参考）

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
│   ├── sdk/                  ← SDK 类型定义
│   ├── core/                 ← Agent 核心引擎
│   ├── shared/               ← 共享工具库
│   └── 15 more core packages
└── apps/
    └── runner/               ← CLI 入口点
```

**Dependencies**:
- Root: vitest 4.0.18
- All packages: Node.js >= 20.0.0
- Package manager: pnpm 9+

#### Plugin Workspace (15 plugins)
```
agent_dev/openstarry_plugin/
├── mcp-server/               ← MCP 协议服务器
├── mcp-client/               ← MCP 客户端集成
├── tui-dashboard/            ← 终端 UI 仪表板
├── workflow-engine/          ← 工作流执行
├── devtools/                 ← 开发工具
├── transport-websocket/      ← WebSocket 传输层
├── transport-http/           ← HTTP 传输层
├── http-static/              ← 静态 HTTP 服务器
├── web-ui/                   ← Web UI 插件
├── provider-gemini-oauth/    ← Gemini OAuth 提供者
├── standard-function-*/ (3)  ← 文件系统、stdio、skills
├── guide-character-init/     ← 角色初始化
└── More plugins in development
```

### Test Status
- **Agent Dev**: 117 test files, 1330 tests, all passing
- **Agent Test**: Ready for sync
- **Expected on Windows**: Same 1330 tests should pass

---

## 6. Windows-Specific Fixes Applied

已应用四项关键修复以确保 Windows 兼容性：

### Fix 1: Plugin Resolver Path Resolution

**File**: `agent_dev/openstarry/apps/runner/src/utils/plugin-resolver.ts` (lines 122-131)

**问题**: 由于 ESM/CJS 路径解析差异，Sandbox worker 无法在 Windows 上解析插件。

**解决方案**: 使用 `import.meta.resolve()`（ESM 原生，Node 20.6+）作为主要策略，`createRequire` 作为回退。

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

**问题**: TypeScript 编译器不会复制非 .ts 文件。`plugin-catalog.json` 必须存在于 `dist/data/` 中供运行时使用。

**解决方案**: 修改构建脚本以显式复制数据目录：

```json
{
  "scripts": {
    "build": "tsc -b && node -e \"require('fs').cpSync('src/data','dist/data',{recursive:true})\""
  }
}
```

### Fix 3: Symlink Dereferencing

**File**: `agent_dev/openstarry/apps/runner/src/utils/plugin-installer.ts` (lines 100, 144)

**问题**: pnpm 在 node_modules 中创建符号链接。Windows 创建符号链接需要特殊权限。`cp()` 函数默认尝试重新创建符号链接。

**解决方案**: 在两个 `cp()` 调用中添加 `dereference: true`，跟随符号链接而不是重新创建它们：

```typescript
// Line 100: workspace copy
cp(src, dest, { dereference: true, recursive: true });

// Line 144: npm pack extract copy
cp(srcPath, destPath, { dereference: true, recursive: true });
```

### Fix 4: Audit Logger Test Timing

**File**: `agent_dev/openstarry/packages/core/src/sandbox/__tests__/audit-logger.test.ts` (lines 256-274)

**问题**: 由于缓冲区刷新的时序问题，测试在 Windows 上不稳定。`bufferSize: 1` 触发自动刷新作为 fire-and-forget，导致无法正确捕获错误。

**解决方案**: 增加 bufferSize 并移除 setTimeout：

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

如果从 Git 克隆：
```powershell
git clone https://github.com/your-repo/openstarry-eco.git
cd openstarry-eco

# Fix line endings if needed
git config core.autocrlf input
for ($f in dir scripts\*.sh) { (Get-Content $f) -replace "`r$", "" | Set-Content $f }
```

或直接将项目目录复制到你的 Windows 机器上。

### Step 3: Configure Claude Code

1. 打开 `$env:USERPROFILE\.claude\settings.json` 并添加：
   ```json
   {
     "alwaysThinkingEnabled": true
   }
   ```

2. 更新 `openstarry_eco\.claude\settings.local.json`：
   - 将 `/data/openstarry_eco` 路径替换为你实际的 Windows 路径
   - 示例：`C:/Users/YourUsername/Desktop/openstarry-eco`

3. 确保 `.claude/agents/` 目录存在且包含 6 个 agent 文件

### Step 4: Install Dependencies

```powershell
cd openstarry_eco
pnpm install
```

如果 pnpm 符号链接创建失败：
- 确保已启用 Developer Mode
- 或以管理员身份运行 PowerShell
- 或使用 `pnpm install --force`

### Step 5: Build Project

```powershell
pnpm build
```

验证构建产物：
```powershell
# Check if plugin-catalog.json was copied
Test-Path agent_dev\openstarry\apps\runner\dist\data\plugin-catalog.json
```

### Step 6: Initialize Claude Code Memory

1. 在 Claude Code 中打开项目
2. 导航到 `openstarry_eco` 目录
3. 这会在 `~\.claude\projects\` 中创建项目路径编码
4. 创建 memory 目录：
   ```powershell
   mkdir $env:USERPROFILE\.claude\projects\{encoded-path}\memory -Force
   ```
5. 将 memory 文件复制到其中

---

## 8. Windows Verification Checklist

设置完成后运行这些检查以验证一切正常：

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

如果所有检查都通过，你的 Windows 设置就完成了！

---

## 9. Windows Troubleshooting

### Issue: `pnpm install` fails with symlink errors

**解决方案**：
1. 启用 Windows Developer Mode：Settings → System → For Developers → Enable Developer Mode
2. 或以管理员身份运行 PowerShell
3. 或使用：`pnpm install --force`

### Issue: `tsc` or `vitest` not found

**解决方案**：
```powershell
# Reinstall with clean cache
pnpm install --force
pnpm build
```

### Issue: Plugin loading fails (sandbox worker error)

**解决方案**：
- 确保 `plugin-catalog.json` 存在：`agent_dev\openstarry\apps\runner\dist\data\plugin-catalog.json`
- 重新构建：`pnpm build`
- 检查日志中的路径解析错误

### Issue: Tests hang or timeout

**解决方案**：
- 增加超时时间：`pnpm test -- --reporter=verbose`
- 检查慢速文件系统（USB 驱动器、网络共享）上的符号链接问题
- 尝试：`pnpm install --prefer-offline`

### Issue: CRLF line ending errors in scripts

**解决方案**：
```bash
# Using Git Bash
for f in scripts/*.sh; do sed -i 's/\r$//' "$f"; done

# Or using dos2unix (if installed)
dos2unix scripts/*.sh
```

### Issue: Path length exceeds Windows 260-character limit

**解决方案**：
1. 在 Windows 中启用长路径：
   ```powershell
   New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -Force
   ```
2. 重启机器
3. 重试构建

### Issue: Claude Code can't find project

**解决方案**：
1. 检查 `openstarry_eco\CLAUDE.md` 是否存在
2. 检查 `openstarry_eco\.claude\settings.local.json` 中的路径是否正确
3. 重启 Claude Code
4. 显式打开项目文件夹

---

## Post-Migration Checklist

完成 Windows 设置后：

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

1. **符号链接**: pnpm 使用符号链接。在 Windows 上，确保已启用 Developer Mode 或以管理员身份运行。
2. **路径分隔符**: 在配置文件中使用正斜杠（`/`），在 PowerShell/cmd 中使用反斜杠（`\`）。
3. **Bash 脚本**: 使用 Git Bash 或 WSL 运行。PowerShell 需要 `bash` 可用。
4. **构建产物**: 首次构建会清理 dist/ 和 tsbuildinfo。后续构建是增量的。
5. **环境变量**: 将 export 语句（`export VAR=val`）调整为 PowerShell 格式（`$env:VAR = "val"`）

---

**有问题？** 请参考故障排除部分或查阅项目主文档。
