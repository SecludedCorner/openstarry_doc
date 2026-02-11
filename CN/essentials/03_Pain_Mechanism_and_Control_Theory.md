# OpenStarry Agent 如何学习：痛觉机制与控制理论

> *"错误是学习的机会。"*

## 错误处理的问题

大多数 AI Agent 框架把错误当成崩溃来处理。某个东西失败了，Agent 就死掉或盲目重试，用户重新启动。没有学习，没有适应，没有对出错的记忆。

OpenStarry 采取了一种受生物学启发的根本性不同做法：**错误即是痛觉信号**。

## 痛觉机制

### 生物痛觉如何运作

当你碰到烫炉子时，你不会宕机。你的神经系统发送痛觉信号，你的大脑解读它（「那很烫，那很痛」），然后你把手抽回来。下次，你会用不同的方式靠近炉子。痛觉不是 Bug——它是生物学中最重要的反馈机制。

### OpenStarry 痛觉如何运作

三个层次，对应生物神经系统：

```
1. Pain Sensing (Core)       — SafetyMonitor 捕捉工具执行失败
        ↓
2. Pain Conduction (Plugin)  — Guide 插件将失败转化为有意义的反馈
        ↓
3. Pain Response (LLM)       — Agent「感受」痛觉，反思，调整策略
```

### 具体情境

Agent 尝试读取受限的文件：

**第一步——行动：** Agent 对 `/root/secret.txt` 调用 `fs.read`

**第二步——失败：** 系统返回 `Error: EPERM (Permission Denied)`

**第三步——痛觉注入：** Core 调用 Guide 插件的 `interpretPain()`：

```typescript
interpretPain: (error) => {
  const severity = calculateSeverity(error);
  return `
【System Pain Alert】
Execution anomaly detected!
Source: ${error.source}
Message: ${error.message}
Pain Level: ${severity}
Status: Your action has been blocked. This causes "discomfort."

Stop repeating this attempt. Analyze the pain source.
Adjust your strategy in the next tick.
  `;
};

calculateSeverity(error) {
  if (error.code === 'EPERM') return '🔥🔥🔥 Critical Pain';
  if (error.code === 'ENOENT') return '⚡ Medium Pain';
  return '💧 Low Pain';
}
```

**第四步——作为工具结果消息注入上下文：**

> 【System Pain Alert】
> Source: fs.read
> Message: EPERM (Permission Denied)
> Pain Level: 🔥🔥🔥 Critical Pain
> Status: Your action has been blocked.

**第五步——自我修正：** LLM 处理这个痛觉信号并响应：

> 「读取失败，触发严重痛觉警报。这表示直接路径被禁止，可能触发安全断路器。我必须停止撞击权限墙。我应该先检查我的许可路径，并寻找替代方案。」

### 三种痛觉类型

| 类型 | 示例 | Agent 响应 | 系统响应 |
|------|------|-----------|---------|
| **执行痛觉** | 参数错误、文件不存在、权限被拒 | 反思，调整做法 | 痛觉注入上下文 |
| **暂态痛觉** | 网络超时、API 频率限制 | 等待并重试 | 指数退避自动重试 |
| **致命痛觉** | 内存不足、安全违规、错误级联 | 无法自我修正 | 程序终止 + Daemon 警报 |

### 挫折计数器

如果 Agent 忽视痛觉，持续犯相同的错误呢？

SafetyMonitor 使用 **SHA-256 指纹识别**来侦测重复的失败：

```typescript
// 工具名称 + 参数的哈希 = 唯一指纹
function fingerprint(toolName: string, argsJson: string): string {
  return createHash("sha256")
    .update(`${toolName}:${argsJson}`)
    .digest("hex")
    .slice(0, 16);
}
```

**升级阶梯：**

| 连续失败次数 | 响应 |
|-------------|------|
| 1-2 | 正常痛觉反馈——Agent 有机会自我修正 |
| 3（相同指纹） | **系统警报注入**："SYSTEM ALERT: You are repeating a failed action. STOP and analyze why." |
| 5 | **挫折阈值**——系统强制暂停命令 |
| 10 次操作中有 8 次错误 | **错误级联侦测**——`EMERGENCY_HALT`，状态 → `ERROR_PAUSED` |

这对应了生物的挫折感：重复的痛觉信号从「哎呀」升级到「停下来想想」再到「你需要帮助」。

### 事实与意义的分离

一个关键的设计原则：**Core 提供失败的事实。Guide 插件提供意义。**

Core 说：`Tool "fs.write" failed with EPERM at /etc/passwd.`
Guide 解读：「你尝试写入一个系统文件。这造成了严重痛觉。停下来重新考虑。」

这种分离意味着不同的 Guide 插件可以对相同的错误给出不同的解读：
- **安全导向**的 Agent 将 EPERM 视为违规——「你不应该尝试这个。」
- **学习型** Agent 将其视为探索边界——「这条路不通。让我们找另一条。」
- **除错型** Agent 将其视为诊断数据——「侦测到权限问题。检查文件所有权。」

