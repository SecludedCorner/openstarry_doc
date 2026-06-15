# Plan28: IVolition v1 + Safety Hardening

> **狀態**: ✅ 已完成 (2026-03-05)
> **目標版本**: v0.28.0-alpha
> **基線版本**: v0.27.0-beta (1654 tests)
> **結果版本**: v0.28.0-alpha (1722 tests)
> **SOP**: Standard

## 目標

IVolition v1 能消費 RouteResult + riskCategory 做風險分級審議，SafetyMonitor.postRouteCheck() 保護 Gear 1 路徑，配置化 VedanaEmergency + Moha，VedanaEmergency 佈線至 ManoAggregator。

---

## 研究團隊修正規格來源

- `cycle02-4_Plan27a/deliver/plan27b_plan28_corrected_specs.md` §二 (P28-A/C/D)
- `cycle02-4_Plan27b/deliver/plan28_supplementary_specs.md` (P28-B DeliberationContext + SEC-027)
- `cycle02-4_Plan27b/todo/tenet_compliance_review_plan28.md` (2🔴 + 2🟡, all have fix specs)
- `cycle02-4_Plan27b/todo/plan28_review_recommendations.md` (2🔴 R1/R2 + 4🟡 Y1-Y4)

---

## Wave 1: SDK Type Foundations

### 1.1 DeliberationContext + Input 擴展
- **Modified**: `packages/sdk/src/types/volition.ts`
- Added `DeliberationContext` interface: `{ routeResult: RouteResult, actionHistory: readonly ActionRecord[] }`
- Extended `PlanDeliberationInput` + `ActionDeliberationInput` with optional `deliberationContext`

### 1.2 RouteResult.riskCategory
- **Modified**: `packages/sdk/src/types/gear-arbiter.ts`
- Added `readonly riskCategory?: RiskCategory` to RouteResult

### 1.3 VedanaEmergencyConfig
- **Modified**: `packages/sdk/src/types/vedana.ts`
- Added `VedanaEmergencyConfig` interface + `DEFAULT_VEDANA_EMERGENCY_CONFIG`
- Parameters: intensityThreshold, sustainedTicks, maxThresholdBoost, cooldownTicks

### 1.4 MohaConfig
- **Modified**: `packages/sdk/src/types/klesha.ts`
- Added `MohaConfig` interface + `DEFAULT_MOHA_CONFIG` (alphaM: 0.02, betaM: 5.0)

### 1.5 PluginHooks.volition
- **Modified**: `packages/sdk/src/types/plugin.ts`
- Added `volition?: IVolition` to PluginHooks (single slot, last plugin wins)

### 1.6 SDK Barrel Exports
- **Modified**: `packages/sdk/src/index.ts`
- Exported: DeliberationContext, VedanaEmergencyConfig, DEFAULT_VEDANA_EMERGENCY_CONFIG, MohaConfig, DEFAULT_MOHA_CONFIG

### 1.7 Tests (12 tests)
- `packages/sdk/src/types/__tests__/volition-plan28.test.ts`

---

## Wave 2: Core Infrastructure

### 2.1 ManoAggregator riskCategory Propagation
- **Modified**: `packages/core/src/mano/mano-aggregator.ts`
- Winning arbiter path: propagate `riskCategory: evaluation.riskCategory` to RouteResult

### 2.2 SafetyMonitor.postRouteCheck()
- **Modified**: `packages/core/src/security/safety-monitor.ts`
- Added `postRouteCheck(routeResult: RouteResult): RouteResult` — v1 passthrough
- Named `postRouteCheck` (not `postCheck`) to avoid future Doc 44 `postLLMCheck` conflict [R2]

### 2.3 SEC-027 Path Leak Fix
- **Modified**: `packages/core/src/execution/loop.ts`
- SHA-256 hash of workingDirectory, sliced to 16 chars for GearContext.sessionId

### 2.4 Tests (10 tests)
- `packages/core/src/mano/__tests__/mano-aggregator-plan28.test.ts` (5 tests)
- `packages/core/src/security/__tests__/safety-monitor-plan28.test.ts` (2 tests)
- `packages/core/src/execution/__tests__/loop-sec027.test.ts` (3 tests)

---

## Wave 3: VedanaEmergency + Moha Enhancement

### 3.1 VedanaEmergency Pure Function
- **New**: `packages/core/src/vijnana/vedana-emergency.ts`
- Pure stateful function: `createVedanaEmergencyState()` + `checkVedanaEmergency(vedana, state, config)`
- No magic numbers — all from VedanaEmergencyConfig
- State: `{ consecutiveDukkhaTicks, cooldownRemaining }`

### 3.2 Moha Action-Based Update
- **Modified**: `packages/core/src/vijnana/klesha.ts`
- Added `MohaConfig` constructor parameter
- New method: `updateFromAction(currentMoha, repetitionRatio): number`
- Diminishing marginal returns via saturation formula: `alphaM * ratio / (1 + betaM * current)`

### 3.3 Barrel Exports
- **New**: `packages/core/src/vijnana/index.ts`
- Exports: createVedanaEmergencyState, checkVedanaEmergency, VedanaEmergencyState, createVitakkaWatchdog, Moha, Drishti, Mana, Sneha, KleshaModulatedDispatcher, createDefaultKleshas

