# How OpenStarry Agents Learn: Pain Mechanism & Control Theory

> *"Errors are opportunities to learn."*

## The Problem with Error Handling

Most AI Agent frameworks treat errors as crashes. When something fails, the Agent either dies or retries blindly, and the user has to restart. There is no learning, no adaptation, and no memory of what went wrong.

OpenStarry takes a fundamentally different, biologically-inspired approach: **Errors are signals of pain**.

## The Pain Mechanism

### How Biological Pain Works

When you touch a hot stove, you don't crash. Your nervous system sends a pain signal, your brain interprets it ("That's hot, that hurts"), and you pull your hand back. Next time, you approach the stove differently. Pain is not a bug—it is the most important feedback mechanism in biology.

### How OpenStarry Pain Works

Three levels, corresponding to the biological nervous system:

```
1. Pain Sensing (Core)       — SafetyMonitor captures tool execution failure
        ↓
2. Pain Conduction (Plugin)  — Guide plugin transforms failure into meaningful feedback
        ↓
3. Pain Response (LLM)       — Agent "feels" pain, reflects, and adjusts strategy
```

### Concrete Scenario

An Agent attempts to read a restricted file:

**Step 1 — Action:** The Agent calls `fs.read` on `/root/secret.txt`.

**Step 2 — Failure:** The system returns `Error: EPERM (Permission Denied)`.

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

**Step 4 — Injection as a tool result message into Context:**

> 【System Pain Alert】
> Source: fs.read
> Message: EPERM (Permission Denied)
> Pain Level: 🔥🔥🔥 Critical Pain
> Status: Your action has been blocked.

**Step 5 — Self-correction:** The LLM processes this pain signal and responds:

> "Read failed, triggering critical pain alert. This indicates direct paths are prohibited, likely triggering a safety circuit breaker. I must stop hitting the permission wall. I should check my permitted paths first and find an alternative approach."

### Three Types of Pain

| Type | Example | Agent Response | System Response |
|------|------|-----------|---------|
| **Execution Pain** | Argument errors, file not found, permission denied | Reflect and adjust approach | Pain injected into Context |
| **Transient Pain** | Network timeout, API rate limit | Wait and retry | Automatic exponential backoff retry |
| **Fatal Pain** | Out of memory, safety violation, error cascade | Cannot self-correct | Process termination + Daemon alert |

### Frustration Counter

What if an Agent ignores the pain and keeps making the same mistake?

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

**Escalation Ladder:**

| Consecutive Failures | Response |
|-------------|------|
| 1-2 | Normal pain feedback—Agent has opportunity to self-correct |
| 3 (Same Fingerprint) | **System Alert Injection**: "SYSTEM ALERT: You are repeating a failed action. STOP and analyze why." |
| 5 | **Frustration Threshold**—System forcibly halts the command |
| 8 errors in 10 ops | **Error Cascade Detection**—`EMERGENCY_HALT`, status → `ERROR_PAUSED` |

This corresponds to biological frustration: repetitive pain signals escalate from "ouch" to "stop and think" to "you need help."

### Separation of Fact and Meaning

A key design principle: **The Core provides the facts of failure. The Guide plugin provides the meaning.**

Core says: `Tool "fs.write" failed with EPERM at /etc/passwd.`
Guide interprets: "You tried to write to a system file. This causes severe pain. Stop and reconsider."

This separation means different Guide plugins can interpret the same error differently:
- A **Safety-oriented** Agent views EPERM as a violation—"You should not have tried that."
- A **Learning** Agent views it as exploring boundaries—"This path is blocked. Let's find another."
- A **Debugging** Agent views it as diagnostic data—"Permission issue detected. Check file ownership."

The Core remains pure. Meaning is the plugin's responsibility.

## Multi-level Circuit Breakers

In addition to pain, OpenStarry implements a three-tier safety system inspired by industrial circuit breakers:

### Level 1: Resource Limits (Resource Level)

Hard limits enforced before every operation:

