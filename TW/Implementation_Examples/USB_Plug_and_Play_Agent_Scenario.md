# 實作範例：USB 即插即用代理人 (USB Plug-and-Play Agent)

本範例展示了 OpenStarry 架構的極致靈活性：將一個完整的 Agent 封裝在 USB 隨身碟中。當 USB 插入主機時，系統自動感知並喚醒該 Agent 執行特定任務（如自動備份、系統診斷），並在任務結束後允許安全移除。

---

## 1. 場景描述

*   **目標：** 製作一個「照片備份棒」。
*   **行為：**
    1.  插入 USB。
    2.  系統提示：「檢測到照片備份助手」。
    3.  用戶確認後，該 Agent 自動掃描主機圖片文件夾，將新增照片備份到 USB 中。
    4.  完成後通知用戶，並自我終止。

---

## 2. USB 內的目錄結構

這是一個標準的 OpenStarry **專案級 Agent** 目錄，只是物理載體變成了 USB 驅動器 (例如 `E:/`)。

```text
E:/ (USB Root)
├── configs/
│   └── agent.json       # 身份與權限定義
├── plugins/
│   └── backup-utils/    # 專用插件：負責增量備份算法
├── logs/                # 運行日誌
└── backup_data/         # 備份文件存放區
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
      "std-fs-tools",       // 繼承系統的文件工具
      "backup-utils"        // 使用 USB 自帶的備份算法
    ],
    "permissions": {
      "fs_allow_paths": ["C:/Users/Public/Pictures", "E:/backup_data"], // 嚴格限制讀寫範圍
      "network_allow_hosts": [] // 禁止聯網，確保隱私
    }
  }
}
```

---

## 3. 系統側實現 (Host Side)

主機上的 **System Agent** 需要具備「感知硬體變更」的能力。

### 步驟 A: 安裝 USB 監聽插件
System Agent 加載 `std-hardware-listener` 插件（屬於受蘊）。

```javascript
// plugins/std-hardware-listener/index.js (示意代碼)
const usb = require('usb-detection');

class HardwareListener {
  start() {
    usb.on('add', (device) => {
      // 1. 檢測到設備插入
      const mountPoint = this.getMountPoint(device);
      // 2. 檢查是否存在 agent.json
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

### 步驟 B: 系統 Agent 的響應邏輯
當 System Agent 的 LLM 收到 `USB_AGENT_DETECTED` 事件時，它會根據 System Prompt 執行以下決策：

1.  **驗證:** 調用 `agent:validate_manifest` 檢查 USB Agent 的權限請求（是否請求了過高的權限？）。
2.  **詢問:** 通過 UI 詢問用戶：「檢測到 USB 備份助手，是否允許啟動？它請求訪問圖片文件夾。」
3.  **孵化:** 用戶同意後，調用 `agent:spawn`：
    ```javascript
    agent_manager.spawn({
      manifest: "E:/configs/agent.json",
      work_dir: "E:/"
    });
    ```

---

## 4. 運行時協作 (Runtime Coordination)

1.  **啟動:** Orchestrator Daemon 啟動一個新的 Node.js 進程，加載 `E:/` 下的代碼。
2.  **執行:** USB Agent (Project Agent) 開始運行。它使用 `std-fs-tools` (系統提供的) 讀取主機硬碟，使用 `backup-utils` (自己攜帶的) 寫入 USB。
3.  **通訊:** USB Agent 通過 MCP 協議向 System Agent 匯報進度：「已備份 50 張照片...」。
4.  **結束:** 任務完成，USB Agent 發送 `TASK_COMPLETE` 信號。
5.  **銷毀:** System Agent 收到信號，調用 `agent:kill` 終止進程，並通知用戶「可以安全拔出」。

---

## 5. 架構總結

此案例完美展示了 OpenStarry 的 **分層繼承** 與 **權限隔離** 機制：
*   **可攜帶性:** Agent 的靈魂 (Prompt) 和肉體 (Plugins) 都在 USB 裡，隨插隨用。
*   **安全性:** 主機 System Agent 充當守門員，嚴格審查 USB Agent 的權限，防止惡意軟體讀取敏感數據。
