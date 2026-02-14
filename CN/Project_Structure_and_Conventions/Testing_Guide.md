# OpenStarry 测试与发布指南

**版本**: v0.20.1-beta (Cycle 26 — Attach UX + Token Persistence Fix)
**日期**: 2026-02-14
**状态**: release 已构建，等待手动测试

---

## 项目结构与 Clone 说明

OpenStarry 分为两个 GitHub repo，**必须放在同一个父文件夹下**:

```
your-workspace/              ← 任意父文件夹
├── openstarry/              ← Repo 1: 核心 monorepo (SDK, Core, Shared, Runner)
│   ├── packages/
│   │   ├── sdk/             ← @openstarry/sdk
│   │   ├── core/            ← @openstarry/core
│   │   ├── shared/          ← @openstarry/shared
│   │   └── plugin-signer/   ← @openstarry/plugin-signer
│   ├── apps/
│   │   └── runner/          ← @openstarry/runner (CLI 入口)
│   ├── configs/             ← Agent 配置文件示例
│   ├── pnpm-workspace.yaml  ← 关键: 包含 ../openstarry_plugin/*
│   └── package.json
└── openstarry_plugin/       ← Repo 2: 插件生态系 (15 个官方插件)
    ├── standard-function-fs/
    ├── standard-function-stdio/
    ├── transport-websocket/
    ├── web-ui/
    ├── workflow-engine/
    ├── ... (其他插件)
    └── package.json
```

### Clone 步骤

```bash
# 创建工作目录
mkdir openstarry-workspace && cd openstarry-workspace

# Clone 两个 repo 到同一层
git clone <openstarry-repo-url> openstarry
git clone <openstarry-plugin-repo-url> openstarry_plugin
```

### 为什么必须同层?

`openstarry/pnpm-workspace.yaml` 中设置了:

```yaml
packages:
  - apps/*
  - packages/*
  - ../openstarry_plugin/*     ← 通过相对路径引用插件 repo
```

所有插件的 `package.json` 也通过 `workspace:*` 或 `link:../../openstarry/packages/sdk` 引用核心包。如果两个 repo 不在同层，pnpm 将无法解析跨 repo 依赖。

---

## 前置条件

- **Node.js** >= 20
- **pnpm** >= 10 (`npm install -g pnpm`)
- 两个 repo 已 clone 到同一个父文件夹 (如上所述)
- 如需测试 Agent 对话功能，需要 Gemini OAuth 登录 (`/provider login gemini`)

---

## Step 1: 清理 (如为重新测试)

如果之前已经构建过，先做完整清理:

```bash
cd openstarry        # 进入核心 monorepo
pnpm clean:all
```

这会清除所有 `node_modules/`、`dist/`、`tsconfig.tsbuildinfo`、`pnpm-lock.yaml`，以及 `../openstarry_plugin/` 下的 `node_modules/` 和 `dist/`。

> `clean:all` 使用跨平台 Node 脚本 (`scripts/clean-all.mjs`)，Windows 和 Linux 均可正常执行。
> 如果是全新 clone，可跳过此步骤。

---

## Step 2: 安装依赖

```bash
cd openstarry        # 进入核心 monorepo (如尚未进入)
pnpm install
```

预期结果: 无错误，所有 21 个 workspace 包 (含 openstarry_plugin 下的插件) 安装完成。

> **注意**: 必须从 `openstarry/` 目录执行，pnpm 会自动通过 workspace 设置解析 `../openstarry_plugin/*` 下的插件。

---

## Step 3: 构建

```bash
pnpm build
```

预期结果: 20 个包全部 build 成功。

---

## Step 4: 执行自动测试

```bash
pnpm test
```

预期结果:
- **Test Files**: 117 passed (117)
- **Tests**: 1339 passed | 3 skipped (1342)
- 已知跳过: attach.test.ts 中 1 个 daemon socket 超时测试 + 2 个环境相关 (非 bug)

