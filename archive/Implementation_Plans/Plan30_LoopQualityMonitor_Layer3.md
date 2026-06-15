# Plan30: Default LoopQualityMonitor Plugin + Layer 3 Integration

- **Version**: v0.29.0-alpha → v0.30.0-alpha
- **SOP**: Standard
- **Cycle**: Scope absorbed into Plan29 + Plan32
- **Status**: ✅ Complete — Scope absorbed (monitor-loop-quality extracted as plugin in Plan32 W2, Layer 3 integration in Plan29)
- **Research Basis**: Cycle 02-6 D3-R5 (19/20 + 1 conditional), D2-R4, D3-R1~R4

---

## 目標

實作預設 LoopQualityMonitor Plugin（四維品質公式）並佈線 Layer 3 threshold 調整，使 Model Delta L0~L4 全部有實際信號路徑。同時提升事件型別常量與 ConfidenceAuditLog SDK 型別。

**成功標準**: Model Delta L0~L4 全部有實際佈線，以整合測試驗證。

---

## Wave 1: Default LoopQualityMonitor Plugin (~120 LOC)

### 新增檔案
| 檔案 | 說明 |
|------|------|
| `packages/core/src/plugins/default-loop-quality-monitor.ts` | DefaultLoopQualityMonitor: ILoopQualityMonitor 實作，4D 公式 + 6 事件訂閱 |

### 修改檔案
| 檔案 | 變更 |
|------|------|
| `packages/core/src/plugins/index.ts` | barrel export |
| `packages/core/src/agents/agent-core.ts` | 預設掛載 DefaultLoopQualityMonitor（若無 plugin 提供 monitor） |

### 設計決策
- **四維公式** (D3-R1, 20/20):
  - coherence = `1 - switchCount / (W - 1)`，輸入 `gear:switch`
  - efficiency = `successCount / proposedCount`（或 1.0），輸入 `action:proposed` + `tool:result`
  - convergence = `longestStreak / W`，輸入 `gear:switch`
  - stability = `1 - min(1, σ² / 0.25)`，輸入 `gear:arbiter_evaluated`
- **滑動窗口**: W=10, W_warmup=5（不足 5 事件時不發射品質分數）
- **複合分數**: Q = Σ(0.25 × d_i) ∈ [0, 1]，Phase 1 固定均等權重 (D3-R4, 20/20)
- **全部 O(1)/event, O(W) 空間**
- **6 事件訂閱** (D3-R2, 20/20): `gear:arbiter_evaluated`, `gear:switch`, `action:proposed`, `tool:result`, `loop:started`, `loop:finished`
- **退化原則**: 缺失事件 → 安全預設值（1.0 或 0.5），不報錯

---

## Wave 2: Layer 3 ManoAggregator Integration (~80 LOC)

### 修改檔案
| 檔案 | 變更 |
|------|------|
| `packages/core/src/mano/mano-aggregator.ts` | Layer 3 threshold adjustment: `loopQualityFn` 回調 + 方案 C 公式 |
| `packages/sdk/src/types/gear-arbiter.ts` | ManoAggregatorConfig 新增 `loopQualityAlpha?`, `monitorStalenessMs?`, `historicalConfidenceSize?` |

### 設計決策
- **方案 C 獨立雙通道** (D2-R4, 20/20):
  ```
  L2 → confidence:  confidence_adjusted = confidence + clampAuditDelta(delta)  [±0.05]
  L3 → threshold:   θ_adjusted = max(thresholdFloor, θ × (1 - α × q))         [α=0.10]
  routing = (confidence_adjusted > θ_adjusted) ? arbiter_gear : default_gear
  ```
- 兩通道獨立，無交叉項，BIBO 穩定可證
- **Layer 順序**: L4(VedanaEmergency) → L3(LoopQuality) → 比較
- **loopQualityFn**: pull 模式回調，注入 `createManoAggregator`
- **ManoAggregatorConfig 新欄位**:
  - `loopQualityAlpha`: 調整係數，預設 0.10
  - `monitorStalenessMs`: 品質信號最大年齡，預設 5000ms
  - `historicalConfidenceSize`: 歷史信心度環形緩衝區大小，預設 10
