# How OpenStarry Agents Learn: Pain Mechanism & Control Theory

> *"Errors are learning opportunities."*

## The Problem with Error Handling

Most AI agent frameworks treat errors as crashes. Something fails, the agent dies or retries blindly, the user restarts. There's no learning, no adaptation, no memory of what went wrong.

OpenStarry takes a fundamentally different approach inspired by biology: **errors are pain signals**.

## The Pain Mechanism

### How Biological Pain Works

When you touch a hot stove, you don't crash. Your nervous system sends a pain signal, your brain interprets it ("that's hot, that hurts"), and you pull your hand away. Next time, you approach the stove differently. Pain is not a bug — it's the most important feedback mechanism in biology.

### How OpenStarry Pain Works

Three layers, mirroring the biological nervous system:

```
1. Pain Sensing (Core)       — SafetyMonitor captures tool execution failure
        ↓
2. Pain Conduction (Plugin)  — Guide plugin transforms failure into meaningful feedback
        ↓
3. Pain Response (LLM)       — Agent "feels" pain, reflects, adjusts strategy
```

### A Concrete Scenario

The agent tries to read a restricted file:

**Step 1 — Action:** Agent calls `fs.read` on `/root/secret.txt`

**Step 2 — Failure:** System returns `Error: EPERM (Permission Denied)`

**Step 3 — Pain Injection:** The Core calls the Guide plugin's `interpretPain()`:

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

**Step 4 — Injected into context as a tool result message:**

> 【System Pain Alert】
> Source: fs.read
> Message: EPERM (Permission Denied)
> Pain Level: 🔥🔥🔥 Critical Pain
> Status: Your action has been blocked.

**Step 5 — Self-correction:** The LLM processes this pain signal and responds:

> "Read failed with critical pain alert. This means the direct path is forbidden and may trigger a safety breaker. I must stop hitting the permission wall. I should first check my allowed paths and find an alternative approach."

### Three Types of Pain

| Type | Example | Agent Response | System Response |
|------|---------|---------------|-----------------|
| **Execution Pain** | Wrong parameters, file not found, permission denied | Reflects, adjusts approach | Pain injected into context |
| **Transient Pain** | Network timeout, API rate limit | Waits and retries | Auto-retry with exponential backoff |
| **Fatal Pain** | Out of memory, safety breach, error cascade | Cannot self-correct | Process termination + Daemon alert |

### The Frustration Counter

What if the agent ignores pain and keeps making the same mistake?

The SafetyMonitor uses **SHA-256 fingerprinting** to detect repetitive failures:

```typescript
// Hash of tool name + arguments = unique fingerprint
function fingerprint(toolName: string, argsJson: string): string {
  return createHash("sha256")
    .update(`${toolName}:${argsJson}`)
    .digest("hex")
    .slice(0, 16);
}
```

**Escalation ladder:**

| Consecutive Failures | Response |
|---------------------|----------|
| 1-2 | Normal pain feedback — agent has a chance to self-correct |
| 3 (same fingerprint) | **System Alert injected**: "SYSTEM ALERT: You are repeating a failed action. STOP and analyze why." |
| 5 | **Frustration threshold** — system forces a pause command |
| 8 errors in 10 operations | **Error cascade detected** — `EMERGENCY_HALT`, state → `ERROR_PAUSED` |

This mirrors biological frustration: repeated pain signals escalate from "ouch" to "stop and think" to "you need help."

### Separation of Fact and Meaning

A critical design rule: **Core provides failure facts. Guide plugins provide meaning.**

The Core says: `Tool "fs.write" failed with EPERM at /etc/passwd.`
The Guide interprets: "You tried to write to a system file. This causes critical pain. Stop and reconsider."

This separation means different Guide plugins can give the same error different interpretations:
- A **security-focused** agent treats EPERM as a violation — "You should never attempt this."
- A **learning agent** treats it as an exploration boundary — "This path is not available. Let's find another."
- A **debugging agent** treats it as diagnostic data — "Permission issue detected. Check file ownership."

The Core stays pure. Meaning is a plugin concern.

