# OpenStarry User Scenario and Workflow Guide

This guide walks you through the core usage scenarios of OpenStarry — from system startup and agent design to interactive conversations and automated management workflows.

---

## 1. System Entry Point: Startup & Monitoring (`openstarry`)

When you type `openstarry` in the terminal, you are launching the **Runtime Dashboard** of this multi-agent operating system.

### Usage Scenario
```bash
$ openstarry
```

### System Behavior
1.  **Auto-connect/Start Daemon:** The system checks whether an `Orchestrator Daemon` is already running in the background. If not, it starts one automatically.
2.  **Enter the Dashboard:** Displays a real-time TUI (Text User Interface) dashboard.

### Visual Experience (Dashboard)
```text
  ___                   _____ _
 / _ \ _ __   ___ _ __ /  ___| |_ __ _ _ __ _ __ _   _
| | | | '_ \ / _ \ '_ \\ `--.| __/ _` | '__| '__| | | |
| |_| | |_) |  __/ | | |`--. \ || (_| | |  | |  | |_| |
 \___/| .__/ \___|_| |_/\____/\__\__,_|_|  |_|   \__, |
      | |                                         __/' |
      |_|                                        |___/

[SYSTEM DASHBOARD] - Connected to Daemon (PID: 14023)
---------------------------------------------------------------------
● System Status:  HEALTHY   |   ● Uptime: 2d 4h 12m
● CPU Usage:      12%       |   ● Memory: 402MB / 8GB
---------------------------------------------------------------------

[RUNNING AGENTS]
ID       NAME            TYPE          STATUS    THOUGHTS/SEC   LAST ACTIVITY
sys-01   Master-Mind     Orchestrator  RUNNING   0.5            Just now
dev-01   Code-Helper     Worker        IDLE      0.0            5m ago
web-01   Search-Bot      Worker        BUSY      1.2            Processing...

[CONTROLS]
(q)uit view  (r)estart daemon  (k)ill agent  (d)esign mode
(a)ttach to agent (select via arrows)
```

---

## 2. Design Mode: Create & Edit (`openstarry design`)

When you need to create new life or modify an existing agent, use design mode. This command is **context-aware**.

### Scenario A: Global Design Mode
*   **Trigger:** Run `openstarry design` outside of any project directory.
*   **Purpose:** This is the "creator's perspective."
    *   **Create New Agent Project:** Hatch a brand new agent project from standard templates (Coding, Writing, Analysis).
    *   **Manage Templates:** Download or manage your agent genome library.
    *   **System Configuration:** Set global API keys or adjust daemon resource limits.

### Scenario B: Project Design Mode
*   **Trigger:** Run `openstarry design` inside an agent project directory.
*   **Purpose:** This is the "operating table" perspective. The system guides you through defining the agent's **Five Aggregates**:
    1.  **Consciousness (Guide):** Inject the soul. Modify the `system_prompt`, set the **persona**, **core objectives**, and **memory strategy**.
    2.  **Form (UI/Body):** Shape the body. Set the agent's **name**, **appearance**, or **interaction interface (UI)**.
    3.  **Sensation (Listeners):** Configure the senses. Set the channels for receiving messages (e.g., `stdio`, `webhook`).
    4.  **Volition (Tools):** Equip the limbs. Install tool plugins (e.g., `fs`, `gemini-search`).
    5.  **Perception (Providers):** Choose the brain. Connect to a specific LLM model (e.g., Gemini 1.5 Pro).

---

## 3. Interactive Conversation: Connecting to an Agent (`openstarry attach`)

To start a conversation with an agent, use the `attach` command. This is the **single entry point** for interaction.

### Scenario A: Connecting from the Dashboard
1.  In the `openstarry` dashboard, use the arrow keys to select the target agent (e.g., `sys-01`).
2.  Press the **`a`** key.
3.  The screen instantly switches to the agent's conversation window.
4.  Press `Ctrl+D` (Detach) to return to the dashboard while the agent continues running in the background.

### Scenario B: Connecting from a Project Directory (Most Common)
In your development directory, simply type:
```bash
$ openstarry attach
```
*   **If the agent is not running:** The system automatically starts it and enters the conversation interface.
*   **If the agent is already running in the background:** The system connects to it directly.

### Scenario C: Connecting to Any Background Agent
```bash
$ openstarry attach web-01
```

### Interaction Tips: Slash Commands
During a conversation, you can use `/` to invoke tools directly without going through the LLM.
*   `/login` -> Calls the `google-login` tool (if an alias has been registered).
*   `/tool google-login` -> Executes the tool directly.
*   `/clear` -> Clears the current Context memory.

---

## 4. Deployment & Auto-Start Workflow

How do you make your agent come online automatically when the system boots?

### Step 1: Sync Standard Plugins
Ensure your system has the latest standard capability library.
```bash
# Assuming you have cloned the openstarry_plugin repo locally
$ openstarry plugin sync ./openstarry_plugin
```

### Step 2: Register
First, tell the daemon where your agent lives. Run this inside the project directory:
```bash
$ openstarry register
```
This writes the project path to `~/.openstarry/registry.json`, making it a "managed agent."

### Step 3: Add to Auto-Start List
There are currently two methods:

**Method A (CLI):**
In the Dashboard's "System Configuration" menu, or using a future planned command:
```bash
$ openstarry system config --autostart add <project-id>
```

**Method B (Manual Configuration):**
Edit `~/.openstarry/daemon.json`:
```json
{
  "autostart": [
    "master-agent",
    "my-coding-bot",
    "daily-news-reporter"
  ]
}
```

### Step 4: Verification
Restart the daemon (`openstarry daemon restart`) or re-enter `openstarry` to open the dashboard. You should see your agent already online in the **[RUNNING AGENTS]** list.

---

## 5. Advanced Operations: Workflows & Multi-Agent Orchestration

OpenStarry's real power lies in its ability to run a "chain" of agents.

### Step 1: Define a Workflow
Create a `workflow.yaml` in your project or system directory:
```yaml
name: Daily Research
steps:
  - id: research
    agent: research-agent
    instruction: "Collect today's AI news"
  - id: summary
    agent: writer-agent
    input_from: research
    instruction: "Summarize into 3 key points"
```

### Step 2: Execute the Workflow
Trigger it from the CLI:
```bash
$ openstarry run ./workflow.yaml
```

### Step 3: Observe the Collaboration
Return to the Dashboard (`openstarry`) and you'll see:
1.  **Dynamic Spawning:** `research-agent` suddenly appears in the list, showing `BUSY`.
2.  **Task Handoff:** After a few seconds, `research-agent` disappears (task complete, instance destroyed) and `writer-agent` appears and begins writing.
3.  **Result Delivery:** The final result is output to your terminal or a designated file.

---

## 6. Troubleshooting & Log Interpretation

When an agent isn't behaving as expected, how do you diagnose the issue?

### Viewing Live Thoughts
After using `attach` to connect to an agent, you can see its "inner monologue":
```text
[THOUGHT] User asked for file search.
[PLAN] I need to use 'fs' tool to list directory.
[CALL] fs.list_dir(path=".")
[RESULT] Success. Found 3 files.
[RESPONSE] I found 3 files...
```
If the agent is stuck, it's usually hanging at a `[CALL]` failure or caught in a `[THOUGHT]` loop.

### Viewing System Logs
If an agent won't start at all, check the daemon logs:
```bash
$ openstarry system logs
```
Common errors:
*   **Template Not Found:** The registry path is wrong. Re-run `register`.
*   **Permission Denied:** The plugin lacks sufficient permissions. Check `manifest.json` or use `openstarry plugin grant`.
