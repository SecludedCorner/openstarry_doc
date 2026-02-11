# 05. Security & Sandboxing Protocol Technical Specification

This document defines the multi-layered defense system of OpenStarry. As an Agent system with execution capabilities, security is a non-negotiable foundation.

## 1. FileSystem Sandboxing

All IO-related operations (read, write, list) must be strictly monitored by the `PathGuard`.

### 1.1 Chroot-like Jail
By default, an Agent can only access its working directory (`workspace_root`). Any attempt to access parent directories will be intercepted.

*   **Allowed:** `./data/file.txt`, `src/index.ts`
*   **Blocked:** `../secret.key`, `/etc/passwd`, `C:\Windows\System32`

### 1.2 Path Normalization
Before checking permissions, all paths must be resolved and normalized to prevent bypasses through symbolic links (Symlinks) or relative path traversal attacks (`/app/workspace/../../etc/passwd`).

---

## 2. Command Execution Whitelist

For high-risk tools such as `shell:exec`, a whitelist strategy must be enforced.

### Security Policy Configuration (`agent.json`)

```json
"security": {
  "allow_shell": true,
  "blocked_commands": ["rm -rf", "mkfs", ":(){:|:&};:", "format"],
  "allowed_commands": ["ls", "cat", "grep", "npm test", "git status"],
  "confirm_dangerous_actions": true
}
```

When `confirm_dangerous_actions` is set to `true`, any command not in `allowed_commands` will cause the core to pause execution and emit `EVENT:REQUEST_CONFIRMATION`, awaiting human approval before proceeding.

---

## 3. Resource Quotas & Circuit Breakers

To prevent DoS attacks or infinite loops, the core enforces the following limits:

| Resource Dimension | Threshold (Default) | Triggered Consequence |
| :--- | :--- | :--- |
| **Max Loop Ticks** | 50 per task | Force task pause (Safety Halt) |
| **Token Budget** | 100k per session | Reject LLM request |
| **Tool Timeout** | 30 seconds | Force-terminate tool process (SIGKILL) |
| **Output Size** | 1 MB | Truncate output |

---

## 4. Plugin Permission Model

Plugins must declare their required permissions at load time (Manifest Declaration).

```json
// plugin.json
{
  "permissions": [
    "fs:read",
    "network:http",
    "core:state:write"
  ]
}
```

During plugin initialization, the core compares the request against the authorization list in `agent.json`. If a plugin requests an unauthorized sensitive permission, loading will be rejected.
