# 深度解析：安全层

本文档深入探讨「无头代理人核心」的安全边界——安全层的实现机制。

## 核心职责

安全层作为核心与外部世界交互的「守门员」，其唯一职责是在 `Tool` 被执行**之前**，拦截调用请求，并根据预设策略进行处理，以确保代理人的行为符合安全与合规要求。

---

## 实现机制：一个多级决策流

1.  **拦截点：** 安全层位于「执行循环」中，在 LLM 解析出工具调用请求之后、实际执行工具之前。

2.  **与策略引擎的交互 (API Contract):**
    *   安全层的第一步是向外部的「策略与安全护栏引擎」发起一次同步的权限检查请求。
    *   **请求体 (Request Body):**
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
    *   **响应体 (Response Body):**
        ```json
        {
          "decision": "ALLOW" | "DENY" | "REQUIRE_USER_CONFIRMATION",
          "reason": "Command 'ls' is in the whitelist.",
          "obligations": [ /* 例如：需要在执行后记录日志 */ ]
        }
        ```
### 3. 参数清洗与验证 (Argument Sanitization & Validation):
*   **在执行之前**，即使策略引擎和用户都已批准，安全层也应执行最后一道检查。
*   **路径限制机制 (Path Scoping - 核心安全特性):**
    *   **定义：** 针对 `fs` (文件系统) 类工具，系统会强制执行「根路径限制」。Agent 被赋予一个或多个「许可根目录」(Permitted Roots，例如 `./workspace/`)。
    *   **路径规范化 (Normalization):** 安全层会先将 Agent 传入的相对路径或绝对路径转化为绝对规范路径，并移除所有 `..` (Parent directory) 引用。
    *   **边界检查 (Boundary Check):** 系统会验证最终路径是否**起始于**许可根目录。如果 Agent 试图通过 `../../../etc/passwd` 逃逸，操作将被立即拦截。
    *   **操作权限：** 在许可的路径范围内，Agent **可以执行增加文件夹、删除文件等操作**。这确保了 Agent 在具备实用能力的同时，不会对宿主机的核心区域造成危害。
*   **示例：**
    *   **路径遍历：** 检查文件路径参数是否包含 `..` 或以 `/` 开头（如果只允许相对路径）。
    *   **命令注入：** 检查 shell 命令参数是否包含 `&&`, `|`, `;` 等恶意连接符。可以使用白名单来限制可执行命令的结构。
    *   **类型验证：** 确保传入的参数类型与工具定义的 `args` schema 一致。

4.  **用户确认请求:**
    *   如果策略引擎的回应是 `REQUIRE_USER_CONFIRMATION`，安全层则通过「双向通信接口」发出 `onToolCallRequest` 事件，将最终决策权交给用户。

5.  **审计日志 (Audit Logging):**
    *   **原则：** 每一个经过安全层的决策，无论最终是允许、拒绝还是等待用户确认，都**必须**被记录下来。
    *   **日志内容：** 日志应至少包含 `时间戳`, `请求主体 (principal)`, `执行的动作 (action)`, `资源 (resource)`, `策略引擎的决策`, `用户的决策` (如果有), 和 `最终结果 (允许/拒绝)`。
    *   **作用：** 用于事后的安全审计、问题追踪和系统行为分析。

---

## 特别考量：高风险工具的安全策略

对于像 `shell_interpreter` 或 `python_interpreter` 这样可以执行任意代码的工具，安全层及其协同的策略引擎必须有更严格的处理机制。

### 1. 强制沙盒 (Mandatory Sandboxing)
*   **首要原则：** `Tool` 插件的**实现本身**必须强制使用沙盒。安全层不应该信任工具的实现，但可以通过某种机制来验证该工具是否声明并运行在沙盒模式下。
*   **策略配置：** 「策略与安全护栏引擎」应包含规则，规定只有被标记为「沙盒化」的代码执行工具才能被允许调用。

### 2. 细粒度策略规则 (Fine-Grained Policy Rules)
策略引擎应针对这些高风险工具定义极其细粒度的规则，例如：
*   **命令白名单/黑名单：**
    *   `ALLOW` `shell:execute` if command starts with `ls`, `cat`, `echo`.
    *   `DENY` `shell:execute` if command contains `rm -rf`, `mv`, `sudo`.
*   **网络访问限制：**
    *   `DENY` `python:execute` if code attempts to make outbound network calls, unless destination is in `[api.example.com]`.
*   **文件系统访问权限：**
    *   `DENY` `python:execute` if code attempts to read files outside of its designated `/workspace` 目录。
*   **资源限制：**
    *   `DENY` `python:execute` if execution time exceeds 30 seconds or memory usage exceeds 512MB.

### 3. 增强的用户确认
当策略引擎决定需要用户确认时 (`REQUIRE_USER_CONFIRMATION`)，发送给 UI 的 `onToolCallRequest` 事件应包含更丰富的警告信息。
*   **数据结构：**
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
*   **UI 行为：** UI 插件在收到带有 `security_warning` 的请求时，应以更醒目的方式（例如，红色高亮的对话框）向用户展示警告，并可能要求用户进行二次确认（例如，输入「我确认」）。
