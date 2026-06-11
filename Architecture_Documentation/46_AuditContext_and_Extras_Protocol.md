# 46. AuditContext 與 Extras 協議 (AuditContext and Extras Protocol)

`[Cycle 02-6 新增]` `[Cycle 02-8 更新: Plan32 影響 — auto-mount 移除 + plugin 提取 + schema 擴展]`

> Cycle 02-6 D2 辯論五項決議全部通過 (20/20 或 19/20)。本文件定義 IConfidenceAuditor 的完整輸入上下文、Plugin 間資料共享的 extras bag 協議、以及防止回饋迴路的 WIENER 約束。
>
> **實作狀態**: IConfidenceAuditor SDK 介面 + ManoAggregator 佈線 + clampAuditDelta ±0.05 已實作 (Plan29, v0.29.0-alpha)。AuditContext 型別 + ThresholdAuditor + Audit Trail JSONL 已實作 (Plan31, v0.31.0-alpha)。**Plan32 將提取 ThresholdAuditor/PassthroughAuditor 為獨立 plugin 包，移除 auto-mount fallback，擴展 audit:completed schema。**
> **核心學者**: WIENER (#12), GUARDIAN (#11), ATHENA (#5), PASCAL (#19), TURING (#17), ARCHIMEDES (#16)
> **依賴文件**: #42 IGearArbiter 介面規格, #43 認知迴路品質監控, #44 安全架構總覽, #45 五蘊 OOP 架構

---

## 1. 概述

AuditContext 是 IConfidenceAuditor.audit() 方法的輸入參數，提供豐富的決策上下文，使 auditor 能做出更精準的信心度調節判斷。extras bag 是 Plugin 間透過 EventBus 共享非結構化資料的通用管道。

### 設計動機

Plan29 (v0.29.0-alpha) 實作的 IConfidenceAuditor 接收的上下文有限（僅 RouteResult）。Cycle 02-6 研究發現，有效的審計需要更豐富的上下文——arbiter 推理過程、歷史信心度、品質指標——但這些資料的傳遞必須避免引入回饋迴路。

### 核心設計原則

1. **資訊豐富但單向**: AuditContext 提供豐富的上下文，但 auditor 的輸出（delta）不回流到 AuditContext
2. **Core 組裝、Plugin 貢獻**: 核心欄位由 ManoAggregator 組裝，extras 由 Plugin 透過 EventBus 貢獻
3. **BIBO 穩定**: Layer 2 (confidence) 和 Layer 3 (threshold) 作用於不同變量，無交叉項

---

## 2. AuditContext 型別定義 [D2-R1, 20/20]

```typescript
export interface AuditContext {
  readonly version: 1;                              // 字面量型別，未來版本升級用
  readonly sparshEvent: SparshEvent;                // 觸發本次路由的原始輸入事件
  readonly gearEvaluation: GearEvaluation;          // IGearArbiter chain 的完整評估結果
  readonly riskCategory: RiskCategory | undefined;  // 風險分類（若可用）
  readonly routeResult: RouteResult;                // 審計前的路由決策快照
  readonly historicalConfidence?: readonly number[]; // 環形緩衝區，最近 N 次原始 arbiter 信心度
  readonly extras: ReadonlyMap<string, unknown>;     // 泛型擴展袋
}
```

### 欄位說明

| 欄位 | 必填 | 來源 | 說明 |
|------|------|------|------|
| version | Yes | Core | 字面量 1，供未來向後相容 |
| sparshEvent | Yes | Core | 觸事件，IConfidenceAuditor 可據此判斷輸入語境 |
| gearEvaluation | Yes | Core | arbiter 完整推理（含齒輪號、信心度、推理文字） |
| riskCategory | Yes | Core | 風險等級（destructive/state_modifying/read_only/informational），undefined 若未分類 |
| routeResult | Yes | Core | 路由結果快照（齒輪選擇 + 信心度，審計前） |
| historicalConfidence | No | Core | 環形緩衝區，預設保留最近 10 筆（可配置 historicalConfidenceSize） |
| extras | Yes | Plugin | ReadonlyMap，由 Plugin 透過 EventBus 貢獻的非結構化資料 |

### historicalConfidence 約束 (WIENER C-1)

歷史信心度陣列**僅含原始 arbiter 信心度**，不含經 audit 調整後的值。這防止 auditor 觀察到自己過去的調整結果，避免正回饋迴路。

```typescript
// CORRECT: 原始 arbiter 信心度
historicalConfidence: [0.72, 0.68, 0.75, 0.71, ...]

// WRONG: 不得包含 audit-adjusted 值
historicalConfidence: [0.77, 0.73, 0.80, ...]  // ← 違反 C-1
```

---

## 3. Extras Bag 協議 [D2-R2, 19/20]

### 3.1 寫入模式：雙事件 + SDK 便利函數

Plugin 透過 EventBus 貢獻 extras 資料：

1. Plugin 在自己的事件（如 `loopQuality:updated`）中發射業務資料
2. 同時發射 `audit:context_contribute` 事件，由 ManoAggregator 收集

SDK 提供 `emitWithExtras()` 便利函數，一次呼叫完成兩個事件：

```typescript
// SDK 便利函數
function emitWithExtras(
  bus: EventBus,
  primaryEvent: { type: string; payload: unknown },
  extras: Array<{ key: string; value: unknown }>
): void;

// 使用範例（LoopQualityMonitor）
emitWithExtras(bus,
  { type: 'loopQuality:updated', payload: report },
  [
    { key: 'loopQuality:vector', value: vector },
    { key: 'loopQuality:composite', value: compositeQ },
  ]
);
```

### 3.2 收集管道

ManoAggregator 訂閱 `audit:context_contribute` 事件，收集所有 extras 到 ReadonlyMap：

```
Plugin A → emitWithExtras() → audit:context_contribute {key, value}
Plugin B → emitWithExtras() → audit:context_contribute {key, value}
                                        ↓
                              ManoAggregator 收集
                                        ↓
                              AuditContext.extras: ReadonlyMap
                                        ↓
                              IConfidenceAuditor.audit(context)
```

### 3.3 讀取：型別安全存取器

```typescript
function getExtra<T>(
  extras: ReadonlyMap<string, unknown>,
  key: string,
  guard: (v: unknown) => v is T
): T | undefined;

// 使用範例
const quality = getExtra(ctx.extras, 'loopQuality:composite', isNumber);
if (quality !== undefined) {
  // type-safe: quality is number
}
```

### 3.4 限制與命名規範

| 規則 | 值 |
|------|---|
| 最大 key 數量 | 32 |
| 最大 key 長度 | 128 chars |
| key 格式 | `{category}:{name}` |
| 禁止前綴 | `audit:`, `core:`, `internal:` |
| 不可變性 | 淺凍結 + ReadonlyMap |

已知 key 命名：
- `loopQuality:vector` — LoopQualityVector (四維品質向量)
- `loopQuality:composite` — number (複合品質分數 Q ∈ [0, 1])

---

## 4. WIENER 回饋迴路約束 [D2-R1/R3]

WIENER (#12) 從控制理論出發，確立三條約束防止正回饋迴路：

| 約束 | 規則 | 防止的回饋路徑 |
|------|------|---------------|
| C-1 | historicalConfidence 僅含原始 arbiter 信心度 | auditor → historicalConfidence → auditor |
| C-2 | AuditContext 不含 previousAuditResult | auditor → context → auditor |
| C-3 | extras key 禁止 `audit:` 前綴 | auditor → extras → auditor |

### BIBO 穩定性保證

Layer 2 和 Layer 3 作用於不同變量（confidence vs threshold），且各自有界：
- L2: clampAuditDelta(delta) ∈ [-0.05, +0.05]
- L3: α × q ∈ [0, 0.10]（q ∈ [0,1], α=0.10）

兩通道無交叉項 → BIBO (Bounded-Input, Bounded-Output) 穩定可證。

---

## 5. ConfidenceAuditLog [D2-R3, 20/20]

完整審計軌跡型別，記錄每次 audit 的完整輸入/輸出：

```typescript
export interface ConfidenceAuditLog {
  readonly inputConfidence: number;     // 審計前信心度
  readonly rawDelta: number;            // auditor 原始建議 delta
  readonly clampedDelta: number;        // 經 clamp 後的 delta
  readonly wasClamped: boolean;         // 是否被 clamp
  readonly reasoning: string;           // 審計理由（截斷 500 chars）
  readonly outputConfidence: number;    // 審計後信心度
  readonly result: 'adjusted' | 'unchanged' | 'error';
  readonly auditDurationMs: number;     // 審計耗時
}
```

### 發射通道

- **EventBus**: `audit:completed` 事件，payload 為 ConfidenceAuditLog
- **JSONL appender**: 可選的持久化寫入（Plan31+）
- **reasoning 截斷**: Core 負責截斷至 500 chars；PII 淨化為 plugin 責任

### GUARDIAN D5 義務兌現

Cycle 02-4 D5 辯論中，GUARDIAN (#11) 以「±0.05 clamp 可能不足」為由保留重新審議權利。Cycle 02-6 D2-R3 決議正式兌現此條件——ConfidenceAuditLog 提供完整的審計追蹤，使 GUARDIAN 不再保留重新審議 ±0.05 限幅的權利。

---

## 6. Model Delta 五層整合 — Option C [D2-R4, 20/20]

### 完整公式

```
θ_base + L1(Klesha) + L4(VedanaEmergency) = θ_intermediate
θ_adjusted = max(thresholdFloor, θ_intermediate × (1 - α × q))     ← L3 (LoopQuality)
confidence_adjusted = confidence + clampAuditDelta(audit.delta)      ← L2 (Audit)
routing = (confidence_adjusted > θ_adjusted) ? arbiter_gear : default_gear
```

### 各層職責

| Layer | 變量 | 機制 | 限幅 |
|-------|------|------|------|
| L0 | θ_base | 基礎閾值 (config) | — |
| L1 | θ | Klesha 增益排程 | [0.3, 0.9] |
| L2 | confidence | IConfidenceAuditor delta | ±0.05 |
| L3 | θ | LoopQualityMonitor q 值 | α=0.10 |
| L4 | θ | VedanaEmergency boost | — |

### L3 數據來源

- **loopQualityFn()** 回調（pull 模式）— ManoAggregator 主動呼叫
- **extras `loopQuality:composite`**（push 模式）— 供 auditor 參考

### 邊界條件

- 無 monitor 安裝: q=0, L3 不生效（保守退化）
- Monitor 過期 (>5000ms): q=0（同上）
- 多 monitor: 簡單平均

---

## 7. 新增事件型別

| 事件 | payload | 用途 |
|------|---------|------|
| `audit:context_contribute` | `{key: string, value: unknown}` | extras 通道 |
| `audit:completed` | ConfidenceAuditLog | 審計完成日誌 |

---

## 8. ManoAggregatorConfig 新增欄位

| 欄位 | 型別 | 預設值 | 決議 | 說明 |
|------|------|--------|------|------|
| historicalConfidenceSize | number | 10 | D2-R1 | 環形緩衝區大小 |
| loopQualityAlpha | number | 0.10 | D2-R4 | L3 閾值調制係數 |
| monitorStalenessMs | number | 5000 | D2-R4 | Monitor 過期閾值 |

---

## 9. Auditor 策略方向 [D2-R5, 20/20]

| Phase | 內容 | 時程 | 約束 |
|-------|------|------|------|
| Phase 0 | PassthroughAuditor (delta=0, 純日誌) | Plan30 (可選) | WIENER C-1/C-2/C-3 |
| Phase 1 | ThresholdAuditor (規則式) | Plan31 | WIENER C-1/C-2/C-3 |
| Phase 2 | LLM-assisted | 長期 | WIENER C-1/C-2/C-3 |

所有 Phase 必須遵守 WIENER 三約束。Phase 0 的 PassthroughAuditor 雖然 delta=0，但仍完整記錄 ConfidenceAuditLog——它的價值在於收集審計基線數據。

---

## 10. 跨文件參照

| 相關文件 | 關係 |
|---------|------|
| Doc 42 (IGearArbiter) | IGearArbiter.evaluate() → GearEvaluation → AuditContext.gearEvaluation |
| Doc 43 (LoopQualityMonitor) | §17.3 extras 整合 → AuditContext.extras |
| Doc 44 (Safety Architecture) | Model Delta 五層公式 → §6 Option C |
| Doc 45 (Five Skandha OOP) | IConfidenceAuditor 型別歸屬 IVijnana |

---

## 附錄: Cycle 02-6 D2 決議索引

| 編號 | 決議內容 | 票數 | 對應章節 |
|------|---------|------|---------|
| D2-R1 | AuditContext 完整型別 | 20/20 | §2 |
| D2-R2 | extras bag 協議 | 19/20 | §3 |
| D2-R3 | ConfidenceAuditLog + GUARDIAN D5 兌現 | 20/20 | §5 |
| D2-R4 | Layer 整合方案 C | 20/20 | §6 |
| D2-R5 | Auditor 策略 Phase 0/1/2 | 20/20 | §9 |

---

*Cycle 02-6 新增文件。AuditContext 為 IConfidenceAuditor 提供豐富的決策上下文，extras bag 為 Plugin 間資料共享建立通用管道，WIENER 三約束確保回饋迴路穩定性。所有決議經 20 位學者投票確認。*

---

## Cycle 02-7 Research Additions

`[Cycle 02-7 新增: Plan31 規格精化 7 項決議，24 位學者]`

以下為 Cycle 02-7 研究成果，將 Cycle 02-6 的 AuditContext 設計精化為可直接實作的工程規格。

### 11. IConfidenceAuditor 簽章變更 [D4-R1, 24/24]

```typescript
// BEFORE (v0.30.0):
audit(routeResult: RouteResult): ConfidenceAuditResult | Promise<ConfidenceAuditResult>;

// AFTER (v0.31.0):
audit(context: AuditContext): ConfidenceAuditResult | Promise<ConfidenceAuditResult>;
```

ManoAggregator.route() 新增可選 sparshEvent 參數，用於組裝 AuditContext.sparshEvent。GearContext 型別不變。

### 12. destructive 安全約束 [D1-R1, 24/24]

新增 Core 級安全地板：當 riskCategory === 'destructive' 時，audit delta 強制 ≤ 0。此約束在 ManoAggregator 的 audit 呼叫後執行，獨立於 clampAuditDelta 通用函數。

```typescript
// ManoAggregator — after clampAuditDelta:
if (evaluation.riskCategory === 'destructive' && clampedDelta > 0) {
  clampedDelta = 0;
}
```

### 13. ThresholdAuditor Phase 1 規則 [D5-R1, 22/24]

| riskCategory | delta | reasoning |
|-------------|-------|-----------|
| destructive | -0.03 | reduce confidence for safety |
| state_modifying | -0.01 | slight caution |
| read_only | 0 | no adjustment |
| informational | 0 | no adjustment |
| undefined | 0 | passthrough |

auto-mount [D3-R1, 20/24]：無 plugin 提供 auditor 時自動掛載 ThresholdAuditor。

### 14. Audit Trail JSONL [D2-R1, 23/24 + D6-R1, 24/24]

Core 內建模組 createAuditTrailWriter(bus, agentId, config)。每行 JSONL 記錄：

```typescript
interface AuditTrailEntry extends ConfidenceAuditLog {
  readonly timestamp: number;
  readonly agentId: string;
  readonly sessionId?: string;
  readonly version: 1;
}
```

旋轉策略：maxSizeBytes=10MB, maxFiles=5, append-only。

### 15. 十大宣言合規性 [D7-R1, 24/24]

Plan31 通過十大核心宣言逐條合規性審查。關鍵合規點：
- Tenet #2：ThresholdAuditor 是可替換 plugin
- Tenet #7：Core 只做組裝和限幅，審計策略在 plugin
- Tenet #8：L2 BIBO 穩定，C-1/C-2/C-3 合規

---

## 附錄 D: Cycle 02-7 決議索引

| 編號 | 決議內容 | 票數 | 對應章節 |
|------|---------|------|---------|
| D1-R1 | destructive delta ≤ 0 | 24/24 | §12 |
| D2-R1 | Audit Trail Writer Core 內建 | 23/24 | §14 |
| D3-R1 | auto-mount ThresholdAuditor | 20/24 | §13 |
| D4-R1 | route() 新增 sparshEvent | 24/24 | §11 |
| D5-R1 | Phase 1 規則矩陣 | 22/24 | §13 |
| D6-R1 | JSONL Schema | 24/24 | §14 |
| D7-R1 | 十大宣言合規 | 24/24 | §15 |

---

## §16. Cycle 02-8 Plan32 影響

`[Cycle 02-8 新增]`

Plan32 對本文件描述的 AuditContext 和 Auditor 架構有以下影響：

### 16.1 Auto-mount 移除 (Plan32 Wave 1)

Cycle 02-7 D3-R1 建立的 `?? createThresholdAuditor()` auto-mount 設計在 Plan32 Wave 1 中被移除：

| 項目 | Plan31 (v0.31.0) | Plan32 (v0.32.0) |
|------|------------------|------------------|
| Auditor 缺席行為 | `?? createThresholdAuditor()` | delta=0（Optional-degraded 級別） |
| Core 導入 | `import { createThresholdAuditor }` | 移除 |
| Tenet #7 合規 | 違規（auto-mount = 內建能力） | 合規 |

**決議**: 02-8_a D1-Q1-R1 (24/24 全票，Option A)。Core 不發出 warning event 當 auditor 缺席（僅 debug-level log）。

### 16.2 Plugin 提取 (Plan32 Wave 2)

ThresholdAuditor 和 PassthroughAuditor 從 `packages/core/src/plugins/` 提取為獨立 plugin 包：

| 原位置 | 新位置 | 包名 |
|--------|--------|------|
| `packages/core/src/plugins/threshold-auditor.ts` | `packages/openstarry-plugin/auditor-threshold/` | `@openstarry-plugin/auditor-threshold` |
| `packages/core/src/plugins/passthrough-auditor.ts` | `packages/openstarry-plugin/auditor-passthrough/` | `@openstarry-plugin/auditor-passthrough` |

兩者宣告 `skandha: 'vijnana'`。不需要 trusted flag（D2-Supp-R1, 24/24）。

### 16.3 audit:completed Schema 擴展 (Plan32 Wave 5)

`audit:completed` EventBus 事件新增兩個 P0 必填欄位：

```typescript
interface AuditCompletedEvent {
  // 既有欄位...
  riskCategory: RiskCategory;          // 新增: 風險分類
  thresholdAtDecision: number;         // 新增: 決策時的閾值
}
```

此擴展為 Doc 50 (Calibration Methodology) 的校準協議提供必要的資料基礎。

---

*Cycle 02-7 更新。Plan31 三大交付物（AuditContext 落地、ThresholdAuditor、Audit Trail JSONL）的工程規格精化。Cycle 02-8 更新：Plan32 影響（auto-mount 移除 + plugin 提取 + schema 擴展）。*
