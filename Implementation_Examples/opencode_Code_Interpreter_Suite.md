# 實現思路：opencode - 代碼解釋器工具套件

本文件詳細描述了為實現 `opencode` 功能而設計的「代碼解釋器工具套件」的內部組件和工作流程。

## 核心設計：沙盒管理器 (Sandbox Manager)

此工具套件的核心不是單個的工具，而是一個共享的、後端的**沙盒管理器 (Sandbox Manager)**。所有代碼執行和文件操作都必須通過這個管理器來進行，以確保安全性和隔離性。

*   **職責：**
    1.  **生命週期管理：** 負責創建、啟動、停止和銷毀安全的執行環境（例如，一個 Docker 容器）。每個用戶會話或每個需要隔離的任務都應該對應一個獨立的沙盒實例。
    2.  **命令執行：** 提供一個接口，用於在指定的沙盒實例中異步地執行命令。
    3.  **文件交互：** 提供接口，用於在沙盒實例和主機之間上傳/下載文件，或直接在沙盒內進行文件操作。

---

## Tool 插件設計

基於 `Sandbox Manager`，我們可以設計出以下 `Tool` 插件。

### 1. `python:execute` 工具

*   **功能：** 在一個隔離的環境中執行 Python 代碼。
*   **實現流程：**
    1.  當 `execute` 方法被調用時，它首先從上下文中獲取當前的 `session_id`。
    2.  它向 `Sandbox Manager` 請求為該 `session_id` 獲取或創建一個沙盒實例。
    3.  它將 `args.code` 寫入沙盒中的一個臨時文件（例如 `/workspace/script.py`）。
    4.  如果 `args.dependencies` 存在，它會讓 `Sandbox Manager` 在沙盒中執行 `pip install ...`。
    5.  它讓 `Sandbox Manager` 在沙盒中執行 `python /workspace/script.py`。
    6.  它異步地等待 `Sandbox Manager` 返回執行的結果（`stdout`, `stderr`, `result`），然後將結果返回給代理人核心。

### 2. `shell:execute` 工具

*   **功能：** 在隔離的環境中執行 Shell 命令。
*   **實現流程：**
    1.  與 `python:execute` 類似，首先獲取 `session_id` 並定位到對應的沙盒實例。
    2.  它直接調用 `Sandbox Manager` 的命令執行接口，傳入 `args.command`。
    3.  `Sandbox Manager` 確保該命令在沙盒的 `/workspace` 目錄下執行，並且使用的是沙盒內部的 Shell。
    4.  等待並返回執行結果。

### 3. `sandbox_fs:*` 系列工具

*   **功能：** 操作沙盒內部的文件系統。
*   **實現流程：**
    *   **`sandbox_fs:list`**: 內部調用 `Sandbox Manager` 在沙盒中執行 `ls -l` 命令。
    *   **`sandbox_fs:read`**: 內部調用 `Sandbox Manager` 在沙盒中執行 `cat` 命令，或使用文件交互接口直接讀取內容。
    *   **`sandbox_fs:write`**: 內部使用文件交互接口將內容寫入到沙盒的指定路徑。

這種設計將安全關鍵的沙盒操作全部集中到了 `Sandbox Manager` 中，使得上層的 `Tool` 插件實現變得簡單且標準化，它們只負責將 LLM 的意圖轉化為對 `Sandbox Manager` 的調用。
