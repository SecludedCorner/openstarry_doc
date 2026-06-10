# CLI 指令

## 指令總覽

```bash
node apps/runner/dist/bin.js --help
```

| 指令 | 說明 |
|------|------|
| `start` | 啟動 agent（預設） |
| `daemon start` | 背景啟動 agent |
| `daemon stop` | 停止背景 agent |
| `attach` | 附加到運行中的 daemon |
| `ps` | 列出運行中的 agent |
| `init` | 互動式建立新配置 |
| `create-plugin` | 腳手架新插件 |
| `plugin sync` | 同步插件到系統目錄 |
| `version` | 顯示版本資訊 |

## 斜線指令（Agent 內部）

啟動後在 CLI 中使用：

| 指令 | 說明 |
|------|------|
| `/help` | 顯示所有可用指令 |
| `/reset` | 重置對話歷史 |
| `/quit` | 退出 Agent |
| `/provider login gemini` | 登入 Gemini OAuth |
| `/provider logout gemini` | 登出 |
| `/provider status` | 查看登入狀態 |
| `/devtools` | 開啟/關閉開發者面板 |
| `/metrics` | 顯示效能指標 |
| `/workflow <file>` | 執行工作流 |
| `/mcp-server-status` | MCP 伺服器狀態 |

## 使用範例

### 基本啟動

```bash
# 使用預設配置
node apps/runner/dist/bin.js

# 使用指定配置
node apps/runner/dist/bin.js --config ./configs/basic-agent.json
```

### Daemon 模式

```bash
# 背景啟動
node apps/runner/dist/bin.js daemon start --config ./configs/basic-agent.json

# 查看運行中的 agent
node apps/runner/dist/bin.js ps

# 附加到背景 agent
node apps/runner/dist/bin.js attach

# 停止
node apps/runner/dist/bin.js daemon stop
```

### Debug 模式

```bash
LOG_LEVEL=debug node apps/runner/dist/bin.js --config ./configs/basic-agent.json
```