---

## Step 5: CLI 命令测试

以下所有命令从 `openstarry/` 根目录执行:

### 5.1 版本信息

```bash
node apps/runner/dist/bin.js version
```

预期: 显示版本号。

### 5.2 帮助信息

```bash
node apps/runner/dist/bin.js --help
```

预期: 显示所有可用命令清单，包括:
- `start`, `daemon start`, `daemon stop`, `attach`, `ps`
- `init`, `create-plugin`
- `plugin install`, `plugin uninstall`, `plugin list`, `plugin search`, `plugin info`, `plugin sync`
- `version`

### 5.3 初始化配置

```bash
node apps/runner/dist/bin.js init
```

预期: 在当前目录生成 `agent.json` 配置文件 (如已存在会提示)。

---

## Step 6: Plugin Marketplace 命令测试

### 6.1 搜索插件

```bash
node apps/runner/dist/bin.js plugin search fs
```

预期: 显示包含 "fs" 的插件列表 (如 `standard-function-fs`)。

```bash
node apps/runner/dist/bin.js plugin search websocket
```

预期: 显示 `transport-websocket` 相关插件。

### 6.2 查看插件信息

```bash
node apps/runner/dist/bin.js plugin info standard-function-fs
```

预期: 显示该插件的 name、version、description、aggregates、tags。

```bash
node apps/runner/dist/bin.js plugin info @openstarry-plugin/web-ui
```

预期: 同上，全名也能解析。

### 6.3 列出所有插件 (目录)

```bash
node apps/runner/dist/bin.js plugin list --all
```

预期: 列出全部 15 个官方插件，标注 `[installed]` 或 `[available]`。

### 6.4 列出已安装插件

```bash
node apps/runner/dist/bin.js plugin list
```

预期: 如果从未安装过，显示 "No plugins installed"。

### 6.5 安装单一插件

```bash
node apps/runner/dist/bin.js plugin install standard-function-fs
```

预期: 从 workspace 解析并安装，更新 lock file (`~/.openstarry/plugins/lock.json`)。

### 6.6 确认安装成功

```bash
node apps/runner/dist/bin.js plugin list
```

预期: 显示刚安装的 `@openstarry-plugin/standard-function-fs`。

### 6.7 安装全部插件

```bash
node apps/runner/dist/bin.js plugin install --all
```

预期: 安装全部 15 个官方插件，跳过已安装的。

### 6.8 再次列出

```bash
node apps/runner/dist/bin.js plugin list
```

预期: 15 个插件全部显示为已安装。

### 6.9 卸载插件

```bash
node apps/runner/dist/bin.js plugin uninstall standard-function-fs
```

预期: 移除该插件，更新 lock file。

### 6.10 强制重新安装

```bash
node apps/runner/dist/bin.js plugin install standard-function-fs --force
```

预期: 即使已安装也会重新安装。

---

## Step 7: Agent 启动测试

### 7.1 Basic Agent (CLI 模式)

```bash
node apps/runner/dist/bin.js start --config configs/basic-agent.json
```

预期: Agent 启动，CLI 等待用户输入。输入任意文字测试对话，`Ctrl+C` 退出。

> **注意**: 首次启动会显示 `[gemini-oauth] Not logged in`，使用 `/provider login gemini` 登录后 token 会持久化，之后不需重新登录。

### 7.2 TUI Agent (终端界面模式)

```bash
node apps/runner/dist/bin.js start --config configs/tui-agent.json
```

预期: Ink-based TUI 界面启动，显示 dashboard。`Ctrl+C` 退出。

### 7.3 WebSocket Agent (无头模式)

```bash
node apps/runner/dist/bin.js start --config configs/websocket-agent.json
```

预期: Agent 启动并监听 `ws://0.0.0.0:8080/ws`。可用 WebSocket 客户端连接测试。

### 7.4 Web Agent (浏览器界面)

