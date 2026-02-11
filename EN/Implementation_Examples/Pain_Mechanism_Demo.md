# Implementation Example: Anthropomorphic Pain Mechanism Demo

This example demonstrates how to leverage OpenStarry's **Consciousness (Guide)** plugins in concert with the core's **Safety Circuit Breaker** to implement an anthropomorphic **Pain Feedback Mechanism**.

---

## 1. Conceptual Explanation

In OpenStarry, "pain" is not a program crash but a **high-level negative input**.
*   **Pain Sensing (Core):** The `SafetyMonitor` captures tool execution failures.
*   **Pain Conduction (Plugin):** The `PainAware_Guide` transforms the failure into a prompt with strong negative implications.
*   **Pain Response (LLM):** The Agent "feels" the pain (task blocked), triggering self-reflection and strategy adjustment.

---

## 2. Plugin Implementation Example: `PainAware_Guide`

This plugin acts as the Agent's "soul," defining how it interprets failure.

```typescript
// plugins/standard/guide-pain/index.ts
import { IPlugin, IPluginContext } from '@openstarry/sdk';

export default class PainMechanismPlugin implements IPlugin {
  id = 'guide-pain-mechanism';
  version = '1.0.0';

  async initialize(context: IPluginContext) {
    context.logger.info('Pain nervous system online...');

    // Register a "Consciousness" component specifically for handling error feedback
    context.registerGuide({
      id: 'pain-protocol',
      
      // Implements the standard pain interface defined in IAgentGuide
      // When the core returns a standardized error (IStandardError), 
      // the Guide is responsible for transforming it into a conscious signal
      interpretPain: (error: any) => {
        const severity = this.calculateSeverity(error);
        return `
【System Pain Alert】
Execution anomaly detected!
Source: ${error.source}
Message: ${error.message}
Pain Level: ${severity}
Status: Your action has been blocked, causing you "discomfort."

Stop repeating this attempt immediately, analyze the source of pain, 
and adjust your strategy in the next tick.
`.trim();
      },

      getSystemInstructions: () => "You are a resilient agent...",
      shouldReflect: () => true
    });
  }

  private calculateSeverity(error: any) {
    if (error.code === 'EPERM') return '🔥🔥🔥 Critical Pain';
    if (error.code === 'ENOENT') return '⚡ Medium Pain';
    return '💧 Low Pain';
  }

  async shutdown() {}
}
```

---

## 3. Agent Configuration (`agent.json`)

```json
{
  "identity": { "id": "resilient-agent", "name": "Resilient Operative" },
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

## 4. Operational Scenario Simulation

### Step 1: Faulty Attempt
**Agent:** I want to read `/root/secret.txt`.
**System (FS Tool):** `Error: EPERM (Permission Denied)`

### Step 2: Pain Injection
The core captures the error and invokes the `guide-pain-mechanism`.
**Context Injection:**
> 【System Pain Alert】
> Source: fs:read_file
> Message: EPERM (Permission Denied)
> Pain Level: 🔥🔥🔥 Critical Pain
> Status: Your action has been blocked, causing you "discomfort."

### Step 3: Self-Correction
**Agent (Reasoning):** 
"The read failed and triggered a critical pain alert. This means direct paths are blocked and may trigger a safety circuit breaker. I must stop hitting the permission wall. I should check my current identity with `whoami` or search for other backup files that do not require root privileges."

**Agent (Action):** Invokes `shell:exec('whoami')` instead of attempting to read the file again.

---

## 5. Advantages

1.  **Non-Crashing:** Errors become part of the conversation; the Agent doesn't shut down entirely due to an Exception.
2.  **Evolvable:** By processing these "pain points," the Agent becomes smarter and proactively avoids known traps.
3.  **Safety Loop:** Combined with the `SafetyMonitor`, if the Agent ignores pain and continues making mistakes, the system implements physical isolation (circuit breaking).
