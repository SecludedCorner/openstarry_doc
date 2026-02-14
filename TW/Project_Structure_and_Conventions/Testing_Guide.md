# OpenStarry 測試與發布指南

**版本**: v0.20.1-beta (Cycle 26 — Attach UX + Token Persistence Fix)
**日期**: 2026-02-14
**狀態**: release 已建置，等待手動測試

---

## 專案結構與 Clone 說明

OpenStarry 分為兩個 GitHub repo，**必須放在同一個父資料夾下**:

```
your-workspace/              ← 任意父資料夾
├── openstarry/              ← Repo 1: 核心 monorepo (SDK, Core, Shared, Runner)
│   ├── packages/
│   │   ├── sdk/             ← @openstarry/sdk
│   │   ├── core/            ← @openstarry/core
│   │   ├── shared/          ← @openstarry/shared
│   │   └── plugin-signer/   ← @openstarry/plugin-signer
│   ├── apps/
│   │   └── runner/          ← @openstarry/runner (CLI 入口)
│   ├── configs/             ← Agent 設定檔範例
│   ├── pnpm-workspace.yaml  ← 關鍵: 包含 ../openstarry_plugin/*
│   └── package.json
└── openstarry_plugin/       ← Repo 2: 外掛生態系 (15 個官方外掛)
    ├── standard-function-fs/
    ├── standard-function-stdio/
    ├── transport-websocket/
    ├── web-ui/
    ├── workflow-engine/
    ├── ... (其他外掛)
    └── package.json
```

### Clone 步驟

```bash
# 建立工作目錄
mkdir openstarry-workspace && cd openstarry-workspace

# Clone 兩個 repo 到同一層
git clone <openstarry-repo-url> openstarry
git clone <openstarry-plugin-repo-url> openstarry_plugin
```

### 為什麼必須同層?

`openstarry/pnpm-workspace.yaml` 中設定了:

```yaml
packages:
  - apps/*
  - packages/*
  - ../openstarry_plugin/*     ← 透過相對路徑引用外掛 repo
```

所有外掛的 `package.json` 也透過 `workspace:*` 或 `link:../../openstarry/packages/sdk` 引用核心套件。如果兩個 repo 不在同層，pnpm 將無法解析跨 repo 依賴。

---

## 前置條件

- **Node.js** >= 20
- **pnpm** >= 10 (`npm install -g pnpm`)
- 兩個 repo 已 clone 到同一個父資料夾 (如上所述)
- 如需測試 Agent 對話功能，需要 Gemini OAuth 登入 (`/provider login gemini`)

---

## Step 1: 清理 (如為重新測試)

如果之前已經建置過，先做完整清理:

```bash
cd openstarry        # 進入核心 monorepo
pnpm clean:all
```

這會清除所有 `node_modules/`、`dist/`、`tsconfig.tsbuildinfo`、`pnpm-lock.yaml`，以及 `../openstarry_plugin/` 下的 `node_modules/` 和 `dist/`。

> `clean:all` 使用跨平台 Node 腳本 (`scripts/clean-all.mjs`)，Windows 和 Linux 均可正常執行。
> 如果是全新 clone，可跳過此步驟。

---

## Step 2: 安裝依賴

```bash
cd openstarry        # 進入核心 monorepo (如尚未進入)
pnpm install
```

預期結果: 無錯誤，所有 21 個 workspace 套件 (含 openstarry_plugin 下的外掛) 安裝完成。

> **注意**: 必須從 `openstarry/` 目錄執行，pnpm 會自動透過 workspace 設定解析 `../openstarry_plugin/*` 下的外掛。

---

## Step 3: 建置

```bash
pnpm build
```

預期結果: 20 個套件全部 build 成功。

---

## Step 4: 執行自動測試

```bash
pnpm test
```

預期結果:
- **Test Files**: 117 passed (117)
- **Tests**: 1339 passed | 3 skipped (1342)
- 已知跳過: attach.test.ts 中 1 個 daemon socket 超時測試 + 2 個環境相關 (非 bug)