- **多 monitor**: 簡單平均 (D3-R3)
- **無 monitor**: q=0, L3 不生效（保守退化）
- **WIENER 約束**:
  - C-1: 歷史信心度只含原始 arbiter 信心度（audit 前）
  - C-2: Layer 2 clamp ±0.05 + Layer 3 α ≤ 0.10
  - C-3: 無 monitor 時 θ 不變

---

## Wave 3: Events + Extras + Types (~60 LOC)

### 新增檔案
| 檔案 | 說明 |
|------|------|
| `packages/sdk/src/types/confidence-audit-log.ts` | ConfidenceAuditLog 型別（inputConfidence, rawDelta, clampedDelta, wasClamped, reasoning, outputConfidence, result, auditDurationMs） |

### 修改檔案
| 檔案 | 變更 |
|------|------|
| `packages/sdk/src/types/events.ts` | AgentEventType 常量提升: 6 既有 Plan27b 事件字串 + `LOOP_QUALITY_UPDATED` |
| `packages/sdk/src/types/loop-quality-monitor.ts` | `MINIMAL_QUALITY_EVENTS` 常量 (= 6) |
| `packages/sdk/src/index.ts` | barrel exports (ConfidenceAuditLog, new AgentEventType constants) |
| `packages/core/src/plugins/default-loop-quality-monitor.ts` | 使用 `emitWithExtras()` 同時發出 `loopQuality:updated` + `audit:context_contribute` |

### 設計決策
- **extras 寫入** (D3-R3, 20/20): Monitor 使用 SDK `emitWithExtras()` 便利函數
  - `loopQuality:vector` — 四維分數物件
  - `loopQuality:composite` — 複合分數 Q
- **雙通道** (D2-R2 + D3-R3): ManoAggregator 訂閱 `loopQuality:updated` 供 `loopQualityFn`（pull）；訂閱 `audit:context_contribute` 供 extras（push）
- **per-routing-cycle 生命週期**: extras 不持久化
- **延遲一 tick 可接受**: 品質是歷史指標
- **ConfidenceAuditLog**: GUARDIAN D5 義務兌現 (D2-R3, 20/20)
  - reasoning 截斷 500 chars（Core 負責）
  - PII 淨化為 plugin 責任
  - 主通道: EventBus `audit:completed` 事件

---

## Wave 4 (Optional): PassthroughAuditor (~30 LOC)

### 新增檔案
| 檔案 | 說明 |
|------|------|
| `packages/core/src/plugins/passthrough-auditor.ts` | PassthroughAuditor: IConfidenceAuditor 實作，delta=0 純日誌 |

### 修改檔案
| 檔案 | 變更 |
|------|------|
| `packages/core/src/plugins/index.ts` | barrel export |

### 設計決策
- **Phase 0 Auditor** (D2-R5, 20/20): delta 永遠為 0，不修改信心度
- 純日誌功能：驗證 audit pipeline 端對端
- 作為參考實作與整合測試 fixture
- GUARDIAN 條件 (D3-R5): W3 已包含 ConfidenceAuditLog 型別

---

## 範圍摘要

| 指標 | 估計 |
|------|------|
| 生產程式碼 | ~260-290 LOC |
| 測試程式碼 | ~130 LOC |
| 合計 | ~390-420 LOC |

---

## 前向路徑

```
Plan30 (本計畫)  → Default LoopQualityMonitor + Layer 3 Integration
Plan31 (下一步)  → AuditContext 落地 + Default Auditor + Audit Trail (~350 LOC)
Plan32 (未來)    → VasanaEngine / SPC / Lyapunov 穩定性
```

---

*[來源: Cycle 02-6 D3-R5 共識, D2-R4 方案 C, D3-R1~R4 公式與規格]*
