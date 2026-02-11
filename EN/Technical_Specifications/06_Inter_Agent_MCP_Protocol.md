# 06. Inter-Agent MCP Protocol Specification

This document defines the communication standards between OpenStarry Agents, as well as between Agents and the external world. We fully adopt and extend the **Model Context Protocol (MCP)**.

## 1. Architectural Roles

Within an MCP network, an OpenStarry Agent can simultaneously assume two roles:

*   **MCP Server:** Exposes Tools and Resources to external consumers. For example, a "Database Agent" exposes SQL query tools for other Agents to use.
*   **MCP Client:** Connects to other Agents and invokes their capabilities.

---

## 2. Discovery & Handshake

### 2.1 Service Declaration
An Agent declares its capabilities through standard JSON-RPC 2.0 messages:

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

### 2.2 Tool Exposure
All registered tools within an Agent are **not** exposed by default. They must be explicitly exported in `agent.json`:

```json
"mcp": {
  "expose_tools": ["fs:read_file", "git:status"], // Only expose these two
  "private_tools": ["admin:reset"] // Hide administrative tools
}
```

---

## 3. Inter-Agent Call Flow

Assume Agent A (Client) calls a tool on Agent B (Server):

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
    Agent B's core receives the request, treats it as an `EXEC:TOOL_CALL` event, and executes the corresponding internal plugin.

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

## 4. Recursion Guard

In multi-agent collaboration scenarios (e.g., A -> B -> C -> A), infinite recursive calls can easily occur.

### Solution: Trace Context Propagation
All MCP requests must include an HTTP Header or Metadata field `X-OpenStarry-Trace-ID`.
The core maintains a call chain depth counter, and when the depth exceeds 5 levels, it automatically trips the circuit breaker and rejects the request.
