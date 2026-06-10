# 插件範例：開發者工具 (Developer Tools)

本文件闡述了與代碼開發相關的高級工具插件的設計思路，如代碼搜索和 LSP。

---

### 1. 代碼搜索 (`code:search`)

*   **插件類型：** `tool`
*   **職責：** 在代理人的工作區內進行快速、精準的代碼搜索。
*   **`plugin.json` 示例：**
    ```json
    {
      "name": "Code-Search",
      "type": "tool",
      "entryPoint": "./CodeSearchTool.js"
    }
    ```
*   **實現思路：**
    1.  `CodeSearchTool.js` 的 `execute` 方法接收搜索模式 `pattern` 和可選的文件類型 `glob` 等參數。
    2.  它在後端調用一個高性能的命令行工具，如 `ripgrep (rg)`。
    3.  為了安全，這個調用也應該通過 `SandboxManager` 在容器內執行，以確保搜索範圍被嚴格限制在當前會話的工作區內。
    4.  返回搜索結果的摘要給 LLM。

---

### 2. LSP (語言服務器協議)

實現 LSP 支持較為複雜，它涉及到管理長時程運行的進程，是一種**有狀態的工具**。

*   **插件類型：** 一個包含多個相關 `tool` 的工具集。
*   **職責：** 讓代理人具備理解代碼語義的能力（如跳轉到定義、獲取類型信息、自動補全等）。
*   **實現思路：**
    1.  **後端 `LspProcessManager`:** 需要一個後端模塊來管理語言服務器的進程。它可以為每個項目/會話啟動並維護一個對應的語言服務器進程（如 `typescript-language-server`）。
    2.  **`lsp:start` Tool:**
        *   功能：在後端為當前項目啟動一個指定語言的 LSP 服務。
        *   返回：一個 `server_id`，用於後續的交互。
    3.  **`lsp:request` Tool:**
        *   功能：向一個已啟動的 LSP 服務發送一個標準的 LSP 請求。
        *   參數：`{ server_id: string, method: string, params: object }`。例如 `method: 'textDocument/definition'`, `params: { textDocument: { uri: '...' }, position: { ... } }`。
        *   實現：該工具通過 `LspProcessManager` 找到對應的服務器進程，通過標準輸入/輸出 (stdin/stdout) 與之通信，發送 JSON-RPC 請求並等待響應。
    4.  **`lsp:stop` Tool:**
        *   功能：關閉一個正在運行的 LSP 服務。

這種設計將有狀態的進程管理放在後端，而提供給 LLM 的只是一系列無狀態的、可調用的 `Tool` 接口。
