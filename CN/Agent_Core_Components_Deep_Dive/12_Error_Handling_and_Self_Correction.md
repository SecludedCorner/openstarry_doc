# 12. 错误处理与自我修正 (Error Handling & Self-Correction)

本文档定义了 OpenStarry 核心 (Core) 中的异常拦截与反馈机制。核心的职责是确保系统的稳定性，并将异常转化为可供上层插件（如 Guide）解析的标准化数据。

## 核心机制：异常拦截层 (Exception Interceptor Layer)

在 Agent Core 的执行回路中，必须存在一个显式的拦截层，用于捕获所有插件执行与内部处理过程中的异常。核心不对异常进行感性诠释，仅进行技术性标准化。

### 1. 错误的标准化 (Normalization)

Core 负责捕获底层异常（如 `ConnectionRefusedError`）并将其封装为标准的 `ToolExecutionResult` 对象。这确保了无论底层发生何种错误，上层接收到的数据格式始终一致。

```typescript
type ToolExecutionResult = {
  success: boolean;
  data?: any;
  error?: {
    code: string;       // 标准化错误代码，例如 "EPERM", "ENOENT"
    message: string;    // 原始错误信息
    source: string;     // 引发错误的组件 ID
    suggestion?: string // 系统级的技术建议
  };
};
```

### 2. 反馈回路流程 (The Feedback Loop)

1.  **Action Attempt:** 执行回路调用工具插件。
2.  **Execution Failure:** 插件执行失败或抛出异常。
3.  **Interception:** Core 捕获异常，提取特征，转化为 `error` 对象。
4.  **Cognitive Injection:** Core 将此标准化对象封装为一条 `ToolMessage` 注入 Context。
5.  **Upstream Interpretation:** 由加载的 **识蕴 (Guide)** 插件决定如何向 LLM 诠释这条信息（例如将其转化为负面反馈或重新规划指令）。

---

## 错误分类学 (Taxonomy of Errors)

核心根据错误的性质将其分类，以决定不同的系统级行为：

### Type A: 逻辑与执行异常 (Execution Anomalies)
*   **定义：** 工具调用参数错误、找不到路径、权限不足。
*   **核心行为：** 将错误数据化并反馈给 LLM 处理。

### Type B: 瞬态环境异常 (Transient Environmental Anomalies)
*   **定义：** 网络超时、API 限流、资源暂时不可用。
*   **核心行为：** 执行自动重试机制 (Backoff Retry)。

### Type C: 致命系统崩溃 (Fatal System Failures)
*   **定义：** 内存溢出、核心逻辑 Bug、安全熔断触发。
*   **核心行为：** 立即终止进程并触发守护进程 (Daemon) 报警。

---

## 防御性机制：挫折计数器 (Frustration Counter)

为了防止执行回路陷入无效的重复尝试，核心内建了技术性的保护措施：

1.  Core 维护一个计数器，记录连续失败的 Tool 调用次数。
2.  如果计数器超过预设阈值（如 5 次），核心将触发安全干预。
3.  **干预动作：** 强制在队列中插入系统级指令，要求 Agent 停止当前路径。

> **注记：** 核心仅提供「连续失败」这一事实的侦测，关于「挫折感」或「痛觉」的拟人化诠释，应完全由外部 Guide 插件实现。
