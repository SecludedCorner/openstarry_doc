# 09. 可観測性とトレーシング (Observability & Tracing)

このドキュメントでは、 OpenStarry システムにおいて標準化されたログ (Logging)、指標 (Metrics)、およびリンク追跡 (Tracing) をどのように実現するかを定義します。これはマルチエージェント協調システムのデバッグにおいて極めて重要です。

## 1. リンク追跡 (Tracing)

マルチエージェント環境では、1つのユーザーリクエストが複数のエージェント間の MCP 通信を誘発することがあります。私たちはリクエストチェーン全体を追跡できなければなりません。

### 1.1 Trace Context (伝送プロトコル)
システムは一意の識別子として `trace_id` を使用します。

*   **TraceID の生成**: リクエストがシステムに入った最初の瞬間に、 `Orchestrator Daemon` または `Gateway` によって生成されます。
*   **伝送方法**: MCP メッセージパケットの `metadata` フィールドに入れて運ばれます。
    ```json
    {
      "source": "master_agent",
      "target": "worker_007",
      "metadata": {
        "trace_id": "tr-xxxx-yyyy-zzzz",
        "parent_span_id": "span-123"
      },
      "payload": { ... }
    }
    ```
*   **伝播の責任**: エージェントコアは `trace_id` を持つイベントを受信した際、その ID を現在実行中の `Execution Loop` コンテキストにバインドし、出力されるログやその後に送信されるメッセージにその ID を透過させなければなりません。

---

## 2. 標準化されたログ (Structured Logging)

システムは、マシンによる解析や自動監視を容易にするために、 **JSON 形式の構造化ログ** の使用を強制します。

### 2.1 ログフィールドの仕様
各ログには以下が含まれるべきです：
*   `timestamp`: ISO8601 形式。
*   `level`: `DEBUG`, `INFO`, `WARN`, `ERROR` 。
*   `agent_id`: 現在のエージェント ID。
*   `trace_id`: 関連するリンク追跡 ID。
*   `module`: `Kernel`, `PluginLoader`, `SafetyMonitor` など。
*   `msg`: 人間が読めるメッセージ。
*   `context`: (任意) 追加のキー・バリューデータ。

### 2.2 重要なイベントの記録ポイント
*   **LLM 呼び出し**: プロンプトの要約、 Token 消費量、遅延を記録。
*   **ツール実行**: ツール名、引数 (マスキング済み)、実行結果を記録。
*   **状態遷移**: ステートマシンの A から B への切り替えを記録。
*   **遮断トリガー**: 遮断の原因と現在のカウンター値を記録。

---

## 3. 指標の収集 (Metrics)

システムの健全性とコストを監視するために使用されます。

*   **コスト指標 (Cost)**: `total_tokens_used`, `cost_usd` （モデルごとの課金）。
*   **性能指標 (Performance)**: `llm_latency_ms`, `tool_execution_time_ms` 。
*   **健全性指標 (Health)**: `loop_error_rate`, `active_agents_count`, `mcp_message_latency` 。

---

## 4. 実装メカニズム：すべてはプラグインである (Plugin-based Observability)

エージェントコアはログサーバーや Prometheus に直接接続しません。 **`logger` プラグイン** を介して出力を行います。

*   **`ConsoleLoggerPlugin`**: JSON ログを stdout/stderr に出力します（デーモンが収集を担当）。
*   **`FileLoggerPlugin`**: ログを特定のプロジェクトディレクトリに書き込みます。
*   **`OpenTelemetryPlugin`**: トレースとメトリクスをバックエンドの分析プラットフォーム（ Jaeger, Grafana Tempo など）に送信します。

---

## 5. まとめ

TraceID による全リンクの貫通と JSON 構造化ログにより、以下を実現します。
1.  **迅速なデバッグ**: 膨大なログの中から、あるタスクのすべてのステップを正確に抽出できます。
2.  **コスト監視**: 各エージェント、各タスクの費用をリアルタイムで把握できます。
3.  **ボトルネック分析**: どのツールやどのエージェントの反応が最も遅いかを特定できます。
