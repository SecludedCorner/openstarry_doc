# 71. Two-Path Config Propagation Architecture

`[Cycle 03-9 新增]`

> **來源**: Cycle 03-9 Fix 12d + GUARDIAN 追溯調查
> **核心學者**: GUARDIAN (#11), KERNEL (#10), SUSSMAN (#22)
> **相關規則**: Rule #63 (L4 Config Activates), Rule #68 (Two-Path Verification)

---

## 1. 概述

OpenStarry plugin 系統有**兩條獨立的 config 傳遞路徑**。這個架構細節在 Cycle 03-9 Fix 12d 的追溯調查中才被完整認識，揭露了 v0.14.0-beta 至 v0.43.0-alpha 的 24 個版本中，Path A 一直斷裂但無 plugin 使用，因此**從未導致實際 bug**。

本文件記錄此雙路徑架構，以防未來類似盲區。

---

## 2. 兩條路徑的完整流向

### Path A: Module-Level Factory Config

```
agent.json
  └── PluginRef { name, path?, config? }
       └── plugin-resolver.ts::resolvePlugin(ref)
            ├── Strategy 1: path-based resolution
            │    └── factory(ref.config)  ← Fix 12d 前：factory()
            ├── Strategy 2: package-based resolution
            │    └── factory(ref.config)  ← Fix 12d 前：factory()
            └── Strategy 3: system-directory resolution
                 └── factory(ref.config)  ← Fix 12d 前：factory()
```

**特性**:
- 在 **Stage 1**（IPlugin 建構期）提供 config
- Factory 函數純函數，接收 config 返回 IPlugin
- **v0.14-v0.43 斷裂**：factory() 未接收 config

### Path B: IPluginContext.config

```
agent.json
  └── IAgentConfig.plugins[i]
       └── agent-core.ts::getPluginContext(pluginRef?.config)
            └── IPluginContext { config, bus, logger, ... }
                 └── IPlugin.factory(ctx)  ← 可訪問 ctx.config
```

**特性**:
- 在 **Stage 2**（IPluginHooks 建構期）提供 config
- Plugin 可透過 `ctx.config` 訪問 runtime config
- **始終正常工作**：無版本缺陷

---

## 3. 兩條路徑的歷史

### Path A 的 24 版本斷裂

| 版本 | plugin-resolver.ts::factory() | 狀態 |
|------|:-----------------------------:|:---:|
| v0.14.0-beta | `factory()` (無參數) | 斷裂 |
| v0.20.0-beta | `factory()` | 斷裂 |
| v0.24.0-beta | `factory()` | 斷裂 |
| v0.30.0-alpha | `factory()` | 斷裂 |
| v0.38.0-alpha | `factory()` | 斷裂 |
| v0.43.0-alpha | `factory()` | 斷裂 |
| **v0.44.0-alpha** | **`factory(ref.config)`** | **修復（Fix 12d）** |

### Path B 的持續正常

`agent-core.ts::getPluginContext()` 自 v0.14.0-beta 起即正確傳遞 `pluginRef?.config`。

### PluginRef.config 型別 vs 實作的時差

`PluginRef.config` 型別欄位在 SDK（`packages/sdk/src/types/agent.ts:192`）**自 v0.14.0-beta 即存在**。型別與實作存在 **24 版本的時差**。

這是一個典型的 "spec-implementation drift"：SDK 承諾了 API，但 runner 實作了另一套。

---

## 4. 為什麼 24 版本的斷裂沒有造成 bug？

### GUARDIAN 追溯調查結論：CLEAN

所有 38 個 plugin 的 factory 函數分析：
- **30 個 plugin** 無 config 參數（不受 Path A 影響）
- **8 個 plugin** 有 config 參數，但全部從 Path B 取得 config
- **0 個 plugin** 依賴 Path A

具體範例（gear-arbiter-dynamic，Plan43）：
```typescript
// v0.43.0-alpha, plugin index.ts
export function createGearArbiterDynamicPlugin(config?: GearArbiterDynamicConfig): IPlugin {
  // config 在 Plan43 時期：
  //   - coldStartGear: 從 ctx 取得（Path B）
  //   - phase3Config: 尚未實作（deferred to Plan44）
  return {
    factory: (ctx) => {
      const coldStartGear = ctx.config?.coldStartGear ?? 1;
      // ... Path B 正常工作
    }
  };
}
```

Plan44 引入 Phase3Config 並同時從 Path A 讀取，首次觸發斷裂問題。

### 為什麼直到 Plan44 才爆發？

- Phase3Config 是第一個在 **Stage 1 factory signature** 層讀取 config 的參數
- 之前所有 plugin 都在 **Stage 2 ctx.config** 讀取
- Plan44 的新設計意外選擇了 Path A，踩到地雷

---

## 5. 為什麼有兩條路徑？架構分析

### SUSSMAN 程式結構觀點

兩階段 factory 模式（two-stage factory pattern）本身是合理設計：

```
Stage 1: Module-level factory
  - 建構 IPlugin object（含 priority, skandha, hooks 等靜態屬性）
  - 純粹建構，無 runtime context
  
Stage 2: Plugin hooks factory
  - IPlugin.factory(ctx) 建構 IPluginHooks
  - 需 runtime context（bus, logger, ...）
```

兩階段對應兩個不同的 config consumption timing。

### 問題在於 "config 應該何時被 consumed"

- 若 config 用於**靜態屬性決定**（如 priority 計算） → Path A
- 若 config 用於**動態行為決定**（如 runtime decision） → Path B

絕大多數 plugin 使用 Path B，因此 Path A 斷裂長期無人察覺。

---

## 6. 設計準則（post Fix 12d）

### DP-1: Plugin 作者的預設

Plugin 作者**優先使用 Path B**（`ctx.config`）。理由：
- Path B 自始正常，無版本風險
- Runtime context 可用（bus, logger 等）

### DP-2: Path A 的使用時機

僅在以下情況使用 Path A：
- 需在 Stage 1 決定靜態屬性（極罕見）
- 明確理由記錄於 plugin 文件

### DP-3: Runner 的義務

`plugin-resolver.ts` 必須**同時支援兩條路徑**（post Fix 12d）。移除任一路徑 = 破壞現有 plugin。

### DP-4: 測試義務（Rule #68）

任何新 plugin 或 config 變更：
- **必須同時測試** Path A 和 Path B
- ENG-FAB v1.4 F-7 項要求

---

## 7. 安全分析

### 攻擊面評估

| 路徑 | 潛在風險 | 現狀 |
|------|---------|-----|
| Path A | Factory 接收 untrusted config，若未驗證可能注入 | Plugin 作者責任；SDK 提供 Zod schema 建議 |
| Path B | `ctx.config` 接收 untrusted config，同上 | 同上 |

### 雙路徑與安全

雙路徑**並未**增加安全風險。Config 的來源（`agent.json`）是單一信任來源。雙路徑只是傳遞機制的冗餘。

### Sensitive Config

對於敏感 config（API keys, HMAC secrets），**兩條路徑都應使用相同的安全處理**：
- 傳遞後即清除記憶體（SEC-002，Plan45 W0）
- 不寫入 logs
- Schema 驗證

---

## 8. 與 Rule #63 (L4 Config Activates) 的關係

Rule #63 要求「新 config 參數須有 factory-level 行為驗證」。

### L4 驗證涵蓋範圍（post Plan44）

| Config 參數 | Path | L4 測試 |
|------------|------|---------|
| coldStartGear (Plan43) | Path B | ✅ NC3 integration test (factory → evaluate()) |
| Phase3Config.enabled (Plan44) | Path A | ✅ NC3 W3-2 test |
| calibrationState (Plan44) | Path A | ✅ factory calibrationState test |

### Rule #68 補全

Rule #68 將 Rule #63 的 L4 驗證擴展為**強制雙路徑**。未來新 config 參數須證明其在兩條路徑的行為均符合 spec。

---

## 9. 對研究方法論的影響

### 追溯調查的啟示

GUARDIAN 的調查證實：
- 單路徑驗證不足
- 型別 vs 實作 drift 可潛伏數年
- 雙路徑架構本身合理，但需文件化

### Plan44 的意外功勞

Fix 12d 雖是「bug fix」，但**揭露了架構盲區**。從 MR-10（錯誤補回）+ MR-11（透明是抉擇）的角度，這是研究過程的正面結果。

---

## 10. 驗證現狀（Cycle 03-9）

| 項目 | 狀態 |
|------|:----:|
| Fix 12d 修復 Path A | ✅ TURING L1-L4 PASS |
| Path B 持續正常 | ✅ GUARDIAN 確認 |
| 雙路徑測試 Plan45 W0-3 | ⏳ 等 Plan45 交付 |
| Rule #68 應用 ENG-FAB v1.4 | ⏳ 03-10 起生效 |

---

## 11. 附錄: Plugin Factory 類型簽名演進

### v0.14.0-beta 至 v0.43.0-alpha

```typescript
// plugin-resolver.ts
function resolvePlugin(ref: { name: string; path?: string }): Promise<IPlugin> {
  // ...
  const factory = await loadModule(ref);
  return factory();  // ← config 從未傳入
}
```

### v0.44.0-alpha（Fix 12d 後）

```typescript
// plugin-resolver.ts
function resolvePlugin(
  ref: { name: string; path?: string; config?: Record<string, unknown> }
): Promise<IPlugin> {
  // ...
  const factory = await loadModule(ref);
  return factory(ref.config);  // ← Fix 12d
}
```

---

*Architecture Documentation #71 — Two-Path Config Propagation*
*Cycle 03-9 R1 (GUARDIAN) + R3 D6, 2026-04-15*
