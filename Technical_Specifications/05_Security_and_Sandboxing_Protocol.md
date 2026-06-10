# 05. 安全與沙盒協議技術規範 (Security & Sandboxing Protocol)

本文件定義了 OpenStarry 的多層防禦體系。作為一個擁有執行能力的 Agent 系統，安全性是不可妥協的基石。

## 1. 檔案系統沙盒 (FileSystem Sandboxing)

所有涉及 IO 的操作（讀、寫、列表）必須受到 `PathGuard` 的嚴格監控。

### 1.1 監獄模式 (Chroot-like Jail)
Agent 預設只能訪問其工作目錄 (`workspace_root`)。任何試圖訪問父目錄的操作都會被攔截。

*   **允許:** `./data/file.txt`, `src/index.ts`
*   **禁止:** `../secret.key`, `/etc/passwd`, `C:\Windows\System32`

### 1.2 路徑規範化
在檢查權限前，所有路徑必須經過解析與規範化，以防止透過符號連結 (Symlink) 或相對路徑欺騙 (`/app/workspace/../../etc/passwd`) 繞過檢查。

---

## 2. 命令執行白名單 (Command Execution Whitelist)

對於 `shell:exec` 這類高風險工具，必須實施白名單策略。

### 安全策略配置 (`agent.json`)

```json
"security": {
  "allow_shell": true,
  "blocked_commands": ["rm -rf", "mkfs", ":(){:|:&};:", "format"],
  "allowed_commands": ["ls", "cat", "grep", "npm test", "git status"],
  "confirm_dangerous_actions": true
}
```

當 `confirm_dangerous_actions` 為 `true` 時，任何不在 `allowed_commands` 中的指令，核心將暫停執行並發出 `EVENT:REQUEST_CONFIRMATION`，等待人類批准後才放行。

---

## 3. 資源配額與熔斷 (Resource Quotas & Circuit Breakers)

為了防止 DoS 攻擊或無限循環，核心強制執行以下限額：

| 資源維度 | 限制閾值 (預設) | 觸發後果 |
| :--- | :--- | :--- |
| **Max Loop Ticks** | 50 次/任務 | 強制暫停任務 (Safety Halt) |
| **Token Budget** | 100k / 會話 | 拒絕 LLM 請求 |
| **Tool Timeout** | 30 秒 | 強制終止工具進程 (SIGKILL) |
| **Output Size** | 1 MB | 截斷輸出 |

---

## 4. 插件權限模型 (Plugin Permission Model)

插件在加載時必須宣告其所需權限 (Manifest Declaration)。

```json
// plugin.json
{
  "permissions": [
    "fs:read",
    "network:http",
    "core:state:write"
  ]
}
```

核心在初始化插件時，會對比 `agent.json` 中的授權列表。若插件請求了未授權的敏感權限，加載將被拒絕。
