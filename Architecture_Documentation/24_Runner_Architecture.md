# 24. Runner 架構 (Runner Architecture)

本文件說明 `apps/runner` 的設計理念與職責邊界。

---

## 1. 什麼是 Runner？

Runner 是 OpenStarry 的**宿主引導程式**（Host Bootstrap Program）。它是 Agent 的進入點，但本身不包含任何業務邏輯。

```
apps/runner/src/bin.ts
```

Runner 的職責僅有四步：
1. 讀取 `agent.json` 配置（或使用內建預設值）
2. 建立 `AgentCore` 實例
3. 動態解析並載入所有插件
4. 啟動 Agent

---

## 2. 為什麼不叫 CLI？

Runner 曾經叫做 `apps/cli`，但在 Plan03 Phase C 重構中更名。原因：

| 問題 | 說明 |
|------|------|
| 名不副實 | CLI 的互動體驗（readline、顏色輸出）由 `standard-function-stdio` 插件提供，不在 Runner 中 |
| 暗示耦合 | 「CLI」這個名字暗示它只能用於命令列，但 Runner 不知道也不關心 UI 是什麼 |
| 微核心精神 | Runner 應該是一個與 UI 無關的純啟動器 |

---

## 3. Runner 的依賴

Runner 只依賴三個核心套件：

```json
{
  "dependencies": {
    "@openstarry/core": "workspace:*",
    "@openstarry/sdk": "workspace:*",
    "@openstarry/shared": "workspace:*"
  }
}
```

**零插件依賴。** 所有 `@openstarry-plugin/*` 套件都在運行時透過 `import()` 動態載入。

---

## 4. 插件如何推送輸入？

在 Runner 解耦之前，stdio 插件依賴宿主注入的 `onInput` 回呼。這迫使 Runner 必須認識 stdio 插件並做特殊處理。

現在的架構：

```
SDK 層：IPluginContext.pushInput(event: InputEvent)
         ↓
Core 層：getPluginContext() 自動注入 pushInput → core.pushInput
         ↓
插件層：stdio listener 收到用戶輸入 → ctx.pushInput({ source: "cli", data: text })
         ↓
Core 層：pushInput → 斜線指令快速路徑 / EventQueue → ExecutionLoop
```

**所有 Listener 插件**都可以透過 `ctx.pushInput` 推送輸入。Runner 不參與這個流程。

---

## 5. 預設配置

當沒有提供 `agent.json` 時，Runner 使用內建的 `defaultConfig()`：

```typescript
function defaultConfig(): IAgentConfig {
  return {
    identity: { id: "openstarry-agent", name: "OpenStarry Agent", ... },
    cognition: { provider: "gemini-oauth", model: "gemini-2.0-flash", ... },
    plugins: [
      { name: "@openstarry-plugin/provider-gemini-oauth" },
      { name: "@openstarry-plugin/standard-function-fs" },
      { name: "@openstarry-plugin/standard-function-stdio" },
    ],
    guide: "default-guide",
  };
}
```

注意：即使是預設配置，插件也是用完整套件名透過動態 import 載入。

---

## 6. 進程級職責

Runner 保留少數進程級關注點，這些不屬於任何插件的責任範圍：

| 職責 | 說明 |
|------|------|
| `process.exit()` | 監聽 `__QUIT__` 事件，執行 process exit |
| `SIGINT / SIGTERM` | 收到終止信號時，優雅關閉 Agent |
| Config 載入 | 讀取 `agent.json`、Zod 驗證、fallback 到預設值 |

---

## 7. 未來演進

Runner 可以被不同的宿主取代，而不需要修改任何插件：

| 宿主 | 說明 |
|------|------|
| `apps/runner` | 當前的 Node.js 進程宿主 |
| `apps/daemon` | 未來的守護進程，管理多個 Agent 實例 |
| `apps/web-server` | 未來的 Web 服務宿主，透過 HTTP 接收輸入 |

每個宿主做的事情都一樣：讀 config → 建 core → 載入插件 → 啟動。差異只在進程管理和外部接口。
