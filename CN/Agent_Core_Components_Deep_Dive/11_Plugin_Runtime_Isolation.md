# 11. 插件运行时隔离与沙箱机制 (Plugin Runtime Isolation)

本文档定义了 OpenStarry 系统如何隔离插件的执行环境，防止恶意或充满 Bug 的插件导致核心崩溃、数据泄露或资源耗尽。

## 1. 风险模型

*   **稳定性风险**: 插件抛出未捕获异常 (Uncaught Exception) 导致主进程退出。
*   **资源风险**: 插件写了 `while(true)` 死循环或内存泄露 (Memory Leak)。
*   **安全风险**: 插件读取环境变量中的 API Key，或随意删除文件系统中的文件。

## 2. 隔离分级策略 (Isolation Levels)

根据部署环境与安全需求，系统支持三种隔离级别。

### Level 1: 函数级包裹 (Function Wrapping) - [开发/轻量模式]
*   **机制**: 所有插件代码在主进程 (Agent Core) 中运行。
*   **防护**:
    *   **Try-Catch**: `PluginLoader` 强制使用 `try-catch` 包裹插件的 `execute` 方法。
    *   **Timeout**: 使用 `Promise.race` 设置执行超时（例如 10秒），超时则抛出错误。
*   **缺点**: 无法防御 `while(true)` (会卡死 Event Loop)，无法限制文件系统访问。
*   **适用**: 内部受信任插件，开发调试阶段。

### Level 2: 虚拟机沙箱 (VM Sandboxing) - [标准模式]
*   **机制**: 使用 Node.js 的 `vm` 模块或 `vm2` 库来运行插件代码。
*   **防护**:
    *   **Context Isolation**: 插件无法访问全局 `process` 对象，无法读取环境变量。
    *   **Limited Access**: 只能调用 Core 明确注入的 API (如 `log`, `fetch`)。
*   **缺点**: 仍然在同一进程内，无法完全防御 CPU 密集型攻击。
*   **适用**: 运行第三方代码，但信任度尚可的场景。

### Level 3: 进程级隔离 (Process Isolation) - [严格/生产模式]
*   **机制**: **Orchestrator Daemon** 负责为每个插件（或一组插件）启动独立的 OS 子进程 (Child Process) 或 Worker Thread。
*   **通讯**: Core 与 Plugin 之间通过 IPC (Inter-Process Communication) 或标准输入输出 (stdio) 传递 JSON 消息。
*   **防护**:
    *   **资源配额**: Daemon 可以使用 OS 机制 (cgroups, Docker) 限制插件进程的 CPU/RAM。
    *   **物理熔断**: 插件死锁不影响 Core，Daemon 可直接 `kill` 插件进程。
*   **缺点**: 性能开销大，IPC 通讯有延迟。
*   **适用**: 执行高风险代码（如 Code Interpreter），或多租户环境。

### Level 4: WebAssembly (WASM) - [未来演进/极致安全]
*   **机制**: 将插件编译为 WASM 模块，并在 Core 内嵌的 WASM 运行时 (如 Wasmtime) 中执行。
*   **优势**:
    *   **内存安全**: 插件无法访问沙箱外的内存空间。
    *   **能力授权 (Capability-based)**: 通过 WASI 接口极其精细地控制插件的文件与网络访问权限。
    *   **极速启动**: 毫秒级冷启动，远快于进程隔离。
*   **场景**: 高性能计算工具、不信任的第三方商务插件。

---

## 3. 建议实作路径 (Implementation Path)

### MVP 阶段
采用 **Level 1 (Try-Catch + Timeout)**。
*   这足以应对大多数非恶意的 Bug。
*   开发成本最低。

### V1.0 生产阶段
针对 **Code Interpreter** 和 **第三方插件**，必须采用 **Level 3 (独立进程)**。
*   `Infrastructure Plugins` (如本地 Broker) 本质上就是 Level 3 的体现，由 Daemon 独立管理。
*   普通的 `Tool` 插件若涉及敏感操作，应通过 `Plugin Runner` 独立执行。

---

## 4. 权限声明 (Permission Manifest)

每个插件的 `plugin.json` 必须声明所需权限，由用户在安装时授权（类似 Android App）。

```json
{
  "name": "FileSearchTool",
  "permissions": [
    "fs:read:./data",  // 允许读取 data 目录
    "network:none"     // 禁止联网
  ]
}
```

Core 在加载插件时，会根据这些声明构建受限的执行上下文 (Context)。
