# 实作示例：USB 即插即用代理人 (USB Plug-and-Play Agent)

本示例展示了 OpenStarry 架构的极致灵活性：将一个完整的 Agent 封装在 USB 闪存盘中。当 USB 插入主机时，系统自动感知并唤醒该 Agent 执行特定任务（如自动备份、系统诊断），并在任务结束后允许安全移除。

---

## 1. 场景描述

*   **目标：** 制作一个「照片备份棒」。
*   **行为：**
    1.  插入 USB。
    2.  系统提示：「检测到照片备份助手」。
    3.  用户确认后，该 Agent 自动扫描主机图片文件夹，将新增照片备份到 USB 中。
    4.  完成后通知用户，并自我终止。

---

## 2. USB 内的目录结构

这是一个标准的 OpenStarry **项目级 Agent** 目录，只是物理载体变成了 USB 驱动器 (例如 `E:/`)。

```text
E:/ (USB Root)
├── configs/
│   └── agent.json       # 身份与权限定义
├── plugins/
│   └── backup-utils/    # 专用插件：负责增量备份算法
├── logs/                # 运行日志
└── backup_data/         # 备份文件存放区
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
      "std-fs-tools",       // 继承系统的文件工具
      "backup-utils"        // 使用 USB 自带的备份算法
    ],
    "permissions": {
      "fs_allow_paths": ["C:/Users/Public/Pictures", "E:/backup_data"], // 严格限制读写范围
      "network_allow_hosts": [] // 禁止联网，确保隐私
    }
  }
}
```

---

## 3. 系统侧实现 (Host Side)

主机上的 **System Agent** 需要具备「感知硬件变更」的能力。

### 步骤 A: 安装 USB 监听插件
System Agent 加载 `std-hardware-listener` 插件（属于受蕴）。

```javascript
// plugins/std-hardware-listener/index.js (示意代码)
const usb = require('usb-detection');

class HardwareListener {
  start() {
    usb.on('add', (device) => {
      // 1. 检测到设备插入
      const mountPoint = this.getMountPoint(device);
      // 2. 检查是否存在 agent.json
      if (fs.existsSync(path.join(mountPoint, 'configs/agent.json'))) {
        // 3. 通知 System Agent
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

### 步骤 B: 系统 Agent 的响应逻辑
当 System Agent 的 LLM 收到 `USB_AGENT_DETECTED` 事件时，它会根据 System Prompt 执行以下决策：

1.  **验证:** 调用 `agent:validate_manifest` 检查 USB Agent 的权限请求（是否请求了过高的权限？）。
2.  **询问:** 通过 UI 询问用户：「检测到 USB 备份助手，是否允许启动？它请求访问图片文件夹。」
3.  **孵化:** 用户同意后，调用 `agent:spawn`：
    ```javascript
    agent_manager.spawn({
      manifest: "E:/configs/agent.json",
      work_dir: "E:/"
    });
    ```

---

## 4. 运行时协作 (Runtime Coordination)

1.  **启动:** Orchestrator Daemon 启动一个新的 Node.js 进程，加载 `E:/` 下的代码。
2.  **执行:** USB Agent (Project Agent) 开始运行。它使用 `std-fs-tools` (系统提供的) 读取主机硬盘，使用 `backup-utils` (自己携带的) 写入 USB。
3.  **通讯:** USB Agent 通过 MCP 协议向 System Agent 汇报进度：「已备份 50 张照片...」。
4.  **结束:** 任务完成，USB Agent 发送 `TASK_COMPLETE` 信号。
5.  **销毁:** System Agent 收到信号，调用 `agent:kill` 终止进程，并通知用户「可以安全拔出」。

---

## 5. 架构总结

此案例完美展示了 OpenStarry 的 **分层继承** 与 **权限隔离** 机制：
*   **可携带性:** Agent 的灵魂 (Prompt) 和肉体 (Plugins) 都在 USB 里，随插随用。
*   **安全性:** 主机 System Agent 充当守门员，严格审查 USB Agent 的权限，防止恶意软件读取敏感数据。
