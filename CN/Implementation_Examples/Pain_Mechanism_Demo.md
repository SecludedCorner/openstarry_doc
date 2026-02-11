# 实作示例：拟人化痛觉机制 (Pain Mechanism Demo)

本示例展示如何利用 OpenStarry 的「识蕴 (Guide)」插件与核心的「安全断路器 (Safety Circuit Breaker)」协同工作，实现拟人化的**痛觉反馈机制**。

---

## 1. 概念说明

在 OpenStarry 中，「痛觉」不是一种程序崩溃，而是一种**高等级的负面输入**。
*   **痛觉感应 (Core):** `SafetyMonitor` 捕获工具执行失败。
*   **痛觉传导 (Plugin):** `PainAware_Guide` 将失败转化为带有强烈负面暗示的 Prompt。
*   **痛觉反应 (LLM):** Agent 感觉到「痛」（任务受阻），触发自我反思与策略调整。

---

## 2. 插件实作示例：`PainAware_Guide`

此插件作为 Agent 的「灵魂」，定义了它如何诠释失败。

```typescript
// plugins/standard/guide-pain/index.ts
import { IPlugin, IPluginContext } from '@openstarry/sdk';

export default class PainMechanismPlugin implements IPlugin {
  id = 'guide-pain-mechanism';
  version = '1.0.0';

  async initialize(context: IPluginContext) {
    context.logger.info('痛觉神经系统已上线...');

    // 注册一个「识蕴」组件，专门处理错误反馈
    context.registerGuide({
      id: 'pain-protocol',
      
      // 实现 IAgentGuide 定义的标准痛觉接口
      // 当核心回传标准化错误 (IStandardError) 时，Guide 负责将其转化为意识信号
      interpretPain: (error: any) => {
        const severity = this.calculateSeverity(error);
        return `
【系统痛觉警报】
侦测到执行异常！
来源：${error.source}
讯息：${error.message}
痛感等级：${severity}
状态：你的行为受到了阻碍，这让你感到「不适」。

请立即停止重复尝试，分析痛觉来源，并在下一个 Tick 中调整你的策略。
`.trim();
      },

      getSystemInstructions: () => "You are a resilient agent...",
      shouldReflect: () => true
    });
  }

  private calculateSeverity(error: any) {
    if (error.code === 'EPERM') return '🔥🔥🔥 剧痛 (Critical)';
    if (error.code === 'ENOENT') return '⚡ 刺痛 (Medium)';
    return '💧 微痛 (Low)';
  }

  async shutdown() {}
}
```

---

## 3. Agent 配置 (`agent.json`)

```json
{
  "identity": { "id": "resilient-agent", "name": "具备韧性的特工" },
  "plugins": [
    { "name": "@openstarry-plugin/standard-function-fs" },
    { "name": "@openstarry-plugin/guide-pain-mechanism" }
  ],
  "policy": {
    "safety": {
      "max_consecutive_errors": 3,
      "pain_threshold": "medium"
    }
  }
}
```

---

## 4. 运行场景模拟 (Scenario)

### Step 1: 错误尝试
**Agent:** 我要读取 `/root/secret.txt`。
**System (FS Tool):** `Error: EPERM (Permission Denied)`

### Step 2: 痛觉注入 (Pain Injection)
核心捕获错误，调用 `guide-pain-mechanism`。
**Context 注入:**
> 【系统痛觉警报】
> 来源：fs:read_file
> 讯息：EPERM (Permission Denied)
> 痛感等级：🔥🔥🔥 剧痛 (Critical)
> 状态：你的行为受到了阻碍，这让你感到「不适」。

### Step 3: 自我修正 (Self-Correction)
**Agent (思考中):** 
「读取失败且触发了剧痛警报。这意味着直接路径是行不通的，而且可能触发安全熔断。我必须停止冲撞权限墙。我应该先检查我目前的身份 `whoami`，或者寻找其他无需 root 权限的备份文件。」

**Agent (行动):** 调用 `shell:exec('whoami')` 而不是再次尝试读取文件。

---

## 5. 优势

1.  **非崩溃性 (Non-Crashing):** 错误变成了对话的一部分，Agent 不会因为一个 Exception 就彻底挂掉。
2.  **演化性 (Evolvable):** Agent 通过处理这些「痛点」，变得越来越聪明，能主动避开已知陷阱。
3.  **安全闭环 (Safety Loop):** 结合 `SafetyMonitor`，如果 Agent 不理会痛觉持续犯错，系统将实施物理隔离（熔断）。
