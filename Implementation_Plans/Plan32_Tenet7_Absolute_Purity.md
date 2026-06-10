# Plan32: Tenet #7 絕對純淨 + Tenet #9 部分修復

- **Version**: v0.31.0-alpha → v0.32.0-alpha
- **SOP**: Standard
- **Cycle**: 02-8_final (Consolidated)
- **Status**: ✅ Complete (2026-03-11, Cycle 20260311_cycle1, 1803 tests)
- **Research Basis**: Cycle 02-8 D5 + 02-8_a D1~D3 + 02-8_b E1_v2 + 02-8_c Wave 6 修正
- **Complete Spec**: `research record/cycle02-8_final/deliver/plan32_final_spec.md`

---

## 目標

達成 Tenet #7「微內核與絕對純淨」的完全合規，並部分修復 Tenet #9「可插拔的記憶策略」。Plan32 完成後，Core 不含任何 policy 常量、任何自動掛載組件、任何 fallback 預設值、以及任何硬編碼策略。所有策略決策外部化至 SDK defaults 和使用者配置；所有內建組件和 context manager 提取為通過 PluginLoader 載入的正式 plugin。

---

## 總覽

| Wave | 修正 | 宣言 | 預估 LOC |
|------|------|------|---------|
| Wave 1 | AC-1: 移除 ThresholdAuditor 自動掛載 | #7 | ~10 |
| Wave 2 | AC-2: 提取 3 個內建組件為 plugin 包 | #2, #7 | ~200 |
| Wave 3 | AC-4 P0: Policy 值遷移（20 值） | #7 | ~150 |
| Wave 4 | AC-4 P1+P2: 剩餘 policy 值遷移（34 值） | #7 | ~100 |
| Wave 5 | Schema 擴展（audit:completed payload） | #8 (C1 支援) | ~20 |
| Wave 6 | AC-8: Context manager 提取 (修正版) | #9 | ~108 |
| **總計** | | | **~588 production + ~280 tests = ~868 total** |

---

## Master 裁定

1. **Tenet #7 絕對純淨**: Core 必須不含任何 policy 常量、任何自動掛載組件、任何 fallback 預設值
2. **MR-6**: 所有 policy 值必須從 Core 移除；Core 必須含零 policy 常量
3. **Context manager**: 缺席時 `throw Error()`（Required 級別），不使用 fallback

---

## Wave 1: AC-1 — 移除 ThresholdAuditor 自動掛載

**問題**: `agent-core.ts` 中 `pluginAuditor ?? createThresholdAuditor()` 違反 Tenet #7。

**修正**:
- 刪除 `import { createThresholdAuditor }`
- 刪除 `?? createThresholdAuditor()` fallback
- 直接傳遞 `pluginAuditor`（可能為 undefined）
- Auditor 缺席時 delta=0（Optional-degraded 級別）

**決議**: D1-Q1-R1 (24/24 全票，Option A)

---

## Wave 2: AC-2 — 提取內建組件為 Plugin 包

**問題**: Core 內含 ThresholdAuditor、PassthroughAuditor、LoopQualityMonitor 的實作代碼。

**修正**:
- 3 個新 plugin 包：
  - `@openstarry-plugin/auditor-threshold`
  - `@openstarry-plugin/auditor-passthrough`
  - `@openstarry-plugin/monitor-loop-quality`
- 刪除 `packages/core/src/plugins/` 目錄
- 所有三個 plugin 宣告 `skandha: 'vijnana'`
- `monitorRegistry.startAll(bus)` → 修復為在 `start()` 中呼叫，`stopAll()` 在 `stop()` 中呼叫

**決議**: D2-Q1-R1 (23/24, Option B 獨立包) + D2-Q3-R1 (24/24 MUST-FIX)

---

## Wave 3: AC-4 P0 — 20 個高優先 Policy 值遷移

**問題**: Core 中有 53 個 policy 常量硬編碼。

**分類方法**: BABBAGE 連續性測試 (continuity test) — 正式的機制/策略分類準則：

> 若移除一個常量後，系統在數學上失去定義（除以零、NaN 傳播等），則該常量是**機制** (mechanism)；否則是**策略** (policy)。

**機制值（保留在 Core 中，共 8 個）**:
- NaN guard
- Forced gear confidence=1
- G-1 confidence=0
- 2× version discriminant
- clamp01 bounds
- coherence=1.0（乘法恆等元）
- efficiency=1.0（乘法恆等元）

**配置架構**: SUSSMAN 三層配置:
```
IAgentConfig (使用者配置)  →  SDK DEFAULT_* (SDK 預設)  →  Core zero-default
```