---

## Step 5: CLI 指令測試

以下所有指令從 `openstarry/` 根目錄執行:

### 5.1 版本資訊

```bash
node apps/runner/dist/bin.js version
```

預期: 顯示版本號。

### 5.2 幫助訊息

```bash
node apps/runner/dist/bin.js --help
```

預期: 顯示所有可用指令清單，包括:
- `start`, `daemon start`, `daemon stop`, `attach`, `ps`
- `init`, `create-plugin`
- `plugin install`, `plugin uninstall`, `plugin list`, `plugin search`, `plugin info`, `plugin sync`
- `version`

### 5.3 初始化設定

```bash
node apps/runner/dist/bin.js init
```

預期: 在當前目錄產生 `agent.json` 設定檔 (如已存在會提示)。

---

## Step 6: Plugin Marketplace 指令測試

### 6.1 搜尋外掛

```bash
node apps/runner/dist/bin.js plugin search fs
```

預期: 顯示包含 "fs" 的外掛列表 (如 `standard-function-fs`)。

```bash
node apps/runner/dist/bin.js plugin search websocket
```

預期: 顯示 `transport-websocket` 相關外掛。

### 6.2 查看外掛資訊

```bash
node apps/runner/dist/bin.js plugin info standard-function-fs
```

預期: 顯示該外掛的 name、version、description、aggregates、tags。

```bash
node apps/runner/dist/bin.js plugin info @openstarry-plugin/web-ui
```

預期: 同上，全名也能解析。

### 6.3 列出所有外掛 (目錄)

```bash
node apps/runner/dist/bin.js plugin list --all
```

預期: 列出全部 15 個官方外掛，標註 `[installed]` 或 `[available]`。

### 6.4 列出已安裝外掛

```bash
node apps/runner/dist/bin.js plugin list
```

預期: 如果從未安裝過，顯示 "No plugins installed"。

### 6.5 安裝單一外掛

```bash
node apps/runner/dist/bin.js plugin install standard-function-fs
```

預期: 從 workspace 解析並安裝，更新 lock file (`~/.openstarry/plugins/lock.json`)。

### 6.6 確認安裝成功

```bash
node apps/runner/dist/bin.js plugin list
```

預期: 顯示剛安裝的 `@openstarry-plugin/standard-function-fs`。

### 6.7 安裝全部外掛

```bash
node apps/runner/dist/bin.js plugin install --all
```

預期: 安裝全部 15 個官方外掛，跳過已安裝的。

### 6.8 再次列出

```bash
node apps/runner/dist/bin.js plugin list
```

預期: 15 個外掛全部顯示為已安裝。

### 6.9 卸載外掛

```bash
node apps/runner/dist/bin.js plugin uninstall standard-function-fs
```

預期: 移除該外掛，更新 lock file。

### 6.10 強制重新安裝

```bash
node apps/runner/dist/bin.js plugin install standard-function-fs --force
```

預期: 即使已安裝也會重新安裝。

---

## Step 7: Agent 啟動測試

### 7.1 Basic Agent (CLI 模式)

```bash
node apps/runner/dist/bin.js start --config configs/basic-agent.json
```

預期: Agent 啟動，CLI 等待使用者輸入。輸入任意文字測試對話，`Ctrl+C` 退出。

> **注意**: 首次啟動會顯示 `[gemini-oauth] Not logged in`，使用 `/provider login gemini` 登入後 token 會持久化，之後不需重新登入。

### 7.2 TUI Agent (終端介面模式)

```bash
node apps/runner/dist/bin.js start --config configs/tui-agent.json
```

預期: Ink-based TUI 介面啟動，顯示 dashboard。`Ctrl+C` 退出。

### 7.3 WebSocket Agent (無頭模式)

```bash
node apps/runner/dist/bin.js start --config configs/websocket-agent.json
```

預期: Agent 啟動並監聽 `ws://0.0.0.0:8080/ws`。可用 WebSocket 客戶端連接測試。

### 7.4 Web Agent (瀏覽器介面)

