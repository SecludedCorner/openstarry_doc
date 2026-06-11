# 配置檔格式

## agent.json 結構

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

### 各區段說明

| 區段 | 說明 |
|------|------|
| `identity` | Agent 的身份識別（ID、名稱、描述、版本） |
| `cognition` | LLM 相關設定（provider、模型、溫度、token 上限） |
| `capabilities` | 允許的工具清單與存取路徑 |
| `policy` | 執行策略（並發工具數、超時時間） |
| `memory` | 記憶設定（滑動視窗大小） |
| `plugins` | 載入的插件清單，可附帶 `config` |
| `guide` | 使用的 Guide 名稱 |

## 插件解析順序

1. `path` 指定的檔案路徑（動態 `import()`）
2. `~/.openstarry/plugins/installed/` 系統目錄搜尋
3. 完整套件名（從 workspace / node_modules 解析）

## 預置配置

預置配置在 `configs/` 目錄：

| 配置檔 | 說明 | 使用方式 |
|--------|------|----------|
| `basic-agent.json` | 最小 CLI agent | 基本聊天 + 檔案操作 |
| `web-agent.json` | 瀏覽器 Web UI | 打開瀏覽器 `http://localhost:8081` |
| `websocket-agent.json` | 純 WebSocket（無 CLI） | 程式化 API 存取 |
| `tui-agent.json` | 終端機全螢幕面板 | 視覺化監控 |
| `mcp-agent.json` | MCP 協議 agent | 與 Claude Code 等 MCP 客戶端整合 |
| `full-agent.json` | 全功能 | 開發、展示用 |

## 環境變數

| 變數 | 說明 | 預設值 |
|------|------|--------|
| `LOG_LEVEL` | 日誌等級：`debug`, `info`, `warn`, `error` | `info` |
| `LOG_FORMAT` | 日誌格式：`text`, `json` | `text` |
| `OPENSTARRY_WS_TOKEN` | WebSocket 認證 token（替代 config 設定） | （無） |

```bash
# Debug 模式
LOG_LEVEL=debug node apps/runner/dist/bin.js --config ./configs/basic-agent.json

# JSON 日誌輸出
LOG_FORMAT=json node apps/runner/dist/bin.js
```
