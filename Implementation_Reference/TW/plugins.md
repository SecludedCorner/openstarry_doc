# 插件一覽

> ⚠️ **[漂移更正 — v0.59.6；v0.59.8 +agent-comm +comm-channel-p2p] 套件數：49 個 package 目錄 = 48 個可載入插件 + 1 個共享函式庫（`mcp-common`）。** 原文「v0.58.0-alpha 共 43 個插件目錄」已陳舊。實測依據：`openstarry_plugin/` 下 `package.json` 共 49 個（`ls -d openstarry_plugin/*/ | wc -l` = 47；pnpm workspace glob `../openstarry_plugin/*` 見 `openstarry/pnpm-workspace.yaml:4` 全數納管）。其中 `mcp-common`（`@openstarry-plugin/mcp-common`，`mcp-common/package.json:1-22`）無 `test` script、不依賴 `@openstarry/sdk`、僅匯出協定 types/constants ＝ 共享函式庫，非可載入 plugin（對照可載入者如 `mcp-server/package.json:17,21` 有 `test` 且依賴 `@openstarry/sdk`）。下方表格為策展精選（非窮舉），其數字與標籤仍正確（`mcp-common` 下方已標「共用庫」）。

OpenStarry 插件工作區於 v0.59.x-alpha 共有 **49 個 package 目錄**——**48 個可載入插件 + 1 個共享函式庫（`mcp-common`）**（計數於 2026-06-27 為 `agent-comm` ＋ `comm-channel-p2p` 更新；下表為精選清單，非完整列表——完整集合見 `openstarry_plugin/`）。

> ⚠️ **[誠實標記 — v0.59.7（2026-06-17）] 「可載入」≠「掛載後提供 agent 能力」。** 46 個皆可被 loader 載入（factory 跑、回傳合法 `PluginHooks`），但其中 **3 個——`api-runtime`（Plan59）、`mesh`（Plan58）、`vasana-engine`（Plan57 Track1）——的 plugin factory 僅做開機驗證後回傳 `{dispose}`，未註冊任何 agent-facing hook/tool/service**（載入時 loader 會發 sigma-4/5/14「Declares skandha but no hook registered」警告，冷啟動 smoke 可見）。它們的真實價值是**被匯入的函式庫**（供其他插件/規格消費、各自有完整 `__tests__/` 測試），而非掛載後 agent 可直接使用的能力。換言之這 3 個是「library 包成 plugin 外殼」，bounded-by-design（見各自 src 註解與 MEMORY 排除清單）。

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
