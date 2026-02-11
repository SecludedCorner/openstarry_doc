# 09. 可观测性与链路追踪 (Observability & Tracing)

本文档定义了 OpenStarry 系统中，如何实现标准化日志 (Logging)、指标 (Metrics) 与链路追踪 (Tracing)，这对于调试多代理人协作系统至关重要。

## 1. 链路追踪 (Tracing)

在多代理人环境下，一个用户请求可能触发多个代理人之间的 MCP 通讯。我们必须能够追踪整个请求链。

### 1.1 Trace Context (传递协议)
系统使用 `trace_id` 作为唯一标识符。

*   **TraceID 生成**: 由 `Orchestrator Daemon` 或 `Gateway` 在请求进入系统的第一刻生成。
*   **传递方式**: 在 MCP 消息封包的 `metadata` 字段中携带。
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
*   **传播责任**: Agent Core 收到带有 `trace_id` 的事件时，必须将该 ID 绑定到当前执行的 `Execution Loop` 上下文中，并在产出的日志与后续发送的消息中透传该 ID。

---

## 2. 标准化日志 (Structured Logging)

系统强制使用 **JSON 格式的结构化日志**，以便于机器解析与自动化监控。

### 2.1 日志字段规范
每条日志应包含：
*   `timestamp`: ISO8601 格式。
*   `level`: `DEBUG`, `INFO`, `WARN`, `ERROR`。
*   `agent_id`: 当前代理人 ID。
*   `trace_id`: 关联的链路追踪 ID。
*   `module`: `Kernel`, `PluginLoader`, `SafetyMonitor` 等。
*   `msg`: 人类可读的消息。
*   `context`: (选填) 额外的键值对数据。

### 2.2 关键事件记录点
*   **LLM 调用**: 记录 Prompt 摘要、Token 消耗、延迟。
*   **工具执行**: 记录工具名称、参数 (脱敏)、执行结果。
*   **状态切换**: 记录状态机从 A 到 B 的切换。
*   **熔断触发**: 记录熔断原因与当前计数器值。

---

## 3. 指标收集 (Metrics)

用于监控系统健康度与成本。

*   **成本指标 (Cost)**: `total_tokens_used`, `cost_usd` (按模型计费)。
*   **性能指标 (Performance)**: `llm_latency_ms`, `tool_execution_time_ms`。
*   **健康指标 (Health)**: `loop_error_rate`, `active_agents_count`, `mcp_message_latency`。

---

## 4. 实现机制：一切皆插件 (Plugin-based Observability)

Agent Core 不直接连接日志服务器或 Prometheus。它通过 **`logger` 插件** 进行输出。

*   **`ConsoleLoggerPlugin`**: 将 JSON 日志输出到 stdout/stderr (Daemon 会负责收集)。
*   **`FileLoggerPlugin`**: 将日志写入特定项目目录。
*   **`OpenTelemetryPlugin`**: 将 Trace 与 Metrics 发送到后端分析平台 (如 Jaeger, Grafana Tempo)。

---

## 5. 总结

通过 TraceID 的全链路贯穿与 JSON 结构化日志，我们实现了：
1.  **快速除错**: 可以在海量日志中精确捞出某次任务的所有步骤。
2.  **成本监控**: 即时掌握每个代理人、每个任务的花费。
3.  **效能瓶颈分析**: 找出哪个工具或哪个代理人反应最慢。