```bash
node apps/runner/dist/bin.js start --config configs/web-agent.json
```

预期:
- WebSocket 服务器在 `ws://0.0.0.0:8080/ws`
- Web UI 在 `http://0.0.0.0:8081`
- 浏览器打开 `http://localhost:8081` 可看到聊天界面

---

## Step 8: Daemon 模式测试

### 8.1 启动 Daemon

```bash
node apps/runner/dist/bin.js daemon start --config configs/basic-agent.json
```

预期: Agent 以后台 daemon 启动。

### 8.2 查看运行中的 Agent

```bash
node apps/runner/dist/bin.js ps
```

预期: 列出 `basic-agent` (或配置中的 agent ID)。

### 8.3 附加到 Daemon

```bash
node apps/runner/dist/bin.js attach basic-agent
```

预期:
- 连接到后台 agent 的 IPC 通道
- 显示欢迎消息 (含 `/help` 提示)
- **自动显示 provider 登录状态** (如 "Gemini OAuth: Logged in" 或 "Not logged in")

### 8.4 停止 Daemon

```bash
node apps/runner/dist/bin.js daemon stop basic-agent
```

预期: 后台 agent 停止。

---

## Step 9: Plugin Scaffolding 测试

```bash
cd /tmp
node <path-to-openstarry>/apps/runner/dist/bin.js create-plugin my-test-plugin
```

预期: 在 `/tmp` 生成 `my-test-plugin/` 目录，包含:
- `package.json`
- `tsconfig.json`
- `src/index.ts` (带 factory 模板)

---

## Step 10: Plugin Sync 测试

```bash
node apps/runner/dist/bin.js plugin sync --source ../openstarry_plugin
```

预期: 将插件从 source 目录同步到系统插件目录 (`~/.openstarry/plugins/system/`)。

---

## Step 11: Workflow Engine 测试 (Cycle 23 功能)

```bash
# 创建测试 workflow YAML
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

使用 workflow 命令需要 agent 启动并加载 workflow-engine 插件。

---

## 完整插件清单 (15 个)

| # | 插件名称 | 五蕴分类 | 说明 |
|---|----------|---------|------|
| 1 | standard-function-fs | ITool | 文件读写操作 |
| 2 | standard-function-stdio | ITool | 标准输入输出 |
| 3 | standard-function-skill | ITool | 技能执行 |
| 4 | guide-character-init | IGuide | 角色初始化引导 |
| 5 | provider-gemini-oauth | IProvider | Gemini OAuth 提供者 |
| 6 | tui-dashboard | IUI | 终端界面 (Ink) |
| 7 | transport-websocket | IListener | WebSocket 传输 |
| 8 | transport-http | IListener | HTTP 传输 |
| 9 | http-static | IListener | 静态文件服务 |
| 10 | web-ui | IUI | 浏览器聊天界面 |
| 11 | mcp-client | IListener/ITool | MCP 客户端 |
| 12 | mcp-server | IListener | MCP 服务器 |
| 13 | mcp-common | (共用) | MCP 共用类型 |
| 14 | devtools | ITool | 开发者工具 |
| 15 | workflow-engine | ITool | 工作流引擎 |

---

## 配置文件说明 (configs/)

| 配置文件 | 说明 | 插件 |
|--------|------|------|
| `basic-agent.json` | 最小 CLI agent | fs + stdio + guide + gemini |
| `example-agent.json` | 示例 agent | (同 basic) |
| `full-agent.json` | 全功能 agent | 所有插件 |
| `mcp-agent.json` | MCP 集成 agent | mcp-client + mcp-server |
| `tui-agent.json` | 终端界面 agent | tui-dashboard |
| `web-agent.json` | 浏览器界面 agent | transport-websocket + web-ui |
| `websocket-agent.json` | WebSocket 无头 agent | transport-websocket |

---

## 测试结果记录

| 步骤 | 项目 | 结果 | 备注 |
|------|------|------|------|
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