```bash
node apps/runner/dist/bin.js start --config configs/web-agent.json
```

預期:
- WebSocket 伺服器在 `ws://0.0.0.0:8080/ws`
- Web UI 在 `http://0.0.0.0:8081`
- 瀏覽器開啟 `http://localhost:8081` 可看到聊天介面

---

## Step 8: Daemon 模式測試

### 8.1 啟動 Daemon

```bash
node apps/runner/dist/bin.js daemon start --config configs/basic-agent.json
```

預期: Agent 以背景 daemon 啟動。

### 8.2 查看執行中的 Agent

```bash
node apps/runner/dist/bin.js ps
```

預期: 列出 `basic-agent` (或設定中的 agent ID)。

### 8.3 附加到 Daemon

```bash
node apps/runner/dist/bin.js attach basic-agent
```

預期:
- 連接到背景 agent 的 IPC 通道
- 顯示歡迎訊息 (含 `/help` 提示)
- **自動顯示 provider 登入狀態** (如 "Gemini OAuth: Logged in" 或 "Not logged in")

### 8.4 停止 Daemon

```bash
node apps/runner/dist/bin.js daemon stop basic-agent
```

預期: 背景 agent 停止。

---

## Step 9: Plugin Scaffolding 測試

```bash
cd /tmp
node <path-to-openstarry>/apps/runner/dist/bin.js create-plugin my-test-plugin
```

預期: 在 `/tmp` 產生 `my-test-plugin/` 目錄，包含:
- `package.json`
- `tsconfig.json`
- `src/index.ts` (帶 factory 模板)

---

## Step 10: Plugin Sync 測試

```bash
node apps/runner/dist/bin.js plugin sync --source ../openstarry_plugin
```

預期: 將外掛從 source 目錄同步到系統外掛目錄 (`~/.openstarry/plugins/system/`)。

---

## Step 11: Workflow Engine 測試 (Cycle 23 功能)

```bash
# 建立測試 workflow YAML
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

使用 workflow 指令需要 agent 啟動並載入 workflow-engine 外掛。

---

## 完整外掛清單 (15 個)

| # | 外掛名稱 | 五蘊分類 | 說明 |
|---|----------|---------|------|
| 1 | standard-function-fs | ITool | 檔案讀寫操作 |
| 2 | standard-function-stdio | ITool | 標準輸入輸出 |
| 3 | standard-function-skill | ITool | 技能執行 |
| 4 | guide-character-init | IGuide | 角色初始化引導 |
| 5 | provider-gemini-oauth | IProvider | Gemini OAuth 供應者 |
| 6 | tui-dashboard | IUI | 終端介面 (Ink) |
| 7 | transport-websocket | IListener | WebSocket 傳輸 |
| 8 | transport-http | IListener | HTTP 傳輸 |
| 9 | http-static | IListener | 靜態檔案伺服 |
| 10 | web-ui | IUI | 瀏覽器聊天介面 |
| 11 | mcp-client | IListener/ITool | MCP 客戶端 |
| 12 | mcp-server | IListener | MCP 伺服器 |
| 13 | mcp-common | (共用) | MCP 共用型別 |
| 14 | devtools | ITool | 開發者工具 |
| 15 | workflow-engine | ITool | 工作流引擎 |

---

## 設定檔說明 (configs/)

| 設定檔 | 說明 | 外掛 |
|--------|------|------|
| `basic-agent.json` | 最小 CLI agent | fs + stdio + guide + gemini |
| `example-agent.json` | 範例 agent | (同 basic) |
| `full-agent.json` | 全功能 agent | 所有外掛 |
| `mcp-agent.json` | MCP 整合 agent | mcp-client + mcp-server |
| `tui-agent.json` | 終端介面 agent | tui-dashboard |
| `web-agent.json` | 瀏覽器介面 agent | transport-websocket + web-ui |
| `websocket-agent.json` | WebSocket 無頭 agent | transport-websocket |

---

## 測試結果記錄

| 步驟 | 項目 | 結果 | 備註 |
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
