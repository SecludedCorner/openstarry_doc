# 實作範例：擬人化痛覺機制 (Pain Mechanism Demo)

本範例展示如何利用 OpenStarry 的「識蘊 (Guide)」插件與核心的「安全斷路器 (Safety Circuit Breaker)」協同工作，實現擬人化的**痛覺回饋機制**。

---

## 1. 概念說明

在 OpenStarry 中，「痛覺」不是一種程式崩潰，而是一種**高等級的負面輸入**。
*   **痛覺感應 (Core):** `SafetyMonitor` 捕獲工具執行失敗。
*   **痛覺傳導 (Plugin):** `PainAware_Guide` 將失敗轉化為帶有強烈負面暗示的 Prompt。
*   **痛覺反應 (LLM):** Agent 感覺到「痛」（任務受阻），觸發自我反思與策略調整。

---

## 2. 插件實作範例：`PainAware_Guide`

此插件作為 Agent 的「靈魂」，定義了它如何詮釋失敗。

```typescript
// plugins/standard/guide-pain/index.ts
import { IPlugin, IPluginContext } from '@openstarry/sdk';

export default class PainMechanismPlugin implements IPlugin {
  id = 'guide-pain-mechanism';
  version = '1.0.0';

  async initialize(context: IPluginContext) {
    context.logger.info('痛覺神經系統已上線...');

    // 註冊一個「識蘊」組件，專門處理錯誤回饋
    context.registerGuide({
      id: 'pain-protocol',
      
      // 實現 IAgentGuide 定義的標準痛覺介面
      // 當核心回傳標準化錯誤 (IStandardError) 時，Guide 負責將其轉化為意識訊號
      interpretPain: (error: any) => {
        const severity = this.calculateSeverity(error);
        return `
【系統痛覺警報】
偵測到執行異常！
來源：${error.source}
訊息：${error.message}
痛感等級：${severity}
狀態：你的行為受到了阻礙，這讓你感到「不適」。

請立即停止重複嘗試，分析痛覺來源，並在下一個 Tick 中調整你的策略。
`.trim();
      },

      getSystemInstructions: () => "You are a resilient agent...",
      shouldReflect: () => true
    });
  }

  private calculateSeverity(error: any) {
    if (error.code === 'EPERM') return '🔥🔥🔥 劇痛 (Critical)';
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
  "identity": { "id": "resilient-agent", "name": "具備韌性的特工" },
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

## 4. 運行場景模擬 (Scenario)

### Step 1: 錯誤嘗試
**Agent:** 我要讀取 `/root/secret.txt`。
**System (FS Tool):** `Error: EPERM (Permission Denied)`

### Step 2: 痛覺注入 (Pain Injection)
核心捕獲錯誤，調用 `guide-pain-mechanism`。
**Context 注入:**
> 【系統痛覺警報】
> 來源：fs:read_file
> 訊息：EPERM (Permission Denied)
> 痛感等級：🔥🔥🔥 劇痛 (Critical)
> 狀態：你的行為受到了阻礙，這讓你感到「不適」。

### Step 3: 自我修正 (Self-Correction)
**Agent (思考中):** 
「讀取失敗且觸發了劇痛警報。這意味著直接路徑是行不通的，而且可能觸發安全熔斷。我必須停止衝撞權限牆。我應該先檢查我目前的身分 `whoami`，或者尋找其他無需 root 權限的備份文件。」

**Agent (行動):** 調用 `shell:exec('whoami')` 而不是再次嘗試讀取文件。

---

## 5. 優勢

1.  **非崩潰性 (Non-Crashing):** 錯誤變成了對話的一部分，Agent 不會因為一個 Exception 就徹底掛掉。
2.  **演化性 (Evolvable):** Agent 透過處理這些「痛點」，變得越來越聰明，能主動避開已知陷阱。
3.  **安全閉環 (Safety Loop):** 結合 `SafetyMonitor`，如果 Agent 不理會痛覺持續犯錯，系統將實施物理隔離（熔斷）。
