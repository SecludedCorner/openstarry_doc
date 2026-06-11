# 11. 插件運行時隔離與沙箱機制 (Plugin Runtime Isolation)

本文件定義了 OpenStarry 系統如何隔離插件的執行環境，防止惡意或充滿 Bug 的插件導致核心崩潰、數據洩露或資源耗盡。

## 1. 風險模型

*   **穩定性風險**: 插件拋出未捕獲異常 (Uncaught Exception) 導致主進程退出。
*   **資源風險**: 插件寫了 `while(true)` 死循環或內存洩露 (Memory Leak)。
*   **安全風險**: 插件讀取環境變量中的 API Key，或隨意刪除文件系統中的文件。

## 2. 隔離分級策略 (Isolation Levels)

根據部署環境與安全需求，系統支持三種隔離級別。

### Level 1: 函數級包裹 (Function Wrapping) - [開發/輕量模式]
*   **機制**: 所有插件代碼在主進程 (Agent Core) 中運行。
*   **防護**:
    *   **Try-Catch**: `PluginLoader` 強制使用 `try-catch` 包裹插件的 `execute` 方法。
    *   **Timeout**: 使用 `Promise.race` 設置執行超時（例如 10秒），超時則拋出錯誤。
*   **缺點**: 無法防禦 `while(true)` (會卡死 Event Loop)，無法限制文件系統訪問。
*   **適用**: 內部受信任插件，開發調試階段。

### Level 2: 虛擬機沙箱 (VM Sandboxing) - [標準模式]
*   **機制**: 使用 Node.js 的 `vm` 模塊或 `vm2` 庫來運行插件代碼。
*   **防護**:
    *   **Context Isolation**: 插件無法訪問全局 `process` 對象，無法讀取環境變量。
    *   **Limited Access**: 只能調用 Core 明確注入的 API (如 `log`, `fetch`)。
*   **缺點**: 仍然在同一進程內，無法完全防禦 CPU 密集型攻擊。
*   **適用**: 運行第三方代碼，但信任度尚可的場景。

### Level 3: 進程級隔離 (Process Isolation) - [嚴格/生產模式]
*   **機制**: **Orchestrator Daemon** 負責為每個插件（或一組插件）啟動獨立的 OS 子進程 (Child Process) 或 Worker Thread。
*   **通訊**: Core 與 Plugin 之間通過 IPC (Inter-Process Communication) 或標準輸入輸出 (stdio) 傳遞 JSON 消息。
*   **防護**:
    *   **資源配額**: Daemon 可以使用 OS 機制 (cgroups, Docker) 限制插件進程的 CPU/RAM。
    *   **物理熔斷**: 插件死鎖不影響 Core，Daemon 可直接 `kill` 插件進程。
*   **缺點**: 性能開銷大，IPC 通訊有延遲。
*   **適用**: 執行高風險代碼（如 Code Interpreter），或多租戶環境。

### Level 4: WebAssembly (WASM) - [未來演進/極致安全]
*   **機制**: 將插件編譯為 WASM 模組，並在 Core 內嵌的 WASM 運行時 (如 Wasmtime) 中執行。
*   **優勢**:
    *   **內存安全**: 插件無法訪問沙箱外的內存空間。
    *   **能力授權 (Capability-based)**: 通過 WASI 接口極其精細地控制插件的文件與網絡訪問權限。
    *   **極速啟動**: 毫秒級冷啟動，遠快於進程隔離。
*   **場景**: 高性能計算工具、不信任的第三方商務插件。

---

## 3. 建議實作路徑 (Implementation Path)

### MVP 階段
採用 **Level 1 (Try-Catch + Timeout)**。
*   這足以應對大多數非惡意的 Bug。
*   開發成本最低。

### V1.0 生產階段
針對 **Code Interpreter** 和 **第三方插件**，必須採用 **Level 3 (獨立進程)**。
*   `Infrastructure Plugins` (如本地 Broker) 本質上就是 Level 3 的體現，由 Daemon 獨立管理。
*   普通的 `Tool` 插件若涉及敏感操作，應透過 `Plugin Runner` 獨立執行。

---

## 4. 權限聲明 (Permission Manifest)

每個插件的 `plugin.json` 必須聲明所需權限，由用戶在安裝時授權（類似 Android App）。

```json
{
  "name": "FileSearchTool",
  "permissions": [
    "fs:read:./data",  // 允許讀取 data 目錄
    "network:none"     // 禁止聯網
  ]
}
```

Core 在加載插件時，會根據這些聲明構建受限的執行上下文 (Context)。
