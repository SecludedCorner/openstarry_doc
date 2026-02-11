# 13. 理论视角：作为控制系统的 Agent Core (Agent Core as a Control System)

本文档提供了一个独特的理论视角，运用 **控制理论 (Control Theory)** 的框架来建模与分析 OpenStarry 的 Agent Core 架构。这不仅有助于理解系统的动态特性，更为系统的稳定性设计提供了数学依据。

## 系统模型映射

我们可以将一个正在执行任务的 Agent 视为一个经典的 **闭环反馈控制系统 (Closed-loop Feedback Control System)**。

### 1. 参考输入 (Reference Input, $r$)
*   **定义：** 用户的意图或任务目标。
*   **例子：** "帮我写一份关于 AI 的报告。"
*   **在架构中：** 这是初始的 System Prompt 和 User Message。它是系统希望达到的理想状态。

### 2. 控制器 (Controller, $C$)
*   **定义：** 根据误差信号计算控制输出的组件。
*   **在架构中：** **LLM (大语言模型)**。
*   **职责：** 它观察当前的 Context（状态），与目标进行比对，然后生成下一步的 Action（工具调用）。

### 3. 控制量 (Control Input, $u$)
*   **定义：** 施加于系统的操纵变量。
*   **在架构中：** **Tool Calls (工具调用)**。
*   **例子：** `google_search('AI trends')`, `write_file('report.md')`。

### 4. 被控对象/过程 (Plant/Process, $P$)
*   **定义：** 受到控制量影响并产生输出的环境。
*   **在架构中：** **外部环境** (互联网、文件系统、第三方 API)。
*   **动态性：** 环境是不确定的、充满噪声的（例如网页内容变动、API 延迟）。

### 5. 传感器 (Sensor, $H$)
*   **定义：** 测量被控对象状态并反馈给控制器的组件。
*   **在架构中：** **Tool Outputs (工具执行结果)** 与 **Observer Plugins**。
*   **职责：** 将环境的变化（如文件被写入、搜索结果返回）转化为文本信息，注入 Context。

### 6. 误差信号 (Error Signal, $e$)
*   **定义：** 当前状态与目标状态的差距 ($e = r - y$)。
*   **在架构中：** 这是隐含在 Context 中的。LLM 的核心能力就是**最小化误差**——它不断生成 Action，直到它认为 "Task Complete"（误差趋近于零）。

---

## 系统稳定性分析

从控制理论角度，我们可以分析 Agent 常见的失效模式：

### 1. 震荡 (Oscillation)
*   **现象：** Agent 在两个状态之间反复横跳（例如：写文件 -> 删文件 -> 写文件）。
*   **原因：** 控制器 (LLM) 过度反应，或者传感器 (Tool Output) 提供了误导性信息，导致系统没有收敛。
*   **解决方案：** 引入 **「积分项 (Integral Term)」** 的概念——即 Context History。通过保留历史记忆，Agent 可以「意识到」自己正在重复，从而抑制震荡。

### 2. 发散 (Divergence)
*   **现象：** Agent 的行为越来越离谱，偏离原始目标。
*   **原因：** 误差积累，或者正反馈回路。通常是因为 Context Window 被无关信息填满，导致原始目标 ($r$) 被遗忘。
*   **解决方案：** **Context 锚定 (Context Anchoring)**。始终将原始任务目标 ($r$) 放在 Context 的高权重区域（System Prompt），防止目标漂移。

### 3. 稳态误差 (Steady-State Error)
*   **现象：** Agent 认为自己完成了任务，但用户觉得没完成。
*   **原因：** 传感器精度不足。Agent 无法准确感知其 Action 的真实效果。
*   **解决方案：** 引入 **「验证步骤 (Verification Step)」**。强制 Agent 在宣布完成前，调用一个 `verify_result()` 工具（类似 PID 控制中的微分项预测），检查是否真正达标。

---

## 结论

将 Agent Core 视为控制系统，让我们明白：**智能不仅仅在于 LLM 的 IQ，更在于整个反馈回路的设计质量。** 高质量的传感器（工具反馈）、稳定的控制器（Prompt Engineering）和清晰的目标输入，缺一不可。
