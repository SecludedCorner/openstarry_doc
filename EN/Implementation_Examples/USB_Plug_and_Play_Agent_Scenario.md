# Implementation Example: USB Plug-and-Play Agent

This example demonstrates the ultimate flexibility of the OpenStarry architecture: encapsulating a complete Agent within a USB flash drive. When the USB is inserted into a host, the system automatically detects and awakens the Agent to perform specific tasks (e.g., automated backup, system diagnostics) and allows for safe removal upon task completion.

---

## 1. Scenario Description

*   **Goal:** Create a "Photo Backup Stick."
*   **Behavior:**
    1.  Insert USB.
    2.  System prompt: "Photo Backup Assistant detected."
    3.  Upon user confirmation, the Agent automatically scans the host's Pictures folder and backs up new photos to the USB.
    4.  Notifies the user upon completion and self-terminates.

---

## 2. Directory Structure within the USB

This is a standard OpenStarry **Project Agent** directory, with the physical carrier being a USB drive (e.g., `E:/`).

```text
E:/ (USB Root)
├── configs/
│   └── agent.json       # Identity and permission definitions
├── plugins/
│   └── backup-utils/    # Specialized plugin: handles incremental backup algorithms
├── logs/                # Operational logs
└── backup_data/         # Backup storage area
```

### `configs/agent.json`

```json
{
  "identity": {
    "id": "usb-photo-backup-01",
    "name": "USB Photo Backup",
    "role": "backup_worker"
  },
  "capabilities": {
    "plugins": [
      "std-fs-tools",       // Inherit system file tools
      "backup-utils"        // Use the USB's own backup algorithm
    ],
    "permissions": {
      "fs_allow_paths": ["C:/Users/Public/Pictures", "E:/backup_data"], // Strict R/W limits
      "network_allow_hosts": [] // Network disabled for privacy
    }
  }
}
```

---

## 3. Host-side Implementation

The **System Agent** on the host must be capable of "detecting hardware changes."

### Step A: Install USB Listener Plugin
The System Agent loads the `std-hardware-listener` plugin (belonging to Sensation).

```javascript
// plugins/std-hardware-listener/index.js (Schematic code)
const usb = require('usb-detection');

class HardwareListener {
  start() {
    usb.on('add', (device) => {
      // 1. Device insertion detected
      const mountPoint = this.getMountPoint(device);
      // 2. Check for existence of agent.json
      if (fs.existsSync(path.join(mountPoint, 'configs/agent.json'))) {
        // 3. Notify System Agent
        this.core.submitSensoryInput({
          sense: 'environmental',
          type: 'USB_AGENT_DETECTED',
          payload: { path: mountPoint, deviceId: device.serialNumber }
        });
      }
    });
  }
}
```

### Step B: System Agent Response Logic
When the System Agent's LLM receives the `USB_AGENT_DETECTED` event, it executes the following decisions based on the System Prompt:

1.  **Verification:** Calls `agent:validate_manifest` to check the USB Agent's permission requests (are they excessive?).
2.  **Inquiry:** Asks the user via the UI: "USB Backup Assistant detected. Allow startup? It requests access to the Pictures folder."
3.  **Spawning:** Upon user consent, calls `agent:spawn`:
    ```javascript
    agent_manager.spawn({
      manifest: "E:/configs/agent.json",
      work_dir: "E:/"
    });
    ```

---

## 4. Runtime Coordination

1.  **Startup:** The Orchestrator Daemon launches a new Node.js process, loading code from `E:/`.
2.  **Execution:** The USB Agent (Project Agent) begins operation. It uses `std-fs-tools` (system-provided) to read the host hard drive and `backup-utils` (self-contained) to write to the USB.
3.  **Communication:** The USB Agent reports progress to the System Agent via the MCP protocol: "Backed up 50 photos..."
4.  **Conclusion:** Task finished; the USB Agent sends a `TASK_COMPLETE` signal.
5.  **Destruction:** The System Agent receives the signal, calls `agent:kill` to terminate the process, and notifies the user that it is "safe to remove."

---

## 5. Architectural Summary

This case perfectly demonstrates OpenStarry's **hierarchical inheritance** and **permission isolation** mechanisms:
*   **Portability:** Both the Agent's soul (Prompt) and body (Plugins) reside within the USB, plug-and-play.
*   **Security:** The host System Agent acts as a gatekeeper, strictly auditing the USB Agent's permissions to prevent malicious software from reading sensitive data.
