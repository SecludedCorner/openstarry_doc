# OpenStarry 代理如何学习：痛觉机制与控制理论

> *"错误是学习的机会。"*

## 错误处理的问题

大多数 AI 代理框架将错误视为崩溃。某个操作失败了，代理要么死掉，要么盲目重试，用户只能重启。没有学习，没有适应，没有对出错原因的记忆。

OpenStarry 采用了一种根本不同的方法，灵感来自生物学：**错误就是痛觉信号**。

## 痛觉机制 (痛觉机制)

### 生物痛觉的工作原理

当你触碰一个热炉子时，你不会崩溃。你的神经系统发送一个痛觉信号，你的大脑解读它（「那很烫，那很痛」），然后你缩回手。下一次，你会以不同的方式接近炉子。痛觉不是 bug——它是生物学中最重要的反馈机制。

### OpenStarry 痛觉的工作原理

三个层次，模拟生物神经系统：

```
1. 痛觉感知 (Core)       — SafetyMonitor 捕获工具执行失败
        ↓
2. 痛觉传导 (Plugin)     — Guide 插件将失败转化为有意义的反馈
        ↓
3. 痛觉响应 (LLM)        — 代理「感受」痛觉，反思，调整策略
```

### 一个具体场景

代理尝试读取一个受限文件：

**步骤 1 — 行动：** 代理对 `/root/secret.txt` 调用 `fs.read`

**步骤 2 — 失败：** 系统返回 `Error: EPERM (Permission Denied)`

**步骤 3 — 痛觉注入：** Core 调用 Guide 插件的 `interpretPain()`：

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

**步骤 4 — 作为工具结果消息注入上下文：**

> 【System Pain Alert】
> Source: fs.read
> Message: EPERM (Permission Denied)
> Pain Level: 🔥🔥🔥 Critical Pain
> Status: Your action has been blocked.

**步骤 5 — 自我修正：** LLM 处理这个痛觉信号并响应：

> "读取失败，触发了严重痛觉警报。这意味着直接路径被禁止，可能触发安全断路器。我必须停止撞击权限墙。我应该先检查我的允许路径并寻找替代方案。"

### 三种痛觉类型

| 类型 | 示例 | 代理响应 | 系统响应 |
|-----|------|---------|---------|
| **执行痛觉** | 参数错误、文件未找到、权限被拒 | 反思，调整方法 | 痛觉注入上下文 |
| **瞬态痛觉** | 网络超时、API 频率限制 | 等待并重试 | 指数退避自动重试 |
| **致命痛觉** | 内存溢出、安全违规、错误级联 | 无法自我修正 | 进程终止 + 守护进程告警 |

### 挫败计数器

如果代理忽略痛觉并持续犯同样的错误怎么办？

SafetyMonitor 使用 **SHA-256 指纹**来检测重复失败：

```typescript
// 工具名 + 参数的哈希 = 唯一指纹
function fingerprint(toolName: string, argsJson: string): string {
  return createHash("sha256")
    .update(`${toolName}:${argsJson}`)
    .digest("hex")
    .slice(0, 16);
}
```

**升级阶梯：**

| 连续失败次数 | 响应 |
|------------|------|
| 1-2 | 正常痛觉反馈——代理有机会自我修正 |
| 3（相同指纹） | **系统警报注入**："SYSTEM ALERT: You are repeating a failed action. STOP and analyze why." |
| 5 | **挫败阈值** — 系统强制暂停命令 |
| 10 次操作中 8 次错误 | **错误级联检测** — `EMERGENCY_HALT`，状态 → `ERROR_PAUSED` |

这模拟了生物挫败感：重复的痛觉信号从「哎呦」升级到「停下来想想」再到「你需要帮助」。

### 事实与意义的分离

一个关键的设计规则：**Core 提供失败事实。Guide 插件提供意义。**

Core 说：`Tool "fs.write" failed with EPERM at /etc/passwd.`
Guide 解读："你试图写入系统文件。这造成了严重痛觉。停下来重新考虑。"

这种分离意味着不同的 Guide 插件可以对同一个错误给出不同的解读：
- 一个**安全导向**的代理将 EPERM 视为违规——"你永远不应该尝试这样做。"
- 一个**学习型**代理将其视为探索边界——"这条路径不可用。让我们找另一条。"
- 一个**调试型**代理将其视为诊断数据——"检测到权限问题。检查文件所有权。"

