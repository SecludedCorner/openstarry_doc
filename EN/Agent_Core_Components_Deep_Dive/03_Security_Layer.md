# Deep Dive: Security Layer

This document delves into the implementation mechanism of the Security Layer—the safety boundary of the "Headless Agent Core."

## Core Responsibility

Functioning as the "gatekeeper" for interactions between the core and the external world, the Security Layer's sole responsibility is to intercept `Tool` invocation requests **before** execution and process them according to preset policies, ensuring the Agent's behavior complies with safety and regulatory requirements.

---

## Implementation Mechanism: A Multi-level Decision Flow

1.  **Interception Point:** The Security Layer is positioned within the "Execution Loop," occurring after the LLM has parsed a tool call request but before the tool is actually executed.

2.  **Interaction with Policy Engine (API Contract):**
    *   The first step taken by the Security Layer is to issue a synchronous permission check request to the external "Policy & Guardrails Engine."
    *   **Request Body:**
        ```json
        {
          "principal": { "id": "user-123", "groups": ["editor"] },
          "action": "tool:execute",
          "resource": {
            "type": "tool",
            "name": "shell:execute",
            "attributes": {
              "args": { "command": "ls -l /data" }
            }
          },
          "context": { "ip_address": "...", "session_id": "..." }
        }
        ```
    *   **Response Body:**
        ```json
        {
          "decision": "ALLOW" | "DENY" | "REQUIRE_USER_CONFIRMATION",
          "reason": "Command 'ls' is in the whitelist.",
          "obligations": [ /* e.g., logging required after execution */ ]
        }
        ```

### 3. Argument Sanitization & Validation
*   **Before execution**, even if approved by the policy engine and the user, the Security Layer performs a final check.
*   **Path Scoping (Core Security Feature):**
    *   **Definition:** For `fs` (file system) tools, the system enforces a "root path restriction." Agents are assigned one or more "Permitted Roots" (e.g., `./workspace/`).
    *   **Normalization:** The Security Layer converts relative or absolute paths provided by the Agent into absolute canonical paths, removing all `..` (parent directory) references.
    *   **Boundary Check:** The system verifies that the final path **starts with** a permitted root. If an Agent attempts to escape via `../../../etc/passwd`, the operation is intercepted immediately.
    *   **Operational Permissions:** Within permitted paths, the Agent **can perform operations like adding folders or deleting files.** This ensures the Agent possesses practical utility without endangering core areas of the host machine.
*   **Examples:**
    *   **Path Traversal:** Checks if file path parameters contain `..` or start with `/` (if only relative paths are permitted).
    *   **Command Injection:** Checks if shell command parameters contain malicious connectors like `&&`, `|`, or `;`. A whitelist can be used to restrict the structure of executable commands.
    *   **Type Validation:** Ensures the types of passed arguments align with the `args` schema defined by the tool.

4.  **User Confirmation Request:**
    *   If the policy engine responds with `REQUIRE_USER_CONFIRMATION`, the Security Layer issues an `onToolCallRequest` event via the "Bi-directional Communication Interface," delegating the final decision to the user.

5.  **Audit Logging:**
    *   **Principle:** Every decision processed by the Security Layer—whether allowed, denied, or pending user confirmation—**must** be recorded.
    *   **Log Content:** Logs should include at minimum a `timestamp`, `principal`, `action`, `resource`, `policy engine decision`, `user decision` (if any), and `final outcome (allow/deny)`.
    *   **Purpose:** Used for post-hoc security audits, issue tracing, and system behavior analysis.

---

## Special Considerations: Security Policies for High-Risk Tools

For tools capable of executing arbitrary code, such as `shell_interpreter` or `python_interpreter`, the Security Layer and its collaborating policy engine must employ more stringent processing mechanisms.

### 1. Mandatory Sandboxing
*   **Primary Principle:** The **implementation itself** of the `Tool` plugin must be forced into a sandbox. The Security Layer should not trust the tool's implementation but can use mechanisms to verify that the tool declares and operates in a sandboxed mode.
*   **Policy Configuration:** The "Policy & Guardrails Engine" should include rules specifying that only code execution tools marked as "sandboxed" are permitted for invocation.

### 2. Fine-Grained Policy Rules
The policy engine should define extremely granular rules for high-risk tools, such as:
*   **Command Whitelist/Blacklist:**
    *   `ALLOW` `shell:execute` if the command starts with `ls`, `cat`, or `echo`.
    *   `DENY` `shell:execute` if the command contains `rm -rf`, `mv`, or `sudo`.
*   **Network Access Restrictions:**
    *   `DENY` `python:execute` if the code attempts outbound network calls unless the destination is in `[api.example.com]`.
*   **File System Access Rights:**
    *   `DENY` `python:execute` if the code attempts to read files outside its designated `/workspace` directory.
*   **Resource Limits:**
    *   `DENY` `python:execute` if execution time exceeds 30 seconds or memory usage exceeds 512MB.

### 3. Enhanced User Confirmation
When the policy engine requires user confirmation (`REQUIRE_USER_CONFIRMATION`), the `onToolCallRequest` event sent to the UI should contain richer warning information.
*   **Data Structure:**
  ```json
  {
    "toolName": "shell:execute",
    "args": { "command": "rm important_file.txt" },
    "confirmationId": "...",
    "security_warning": {
      "level": "CRITICAL",
      "message": "The agent is attempting to execute a command that deletes a file. This action may be irreversible."
    }
  }
  ```
*   **UI Behavior:** Upon receiving a request with a `security_warning`, the UI plugin should display the warning prominently (e.g., a red highlighted dialog) and may require a second level of confirmation (e.g., typing "I confirm").