Core 保持纯净。意义是插件的职责。

## 多层级断路器

除了痛觉之外，OpenStarry 还实现了受工业断路器启发的三层安全系统：

### 第一级：资源限制（资源级）

在每次操作前强制执行的硬性限制：

| 资源 | 默认限制 | 触发后果 |
|------|---------|---------|
| Token 预算 | 每个 Agent 100,000 tokens | 强制终止循环，进入 `STOPPED` 状态 |
| 循环迭代次数 | 每个任务 50 ticks | 「侦测到无限循环」→ 暂停等待人为介入 |
| 工具超时 | 每次执行 30 秒 | `Promise.race()` 终止工具调用 |

```typescript
// 每次 LLM 调用前：
const tokenCheck = safetyMonitor.beforeLLMCall();
if (tokenCheck.halt) {
  setState("SAFETY_LOCKOUT");
  return; // 预算耗尽——不再思考
}
```

### 第二级：行为分析（行为级）

启发式侦测问题行为模式：

- **重复工具调用**：相同指纹 + 失败 × 3 = 注入警报
- **错误级联**：滑动视窗 10 次操作中 80% 错误率 = `EMERGENCY_HALT`
- **输出异常**：连续无效 JSON 或调用不存在的工具 = 升级处理

### 第三级：人为覆写（指令级）

终极开关。`SYSTEM_HALT` 事件被标记为 **Priority 0**（最高优先级）。即使队列中有 100 个待处理任务，暂停命令也会在下一次循环迭代开始时立即被处理：

```
EXECUTING --[limit reached / anomaly]--> SAFETY_LOCKOUT
SAFETY_LOCKOUT --["admin:unlock"]--> WAITING_FOR_EVENT
```

Agent 只能通过明确的人为介入才能解锁。

## Agent 即控制系统

除了痛觉和安全机制之外，OpenStarry 将整个 Agent 建模为一个**反馈控制系统**——与设计自动驾驶、恒温器和工业机器人所用的同一套数学框架。

### 控制回路

```
                    ┌───────────────────────────────────┐
                    │                                   │
  User Goal ──────► │  Error = Goal - Current State     │
  (reference)       │          ↓                        │
                    │     Controller (LLM)              │
                    │          ↓                        │
                    │     Control Input (Tool Calls)    │
                    │          ↓                        │
                    │     Plant (External World)        │
                    │          ↓                        │
                    │     Sensor (Tool Results)   ──────┘
                    │          ↓
                    │     Measured Output (current state)
                    └───────────────────────────────────┘
```

### 对应关系

| 控制理论 | OpenStarry 组件 | 具体范例 |
|---------|----------------|---------|
| 参考输入 (r) | System Prompt + 用户消息 | 「找出并修复 auth.ts 中的 Bug」 |
| 控制器 (C) | LLM | Gemini 2.0 Flash 分析问题 |
| 控制输入 (u) | 工具调用 | `fs.read("auth.ts")`、`fs.write("auth.ts", fixedCode)` |
| 受控对象 (P) | 外部世界 | 文件系统、代码库 |
| 传感器 (H) | 工具结果 | 文件内容、错误消息、测试输出 |
| 误差信号 (e) | 上下文差距 | 「Bug 仍然存在」→ 继续迭代 |

### 三个稳定性问题（与解方）

**1. 振荡**——Agent 在两个状态之间来回跳动（撤销/重做循环）
- *成因：*过度反应或误导性的传感器数据
- *解方：*上下文历史作为**积分项**——Agent 记住过去的尝试，避免重蹈覆辙。滑动视窗保持近期历史可见。

**2. 发散**——Agent 偏离原始目标
- *成因：*上下文充满噪声，原始意图被淹没
- *解方：***上下文锚定**——System Prompt 在上下文组装中拥有最高权重，即使滑动视窗丢弃较旧的消息，System Prompt 也永远不会被修剪。

**3. 稳态误差**——Agent 自认为完成了，但实际上没有
- *成因：*验证不足（传感器盲点）
- *解方：***验证步骤**——在宣布完成前强制进行检查。如同 PID 的微分项，测量的是误差的*变化率*，而非仅仅是误差本身。

### 关键洞察

> **智慧不仅仅在于拥有强大的 LLM。它在于反馈回路的质量。**

一个搭配优秀反馈（详细的工具结果、准确的痛觉信号、适当的上下文管理）的普通 LLM，会胜过搭配糟糕反馈（被吞掉的错误、没有上下文、没有验证）的顶尖 LLM。

这就是为什么 OpenStarry 在以下方面投入大量心力：
- **错误标准化**——每次失败都产生一致的、可解析的输出
- **痛觉解读**——Guide 插件赋予错误意义
- **上下文管理**——可插拔的策略决定 LLM 看到什么
- **安全监控**——行为分析防止病态回路

LLM 只是其中一个组件——控制器。反馈回路才是整个系统。

> *"当工具调用失败，Core 不会崩溃，而是将错误信息作为「感官输入」反馈给 LLM。"*
