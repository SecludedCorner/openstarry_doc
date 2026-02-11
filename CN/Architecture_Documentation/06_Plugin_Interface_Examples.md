# 05. 插件接口定义 (Plugin Interface Definitions)

本文档定义了各种插件需要实现的抽象接口，以确保它们能与「无头代理人核心」正确交互。具体的代码实现示例，请参考 `Implementation_Examples/` 文件夹下的对应文档。

---

## 1. UI 插件接口

UI 插件的核心是实现「双向通信接口」的客户端。它需要监听来自核心的事件，并向核心发送用户命令。

### 核心 -> UI 的事件 (UI Plugin 需监听)

*   `onNewMessage(message: object)`: 核心生成了最终的、要直接显示给用户的文本。
    *   `message`: `{ content: string, format: 'markdown' | 'text' }`
*   `onToolCallRequest(request: object)`: 核心的「安全层」拦截到一个工具调用，需要用户授权。
    *   `request`: `{ toolName: string, args: object, confirmationId: string, security_warning?: object }`
*   `onReadyForInput`: 通知 UI，核心已准备好接收新输入。
*   `onAgentStateChange(state: string)`: 广播核心内部状态变更 (如 `thinking`, `executing_tool`)。

### UI -> 核心的命令 (UI Plugin 需发送)

*   `submitUserInput(input: string)`: 用户提交了对话输入，触发执行循环。
*   `provideConfirmation(confirmationId: string, approved: boolean)`: 用户对工具调用授权的回应。

### 具体实现示例

*   请参考 `Implementation_Examples/UI_Plugin_Example.md`

---

## 2. Tool 插件接口

Tool 插件相对简单，通常只需要导出一个包含元数据和 `execute` 方法的对象或类。

*   `name: string`: 工具的唯一标识符，供 LLM 调用。
*   `description: string`: 工具功能的详细描述，供 LLM 理解其用途。
*   `args: object`: 描述工具需要的参数，建议使用 JSON Schema 格式。
*   `execute(args: object): Promise<any>`: 执行工具的核心逻辑。

### 具体实现示例

*   **文件系统读取工具：** 请参考 `Implementation_Examples/Tool_ReadFile_Example.md`
*   **代码解释器工具套件：** 请参考 `Implementation_Examples/Tool_CodeInterpreter_Example.md`

---

## 3. Provider 插件接口

Provider 插件负责与具体的 LLM API 进行通信。

*   `constructor(config: object)`: 初始化 API 客户端。
*   `generate(history: Array<object>, tools: Array<object>): Promise<object>`:
    *   生成内容的核心方法。
    *   `history`: 核心传入的对话历史。
    *   `tools`: 可用工具的描述列表。
    *   **返回格式：** `{ is_final: boolean, tool_calls?: Array<object>, text_content?: string }`
        *   `is_final`: 如果为 true，表示这是给用户的最终答复。
        *   `tool_calls`: 如果 LLM 决定调用工具，则包含工具调用请求的列表。
        *   `text_content`: 如果是最终答复，则包含文本内容。

### 具体实现示例

*   请参考 `Implementation_Examples/Provider_Gemini_Example.md`

---

## 4. Listener 插件接口 (受蕴)

Listener 负责监听外部世界的信号，并将其转化为 Core 可理解的事件。

*   `name: string`: 监听器的唯一标识符。
*   `onStart(context: IAgentContext): Promise<void>`:
    *   当 Agent 启动时被调用。
    *   在此方法中启动 HTTP Server、WebSocket 连接或挂载 `process.stdin`。
    *   使用 `context.emitInput(data)` 将接收到的信号传递给 Core。
*   `onStop(): Promise<void>`:
    *   当 Agent 停止时被调用。
    *   在此方法中关闭连接、释放资源。

## 5. Guide 插件接口 (识蕴)

Guide 负责注入 Agent 的灵魂与意识配置。

*   `systemPrompt: string`: 定义 Agent 的核心人设、价值观与行为准则。
*   `temperature?: number`: 定义思考的发散程度。
*   `memoryPolicy?: string`: 指定使用的记忆策略 (如 `sliding-window`, `vector-db`)。
*   `initialMemories?: string[]`: 注入初始记忆，用于建立上下文背景 (Priming)。

---

## 6. Tool 插件套件示例：代码解释器

为了实现类似 `opencode` 的功能，代理人需要一套功能强大的代码执行和文件管理工具。**此套件的所有工具在实现时，都必须在一个安全的、与主机隔离的沙盒环境中（如 Docker 容器）运行，这是保障安全的最高前提。**

### 4.1 代码执行工具

```javascript
// 示例：一个用于执行 Python 代码的 Tool 插件
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
    // 实现细节：
    // 1. 启动一个新的、干净的 Docker 容器。
    // 2. 如果提供了 dependencies，在容器中运行 pip install。
    // 3. 将 args.code 写入容器中的一个 .py 文件。
    // 4. 执行该 .py 文件。
    // 5. 捕获 stdout, stderr, 和可能的返回值。
    // 6. 销毁容器。
    // 7. 返回捕获的输出。
    const result = await this.runInSandbox(args.code, args.dependencies);
    return result;
  }

  async runInSandbox(code, deps) { /* ... sandboxing logic ... */ }
}
```

### 4.2 沙盒文件系统工具

这些工具操作的是代码解释器所在**沙盒内部**的文件系统。

```javascript
// 示例：一个用于列出沙盒内目录的 Tool 插件
class Sandbox_FS_List_Tool {
  name = "sandbox_fs:list";
  description = "Lists the files and directories inside the sandbox at a given path.";
  args = {
    path: { type: "string", description: "The path inside the sandbox." }
  };

  async execute(args) {
    // 实现细节：
    // 通过 'docker exec' 或类似命令在对应的沙盒容器中执行 'ls -l' 或 'dir'
    const result = await this.executeCommandInSandbox(`ls -l ${args.path}`);
    return result.stdout;
  }

  async executeCommandInSandbox(cmd) { /* ... sandboxing logic ... */ }
}
```
*其他文件系统工具 (`read`, `write`, `mkdir`, `remove`) 的实现与此类似。*
