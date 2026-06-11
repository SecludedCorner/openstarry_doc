# OpenStarry Implementation Plan 03 — 品質補強與架構純化

> **狀態**: ✅ 已完成 (2026-02-05)

## 背景

Plan01 完成 MVP 骨架，Plan02 補齊事件驅動架構與安全熔斷。
測試人員於 2026-02-05 確認所有測試通過，並提出下一步建議。
本計畫根據設計文件差距分析結果，聚焦於 **v0.1 → v0.2 的品質升級與架構純化**。

---

## 目標

1. **品質補強**：讓現有系統更健壯，避免配置錯誤崩潰、工具卡死、除錯困難
2. **能力擴展**：實作 Markdown 技能解析，讓 Agent 能動態載入人設與行為準則
3. **架構純化**：Runner 完全解耦插件、所有插件動態載入、統一完整套件名

---

## Phase A：品質補強（v0.2 基礎） ✅ 已完成

### A1. agent.json Zod 運行時驗證 ✅

- 路徑：`packages/shared/src/utils/config-schema.ts`
- 用 Zod 定義 `AgentConfigSchema`，涵蓋所有配置欄位
- 在 `apps/runner/src/bin.ts` 的 `loadConfig()` 中使用 schema 驗證
- 驗證失敗時輸出清楚的錯誤訊息

### A2. 工具調用 Timeout ✅

- 路徑：`packages/core/src/execution/loop.ts` 的 `executeTool()`
- 用 `Promise.race([tool.execute(...), timeoutPromise])` 包裹
- Timeout 值從 `config.policy.toolTimeout`（預設 30000ms）取得

### A3. TraceID 機制 ✅

- 在 `processEvent()` 開頭生成 `traceId`
- 注入 logger 和事件 payload，串聯一次完整處理週期

### A4. CI 純淨性檢查 ✅

- `scripts/check-purity.sh` + `pnpm test:purity`
- core 不引用 plugin/apps、sdk 不引用 core/shared

---

## Phase B：能力擴展 ✅ 已完成

### B1. standard-function-skill 插件 ✅

- 路徑：`openstarry_plugin/standard-function-skill/`
- 讀取 `.md` 技能檔案，解析 YAML Frontmatter + Markdown Body
- 作為 Guide 插件註冊到 GuideRegistry
- **動態載入**：不寫死在 Runner 中，透過 `agent.json` 配置：
  ```json
  {
    "name": "@openstarry-plugin/standard-function-skill",
    "config": { "skillPath": "./skills/my-agent.md" }
  }
  ```
- 範例技能檔案位於 `openstarry_plugin/standard-function-skill/examples/coder.md`
- 7 項單元測試覆蓋 frontmatter 解析

### B2. system_prompt 外部檔案引用 ✅

- `IAgentConfig` 新增 `guideFile?: string` 欄位
- AgentCore 啟動時讀取外部 `.md` 檔案，註冊為 FileGuide
- 兩種方式並存：`guide`（Guide ID）或 `guideFile`（檔案路徑）

---

## Phase C：架構純化 ✅ 已完成

### C1. IPluginContext.pushInput ✅

**問題：** stdio 插件依賴宿主注入的 `onInput` 回呼來推送用戶輸入。這讓 CLI 必須認識 stdio 插件並做特殊處理，違反了完全動態載入的目標。

**解決：**
- SDK 的 `IPluginContext` 新增 `pushInput(event: InputEvent)` 方法
- Core 的 `getPluginContext()` 自動注入 `pushInput → core.pushInput`
- 所有 Listener 插件透過標準化的 context 推送輸入，不再依賴宿主回呼

### C2. 移除 BUILTIN_FACTORIES ✅

**問題：** Runner 的 `BUILTIN_FACTORIES` 寫死了三個插件的 import，每新增插件就要改 Runner。

**解決：**
- 完全移除 `BUILTIN_FACTORIES` 和所有 `@openstarry-plugin/*` import
- 插件解析只剩兩層：
  1. `ref.path` → 檔案路徑直接載入
  2. `import(ref.name)` → 完整套件名從 workspace / node_modules 動態載入
- Runner 的 `package.json` 不再依賴任何插件套件

### C3. 統一完整套件名 ✅

- `agent.json` 中所有插件使用完整套件名：`@openstarry-plugin/xxx`
- defaultConfig 的 plugins 列表也使用完整套件名
- 消除短名/完整名混用的問題

### C4. apps/cli → apps/runner ✅

- 重新命名為 `apps/runner`，package name 改為 `@openstarry/runner`
- 反映其「純啟動器」角色：讀 config → 建 core → 動態載入插件 → 啟動
- 不認識任何具體插件

---

## 實作順序（已全部完成）

```
A1 agent.json Zod 驗證          ✅
A2 工具調用 Timeout             ✅
A3 TraceID 機制                 ✅
A4 CI 純淨性檢查               ✅
B1 standard-function-skill      ✅
B2 system_prompt 外部檔案引用   ✅
C1 IPluginContext.pushInput     ✅
C2 移除 BUILTIN_FACTORIES       ✅
C3 統一完整套件名               ✅
C4 apps/cli → apps/runner       ✅
```

---

## 驗證結果

1. `pnpm build` — ✅ 全部 8 個 workspace project 編譯通過
2. `pnpm test` — ✅ 56 項測試通過（7 個測試檔案）
3. `pnpm test:purity` — ✅ 純淨性檢查通過
4. Runner 零插件依賴 — ✅ `apps/runner/package.json` 只依賴 core/sdk/shared
