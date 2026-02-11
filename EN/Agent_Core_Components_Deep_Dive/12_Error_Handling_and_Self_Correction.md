# 12. Error Handling & Self-Correction

This document defines the exception interception and feedback mechanisms within the OpenStarry Core. The Core's responsibility is to ensure system stability and transform exceptions into standardized data that upper-layer plugins (e.g., Guide) can parse.

## Core Mechanism: Exception Interceptor Layer

Within the Agent Core's execution loop, an explicit interception layer must exist to capture exceptions occurring during plugin execution and internal processing. The Core does not interpret exceptions subjectively; it only performs technical standardization.

### 1. Error Normalization

The Core is responsible for catching low-level exceptions (e.g., `ConnectionRefusedError`) and wrapping them into standardized `ToolExecutionResult` objects. This ensures that the data format received by upper layers remains consistent regardless of the underlying error.

```typescript
type ToolExecutionResult = {
  success: boolean;
  data?: any;
  error?: {
    code: string;       // Standardized error code, e.g., "EPERM", "ENOENT"
    message: string;    // Raw error message
    source: string;     // ID of the component that raised the error
    suggestion?: string // System-level technical suggestion
  };
};
```

### 2. The Feedback Loop Workflow

1.  **Action Attempt:** The execution loop invokes a tool plugin.
2.  **Execution Failure:** The plugin fails to execute or throws an exception.
3.  **Interception:** The Core catches the exception, extracts features, and transforms it into an `error` object.
4.  **Cognitive Injection:** The Core wraps this standardized object as a `ToolMessage` and injects it into the Context.
5.  **Upstream Interpretation:** The loaded **Consciousness (Guide)** plugin determines how to interpret this message for the LLM (e.g., transforming it into negative feedback or re-planning instructions).

---

## Taxonomy of Errors

The core categorizes errors based on their nature to determine different system-level behaviors:

### Type A: Execution Anomalies
*   **Definition:** Tool invocation parameter errors, path not found, insufficient permissions.
*   **Core Behavior:** Data-fies the error and feeds it back to the LLM for processing.

### Type B: Transient Environmental Anomalies
*   **Definition:** Network timeouts, API rate limits, resources temporarily unavailable.
*   **Core Behavior:** Executes an automatic retry mechanism (Backoff Retry).

### Type C: Fatal System Failures
*   **Definition:** Memory overflow, core logic bugs, security circuit breaker triggers.
*   **Core Behavior:** Terminates the process immediately and triggers a daemon alarm.

---

## Defensive Mechanism: Frustration Counter

To prevent the execution loop from falling into futile repetitive attempts, the core includes built-in technical safeguards:

1.  The Core maintains a counter recording the number of consecutive failed tool calls.
2.  If the counter exceeds a preset threshold (e.g., 5 times), the core triggers a safety intervention.
3.  **Intervention Action:** Forcibly inserts a system-level instruction into the queue, requiring the Agent to stop the current path.

> **Note:** The core only provides detection of the fact of "consecutive failures." Anthropomorphic interpretations such as "frustration" or "pain" should be implemented entirely by external Guide plugins.
