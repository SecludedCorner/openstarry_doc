# 05. 插件接口定義 (Plugin Interface Definitions)

本文件定義了各種插件需要實現的抽象接口，以確保它們能與「無頭代理人核心」正確交互。具體的程式碼實現範例，請參考 `Implementation_Examples/` 資料夾下的對應文件。

---

## 1. UI 插件接口

UI 插件的核心是實現「雙向通信接口」的客戶端。它需要監聽來自核心的事件，並向核心發送用戶命令。

### 核心 -> UI 的事件 (UI Plugin 需監聽)

*   `onNewMessage(message: object)`: 核心生成了最終的、要直接顯示給用戶的文本。
    *   `message`: `{ content: string, format: 'markdown' | 'text' }`
*   `onToolCallRequest(request: object)`: 核心的「安全層」攔截到一個工具調用，需要用戶授權。
    *   `request`: `{ toolName: string, args: object, confirmationId: string, security_warning?: object }`
*   `onReadyForInput`: 通知 UI，核心已準備好接收新輸入。
*   `onAgentStateChange(state: string)`: 廣播核心內部狀態變更 (如 `thinking`, `executing_tool`)。

### UI -> 核心的命令 (UI Plugin 需發送)

*   `submitUserInput(input: string)`: 用戶提交了對話輸入，觸發執行循環。
*   `provideConfirmation(confirmationId: string, approved: boolean)`: 用戶對工具調用授權的回應。

### 具體實現範例

*   請參考 `Implementation_Examples/UI_Plugin_Example.md`

---

## 2. Tool 插件接口

Tool 插件相對簡單，通常只需要導出一個包含元數據和 `execute` 方法的對象或類。

*   `name: string`: 工具的唯一標識符，供 LLM 調用。
*   `description: string`: 工具功能的詳細描述，供 LLM 理解其用途。
*   `args: object`: 描述工具需要的參數，建議使用 JSON Schema 格式。
*   `execute(args: object): Promise<any>`: 執行工具的核心邏輯。

### 具體實現範例

*   **文件系統讀取工具：** 請參考 `Implementation_Examples/Tool_ReadFile_Example.md`
*   **代碼解釋器工具套件：** 請參考 `Implementation_Examples/Tool_CodeInterpreter_Example.md`

---

## 3. Provider 插件接口

Provider 插件負責與具體的 LLM API 進行通信。

*   `constructor(config: object)`: 初始化 API 客戶端。
*   `generate(history: Array<object>, tools: Array<object>): Promise<object>`:
    *   生成內容的核心方法。
    *   `history`: 核心傳入的對話歷史。
    *   `tools`: 可用工具的描述列表。
    *   **返回格式：** `{ is_final: boolean, tool_calls?: Array<object>, text_content?: string }`
        *   `is_final`: 如果為 true，表示這是給用戶的最終答覆。
        *   `tool_calls`: 如果 LLM 決定調用工具，則包含工具調用請求的列表。
        *   `text_content`: 如果是最終答覆，則包含文本內容。

### 具體實現範例

*   請參考 `Implementation_Examples/Provider_Gemini_Example.md`

---

## 4. Listener 插件接口 (受蘊)

Listener 負責監聽外部世界的訊號，並將其轉化為 Core 可理解的事件。

*   `name: string`: 監聽器的唯一標識符。
*   `onStart(context: IAgentContext): Promise<void>`:
    *   當 Agent 啟動時被調用。
    *   在此方法中啟動 HTTP Server、WebSocket 連接或掛載 `process.stdin`。
    *   使用 `context.emitInput(data)` 將接收到的訊號傳遞給 Core。
*   `onStop(): Promise<void>`:
    *   當 Agent 停止時被調用。
    *   在此方法中關閉連接、釋放資源。

## 5. Guide 插件接口 (識蘊)

Guide 負責注入 Agent 的靈魂與意識配置。

*   `systemPrompt: string`: 定義 Agent 的核心人設、價值觀與行為準則。
*   `temperature?: number`: 定義思考的發散程度。
*   `memoryPolicy?: string`: 指定使用的記憶策略 (如 `sliding-window`, `vector-db`)。
*   `initialMemories?: string[]`: 注入初始記憶，用於建立上下文背景 (Priming)。

---

## 6. Tool 插件套件範例：代碼解釋器

為了實現類似 `opencode` 的功能，代理人需要一套功能強大的代碼執行和文件管理工具。**此套件的所有工具在實現時，都必須在一個安全的、與主機隔離的沙盒環境中（如 Docker 容器）運行，這是保障安全的最高前提。**

### 4.1 代碼執行工具

```javascript
// 示例：一個用於執行 Python 代碼的 Tool 插件
class Python_Interpreter_Tool {
  name = "python:execute";
  description = "Executes a block of Python code in a sandboxed environment. The environment has a temporary file system and can install packages if specified. Returns the stdout, stderr, and the final result.";
  args = {
    code: {
      type: "string",
      description: "The Python code to execute."
    },
    dependencies: {
      type: "array",
      description: "An optional list of pip packages to install before execution.",
      items: { type: "string" }
    }
  };

  /**
   * @returns {Promise<{stdout: string, stderr: string, result: any}>}
   */
  async execute(args) {
    // 實現細節：
    // 1. 啟動一個新的、乾淨的 Docker 容器。
    // 2. 如果提供了 dependencies，在容器中運行 pip install。
    // 3. 將 args.code 寫入容器中的一個 .py 文件。
    // 4. 執行該 .py 文件。
    // 5. 捕獲 stdout, stderr, 和可能的返回值。
    // 6. 銷毀容器。
    // 7. 返回捕獲的輸出。
    const result = await this.runInSandbox(args.code, args.dependencies);
    return result;
  }

  async runInSandbox(code, deps) { /* ... sandboxing logic ... */ }
}
```

### 4.2 沙盒文件系統工具

這些工具操作的是代碼解釋器所在**沙盒內部**的文件系統。

```javascript
// 示例：一個用於列出沙盒內目錄的 Tool 插件
class Sandbox_FS_List_Tool {
  name = "sandbox_fs:list";
  description = "Lists the files and directories inside the sandbox at a given path.";
  args = {
    path: { type: "string", description: "The path inside the sandbox." }
  };

  async execute(args) {
    // 實現細節：
    // 通過 'docker exec' 或類似命令在對應的沙盒容器中執行 'ls -l' 或 'dir'
    const result = await this.executeCommandInSandbox(`ls -l ${args.path}`);
    return result.stdout;
  }

  async executeCommandInSandbox(cmd) { /* ... sandboxing logic ... */ }
}
```
*其他文件系統工具 (`read`, `write`, `mkdir`, `remove`) 的實現與此類似。*

