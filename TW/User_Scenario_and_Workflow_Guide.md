# OpenStarry 用戶情境與操作工作流指南 (User Scenario and Workflow Guide)

本文件將帶您深入體驗 OpenStarry 的核心操作情境，從系統啟動、設計 Agent 到實際對話互動，以及如何將 Agent 納入系統的自動化管理流程。

---

## 1. 系統入口：啟動與監控 (`openstarry`)

當您在終端機輸入 `openstarry` 時，您正在啟動這個多智能體操作系統的**「總控制台 (Runtime Dashboard)」**。

### 操作情境
```bash
$ openstarry
```

### 系統行為
1.  **自動連接/啟動守護進程 (Daemon):** 系統會檢查背景是否已有 `Orchestrator Daemon` 在運行。若無，則自動啟動它。
2.  **進入上帝視角 (Dashboard):** 顯示一個即時更新的 TUI (Text User Interface) 儀表板。

### 視覺體驗 (Dashboard)
```text
  ___                   _____ _
 / _ \ _ __   ___ _ __ /  ___| |_ __ _ _ __ _ __ _   _
| | | | '_ \ / _ \ '_ \\ `--.| __/ _` | '__| '__| | | |
| |_| | |_) |  __/ | | |`--. \ || (_| | |  | |  | |_| |
 \___/| .__/ \___|_| |_/\____/\__\__,_|_|  |_|   \__, |
      | |                                         __/' |
      |_|                                        |___/

[SYSTEM DASHBOARD] - Connected to Daemon (PID: 14023)
---------------------------------------------------------------------
● System Status:  HEALTHY   |   ● Uptime: 2d 4h 12m
● CPU Usage:      12%       |   ● Memory: 402MB / 8GB
---------------------------------------------------------------------

[RUNNING AGENTS]
ID       NAME            TYPE          STATUS    THOUGHTS/SEC   LAST ACTIVITY
sys-01   Master-Mind     Orchestrator  RUNNING   0.5            Just now
dev-01   Code-Helper     Worker        IDLE      0.0            5m ago
web-01   Search-Bot      Worker        BUSY      1.2            Processing...

[CONTROLS]
(q)uit view  (r)estart daemon  (k)ill agent  (d)esign mode
(a)ttach to agent (select via arrows)
```

---

## 2. 設計模式：創造與編輯 (`openstarry design`)

當您需要創建新生命或修改現有 Agent 時，請使用設計模式。該指令具備**上下文感知**能力。

### 場景 A：全域設計模式 (Global Design Mode)
*   **觸發條件：** 在非專案目錄下執行 `openstarry design`。
*   **功能：** 這是「造物主視角」。
    *   **Create New Agent Project:** 從標準模板 (Coding, Writing, Analysis) 孵化一個全新的 Agent 專案。
    *   **Manage Templates:** 下載或管理您的 Agent 基因庫。
    *   **System Configuration:** 設定全域 API Key 或調整 Daemon 資源限制。

### 場景 B：專案設計模式 (Project Design Mode)
*   **觸發條件：** 在 Agent 專案目錄下執行 `openstarry design`。
*   **功能：** 這是「手術台視角」。系統會引導您定義 Agent 的 **五蘊 (Five Aggregates)**：
    1.  **識 (Guide):** 注入靈魂。修改 `system_prompt`，設定**人設**、**核心目標**與**記憶策略**。
    2.  **色 (UI/Body):** 塑造肉體。設定代理人的**名稱**、**外觀**或**互動介面 (UI)**。
    3.  **受 (Listeners):** 配置感官。設定接收訊息的頻道 (如 `stdio`, `webhook`)。
    4.  **行 (Tools):** 裝配肢體。安裝工具插件 (如 `fs`, `gemini-search`)。
    5.  **想 (Providers):** 選擇大腦。連接特定的 LLM 模型 (如 Gemini 1.5 Pro)。

---

## 3. 對話互動：與 Agent 連線 (`openstarry attach`)

要與 Agent 開始對話，我們使用 `attach` 指令。這是互動的**單一入口**。

### 場景 A：在 Dashboard 中連線
1.  在 `openstarry` 儀表板中，使用方向鍵選中目標 Agent (例如 `sys-01`)。
2.  按下鍵盤上的 **`a`** 鍵。
3.  畫面將瞬間切換到該 Agent 的對話視窗。
4.  按下 `Ctrl+D` (Detach) 可返回儀表板，Agent 繼續在後台運行。

### 場景 B：在專案目錄下連線 (最常用)
在您的開發目錄下，直接輸入：
```bash
$ openstarry attach
```
*   **若 Agent 未運行：** 系統會自動啟動它，並直接進入對話介面。
*   **若 Agent 已在後台：** 系統會直接連線進去。

### 場景 C：連線到任意後台 Agent
```bash
$ openstarry attach web-01
```

### 互動技巧：Slash Commands
在對話中，您可以使用 `/` 來直接呼叫工具，而不必請求 LLM。
*   `/login` -> 呼叫 `google-login` 工具 (若有註冊別名)。
*   `/tool google-login` -> 直接執行工具。
*   `/clear` -> 清除當前 Context 記憶。

---

## 4. 部署與自動啟動流程 (Deployment Workflow)

如何讓您的 Agent 在系統啟動時自動上線？

### 步驟 1：同步標準插件 (Sync)
確保您的系統擁有最新的標準能力庫。
```bash
# 假設您已將 openstarry_plugin 倉庫 clone 到本地
$ openstarry plugin sync ./openstarry_plugin
```

### 步驟 2：註冊 (Register)
首先，告訴守護進程 (Daemon) 您的 Agent 在哪裡。在專案目錄下執行：
```bash
$ openstarry register
```
這會將專案路徑寫入 `~/.openstarry/registry.json`，使其成為「託管 Agent」。

### 步驟 3：加入自動啟動列表 (Autostart)
目前有兩種方式：

**方式 A (CLI):**
在 Dashboard 的「系統配置」選單中，或使用未來規劃的指令：
```bash
$ openstarry system config --autostart add <project-id>
```

**方式 B (手動配置):**
編輯 `~/.openstarry/daemon.json`：
```json
{
  "autostart": [
    "master-agent",
    "my-coding-bot",
    "daily-news-reporter"
  ]
}
```

### 步驟 4：驗證 (Verification)
重啟 Daemon (`openstarry daemon restart`) 或重新輸入 `openstarry` 進入儀表板。您應該會看到您的 Agent 已經在 **[RUNNING AGENTS]** 列表中自動上線了。

---

## 5. 進階操作：工作流與多智能體協作 (Advanced Orchestration)

OpenStarry 的強大之處在於能執行「一連串」的 Agent。

### 步驟 1：定義工作流 (Define Workflow)
在專案或系統目錄下創建 `workflow.yaml`：
```yaml
name: Daily Research
steps:
  - id: research
    agent: research-agent
    instruction: "搜集今日 AI 新聞"
  - id: summary
    agent: writer-agent
    input_from: research
    instruction: "總結為 3 點並翻譯成中文"
```

### 步驟 2：執行工作流 (Execute)
在 CLI 中觸發：
```bash
$ openstarry run ./workflow.yaml
```

### 步驟 3：觀察協作 (Observe)
回到 Dashboard (`openstarry`)，您會看到：
1.  **動態孵化：** `research-agent` 突然出現在列表並顯示 `BUSY`。
2.  **任務交接：** 幾秒後，`research-agent` 消失（任務完成被銷毀），`writer-agent` 出現並開始寫作。
3.  **結果交付：** 最終結果會輸出到您的終端機或指定的檔案中。

---

## 6. 故障排除與日誌解讀 (Troubleshooting & Logs)

當 Agent 行為不如預期時，如何診斷？

### 查看即時思考 (Live Thoughts)
使用 `attach` 進入 Agent 後，您會看到它的「內心獨白」：
```text
[THOUGHT] User asked for file search.
[PLAN] I need to use 'fs' tool to list directory.
[CALL] fs.list_dir(path=".")
[RESULT] Success. Found 3 files.
[RESPONSE] I found 3 files...
```
如果卡住了，通常是卡在 `[CALL]` 失敗或 `[THOUGHT]` 循環。

### 查看系統日誌 (System Logs)
若 Agent 根本啟動不了，請查閱 Daemon 日誌：
```bash
$ openstarry system logs
```
常見錯誤：
*   **Template Not Found:** 註冊表路徑錯誤，請重新 `register`。
*   **Permission Denied:** 插件權限不足，請檢查 `manifest.json` 或使用 `openstarry plugin grant`。