| Resource | Default Limit | Consequence of Trigger |
|------|---------|---------|
| Token Budget | 100,000 tokens per Agent | Forcibly terminates loop, enters `STOPPED` state |
| Loop Iterations | 50 ticks per task | "Infinite loop detected" → Pause for human intervention |
| Tool Timeout | 30 seconds per execution | `Promise.race()` terminates tool call |

```typescript
// Before each LLM call:
const tokenCheck = safetyMonitor.beforeLLMCall();
if (tokenCheck.halt) {
  setState("SAFETY_LOCKOUT");
  return; // Budget exhausted—stop thinking
}
```

### Level 2: Behavioral Analysis (Behavioral Level)

Heuristic detection of problematic behavior patterns:

- **Repetitive Tool Calls**: Same fingerprint + failure × 3 = Alert injection.
- **Error Cascade**: 80% error rate in a sliding window of 10 operations = `EMERGENCY_HALT`.
- **Output Anomalies**: Consecutive invalid JSON or calling non-existent tools = Escalation.

### Level 3: Human Override (Instruction Level)

The ultimate kill switch. the `SYSTEM_HALT` event is marked as **Priority 0** (highest priority). Even if there are 100 pending tasks in the queue, the halt command is processed immediately at the start of the next loop iteration:

```
EXECUTING --[limit reached / anomaly]--> SAFETY_LOCKOUT
SAFETY_LOCKOUT --["admin:unlock"]--> WAITING_FOR_EVENT
```

The Agent can only be unlocked through explicit human intervention.

## Agent as a Control System

Beyond pain and safety mechanisms, OpenStarry models the entire Agent as a **feedback control system**—the same mathematical framework used to design self-driving cars, thermostats, and industrial robots.

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

### Mappings

| Control Theory | OpenStarry Component | Concrete Example |
|---------|----------------|---------|
| Reference Input (r) | System Prompt + User Message | "Find and fix the bug in auth.ts" |
| Controller (C) | LLM | Gemini 2.0 Flash analyzing the problem |
| Control Input (u) | Tool Calls | `fs.read("auth.ts")`, `fs.write("auth.ts", fixedCode)` |
| Plant (P) | External World | File system, codebase |
| Sensor (H) | Tool Results | File content, error messages, test output |
| Error Signal (e) | Context Gap | "Bug still exists" → continue iterating |

### Three Stability Issues (and Solutions)

**1. Oscillation**—Agent bouncing between two states (undo/redo loops).
- *Cause:* Overreaction or misleading sensor data.
- *Solution:* Context history as an **integral term**—Agent remembers past attempts to avoid repeats. Sliding window keeps recent history visible.

**2. Divergence**—Agent drifting from the original goal.
- *Cause:* Context full of noise, drowning out original intent.
- *Solution:***Context Anchoring**—System Prompt has the highest weight in context assembly; it is never trimmed even as older messages are discarded.

**3. Steady-state Error**—Agent thinks it's done, but it isn't.
- *Cause:* Insufficient verification (sensor blind spot).
- *Solution:***Verification Step**—Forced check before declaring completion. Like the derivative term in PID, it measures the *rate of change* of error, not just error itself.

### Key Insight

> **Intelligence is not just about having a powerful LLM. It's about the quality of the feedback loop.**

An average LLM with excellent feedback (detailed tool results, accurate pain signals, proper context management) will outperform a top-tier LLM with poor feedback (swallowed errors, no context, no verification).

This is why OpenStarry invests heavily in:
- **Error Normalization**—consistent, parseable output for every failure.
- **Pain Interpretation**—Guide plugins giving meaning to errors.
- **Context Management**—pluggable strategies determining what the LLM sees.
- **Safety Monitoring**—behavioral analysis preventing pathological loops.

The LLM is just one component—the controller. The feedback loop is the entire system.

> *"When a tool call fails, the Core doesn't crash; it feeds the error message back to the LLM as 'sensory input'."*
