# 05. Plugin Interface Definitions

This document defines the abstract interfaces that various plugins need to implement to ensure correct interaction with the "Headless Agent Core." For specific code implementation examples, please refer to the corresponding documents in the `Implementation_Examples/` folder.

---

## 1. UI Plugin Interface

The core of a UI plugin is the implementation of a client for the "bi-directional communication interface." It needs to listen for events from the Core and send user commands to the Core.

### Events from Core -> UI (UI Plugin must listen)

*   `onNewMessage(message: object)`: The Core has generated the final text to be displayed directly to the user.
    *   `message`: `{ content: string, format: 'markdown' | 'text' }`
*   `onToolCallRequest(request: object)`: The Core's "Security Layer" has intercepted a tool call and requires user authorization.
    *   `request`: `{ toolName: string, args: object, confirmationId: string, security_warning?: object }`
*   `onReadyForInput`: Notifies the UI that the Core is ready to receive new input.
*   `onAgentStateChange(state: string)`: Broadcasts changes in the Core's internal state (e.g., `thinking`, `executing_tool`).

### Commands from UI -> Core (UI Plugin must send)

*   `submitUserInput(input: string)`: The user submitted conversational input, triggering the execution loop.
*   `provideConfirmation(confirmationId: string, approved: boolean)`: The user's response to a tool call authorization request.

### Implementation Example

*   Please refer to `Implementation_Examples/UI_Plugin_Example.md`

---

## 2. Tool Plugin Interface

Tool plugins are relatively simple, typically requiring only the export of an object or class containing metadata and an `execute` method.

*   `name: string`: A unique identifier for the tool, used by the LLM for calls.
*   `description: string`: A detailed description of the tool's functionality for the LLM to understand its purpose.
*   `args: object`: Describes the parameters required by the tool, preferably in JSON Schema format.
*   `execute(args: object): Promise<any>`: The core logic for executing the tool.

### Implementation Examples

*   **File System Read Tool:** Please refer to `Implementation_Examples/Tool_ReadFile_Example.md`
*   **Code Interpreter Tool Suite:** Please refer to `Implementation_Examples/Tool_CodeInterpreter_Example.md`

---

## 3. Provider Plugin Interface

Provider plugins are responsible for communicating with specific LLM APIs.

*   `constructor(config: object)`: Initializes the API client.
*   `generate(history: Array<object>, tools: Array<object>): Promise<object>`:
    *   The core method for generating content.
    *   `history`: The conversation history passed in by the Core.
    *   `tools`: A list of descriptions for available tools.
    *   **Return Format:** `{ is_final: boolean, tool_calls?: Array<object>, text_content?: string }`
        *   `is_final`: If true, indicates this is the final response to the user.
        *   `tool_calls`: If the LLM decides to call tools, this contains a list of tool call requests.
        *   `text_content`: If it is a final response, this contains the text content.

### Implementation Example

*   Please refer to `Implementation_Examples/Provider_Gemini_Example.md`

---

## 4. Listener Plugin Interface (Sensation / Vedana)

Listeners are responsible for monitoring signals from the external world and transforming them into events understandable by the Core.

*   `name: string`: A unique identifier for the listener.
*   `onStart(context: IAgentContext): Promise<void>`:
    *   Invoked when the Agent starts.
    *   Start an HTTP Server, WebSocket connection, or mount `process.stdin` in this method.
    *   Use `context.emitInput(data)` to pass received signals to the Core.
*   `onStop(): Promise<void>`:
    *   Invoked when the Agent stops.
    *   Close connections and release resources in this method.

## 5. Guide Plugin Interface (Consciousness / Vijnana)

Guides are responsible for injecting the Agent's soul and consciousness configuration.

*   `systemPrompt: string`: Defines the Agent's core persona, values, and behavioral guidelines.
*   `temperature?: number`: Defines the degree of divergence in thinking.
*   `memoryPolicy?: string`: Specifies the memory strategy to use (e.g., `sliding-window`, `vector-db`).
*   `initialMemories?: string[]`: Injects initial memories to establish context background (Priming).

---

## 6. Tool Plugin Suite Example: Code Interpreter

To achieve functionality similar to `opencode`, the Agent requires a powerful set of tools for code execution and file management. **All tools in this suite must be implemented to run within a secure sandbox environment isolated from the host (such as a Docker container), which is the primary prerequisite for ensuring security.**

### 4.1 Code Execution Tool

```javascript
// Example: A Tool plugin for executing Python code
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
    // Implementation Details:
    // 1. Start a new, clean Docker container.
    // 2. If dependencies are provided, run pip install in the container.
    // 3. Write args.code into a .py file within the container.
    // 4. Execute the .py file.
    // 5. Capture stdout, stderr, and any possible return value.
    // 6. Destroy the container.
    // 7. Return the captured output.
    const result = await this.runInSandbox(args.code, args.dependencies);
    return result;
  }

  async runInSandbox(code, deps) { /* ... sandboxing logic ... */ }
}
```

### 4.2 Sandbox File System Tool

These tools operate on the file system **inside the sandbox** where the code interpreter resides.

```javascript
// Example: A Tool plugin for listing directories inside the sandbox
class Sandbox_FS_List_Tool {
  name = "sandbox_fs:list";
  description = "Lists the files and directories inside the sandbox at a given path.";
  args = {
    path: { type: "string", description: "The path inside the sandbox." }
  };

  async execute(args) {
    // Implementation Details:
    // Execute 'ls -l' or 'dir' in the corresponding sandbox container via 'docker exec' or similar command.
    const result = await this.executeCommandInSandbox(`ls -l ${args.path}`);
    return result.stdout;
  }

  async executeCommandInSandbox(cmd) { /* ... sandboxing logic ... */ }
}
```
*Implementations for other file system tools (`read`, `write`, `mkdir`, `remove`) are similar.*
