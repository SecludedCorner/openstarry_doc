# 23. 動態插件載入與命名規範 (Dynamic Plugin Loading & Naming Convention)

本文件說明 OpenStarry 的插件解析機制與套件命名規範。

---

## 1. 設計原則

Runner（`apps/runner`）是一個**純啟動器**，不認識任何具體插件。所有插件——包括 Provider、Tool、Listener、Guide——都透過 `agent.json` 配置並在運行時動態載入。

這意味著：
- Runner 的 `package.json` **不依賴**任何 `@openstarry-plugin/*` 套件
- Runner 的原始碼**不 import** 任何插件模組
- 新增插件不需要修改 Runner 的任何程式碼

---

## 2. 套件命名規範

所有官方插件統一使用 `@openstarry-plugin/` scope：

```
@openstarry-plugin/{plugin-name}
```

| 完整套件名 | 類型 | 說明 |
|-----------|------|------|
| `@openstarry-plugin/provider-gemini-oauth` | Provider (想) | Gemini LLM + OAuth 認證 |
| `@openstarry-plugin/standard-function-fs` | Tool (行) | 檔案系統操作 |
| `@openstarry-plugin/standard-function-stdio` | Listener + Guide (受 + 識) | CLI 終端 I/O + 預設人設 |
| `@openstarry-plugin/standard-function-skill` | Guide (識) | Markdown 技能檔案載入 |

### 在 agent.json 中的使用

```json
{
  "plugins": [
    { "name": "@openstarry-plugin/provider-gemini-oauth" },
    { "name": "@openstarry-plugin/standard-function-fs" },
    { "name": "@openstarry-plugin/standard-function-stdio" },
    {
      "name": "@openstarry-plugin/standard-function-skill",
      "config": { "skillPath": "./skills/coder.md" }
    }
  ]
}
```

---

## 3. 插件解析策略（兩層）

Runner 的 `resolvePlugins()` 對 `agent.json` 中的每個 plugin entry 按順序嘗試：

### Strategy 1：檔案路徑載入

當 `ref.path` 有值時，直接 `import()` 該檔案路徑。

```json
{
  "name": "my-custom-plugin",
  "path": "../my-plugins/custom-tool/dist/index.js"
}
```

**適用場景：**
- 本地開發中的插件（還沒發布到 npm）
- 第三方插件不在 pnpm workspace 中
- 別人給你的單一 `.js` 插件檔案

### Strategy 2：套件名動態載入

嘗試 `import(ref.name)`，由 Node.js 的模組解析機制找到套件。

```json
{
  "name": "@openstarry-plugin/standard-function-fs"
}
```

**解析路徑：**
- pnpm workspace 連結（開發環境）
- `node_modules/`（生產環境、npm install 後）

---

## 4. 插件工廠模式

每個插件套件必須 export 一個工廠函數（factory function）：

```typescript
// 具名 export
export function createMyPlugin(): IPlugin { ... }

// 或 default export
export default function createMyPlugin(): IPlugin { ... }
```

Runner 會嘗試 `mod.default ?? mod.createPlugin` 來取得工廠函數。

工廠函數不接受外部參數。插件的配置透過 `IPluginContext.config`（來自 `agent.json` 的 `plugins[].config`）在 factory 階段注入。

---

## 5. 歷史沿革

| 版本 | 載入方式 |
|------|---------|
| Plan01 | `BUILTIN_FACTORIES` 寫死在 CLI 中 + 短名 |
| Plan02 | 新增 `ref.path` 動態載入（三層策略） |
| Plan03 Phase C | 移除 `BUILTIN_FACTORIES`，全面動態載入，統一完整套件名 |