P0 20 個值 = 校準 + 安全關鍵值，優先遷移。

**決議**: D3-Q1-R1 ~ D3-Q8-R1 (全部通過)

---

## Wave 4: AC-4 P1+P2 — 剩餘 34 個 Policy 值遷移

P1 (19 值) + P2 (15 值)，按相同的三層配置架構遷移。

---

## Wave 5: Schema 擴展

`audit:completed` EventBus 事件新增兩個 P0 必填欄位：
- `riskCategory: RiskCategory` — 風險分類
- `thresholdAtDecision: number` — 決策時的閾值

為 C1 校準 (Doc 50) 提供必要的資料基礎。

---

## Wave 6: AC-8 — Context Manager 提取 (02-8_c 修正版)

**問題**: Context manager (createContextManager) 硬編碼在 Core 中。

**02-8_b 原始設計** (已否決): `pluginContextManager ?? createContextManager()` — 與 Wave 1 矛盾。

**02-8_c 修正設計**:
- 新 plugin: `@openstarry-plugin/context-sliding-window`
- Core 移除 `createContextManager()`
- Context manager 缺席時 `throw Error('Context manager plugin required')`
- **Required 級別**（三級關鍵性模型）

**三級關鍵性模型**:

| 級別 | 缺席行為 | 範例 |
|------|---------|------|
| Required | throw Error() at start() | Context manager |
| Optional (degraded) | 中性值 (delta=0) | Auditor、monitors |
| Optional (no-effect) | 功能不可用 | VedanaSensors |

---

## 驗收標準

| AC | 標準 | 測試方法 |
|----|------|---------|
| AC-1 | Core 無 auto-mount fallback | grep -r 'createThresholdAuditor' → 0 results |
| AC-2 | packages/core/src/plugins/ 不存在 | ls 驗證 |
| AC-3 | Core policy 常量 = 0（僅含 8 mechanism 值） | automated scan |
| AC-4 | context-sliding-window 通過 PluginLoader 載入 | integration test |
| AC-5 | 無 auditor 時 delta=0、無 error | unit test |
| AC-6 | 無 context manager 時 throw Error | unit test |
| AC-7 | audit:completed 含 riskCategory + thresholdAtDecision | schema validation |

---

## 4 個新 Plugin 包

| 包名 | skandha | 級別 |
|------|---------|------|
| `@openstarry-plugin/auditor-threshold` | vijnana | Optional (degraded) |
| `@openstarry-plugin/auditor-passthrough` | vijnana | Optional (degraded) |
| `@openstarry-plugin/monitor-loop-quality` | vedana, samjna, vijnana | Optional (degraded) |
| `@openstarry-plugin/context-sliding-window` | — | Required |

---

*完整工程規格請參閱 `research record/cycle02-8_final/deliver/plan32_final_spec.md`*
*紀錄時間：2026-03-10*

---

## 完成狀態 (Completion Status)

**Cycle**: 20260311_cycle1
**Date**: 2026-03-11
**All 6 Waves Complete**: ✅

### Wave 4 交付總結 (P1/P2 Value Migration)

- **P1 Values**: 19 values migrated to SDK DEFAULT_* constants
- **P2 Values**: 15 values migrated to SDK DEFAULT_* constants
- **Total Policy Values**: 34 values + 8 mechanism values = 42 total values processed
- **SDK Constants Created**: 8 new DEFAULT_* constants (Core zero-default architecture)

### IAgentConfig 擴展

新增三個執行控制欄位：
- `execution: ExecutionConfig` — 執行模式和超時設定
- `kleshaFilter?: string[]` — Klesha 過濾清單（可選）
- `sandbox?: SandboxConfig` — 沙箱執行配置

### Build & Test 成績

- **Build**: 34 projects PASS
- **Test**: 1803 tests passed
- **Purity**: All microkernel compliance checks PASS
- **Version**: v0.32.0-alpha

### BABBAGE 分類驗證

機制值（8 個）正確保留在 Core：
1. NaN guard
2. Forced gear confidence=1
3. G-1 confidence=0
4. 2× version discriminant
5. clamp01 bounds
6. coherence=1.0（乘法恆等元）
7. efficiency=1.0（乘法恆等元）
8. Plugin context proxy required behavior

### SUSSMAN 三層配置模式驗證

SUSSMAN 三層配置架構在所有 6 Wave 中完全驗證：
```
IAgentConfig (使用者配置)
  ↓
SDK DEFAULT_* (SDK 預設)
  ↓
Core zero-default (核心零預設)
```

該模式已在所有子系統（Auditor、Monitor、Context Manager、Policy Values）中獲得驗證，成為標準化的配置模式。
