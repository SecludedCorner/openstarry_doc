# 深度解析：安全層

本文件深入探討「無頭代理人核心」的安全邊界——安全層的實現機制。

## 核心職責

安全層作為核心與外部世界交互的「守門員」，其唯一職責是在 `Tool` 被執行**之前**，攔截調用請求，並根據預設策略進行處理，以確保代理人的行為符合安全與合規要求。

---

## 實現機制：一個多級決策流

1.  **攔截點：** 安全層位於「執行循環」中，在 LLM 解析出工具調用請求之後、實際執行工具之前。

2.  **與策略引擎的交互 (API Contract):**
    *   安全層的第一步是向外部的「策略與安全護欄引擎」發起一次同步的權限檢查請求。
    *   **請求體 (Request Body):**
        ```json
        {
          "principal": { "id": "user-123", "groups": ["editor"] },
          "action": "tool:execute",
          "resource": {
            "type": "tool",
            "name": "shell:execute",
            "attributes": {
              "args": { "command": "ls -l /data" }
            }
          },
          "context": { "ip_address": "...", "session_id": "..." }
        }
        ```
    *   **響應體 (Response Body):**
        ```json
        {
          "decision": "ALLOW" | "DENY" | "REQUIRE_USER_CONFIRMATION",
          "reason": "Command 'ls' is in the whitelist.",
          "obligations": [ /* 例如：需要在執行後記錄日誌 */ ]
        }
        ```
### 3. 參數清洗與驗證 (Argument Sanitization & Validation):
*   **在執行之前**，即使策略引擎和用戶都已批准，安全層也應執行最後一道檢查。
*   **路徑限制機制 (Path Scoping - 核心安全特性):**
    *   **定義：** 針對 `fs` (檔案系統) 類工具，系統會強制執行「根路徑限制」。Agent 被賦予一個或多個「許可根目錄」(Permitted Roots，例如 `./workspace/`)。
    *   **路徑規範化 (Normalization):** 安全層會先將 Agent 傳入的相對路徑或絕對路徑轉化為絕對規範路徑，並移除所有 `..` (Parent directory) 引用。
    *   **邊界檢查 (Boundary Check):** 系統會驗證最終路徑是否**起始於**許可根目錄。如果 Agent 試圖透過 `../../../etc/passwd` 逃逸，操作將被立即攔截。
    *   **操作權限：** 在許可的路徑範圍內，Agent **可以執行增加資料夾、刪除檔案等操作**。這確保了 Agent 在具備實用能力的同時，不會對宿主機的核心區域造成危害。
*   **範例：**
    *   **路徑遍歷：** 檢查文件路徑參數是否包含 `..` 或以 `/` 開頭（如果只允許相對路徑）。
    *   **命令注入：** 檢查 shell 命令參數是否包含 `&&`, `|`, `;` 等惡意連接符。可以使用白名單來限制可執行命令的結構。
    *   **類型驗證：** 確保傳入的參數類型與工具定義的 `args` schema 一致。

4.  **用戶確認請求:**
    *   如果策略引擎的回應是 `REQUIRE_USER_CONFIRMATION`，安全層則通過「雙向通信接口」發出 `onToolCallRequest` 事件，將最終決策權交給用戶。

5.  **審計日誌 (Audit Logging):**
    *   **原則：** 每一個經過安全層的決策，無論最終是允許、拒絕還是等待用戶確認，都**必須**被記錄下來。
    *   **日誌內容：** 日誌應至少包含 `時間戳`, `請求主體 (principal)`, `執行的動作 (action)`, `資源 (resource)`, `策略引擎的決策`, `用戶的決策` (如果有), 和 `最終結果 (允許/拒絕)`。
    *   **作用：** 用於事後的安全審計、問題追蹤和系統行為分析。

---

## 特別考量：高風險工具的安全策略

對於像 `shell_interpreter` 或 `python_interpreter` 這樣可以執行任意代碼的工具，安全層及其協同的策略引擎必須有更嚴格的處理機制。

### 1. 強制沙盒 (Mandatory Sandboxing)
*   **首要原則：** `Tool` 插件的**實現本身**必須強制使用沙盒。安全層不應該信任工具的實現，但可以通過某種機制來驗證該工具是否聲明並運行在沙盒模式下。
*   **策略配置：** 「策略與安全護欄引擎」應包含規則，規定只有被標記為「沙盒化」的代碼執行工具才能被允許調用。

### 2. 細粒度策略規則 (Fine-Grained Policy Rules)
策略引擎應針對這些高風險工具定義極其細粒度的規則，例如：
*   **命令白名單/黑名單：**
    *   `ALLOW` `shell:execute` if command starts with `ls`, `cat`, `echo`.
    *   `DENY` `shell:execute` if command contains `rm -rf`, `mv`, `sudo`.
*   **網絡訪問限制：**
    *   `DENY` `python:execute` if code attempts to make outbound network calls, unless destination is in `[api.example.com]`.
*   **文件系統訪問權限：**
    *   `DENY` `python:execute` if code attempts to read files outside of its designated `/workspace` 目錄。
*   **資源限制：**
    *   `DENY` `python:execute` if execution time exceeds 30 seconds or memory usage exceeds 512MB.

### 3. 增強的用戶確認
當策略引擎決定需要用戶確認時 (`REQUIRE_USER_CONFIRMATION`)，發送給 UI 的 `onToolCallRequest` 事件應包含更豐富的警告信息。
*   **數據結構：**
  ```json
  {
    "toolName": "shell:execute",
    "args": { "command": "rm important_file.txt" },
    "confirmationId": "...",
    "security_warning": {
      "level": "CRITICAL",
      "message": "The agent is attempting to execute a command that deletes a file. This action may be irreversible."
    }
  }
  ```
*   **UI 行為：** UI 插件在收到帶有 `security_warning` 的請求時，應以更醒目的方式（例如，紅色高亮的對話框）向用戶展示警告，並可能要求用戶進行二次確認（例如，輸入「我確認」）。
