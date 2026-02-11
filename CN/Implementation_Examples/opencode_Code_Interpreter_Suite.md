# 实现思路：opencode - 代码解释器工具套件

本文档详细描述了为实现 `opencode` 功能而设计的「代码解释器工具套件」的内部组件和工作流程。

## 核心设计：沙盒管理器 (Sandbox Manager)

此工具套件的核心不是单个的工具，而是一个共享的、后端的**沙盒管理器 (Sandbox Manager)**。所有代码执行和文件操作都必须通过这个管理器来进行，以确保安全性和隔离性。

*   **职责：**
    1.  **生命周期管理：** 负责创建、启动、停止和销毁安全的执行环境（例如，一个 Docker 容器）。每个用户会话或每个需要隔离的任务都应该对应一个独立的沙盒实例。
    2.  **命令执行：** 提供一个接口，用于在指定的沙盒实例中异步地执行命令。
    3.  **文件交互：** 提供接口，用于在沙盒实例和主机之间上传/下载文件，或直接在沙盒内进行文件操作。

---

## Tool 插件设计

基于 `Sandbox Manager`，我们可以设计出以下 `Tool` 插件。

### 1. `python:execute` 工具

*   **功能：** 在一个隔离的环境中执行 Python 代码。
*   **实现流程：**
    1.  当 `execute` 方法被调用时，它首先从上下文中获取当前的 `session_id`。
    2.  它向 `Sandbox Manager` 请求为该 `session_id` 获取或创建一个沙盒实例。
    3.  它将 `args.code` 写入沙盒中的一个临时文件（例如 `/workspace/script.py`）。
    4.  如果 `args.dependencies` 存在，它会让 `Sandbox Manager` 在沙盒中执行 `pip install ...`。
    5.  它让 `Sandbox Manager` 在沙盒中执行 `python /workspace/script.py`。
    6.  它异步地等待 `Sandbox Manager` 返回执行的结果（`stdout`, `stderr`, `result`），然后将结果返回给代理人核心。

### 2. `shell:execute` 工具

*   **功能：** 在隔离的环境中执行 Shell 命令。
*   **实现流程：**
    1.  与 `python:execute` 类似，首先获取 `session_id` 并定位到对应的沙盒实例。
    2.  它直接调用 `Sandbox Manager` 的命令执行接口，传入 `args.command`。
    3.  `Sandbox Manager` 确保该命令在沙盒的 `/workspace` 目录下执行，并且使用的是沙盒内部的 Shell。
    4.  等待并返回执行结果。

### 3. `sandbox_fs:*` 系列工具

*   **功能：** 操作沙盒内部的文件系统。
*   **实现流程：**
    *   **`sandbox_fs:list`**: 内部调用 `Sandbox Manager` 在沙盒中执行 `ls -l` 命令。
    *   **`sandbox_fs:read`**: 内部调用 `Sandbox Manager` 在沙盒中执行 `cat` 命令，或使用文件交互接口直接读取内容。
    *   **`sandbox_fs:write`**: 内部使用文件交互接口将内容写入到沙盒的指定路径。

这种设计将安全关键的沙盒操作全部集中到了 `Sandbox Manager` 中，使得上层的 `Tool` 插件实现变得简单且标准化，它们只负责将 LLM 的意图转化为对 `Sandbox Manager` 的调用。