## The Multi-Level Circuit Breaker

Beyond pain, OpenStarry implements a three-level safety system inspired by industrial circuit breakers:

### Level 1: Resource Limits

Hard limits enforced before every operation:

| Resource | Default Limit | What Happens |
|----------|--------------|--------------|
| Token Budget | 100,000 tokens per agent | Force-terminate loop, enter `STOPPED` state |
| Loop Iterations | 50 ticks per task | "Infinite loop detected" → pause for human intervention |
| Tool Timeout | 30 seconds per execution | `Promise.race()` kills the tool call |

```typescript
// Before every LLM call:
const tokenCheck = safetyMonitor.beforeLLMCall();
if (tokenCheck.halt) {
  setState("SAFETY_LOCKOUT");
  return; // Budget exhausted — no more thinking
}
```

### Level 2: Behavioral Analysis

Heuristic detection of problematic patterns:

- **Repetitive Tool Call**: Same fingerprint + failure × 3 = inject alert
- **Error Cascade**: 80% error rate in sliding window of 10 operations = `EMERGENCY_HALT`
- **Output Anomaly**: Consecutive invalid JSON or non-existent tool calls = escalation

### Level 3: Human Override

The kill switch. A `SYSTEM_HALT` event is marked **Priority 0** (highest). Even if there are 100 pending tasks in the queue, the halt is processed immediately at the next loop iteration start:

```
EXECUTING --[limit reached / anomaly]--> SAFETY_LOCKOUT
SAFETY_LOCKOUT --["admin:unlock"]--> WAITING_FOR_EVENT
```

The agent can only be unlocked by explicit human intervention.

## Agent as Control System

Beyond pain and safety, OpenStarry models the entire agent as a **feedback control system** — the same mathematical framework used to design autopilots, thermostats, and industrial robots.

### The Control Loop

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

### The Mapping

| Control Theory | OpenStarry Component | Concrete Example |
|---------------|---------------------|------------------|
| Reference Input (r) | System Prompt + User Message | "Find and fix the bug in auth.ts" |
| Controller (C) | LLM | Gemini 2.0 Flash analyzing the problem |
| Control Input (u) | Tool Calls | `fs.read("auth.ts")`, `fs.write("auth.ts", fixedCode)` |
| Plant (P) | External world | The filesystem, the codebase |
| Sensor (H) | Tool Results | File contents, error messages, test output |
| Error Signal (e) | Context gap | "Bug still exists" → keeps iterating |

### Three Stability Problems (and Solutions)

**1. Oscillation** — Agent loops between two states (undo/redo cycle)
- *Cause:* Over-response or misleading sensor data
- *Solution:* Context history acts as **integral term** — the agent remembers past attempts and avoids repeating them. The sliding window keeps recent history visible.

**2. Divergence** — Agent drifts away from the original goal
- *Cause:* Context fills with noise, original intent gets buried
- *Solution:* **Context anchoring** — System Prompt has the highest weight in context assembly and is never pruned, even when the sliding window drops older messages.

**3. Steady-State Error** — Agent thinks it's done, but it's not
- *Cause:* Insufficient verification (sensor blindness)
- *Solution:* **Verification step** — force a check before claiming completion. Like a PID derivative term, measure the *rate of change* of the error, not just the error itself.

### The Key Insight

> **Intelligence is not just about having a powerful LLM. It's about the quality of the feedback loop.**

A mediocre LLM with excellent feedback (detailed tool results, accurate pain signals, proper context management) will outperform a brilliant LLM with poor feedback (swallowed errors, no context, no verification).

This is why OpenStarry invests so heavily in:
- **Error standardization** — every failure produces consistent, parseable output
- **Pain interpretation** — Guide plugins give errors meaning
- **Context management** — pluggable strategies for what the LLM sees
- **Safety monitoring** — behavioral analysis prevents pathological loops

The LLM is just one component — the controller. The feedback loop is the whole system.

> *"When a tool call fails, the Core doesn't crash — it treats the error as 'sensory input' fed back to the LLM."*
