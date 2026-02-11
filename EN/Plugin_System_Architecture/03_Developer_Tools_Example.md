# Plugin Example: Developer Tools

This document outlines the design concepts for advanced tool plugins related to code development, such as code search and LSP.

---

### 1. Code Search (`code:search`)

*   **Plugin Type:** `tool`
*   **Responsibility:** Performs fast and precise code searches within the Agent's workspace.
*   **`plugin.json` Example:**
    ```json
    {
      "name": "Code-Search",
      "type": "tool",
      "entryPoint": "./CodeSearchTool.js"
    }
    ```
*   **Implementation Strategy:**
    1.  The `execute` method in `CodeSearchTool.js` receives parameters such as the search `pattern` and optional file type `glob`.
    2.  It invokes a high-performance command-line tool, such as `ripgrep (rg)`, on the backend.
    3.  For security, this invocation should also be executed within a container via the `SandboxManager` to ensure the search scope is strictly limited to the current session's workspace.
    4.  A summary of the search results is returned to the LLM.

---

### 2. LSP (Language Server Protocol)

Implementing LSP support is complex as it involves managing long-running processes, making it a **stateful tool**.

*   **Plugin Type:** A toolset containing multiple related `tool`s.
*   **Responsibility:** Empowers the Agent with the ability to understand code semantics (e.g., jump to definition, retrieve type information, auto-completion).
*   **Implementation Strategy:**
    1.  **Backend `LspProcessManager`**: A backend module is required to manage the lifecycle of language server processes. it can start and maintain a corresponding language server process (e.g., `typescript-language-server`) for each project or session.
    2.  **`lsp:start` Tool**:
        *   Function: Starts an LSP service for a specified language for the current project on the backend.
        *   Returns: A `server_id` for subsequent interactions.
    3.  **`lsp:request` Tool**:
        *   Function: Sends a standard LSP request to a started LSP service.
        *   Parameters: `{ server_id: string, method: string, params: object }`. For example, `method: 'textDocument/definition'`, `params: { textDocument: { uri: '...' }, position: { ... } }`.
        *   Implementation: The tool locates the corresponding server process via the `LspProcessManager`, communicates with it through standard input/output (stdin/stdout), sends JSON-RPC requests, and waits for responses.
    4.  **`lsp:stop` Tool**:
        *   Function: Shuts down a running LSP service.

This design places stateful process management on the backend while providing the LLM with only a series of stateless, callable `Tool` interfaces.
