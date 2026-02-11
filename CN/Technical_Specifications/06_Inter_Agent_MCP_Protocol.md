# 06. 多代理协作协议规范 (Inter-Agent MCP Protocol)

本文件定义了 OpenStarry Agent 之间，以及 Agent 与外部世界之间的通信标准。我们完全采用并扩展了 **Model Context Protocol (MCP)**。

## 1. 架构角色 (Architectural Roles)

在 MCP 网络中，OpenStarry Agent 可以同时扮演两种角色：

*   **MCP Server:** 对外暴露工具 (Tools) 与资源 (Resources)。例如，一个「数据库 Agent」暴露 SQL 查询工具给其他 Agent 使用。
*   **MCP Client:** 连接到其他 Agent，调用其能力。

---

## 2. 发现与握手 (Discovery & Handshake)

### 2.1 服务宣告
Agent 通过标准 JSON-RPC 2.0 消息宣告其能力：

```json
// Response to "initialize"
{
  "protocolVersion": "2024-11-05",
  "capabilities": {
    "tools": { "listChanged": true },
    "resources": { "listChanged": true }
  },
  "serverInfo": {
    "name": "openstarry-coder",
    "version": "1.0.0"
  }
}
```

### 2.2 工具暴露 (Tool Exposure)
Agent 内部的所有已注册工具，预设**不会**全部暴露。必须在 `agent.json` 中显式导出：

```json
"mcp": {
  "expose_tools": ["fs:read_file", "git:status"], // 只暴露这两个
  "private_tools": ["admin:reset"] // 隐藏管理工具
}
```

---

## 3. 跨代理调用流程 (Inter-Agent Call Flow)

假设 Agent A (Client) 呼叫 Agent B (Server) 的工具：

1.  **Request (A -> B):**
    ```json
    {
      "jsonrpc": "2.0",
      "method": "tools/call",
      "params": {
        "name": "fs:read_file",
        "arguments": { "path": "README.md" }
      },
      "id": 1
    }
    ```

2.  **Execution (B):**
    Agent B 的核心接收到请求，将其视为一个 `EXEC:TOOL_CALL` 事件，执行对应的内部插件。

3.  **Response (B -> A):**
    ```json
    {
      "jsonrpc": "2.0",
      "result": {
        "content": [{ "type": "text", "text": "# Project Readme..." }]
      },
      "id": 1
    }
    ```

---

## 4. 嵌套递归保护 (Recursion Guard)

在多代理协作（如 A -> B -> C -> A）中，容易发生无限递归调用。

### 解决方案：Trace Context 透传
所有 MCP 请求必须包含 HTTP Header 或 Metadata `X-OpenStarry-Trace-ID`。
核心维护一个调用链计数器，当深度超过 5 层时，自动熔断并拒绝请求。
