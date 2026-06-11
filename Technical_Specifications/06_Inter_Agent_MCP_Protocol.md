# 06. 多代理協作協議規範 (Inter-Agent MCP Protocol)

本文件定義了 OpenStarry Agent 之間，以及 Agent 與外部世界之間的通訊標準。我們完全採用並擴展了 **Model Context Protocol (MCP)**。

## 1. 架構角色 (Architectural Roles)

在 MCP 網路中，OpenStarry Agent 可以同時扮演兩種角色：

*   **MCP Server:** 對外暴露工具 (Tools) 與資源 (Resources)。例如，一個「資料庫 Agent」暴露 SQL 查詢工具給其他 Agent 使用。
*   **MCP Client:** 連接到其他 Agent，調用其能力。

---

## 2. 發現與握手 (Discovery & Handshake)

### 2.1 服務宣告
Agent 透過標準 JSON-RPC 2.0 消息宣告其能力：

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
Agent 內部的所有已註冊工具，預設**不會**全部暴露。必須在 `agent.json` 中顯式導出：

```json
"mcp": {
  "expose_tools": ["fs:read_file", "git:status"], // 只暴露這兩個
  "private_tools": ["admin:reset"] // 隱藏管理工具
}
```

---

## 3. 跨代理調用流程 (Inter-Agent Call Flow)

假設 Agent A (Client) 呼叫 Agent B (Server) 的工具：

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
    Agent B 的核心接收到請求，將其視為一個 `EXEC:TOOL_CALL` 事件，執行對應的內部插件。

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

## 4. 嵌套遞歸保護 (Recursion Guard)

在多代理協作（如 A -> B -> C -> A）中，容易發生無限遞歸調用。

### 解決方案：Trace Context 透傳
所有 MCP 請求必須包含 HTTP Header 或 Metadata `X-OpenStarry-Trace-ID`。
核心維護一個調用鏈計數器，當深度超過 5 層時，自動熔斷並拒絕請求。
