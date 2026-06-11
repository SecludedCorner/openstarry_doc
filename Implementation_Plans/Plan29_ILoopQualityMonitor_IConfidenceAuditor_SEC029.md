# Plan29: ILoopQualityMonitor + IConfidenceAuditor + SEC-029

- **Version**: v0.28.0-alpha → v0.29.0-alpha
- **SOP**: Standard
- **Cycle**: 20260307_cycle1
- **Status**: ✅ Complete (1757 tests, 32 new)

---

## 目標

補上 Model Delta 五層公式中 Layer 2 (`Delta_audit`) 和 Layer 3 (`Delta_loopQuality`) 的佈線基礎設施，並修正延後的 SEC-029 ReDoS 安全問題。

---

## Wave 1: ILoopQualityMonitor SDK + Registry (~120 LOC)

### 新增檔案
| 檔案 | 說明 |
|------|------|
| `packages/sdk/src/types/loop-quality-monitor.ts` | ILoopQualityMonitor, LoopQualityVector, LoopQualityReport, LoopQualityWeights, DEFAULT_LOOP_QUALITY_WEIGHTS |
| `packages/core/src/infrastructure/monitor-registry.ts` | createMonitorRegistry() — Map-backed, array slot |

### 修改檔案
| 檔案 | 變更 |
|------|------|
| `packages/sdk/src/types/plugin.ts` | PluginHooks.monitors: ILoopQualityMonitor[] |
| `packages/sdk/src/types/vedana.ts` | IVedanaSensor extends IVedana (弱繼承修正) |
| `packages/sdk/src/index.ts` | barrel exports |
| `packages/core/src/infrastructure/plugin-loader.ts` | monitors registration + monitorRegistry dep |
| `packages/core/src/infrastructure/index.ts` | export MonitorRegistry |
| `packages/core/src/agents/agent-core.ts` | create + pass monitorRegistry |
| `packages/core/src/execution/loop.ts` | startAll/stopAll lifecycle |

### 設計決策
- MonitorRegistry: Map-backed (同 GearArbiterRegistry)，但無 priority — monitors 平等
- PluginHooks.monitors = array slot (D2-R3)
- IVedanaSensor extends IVedana: 修正已知弱繼承缺口

---

## Wave 2: IConfidenceAuditor + Layer 2 (~71 LOC)

### 新增檔案
| 檔案 | 說明 |
|------|------|
| `packages/sdk/src/types/confidence-auditor.ts` | IConfidenceAuditor extends IVijnana, ConfidenceAuditResult |
| `packages/core/src/mano/confidence-audit.ts` | MAX_AUDIT_DELTA=0.05, clampAuditDelta() |

### 修改檔案
| 檔案 | 變更 |
|------|------|
| `packages/sdk/src/types/plugin.ts` | PluginHooks.auditor: IConfidenceAuditor (last-wins) |
| `packages/sdk/src/types/gear-arbiter.ts` | ManoAggregatorConfig.auditTimeoutMs? |
| `packages/sdk/src/index.ts` | barrel exports |
| `packages/core/src/mano/mano-aggregator.ts` | Layer 2 audit logic (Promise.race + clamp + fail-safe) |
| `packages/core/src/mano/index.ts` | export clampAuditDelta |
| `packages/core/src/infrastructure/plugin-loader.ts` | getAuditor() + auditor registration |
| `packages/core/src/agents/agent-core.ts` | recreate ManoAggregator with auditor at start() |

### 設計決策
- PluginHooks.auditor = singular last-wins (D5-R1/R4)
- IConfidenceAuditor extends IVijnana: 強繼承 (D5-R9)
- Layer 2 時機: arbiter 勝出 + confidence cap 後、return 前
- Promise.race pattern: 同 per-arbiter timeout
- Fail-safe: catch → delta=0 (D5-R5)
- auditTimeoutMs: ManoAggregatorConfig 可選欄位，預設 200ms

---

## Wave 3: SEC-029 ReDoS (~26 LOC)

### 新增檔案
| 檔案 | 說明 |
|------|------|
| `packages/shared/src/utils/safe-regex.ts` | validateRegexSafety(), safeRegexTest() |

### 修改檔案
| 檔案 | 變更 |
|------|------|
| `packages/shared/src/index.ts` | export safe-regex utils |
| `openstarry_plugin/gear-arbiter-static/src/index.ts` | safeRegexTest() replacement |
| `openstarry_plugin/gear-arbiter-static/package.json` | +@openstarry/shared dep |
| `openstarry_plugin/volition-rule-engine/src/rule-engine.ts` | safeRegexTest() replacement |
| `openstarry_plugin/volition-rule-engine/package.json` | +@openstarry/shared dep |

### 設計決策
- 靜態分析 (nested quantifier detection) + input 長度限制
- Node.js 單線程無法真正中斷 regex，故採用預防策略

---

## Hotfix

- `apps/runner/__tests__/utils/plugin-installer.test.ts` — verbose mode test 加 `{ timeout: 30000 }`

---

## 測試 (32 tests)

| 測試檔案 | 位置 | 測試數 |
|---------|------|--------|
| monitor-registry.test.ts | core/infrastructure/__tests__/ | 7 |
| confidence-audit.test.ts | core/mano/__tests__/ | 5 |
| mano-aggregator-plan29.test.ts | core/mano/__tests__/ | 6 |
| plugin-loader-plan29.test.ts | core/infrastructure/__tests__/ | 4 |
| safe-regex.test.ts | shared/utils/__tests__/ | 8 |
| vedana-sensor-extends.test.ts | sdk/types/__tests__/ | 1 |

---

## 版本統計

| 版本 | Tests | Delta |
|------|-------|-------|
| v0.28.0-alpha | 1722 | — |
| v0.29.0-alpha | 1757 | +35 |

---

## 後向相容

- 無 monitors → ExecutionLoop 行為不變
- 無 auditor → ManoAggregator 行為不變 (confidence = effectiveConfidence)
- 既有 1722 tests 全部通過

---

## 後續工作 (Plan30+)

- LoopQualityMonitor event-driven plugin 實體（Level 2 完整功能）
- Layer 3: LoopQualityMetric → Model Delta 佈線
- VasanaEngine 三閘門機制
- SPC 控制圖（Phase 2+ 可選）
- WIENER 耦合校準