Core 保持纯净。意义是插件的职责。

## 多级断路器

除了痛觉之外，OpenStarry 还实现了一个受工业断路器启发的三级安全系统：

### 第一级：资源限制 (资源级)

在每次操作前强制执行的硬限制：

| 资源 | 默认限制 | 触发行为 |
|-----|---------|---------|
| Token 预算 | 每个代理 100,000 个 Token | 强制终止循环，进入 `STOPPED` 状态 |
| 循环迭代 | 每个任务 50 个 Tick | "检测到无限循环" → 暂停等待人工干预 |
| 工具超时 | 每次执行 30 秒 | `Promise.race()` 终止工具调用 |

```typescript
// 每次 LLM 调用之前：
const tokenCheck = safetyMonitor.beforeLLMCall();
if (tokenCheck.halt) {
  setState("SAFETY_LOCKOUT");
  return; // 预算耗尽 — 不再思考
}
```

### 第二级：行为分析 (行为级)

对问题模式的启发式检测：

- **重复工具调用**：相同指纹 + 失败 × 3 = 注入警报
- **错误级联**：10 次操作滑动窗口中 80% 错误率 = `EMERGENCY_HALT`
- **输出异常**：连续的无效 JSON 或调用不存在的工具 = 升级

### 第三级：人工覆盖 (指令级)

终极开关。`SYSTEM_HALT` 事件被标记为**优先级 0**（最高）。即使队列中有 100 个待处理任务，停止指令也会在下一次循环迭代开始时立即处理：

```
EXECUTING --[limit reached / anomaly]--> SAFETY_LOCKOUT
SAFETY_LOCKOUT --["admin:unlock"]--> WAITING_FOR_EVENT
```

代理只能通过明确的人工干预来解锁。

## 代理即控制系统

除了痛觉和安全之外，OpenStarry 将整个代理显式建模为一个**反馈控制系统** — 与设计自动驾驶仪、恒温器和工业机器人所使用的数学框架相同。

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

### 映射关系

| 控制理论 | OpenStarry 组件 | 具体示例 |
|---------|----------------|---------|
| 参考输入 (r) | 系统提示词 + 用户消息 | "查找并修复 auth.ts 中的 bug" |
| 控制器 (C) | LLM | Gemini 2.0 Flash 分析问题 |
| 控制输入 (u) | 工具调用 | `fs.read("auth.ts")`, `fs.write("auth.ts", fixedCode)` |
| 被控对象 (P) | 外部世界 | 文件系统、代码库 |
| 传感器 (H) | 工具结果 | 文件内容、错误消息、测试输出 |
| 误差信号 (e) | 上下文差距 | "Bug 仍然存在" → 继续迭代 |

### 三个稳定性问题（及解决方案）

**1. 振荡** — 代理在两个状态之间循环（撤销/重做循环）
- *原因：* 过度响应或误导性传感器数据
- *解决方案：* 上下文历史作为**积分项** — 代理记住过去的尝试并避免重复。滑动窗口保持最近历史可见。

**2. 发散** — 代理偏离原始目标
- *原因：* 上下文被噪声填满，原始意图被淹没
- *解决方案：* **上下文锚定** — 系统提示词在上下文组装中具有最高权重，即使滑动窗口丢弃了较旧的消息也永不被剪除。

**3. 稳态误差** — 代理以为自己完成了，但实际没有
- *原因：* 验证不足（传感器盲区）
- *解决方案：* **验证步骤** — 在声称完成之前强制进行检查。类似于 PID 微分项，测量误差的*变化率*，而不仅仅是误差本身。

### 核心洞察

> **智能不仅仅在于拥有强大的 LLM。它在于反馈回路的质量。**

一个配备优秀反馈（详细的工具结果、准确的痛觉信号、适当的上下文管理）的普通 LLM 将胜过一个配备糟糕反馈（吞掉的错误、没有上下文、没有验证）的出色 LLM。

这就是为什么 OpenStarry 在以下方面投入如此之多：
- **错误标准化** — 每次失败都产生一致的、可解析的输出
- **痛觉解读** — Guide 插件赋予错误意义
- **上下文管理** — 可插拔的策略决定 LLM 看到什么
- **安全监控** — 行为分析防止病理性循环

LLM 只是一个组件——控制器。反馈回路才是整个系统。

> *"当工具调用失败，Core 不会崩溃，而是将错误信息作为『感官输入』反馈给 LLM。"*
