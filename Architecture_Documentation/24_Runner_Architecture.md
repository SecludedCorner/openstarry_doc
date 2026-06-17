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

> ⚠️ **[漂移更正 — v0.59.6 / stale-claim]** 本節原文宣稱「Runner 只依賴三個核心套件」「零插件依賴」，與代碼**相反**且該相反事實是 CI 強制的。真實狀況：`apps/runner/package.json` 的 `dependencies` 除三個核心套件（`@openstarry/core` / `sdk` / `shared`）外，**還明列 38 個 `@openstarry-plugin/*` workspace 依賴**（`apps/runner/package.json:22-59`）；且 `apps/runner/scripts/verify-plugin-deps.mjs` 是 pre-publish 守衛——只要任一 workspace plugin 未列入 runner 的 `dependencies`，它即 `process.exit(1)` **讓 build 失敗**（`verify-plugin-deps.mjs:63-71`，CC-1 / K-2 HIGH check）。換言之，「零插件依賴」非但不成立，「插件依賴完整」反而是被自動化驗證強制的。
>
> ✅ **仍然成立的部分（不變）**：plugin **載入**確實是運行時動態 `import()`，Runner 不靜態 import 任何 plugin 的程式碼路徑——見 `apps/runner/src/utils/plugin-resolver.ts:97/111/143`（依 file URL → package name → system path 三策略 `await import()`）。`package.json` 的依賴宣告用途是 **workspace 解析 + CI 完整性守衛 + 打包確保套件存在**，與「程式碼不靜態耦合 plugin」並不矛盾：宣告依賴（套件就位）≠ 靜態 import（編譯期耦合）。下方原文保留作歷史，「零插件依賴」一句已不準確。

Runner 在 `package.json` 宣告三個核心套件，外加全部 plugin 套件以供 workspace 解析與 CI 守衛：

```json
{
  "dependencies": {
    "@openstarry/core": "workspace:*",
    "@openstarry/sdk": "workspace:*",
    "@openstarry/shared": "workspace:*",
    "@openstarry-plugin/provider-gemini-oauth": "workspace:*",
    "@openstarry-plugin/standard-function-fs": "workspace:*",
    "...（其餘 36 個 @openstarry-plugin/* 套件，共 38 個）": "workspace:*"
  }
}
```

所有 `@openstarry-plugin/*` 套件在**程式碼層級**都不被靜態 import，而是在運行時透過 `import()` 動態載入（plugin-resolver）。但它們必須出現在 `package.json` 依賴中——`verify-plugin-deps.mjs` 會在缺漏時讓 build 失敗。

> 以下為原文（歷史保留，「零插件依賴」已被上方更正取代）：
>
> Runner 只依賴三個核心套件（`@openstarry/core` / `@openstarry/sdk` / `@openstarry/shared`）。**零插件依賴。** 所有 `@openstarry-plugin/*` 套件都在運行時透過 `import()` 動態載入。

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
