# 插件示例：开发者工具 (Developer Tools)

本文档阐述了与代码开发相关的高级工具插件的设计思路，如代码搜索和 LSP。

---

### 1. 代码搜索 (`code:search`)

*   **插件类型：** `tool`
*   **职责：** 在代理人的工作区内进行快速、精准的代码搜索。
*   **`plugin.json` 示例：**
    ```json
    {
      "name": "Code-Search",
      "type": "tool",
      "entryPoint": "./CodeSearchTool.js"
    }
    ```
*   **实现思路：**
    1.  `CodeSearchTool.js` 的 `execute` 方法接收搜索模式 `pattern` 和可选的文件类型 `glob` 等参数。
    2.  它在后端调用一个高性能的命令行工具，如 `ripgrep (rg)`。
    3.  为了安全，这个调用也应该通过 `SandboxManager` 在容器内执行，以确保搜索范围被严格限制在当前会话的工作区内。
    4.  返回搜索结果的摘要给 LLM。

---

### 2. LSP (语言服务器协议)

实现 LSP 支持较为复杂，它涉及到管理长时程运行的进程，是一种**有状态的工具**。

*   **插件类型：** 一个包含多个相关 `tool` 的工具集。
*   **职责：** 让代理人具备理解代码语义的能力（如跳转到定义、获取类型信息、自动补全等）。
*   **实现思路：**
    1.  **后端 `LspProcessManager`:** 需要一个后端模块来管理语言服务器的进程。它可以为每个项目/会话启动并维护一个对应的语言服务器进程（如 `typescript-language-server`）。
    2.  **`lsp:start` Tool:**
        *   功能：在后端为当前项目启动一个指定语言的 LSP 服务。
        *   返回：一个 `server_id`，用于后续的交互。
    3.  **`lsp:request` Tool:**
        *   功能：向一个已启动的 LSP 服务发送一个标准的 LSP 请求。
        *   参数：`{ server_id: string, method: string, params: object }`。例如 `method: 'textDocument/definition'`, `params: { textDocument: { uri: '...' }, position: { ... } }`。
        *   实现：该工具通过 `LspProcessManager` 找到对应的服务器进程，通过标准输入/输出 (stdin/stdout) 与之通信，发送 JSON-RPC 请求并等待响应。
    4.  **`lsp:stop` Tool:**
        *   功能：关闭一个正在运行的 LSP 服务。

这种设计将有状态的进程管理放在后端，而提供给 LLM 的只是二系列无状态的、可调用的 `Tool` 接口。
