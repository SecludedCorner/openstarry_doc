# 插件一覽

OpenStarry 目前提供 **15 個官方插件**，按功能分類如下。

> 插件倉庫位於 `openstarry_plugin/`，與核心框架同層目錄，透過 `pnpm-workspace.yaml` 統一管理。

## 標準功能

| 插件 | 五蘊 | 說明 |
|------|------|------|
| `standard-function-fs` | 行（ITool） | 檔案讀寫、目錄操作 |
| `standard-function-stdio` | 色+受（IUI+IListener） | CLI 文字輸入/輸出 |
| `standard-function-skill` | 行+識（ITool+IGuide） | Markdown 技能檔案載入 |
| `guide-character-init` | 識（IGuide） | 預設系統提示詞/人設 |

## LLM Provider

| 插件 | 五蘊 | 說明 |
|------|------|------|
| `provider-gemini-oauth` | 想（IProvider） | Google Gemini + PKCE OAuth |

## 傳輸層

| 插件 | 五蘊 | 說明 |
|------|------|------|
| `transport-websocket` | 色+受（IUI+IListener） | WebSocket 雙向通訊（含 token 驗證、CORS） |
| `transport-http` | 色+受（IUI+IListener） | HTTP REST + SSE 串流 |

## 使用者介面

| 插件 | 五蘊 | 說明 |
|------|------|------|
| `web-ui` | 色（IUI） | 瀏覽器聊天介面（搭配 transport-websocket） |
| `tui-dashboard` | 色（IUI） | 終端機全螢幕監控面板 |
| `http-static` | 色（IUI） | 通用靜態檔案伺服器 |

## 開發者工具

| 插件 | 五蘊 | 說明 |
|------|------|------|
| `devtools` | 色+受（IUI+IListener） | 執行時指標、事件日誌 |
| `workflow-engine` | 行（ITool） | YAML 宣告式工作流引擎 |

## MCP 協議

| 插件 | 五蘊 | 說明 |
|------|------|------|
| `mcp-server` | 受（IListener） | 將 Agent 工具暴露給 MCP 客戶端 |
| `mcp-client` | 想（IProvider） | 連接外部 MCP 伺服器 |
| `mcp-common` | （共用庫） | MCP 協議型別與常數 |

## 插件命名

所有官方插件的 npm scope 為 `@openstarry-plugin/`，例如：

```
@openstarry-plugin/standard-function-fs
@openstarry-plugin/transport-websocket
@openstarry-plugin/web-ui
```
