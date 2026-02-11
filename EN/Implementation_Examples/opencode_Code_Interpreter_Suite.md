# Implementation Strategy: opencode - Code Interpreter Tool Suite

This document details the internal components and workflow of the "Code Interpreter Tool Suite" designed to achieve the `opencode` functionality.

## Core Design: Sandbox Manager

The core of this tool suite is not a single tool but a shared, backend **Sandbox Manager**. All code execution and file operations must pass through this manager to ensure security and isolation.

*   **Responsibilities:**
    1.  **Lifecycle Management:** Responsible for creating, starting, stopping, and destroying secure execution environments (e.g., a Docker container). Each user session or task requiring isolation should correspond to an independent sandbox instance.
    2.  **Command Execution:** Provides an interface for asynchronously executing commands within a specified sandbox instance.
    3.  **File Interaction:** Provides interfaces for uploading/downloading files between the sandbox instance and the host, or performing file operations directly within the sandbox.

---

## Tool Plugin Design

Based on the `Sandbox Manager`, we can design the following `Tool` plugins:

### 1. `python:execute` Tool

*   **Function:** Executes Python code in an isolated environment.
*   **Implementation Flow:**
    1.  When the `execute` method is invoked, it first retrieves the current `session_id` from the context.
    2.  It requests the `Sandbox Manager` to retrieve or create a sandbox instance for that `session_id`.
    3.  It writes `args.code` into a temporary file within the sandbox (e.g., `/workspace/script.py`).
    4.  If `args.dependencies` are present, it instructs the `Sandbox Manager` to execute `pip install ...` within the sandbox.
    5.  It instructs the `Sandbox Manager` to execute `python /workspace/script.py` within the sandbox.
    6.  It asynchronously waits for the `Sandbox Manager` to return the execution results (`stdout`, `stderr`, `result`), and then returns those results to the Agent Core.

### 2. `shell:execute` Tool

*   **Function:** Executes Shell commands in an isolated environment.
*   **Implementation Flow:**
    1.  Similar to `python:execute`, it first retrieves the `session_id` and locates the corresponding sandbox instance.
    2.  It directly calls the command execution interface of the `Sandbox Manager`, passing in `args.command`.
    3.  The `Sandbox Manager` ensures the command is executed within the sandbox's `/workspace` directory using the sandbox's internal Shell.
    4.  Wait for and return the execution results.

### 3. `sandbox_fs:*` Series of Tools

*   **Function:** Operates on the filesystem inside the sandbox.
*   **Implementation Flow:**
    *   **`sandbox_fs:list`**: Internally calls the `Sandbox Manager` to execute the `ls -l` command within the sandbox.
    *   **`sandbox_fs:read`**: Internally calls the `Sandbox Manager` to execute the `cat` command within the sandbox, or uses the file interaction interface to read content directly.
    *   **`sandbox_fs:write`**: Internally uses the file interaction interface to write content to a specified path within the sandbox.

This design centralizes all security-critical sandbox operations within the `Sandbox Manager`, making the implementation of upper-layer `Tool` plugins simple and standardized—they are only responsible for transforming LLM intent into calls to the `Sandbox Manager`.
