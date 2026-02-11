# 09. CLI 設計與管理指令 (CLI Design & Management Commands)

OpenStarry CLI (`openstarry`) 是用戶與 Agent 系統互動的統一入口。它具備上下文感知能力，能根據執行位置自動切換「專案模式」與「系統模式」。

## 1. 核心設計原則

### 上下文感知 (Context Awareness)
CLI 的「設計模式」(`openstarry design`) 具備強大的上下文感知能力，其行為取決於**您在哪裡執行它**。
*   **全域設計模式 (Global Design Mode):** 在非專案目錄下執行時，提供系統級管理功能（如創建新專案、管理全域模板）。
*   **專案設計模式 (Project Design Mode):** 在包含 `agent.json` 的目錄下執行時，自動進入該 Agent 的編輯與配置介面。

### 運行時與設計時分離 (Runtime vs Design)
*   **`openstarry` (Runtime):** 專注於**執行與監控**。無論在哪裡，它都連接到守護進程，提供一個即時的系統儀表板 (Dashboard)。
*   **`openstarry design` (Design):** 專注於**定義與配置**。它是您構建五蘊、組裝 Agent 的工作台。

### 前台與後台 (Foreground vs Background)
*   **專案模式 (Foreground):** Agent 作為當前 Shell 的子進程運行。關閉終端機 = Agent 停止。
*   **守護模式 (Background):** 通過 Daemon 啟動，即便關閉終端機，Agent 仍在背景運行。

---

## 2. 指令集詳解

### A. 核心入口 (Core Entry Points)

```bash
# 啟動 OpenStarry 運行時環境 (The Runtime Environment)
$ openstarry
# -> 檢查 Orchestrator Daemon 是否運行。
# -> 若未運行，則自動啟動 Daemon 與 Master Agent (System Mode)。
# -> 若已運行，則進入互動式控制台 (Console)，顯示當前運行的 Agent 狀態與系統指標。
# -> [Console Feature] 可在控制台中選擇特定 Agent 並按下 'a' 鍵直接連線 (Attach) 進行對話。

# -> [Context Aware] 根據當前目錄自動切換：
#    - 全域模式：顯示「創建新專案」、「下載模板」、「管理全域設定」選單。
#    - 專案模式：顯示當前 Agent 的結構，引導用戶定義 **五蘊** (色受想行識)：
#      * 設定人設與記憶 (識)
#      * 配置介面與名稱 (色)
#      * 添加工具插件 (行)
#      * 選擇感官監聽器 (受)
#      * 指定 AI 模型 (想)
# -> 啟動互動式 TUI (Text User Interface)。
# -> 自動生成或更新 agent.json 與目錄結構。
```

### B. 專案生命週期 (Project Lifecycle)

這些指令用於管理當前目錄下的「專案級 Agent」。

```bash
# 初始化一個新 Agent 專案
$ openstarry init [template-name]
# -> 創建 agent.json, package.json, .openstarry/ 結構

# [互動模式] 啟動並進入對話
$ openstarry attach
# -> 在專案目錄下執行時：
#    - 若 Agent 尚未運行：則自動啟動 Agent 並立即進入互動介面。
#    - 若 Agent 已在後台運行：則直接連線 (Attach) 到該 Agent。
# -> 這是與 Agent 開啟對話的最常用指令。

# [啟動模式] 僅啟動進程到後台
$ openstarry start
# -> 讀取 ./agent.json，將 Agent 啟動為背景守護行程。
# -> 寫入 ./openstarry/agent.pid，日誌記錄到檔案。
# -> 不會佔用當前終端機。

# 直接執行工具 (Manual Tool Invocation)
$ openstarry run-tool <tool-name> [args...]
# -> 不經過 LLM，直接調用指定的 ITool。
# -> 常用於：登入 (google-login)、初始化設定、或是測試工具功能。
# -> 範例：openstarry run-tool google-login

# 清理專案運行數據
$ openstarry clean
# -> 刪除 ./.openstarry/ 中的臨時文件 (logs, state cache)
```

### B. 系統守護管理 (Daemon Management)

這些指令用於與 `~/.openstarry` 中的守護進程交互。

```bash
# 啟動系統守護進程 (如果尚未運行)
$ openstarry daemon start

# 查看系統狀態 (列出所有託管的 Agents)
$ openstarry ps
# OUTPUT:
# ID      NAME        STATUS    PID    MODE      LOCATION
# sys-01  Scheduler   RUNNING   1024   System    ~/.openstarry/agents/sys-01
# prj-01  MyBot       RUNNING   4521   Project   D:/Code/MyBot (Registered)

# 停止守護進程 (及所有子 Agent)
$ openstarry daemon stop
```

### C. 註冊與託管 (Registration & Hosting)

將一個「專案級 Agent」升級為「系統級服務」。

```bash
# 將當前目錄的 Agent 註冊到 Daemon
$ openstarry register
# -> Daemon 會記錄路徑 "D:/Code/MyBot" 到 ~/.openstarry/registry.json
# -> Daemon 現在負責該 Agent 的重啟與監控

# 連線至全域或特定 Agent (Attach)
$ openstarry attach [agent-id]
# -> 若帶有 agent-id：連線至指定的後台 Agent。
# -> 若在專案目錄下且無 agent-id：連線至當前專案 Agent。
# -> 按 Ctrl+C 斷開連接（Detach），但 Agent 繼續在後台運行。

# 刷新插件註冊表 (Manual Refresh)
$ openstarry plugin refresh
# -> 強制 Daemon 重新掃描系統插件目錄。
# -> 用於偵測手動複製進去的新插件。
```

## 3. 數據流向總結

| 模式 | 指令範例 | 配置讀取位置 | 運行狀態 (PID/State) | 日誌位置 | 生命週期 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **專案 (前台)** | `openstarry start` | `./agent.json` | `./.openstarry/` | `./.openstarry/logs/` | 隨 Shell 結束 |
| **專案 (託管)** | `openstarry start -d` | `./agent.json` | `~/.openstarry/state/` | `~/.openstarry/logs/` | 由 Daemon 管理 |
| **系統 (內建)** | `openstarry system start` | `~/.openstarry/agents/` | `~/.openstarry/state/` | `~/.openstarry/logs/` | 常駐服務 |

此設計確保了開發時的靈活性（專案模式）與部署時的穩定性（託管模式）完美共存。