### 3.4 Tests (15 tests)
- `packages/core/src/vijnana/__tests__/vedana-emergency.test.ts` (9 tests)
- `packages/core/src/vijnana/__tests__/moha-update.test.ts` (6 tests)

---

## Wave 4: ExecutionLoop Wiring

### 4.1 postRouteCheck Wiring
- **Modified**: `packages/core/src/execution/loop.ts`
- After Phase 2.5 route(): `routeResult = deps.safetyMonitor.postRouteCheck(routeResult)`

### 4.2 DeliberationContext Wiring
- **Modified**: `packages/core/src/execution/loop.ts`
- Phase 3.5 deliberatePlan + Phase 4 deliberateAction: added `deliberationContext: routeResult ? { routeResult, actionHistory } : undefined`

### 4.3 Plugin-Loader Volition Registration
- **Modified**: `packages/core/src/infrastructure/plugin-loader.ts`
- Added `registeredVolition` slot + `getVolition()` getter
- Last plugin wins semantics

### 4.4 Agent-Core Volition DI [Y3]
- **Modified**: `packages/core/src/agents/agent-core.ts`
- Wraps plugin volition with neutral klesha/vedana getters
- Klesha/Vedana 不混入 volition，蘊歸屬清晰

### 4.5 ManoAggregator VedanaEmergency Wiring [R1]
- **Modified**: `packages/core/src/mano/mano-aggregator.ts`
- Added `vedanaFn?: () => ChannelVedana` + `vedanaEmergencyConfig` constructor params
- thresholdBoost 疊加至 effectiveBaseThreshold

### 4.6 Tests (12 tests)
- `packages/core/src/execution/__tests__/loop-plan28.test.ts` (4 tests)
- `packages/core/src/infrastructure/__tests__/plugin-loader-volition.test.ts` (4 tests)
- `packages/core/src/mano/__tests__/mano-vedana-emergency.test.ts` (4 tests)

---

## Wave 5: IVolition v1 Plugin (volition-rule-engine)

### 5.1 Plugin Structure
- **New**: `openstarry_plugin/volition-rule-engine/`
- Package: `@openstarry-plugin/volition-rule-engine@0.1.0-alpha`

### 5.2 Three-Layer Rule Engine
- **Hard layer**: mustAuditCategories — MUST audit (非 MUST veto) [Y2]
- **Soft layer**: Pattern match tool name (first match wins)
- **Heuristic layer**: Repetition detection (maxRepetitions + windowSize)
- Precedence: hard > soft > heuristic

### 5.3 Key Design Decisions
- riskCategory undefined 回退 [Y1]: `config.defaultRiskCategory ?? 'read_only'`
- Hard rule = MUST audit, 非 MUST veto (D5-R5 投票 16:4 否決無條件否決方案) [Y2]
- Default config: hardRules `['destructive', 'state_modifying']`, heuristicRules `{ maxRepetitions: 5, windowSize: 10 }`

### 5.4 Tests (18 tests)
- `volition-rule-engine/src/__tests__/rule-engine.test.ts` (16 tests)
- `volition-rule-engine/src/__tests__/plugin.test.ts` (2 tests)

---

## 研究團隊審閱修正追蹤

| # | 等級 | 問題 | 修正 | 確認 |
|---|------|------|------|------|
| R1 | 🔴 | VedanaEmergency 未佈線至 ManoAggregator | Wave 4 §4.5: vedanaFn + thresholdBoost wiring | ✅ |
| R2 | 🔴 | postCheck → postRouteCheck 命名修正 | 全域使用 postRouteCheck | ✅ |
| Y1 | 🟡 | riskCategory undefined 回退 | defaultRiskCategory 可配置化 | ✅ |
| Y2 | 🟡 | Hard rule = MUST audit 非 MUST veto | 規則引擎語義修正 | ✅ |
| Y3 | 🟡 | Agent-Core 不混入 Klesha/Vedana 到 volition | loop.ts 自行取得，agent-core wrap with neutral defaults | ✅ |
| Y4 | 🟡 | ActionRecord 標註為既有型別 | Wave 1 §1.1 import from gear-arbiter.ts | ✅ |

---

## 統計

| 類別 | 數量 |
|------|------|
| 修改檔案 | 12 |
| 新增檔案 | 15+ (含 plugin + tests) |
| SDK type 新增 | 5 interfaces + 2 defaults |
| Core 新模組 | 1 (vedana-emergency) |
| 新 plugin | 1 (volition-rule-engine) |
| 新增測試 | ~68 (1654 → 1722) |
| Workspace 數 | 29 → 30 |

---

## 關鍵設計決策

1. **Backward compat**: deliberationContext 為 optional field，v0 implementations 不受影響
2. **PluginHooks.volition**: Single slot, last plugin wins
3. **postRouteCheck v1**: Passthrough implementation, plugins can wrap/replace
4. **VedanaEmergency**: Pure stateful function, config-injected, 佈線至 ManoAggregator [R1]
5. **SEC-027**: SHA-256 hash of workingDirectory → 16 char hex string
6. **三層優先**: hard (śīla) > soft (upāya) > heuristic (smṛti)
7. **蘊歸屬**: volition = 識蘊, SafetyMonitor = 識蘊, VedanaEmergency = 受蘊→識蘊 bridge
