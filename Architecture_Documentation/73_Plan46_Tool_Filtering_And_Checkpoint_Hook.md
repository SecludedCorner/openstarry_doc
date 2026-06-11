# 73. Plan46 — Tool Filtering & K-3 Checkpoint Hook

`[Cycle 03-10 新增]`

> **來源**: Cycle 03-10 Plan46 交付
> **核心學者**: VITRUVIUS (#3), KERNEL (#10), GUARDIAN (#11), SUSSMAN (#22), ARCHIMEDES (#16)
> **相關規則**: Rule #45 (PluginHooks 解凍規程), Rule #59 (HYPOTHESIS 標註), Rule #68 (Two-Path Verification)
> **相關文件**: Doc 71 (Two-Path Config Propagation), Doc 72 (WIENER L2+L3)

---

## 1. 概述

Plan46 落地三項 deferred items：

1. **Tool-Level Capability Filtering** — 在 runner 層以工廠包裝模式（factory-wrapper）強制執行 `PluginCapabilities.allowedTools`，**0 Core 修改（Tenet #7 / C46-1）**。
2. **K-3 SDK Framework Hook** — 在 `PluginHooks` 新增 `onCheckpoint` / `onRestore` 兩個 optional 方法與 `PluginSnapshot` 型別。Rule #45 re-freeze 經 R3 D10-Q32 24/24 一致通過；交付後立即再凍結。
3. **Checkpoint Orchestration** — 在 runner 層新增 `CheckpointManager` 與 `capturePluginHooks` wrapper，聚合並轉發 checkpoint/restore 呼叫。

此文件記錄兩項架構決定的細節與 invariant，避免未來重犯或誤解。

---

## 2. Tool Filtering：為什麼用 Factory Wrapper 而不是擴充 Core

### 2.1 對比現況

既有的 provider 過濾（`allowedProviders`）落在 Core：`sandbox-manager.ts:474–491` 在 ProviderRegistry 封裝時即鎖定白名單。Plan46 **沒有跟隨這個模式**，原因：

- **C46-1**：Plan46 硬性限制不得修改 Core；
- **Tenet #2（Everything is a Plugin）**：tool 過濾屬於 runner 配置，放在 Core 反而把一項「runtime 策略」硬編碼進 minimal kernel；
- **Path A/B 可驗證性**：放在 runner 可由 integration test 直接驗證 manifest → ctx.tools 的完整傳遞。

### 2.2 包裝鏈

```
resolvePlugins()
  → wrapPluginWithToolFilter(plugin, onDenied)   // W1
  → capturePluginHooks(plugin, hookMap)           // W2
  → core.loadPlugin(plugin)
```

兩層包裝都只是 `IPlugin → IPlugin`，不修改 Core，不改變 load 順序。

### 2.3 C46-4 Default Permissive

`wrapPluginWithToolFilter` 在 `allowedTools` 為 `undefined` 或空陣列時**直接回傳原 plugin 物件**（`===` 相等），因此：

- 過往所有 plugin（從 v0.1.0 到 v0.45.0-alpha）不需要改動 manifest；
- 向後相容不僅於行為層面，型別與物件 identity 層面也保持不變；
- 只有**顯式宣告** `allowedTools` 的 plugin 才會被過濾。

### 2.4 Proxy 語意

過濾發生在 `ctx.tools` 本身（access time），不是在 tool registry 源頭：

| 操作 | 允許 id 行為 | 被阻擋 id 行為 |
|------|--------------|-----------------|
| `list()` | 回傳 `t.id ∈ allowedSet` 的子集 | 不包含 |
| `get(id)` | 回傳 `ITool` 或 `undefined`（若 registry 本身無此 tool） | 回傳 `undefined` + 觸發 `onDenied` |

`onDenied` 事件格式（runner wire 到 `core.bus`）：

```json
{
  "type": "audit:capability_denied",
  "plugin": "<manifest.name>",
  "tool": "<requested id>",
  "allowedTools": ["<list copy>"],
  "timestamp": "<ISO 8601>"
}
```

### 2.5 Rule #68 Two-Path Verification

Plan46 W1 的 config 只有一條實質流路（manifest → runner → ctx.tools），但仍可用 Rule #68 的 two-path 角度驗證：

- **Path A (Manifest)**: `plugin.manifest.capabilities.allowedTools` 宣告於靜態 metadata。
- **Path B (Runtime)**: factory 收到的 `ctx.tools` accessor 反映同一份清單。

`apps/runner/__tests__/integration/tool-capability-filtering.test.ts` 內的
`Path A (manifest) → Path B (ctx.tools) filtered` test case 用單一斷言覆蓋兩端，符合 Rule #68。

### 2.6 未被包裝的路徑

兩類 plugin 不會走這個過濾：

1. 未宣告 `allowedTools` 或空陣列（C46-4 default permissive）；
2. 在 sandbox 之下以 RPC 方式取得 tools — 沙箱 worker 內的 `ctx.tools` proxy 目前是 Plan46 範圍之外，未來 plan 如需一致化可擴充 `createSandboxPluginContext` 內部亦套用相同 wrapper。

---

## 3. K-3 SDK Framework Hook

### 3.1 PluginSnapshot 型別

```ts
export interface PluginSnapshot {
  readonly pluginName: string;
  readonly schemaVersion: number;
  readonly state: Readonly<Record<string, unknown>>;
  readonly timestamp: number;
}
```

- `pluginName` 為 wire identity（與 `manifest.name` 未必一致；例如 `spc-safety-gate`、`state-tracker`、`shewhart-chart` 是元件而非 plugin package name）。
- `schemaVersion` 是**元件層級**的版本號，不跟隨 SDK / runtime 版本。
- `state` 對框架透明（opaque），只由元件的 `onCheckpoint` / `onRestore` 解讀。
- `timestamp` 以 `Date.now()` 記錄建立時刻，方便 replay / audit。

### 3.2 PluginHooks 新增方法

```ts
onCheckpoint?: () => PluginSnapshot | null;
onRestore?: (snapshot: PluginSnapshot) => void;
```

- `onCheckpoint` **合約上不應拋錯**；若內部失敗請回 `null`。框架仍以 `try/catch` 防禦。
- `onRestore` 允許拋錯；框架捕捉後 plugin 回到 fresh state（與 SafetyGate / StateTracker 既有 `fromSnapshot` 語意一致）。
- 兩者皆為 optional — 沒有 snapshot 能力的 plugin 毋需變動。

### 3.3 Rule #45 Re-freeze

**凍結範圍解除**：僅限 `PluginHooks` interface 的**加法**（2 個 optional 方法）＋ 1 個新型別 `PluginSnapshot`。

**授權**：R3 D10-Q32，24/24 一致通過（見 `research_team_suggestion/cycle03-10/deliver/R3_decision_log.md`）。

**N=0 驗證**：Plan46 實作前 grep `onCheckpoint|onRestore` 於兩個 monorepo 無命中；BCT（Backward Compat Test）trivially 滿足。

**立即再凍結**：Plan46 交付後 `PluginHooks` 立即 RE-FROZEN，任何新增需另行走 Rule #45 解凍流程。

### 3.4 元件遷移

| 元件 | 檔案 | onCheckpoint | onRestore |
|------|------|--------------|-----------|
| SafetyGate (L3) | `spc-monitor/safety-gate.ts` | 包裝 `serialize()`，pluginName=`spc-safety-gate` | 驗證 pluginName + schemaVersion，委派 `fromSnapshot()`，將還原狀態反寫回此實例 |
| StateTracker | `gear-arbiter-dynamic/state-tracker.ts` | 包裝 `serialize()`，pluginName=`state-tracker` | 委派 `fromSnapshot()` 取得新 tracker 後將其狀態 in-place 複製到 `this` |
| ShewhartChart (L1) | `spc-monitor/shewhart-chart.ts` | 以單欄位 `state.windows: JSON-string` 包裝 `serialize()`，pluginName=`shewhart-chart` | 驗證 pluginName + schemaVersion 後委派 `deserialize()` — **C46-3：SEC-003 驗證路徑全程保留** |

### 3.5 為何採「in-place mutation」而非 `new Instance`？

SafetyGate / StateTracker / ShewhartChart 在 plugin hooks 中是被建構一次、長期持有的物件，其參照分佈於 SpcMonitor / DynamicArbiter 內部。若 `onRestore` 回傳新 instance，呼叫端必須全面 swap 引用，風險極高。因此三者採 **in-place mutation**：

- SafetyGate：手動複製 `lastTriggerMs` + `shadowDecisionsSinceTrigger`。
- StateTracker：`fromSnapshot()` 產生新 tracker，再經 `serialize()` 將三張 Map 內容逐一搬回 `this`。
- ShewhartChart：`deserialize()` 產生新 chart，再將內部 `windows` Map 搬回 `this`；保留相同參照以利 SpcMonitor 繼續觀察。

---

## 4. CheckpointManager 與 Hook-Capture Wrapper

### 4.1 CheckpointManager 介面

```ts
createCheckpointManager(plugins: Map<string, PluginHooks>): {
  checkpoint(): Map<string, PluginSnapshot>;
  restore(snapshots: Map<string, PluginSnapshot>): void;
}
```

- `checkpoint()`：迭代 plugin → 呼叫 `onCheckpoint()` → 收集非 null 結果 → try/catch 防禦。
- `restore()`：依 plugin name 索引 snapshots，對應呼叫 `onRestore()`；plugin 名稱不存在時 no-op，對映 plugin 拋錯則 catch（fresh-state fallback）。

### 4.2 Hook-Capture Wrapper 的必要性（CONDITIONAL-2 解）

`core.loadPlugin()` 簽名為 `Promise<void>`：

```ts
async loadPlugin(plugin: IPlugin): Promise<void>
```

Core 不把 `PluginHooks` 回傳給呼叫者，因此 runner 無法以自然方式得到 hook references 來交給 CheckpointManager。解法：

```ts
export function capturePluginHooks(
  plugin: IPlugin,
  hookMap: Map<string, PluginHooks>,
): IPlugin {
  return {
    manifest: plugin.manifest,
    factory: async (ctx) => {
      const hooks = await plugin.factory(ctx);
      if (hooks.onCheckpoint || hooks.onRestore) {
        hookMap.set(plugin.manifest.name, hooks);
      }
      return hooks;
    },
  };
}
```

- **只**截取具 `onCheckpoint` / `onRestore` 的 plugin，其餘照常不進入 map；
- 對 Core 透明：回傳值與無包裝版本逐字相同；
- 與 W1 tool-filter wrapper 同一檔，語意可組合（`wrap → capture → load`）。

### 4.3 當前尚未串接到 daemon resume

Plan46 `start.ts` 內以 `const _checkpointMgr = createCheckpointManager(hookMap); void _checkpointMgr;` 保留建構但不曝光。真正的 checkpoint/restore 觸發時機（daemon session resume、CLI resume flag 等）將由 Plan47 決定後再接線。此決定記錄於 delivery_report §13。

---

## 5. 測試覆蓋

| Layer | 測試檔案 | 重點 |
|-------|---------|------|
| SDK build | `pnpm build` | `PluginSnapshot` + `onCheckpoint/onRestore` 型別可編譯、re-export 正確 |
| W1 integration | `apps/runner/__tests__/integration/tool-capability-filtering.test.ts` | 10 tests — 含 Path A → Path B 驗證 |
| W2 orchestration | `apps/runner/__tests__/integration/checkpoint-manager.test.ts` | 9 tests — 含 throw 隔離、null skip、round-trip |
| SafetyGate hook | `spc-monitor/__tests__/safety-gate.test.ts` | 3 tests — snapshot identity、pluginName mismatch rejection、round-trip |
| ShewhartChart hook | `spc-monitor/__tests__/shewhart-deserialize.test.ts` | 6 tests — **含 C46-3 SEC-003 corrupt-JSON rejection** |
| StateTracker hook | `gear-arbiter-dynamic/__tests__/dynamic-arbiter.test.ts` | 3 tests — onCheckpoint/onRestore + pluginName rejection |

workspace 整體 2580 passed / 3 skipped，+155 新 tests / +14 新檔案。

---

## 6. 反例：這個架構無法處理什麼

為免未來再被要求重複論證，記下已知限制：

- **跨機器 checkpoint**：`state: Record<string, unknown>` 可序列化，但 framework 不保證 plugin 內部是否藏了 non-serializable 物件（閉包、File handle、WebSocket 等）。Plan46 僅涵蓋 JSON-safe 的既有三元件。
- **schemaVersion 遷移**：`onRestore` 收到舊 schemaVersion 時一律 throw，框架退到 fresh state。沒有 migration step / transformer — 任何跨版本 schema 遷移須由 plugin 自行處理或走 data-migration script。
- **並行 checkpoint**：CheckpointManager 假設 `onCheckpoint` 為同步且快速，不處理鎖定 / 配對語義。多 agent / daemon cluster 情境需由上層（例如 alaya runtime）協調。
- **Sandbox 之下**：當 plugin 於 sandbox worker 執行時，`onCheckpoint` / `onRestore` 會跨 RPC。Plan46 沒有針對此路徑做額外 wrapper；若需要需於 sandbox RPC 層新增 serialization 策略，是未來 plan 的工作。

---

*Architecture Doc 73 — Plan46 Tool Filtering & Checkpoint Hook, Cycle 03-10, 2026-04-17.*
