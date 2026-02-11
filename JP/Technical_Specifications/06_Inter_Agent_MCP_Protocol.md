# 06. マルチエージェント協調プロトコル仕様 (Inter-Agent MCP Protocol)

本ドキュメントは、OpenStarry Agent 間、および Agent と外部世界との間の通信標準を定義します。我々は **Model Context Protocol (MCP)** を完全に採用し、拡張しています。

## 1. アーキテクチャロール (Architectural Roles)

MCP ネットワークにおいて、OpenStarry Agent は同時に二つのロールを担うことができます：

*   **MCP Server:** ツール (Tools) とリソース (Resources) を外部に公開します。例えば、「データベース Agent」が SQL クエリツールを他の Agent に公開します。
*   **MCP Client:** 他の Agent に接続し、その機能を呼び出します。

---

## 2. ディスカバリーとハンドシェイク (Discovery & Handshake)

### 2.1 サービス宣言
Agent は標準の JSON-RPC 2.0 メッセージを通じてその機能を宣言します：

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

### 2.2 ツール公開 (Tool Exposure)
Agent 内部のすべての登録済みツールは、デフォルトでは全て公開**されません**。`agent.json` で明示的にエクスポートする必要があります：

```json
"mcp": {
  "expose_tools": ["fs:read_file", "git:status"], // この二つだけ公開
  "private_tools": ["admin:reset"] // 管理ツールを非公開
}
```

---

## 3. エージェント間呼び出しフロー (Inter-Agent Call Flow)

Agent A（クライアント）が Agent B（サーバー）のツールを呼び出す場合を想定します：

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
    Agent B のコアがリクエストを受信し、これを `EXEC:TOOL_CALL` イベントとして扱い、対応する内部プラグインを実行します。

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

## 4. ネスト再帰ガード (Recursion Guard)

マルチエージェント協調（例：A -> B -> C -> A）において、無限再帰呼び出しが容易に発生する可能性があります。

### 解決策：トレースコンテキストの伝播
すべての MCP リクエストには HTTP ヘッダーまたはメタデータ `X-OpenStarry-Trace-ID` を含める必要があります。
コアは呼び出しチェーンの深さカウンターを維持し、深さが 5 レベルを超えた場合、自動的にサーキットブレーカーを発動してリクエストを拒否します。
