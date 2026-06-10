# 18. 代理人運行時配置邏輯 (Agent Runtime Configuration)

本文件補充了 `05_Agent_Manifest_Specification.md`，專注於 **Runtime (運行時)** 如何解析、驗證並應用 `agent.json` 中的配置，特別是插件的啟用與參數注入。

## 1. 配置加載流程

當 `openstarry start` 或 `Daemon` 啟動一個 Agent 時：

1.  **讀取：** 從 `configs/agent.json` 讀取原始 JSON。
2.  **合併 (Merger):**
    *   如果 `agent.json` 定義了 `extends: "generic-assistant-v1"`，系統會先加載該模板的配置，然後將當前專案的配置覆蓋上去 (Deep Merge)。
    *   這允許專案只定義差異部分（例如：只修改了 Prompt 或增加了一個 Tool）。
3.  **環境變數注入：** 解析 `${ENV_VAR}` 語法，注入 API Key 等敏感資訊。

## 2. 插件啟用與參數注入

這是配置中最關鍵的部分：如何讓 Agent 知道自己有哪些能力。

### 配置結構

```json
"capabilities": {
  "plugins": {
    "@openstarry-plugin/fs": {
      "enabled": true,
      "config": { "root": "./workspace" } // 注入給 initialize(context)
    },
    "@openstarry-plugin/weather": {
      "enabled": false // 顯式禁用
    }
  }
}
```

### 運行時邏輯

1.  **過濾：** Loader 遍歷 `plugins` 列表，過濾掉 `enabled: false` 的項目。
2.  **上下文構建：**
    *   對於每個啟用的插件（例如 `@openstarry-plugin/fs`），Loader 提取其 `config` 對象（例如 `{ "root": "./workspace" }`）。
    *   將此對象放入 `IPluginContext.config` 中。
3.  **初始化：** 呼叫插件的 `initialize(context)`。
    *   插件內部透過 `context.config.root` 獲取參數，並據此設定 `fs` 工具的路徑限制。

## 3. 五蘊配置映射

`agent.json` 的 `cognition` 與 `identity` 區塊，實際上是直接配置 **Guide (識)** 類型的插件。

*   **System Prompt:** 如果 `agent.json` 定義了 `system_prompt`，Core 會將其封裝為一個臨時的 `AnonymousGuidePlugin`，並優先級設為最高。
*   **Provider:** `cognition.provider` 的配置會直接傳遞給對應的 Provider 插件（如 `gemini`）。

## 4. 錯誤處理與缺省行為

*   **無插件啟動 (Zero Plugin State):**
    *   若系統啟動時偵測不到任何 Plugin (或 `~/.openstarry/plugins` 為空)，Agent Core **不應崩潰**。
    *   系統應進入 **「等待模式 (Idle Mode)」**，並在終端機或 Dashboard 顯示醒目提示：
        > "⚠️ No plugins found. Please run `openstarry plugin sync` to install standard capabilities."
*   **缺少依賴：** 如果配置啟用了 `plugin-x` 但系統找不到，Agent 啟動應報錯並終止（Fail Fast），除非該插件被標記為 `optional: true`。
*   **配置無效：** 插件應在 `initialize` 階段驗證傳入的 `config`，若無效則拋出 `ConfigurationError`。
