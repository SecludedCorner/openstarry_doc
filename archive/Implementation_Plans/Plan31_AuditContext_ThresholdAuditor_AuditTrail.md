# Plan31: AuditContext 落地 + ThresholdAuditor + Audit Trail JSONL

- **Version**: v0.30.0-alpha → v0.31.0-alpha
- **SOP**: Standard
- **Cycle**: 02-7
- **Status**: Pending
- **Research Basis**: Cycle 02-6 D2-R1~R5, D3-R5; Cycle 02-7 D1-R1~D7-R1

---

## 目標

將 IConfidenceAuditor.audit() 的輸入從 RouteResult 升級為完整的 AuditContext，實作規則式 ThresholdAuditor 作為 Phase 1 預設審計員（取代 PassthroughAuditor），並新增 Audit Trail JSONL 持久化模組。

**成功標準**:
- AuditContext 型別可編譯
- ThresholdAuditor 通過所有 RiskCategory × gear 組合測試
- destructive delta ≤ 0 約束有測試覆蓋
- WIENER C-1 有測試覆蓋（historicalConfidence 不含 audit-adjusted 值）
- Audit Trail JSONL 可寫入可旋轉
- 所有現有測試通過（不破壞 Plan30 功能）

---

## Wave 1: AuditContext 型別落地 + ManoAggregator 佈線 (~120 LOC)

### 新增檔案

| 檔案 | 說明 |
|------|------|
| `packages/sdk/src/types/audit-context.ts` | AuditContext 介面定義 |

### 修改檔案

| 檔案 | 變更 |
|------|------|
| `packages/sdk/src/types/confidence-auditor.ts` | audit() 簽名改為 `audit(context: AuditContext)` |
| `packages/sdk/src/index.ts` | 匯出 AuditContext |
| `packages/core/src/mano/mano-aggregator.ts` | extras 收集 + historicalConfidence buffer + buildAuditContext() + route() sparshEvent 可選參數 + destructive delta ≤ 0 安全約束 |
| `packages/core/src/plugins/passthrough-auditor.ts` | 參數型別從 RouteResult 遷移至 AuditContext |

### AuditContext 介面 [D2-R1, 20/20 from Cycle 02-6]

```typescript
export interface AuditContext {
  readonly version: 1;
  readonly sparshEvent: SparshEvent;
  readonly gearEvaluation: GearEvaluation;
  readonly riskCategory: RiskCategory | undefined;
  readonly routeResult: RouteResult;
  readonly historicalConfidence?: readonly number[];
  readonly extras: ReadonlyMap<string, unknown>;
}
```

### IConfidenceAuditor 簽名變更 [D4-R1, 24/24]

```typescript
// BEFORE (v0.30.0):
audit(routeResult: RouteResult): ConfidenceAuditResult | Promise<ConfidenceAuditResult>;

// AFTER (v0.31.0):
audit(context: AuditContext): ConfidenceAuditResult | Promise<ConfidenceAuditResult>;
```

Breaking change at type level, zero impact in practice (no external auditor exists).

### ManoAggregator 修改

#### route() 簽名變更 [D4-R1]
```typescript
route(context: GearContext, arbiters: IGearArbiter[], sparshEvent?: SparshEvent): Promise<RouteResult>
```

#### historicalConfidence 環形緩衝區 [WIENER C-1]
```typescript
const historicalBuffer: number[] = [];
const maxHistory = config.historicalConfidenceSize ?? 10;

// After arbiter wins, before audit:
historicalBuffer.push(evaluation.confidence);  // WIENER C-1: raw arbiter confidence only
if (historicalBuffer.length > maxHistory) historicalBuffer.shift();
```

#### extras 收集管道 [D2-R2]
```typescript
const extrasMap = new Map<string, unknown>();
const unsubExtras = bus.on('audit:context_contribute', (event) => {
  const p = event.payload as { key: string; value: unknown } | undefined;
  if (p && typeof p.key === 'string') extrasMap.set(p.key, p.value);
});
// Clear extras at start of each route():
extrasMap.clear();
```

#### buildAuditContext 組裝
```typescript
const auditContext: AuditContext = {
  version: 1,
  sparshEvent: sparshEvent ?? { type: 'unknown', timestamp: Date.now() } as SparshEvent,
  gearEvaluation: evaluation,
  riskCategory: evaluation.riskCategory,
  routeResult: preliminaryResult,
  historicalConfidence: Object.freeze([...historicalBuffer]),
  extras: new Map(extrasMap) as ReadonlyMap<string, unknown>,
};
```

#### destructive 安全約束 [D1-R1, 24/24]
```typescript
// After clampAuditDelta, before using auditedConfidence:
if (evaluation.riskCategory === 'destructive' && clampedDelta > 0) {
  clampedDelta = 0;
  // Update reasoning to note safety override
}
```

### 測試 (~15)

| 測試 | 數量 | 焦點 |
|------|------|------|
| AuditContext 組裝 | 5 | 欄位正確填充 |
| WIENER C-1 | 3 | historicalConfidence 僅含原始值 |
| destructive 安全約束 | 3 | delta ≤ 0 for destructive |
| PassthroughAuditor 遷移 | 2 | 向後相容 |
| route() sparshEvent 傳遞 | 2 | 可選參數正確傳遞 |

---

## Wave 2: Default ThresholdAuditor (~100 LOC)

### 新增檔案

| 檔案 | 說明 |
|------|------|
| `packages/core/src/plugins/threshold-auditor.ts` | ThresholdAuditor 實作 |

### 修改檔案

| 檔案 | 變更 |
|------|------|
| `packages/core/src/plugins/index.ts` | 匯出 createThresholdAuditor |
| `packages/core/src/agents/agent-core.ts` | auto-mount ThresholdAuditor（替代 PassthroughAuditor） |

### 規則矩陣 [D5-R1, 22/24]

```typescript
export interface ThresholdAuditRule {
  readonly riskCategory?: RiskCategory;
  readonly gearMin?: number;
  readonly gearMax?: number;
  readonly delta: number;
  readonly reasoning: string;
}

export const DEFAULT_THRESHOLD_RULES: readonly ThresholdAuditRule[] = [
  { riskCategory: 'destructive', delta: -0.03, reasoning: 'destructive action: reduce confidence for safety' },
  { riskCategory: 'state_modifying', delta: -0.01, reasoning: 'state-modifying: slight caution' },
  { riskCategory: 'read_only', delta: 0, reasoning: 'read-only: no adjustment needed' },
  { riskCategory: 'informational', delta: 0, reasoning: 'informational: no adjustment needed' },
];
```

### createThresholdAuditor 工廠函數

```typescript
export function createThresholdAuditor(
  rules: readonly ThresholdAuditRule[] = DEFAULT_THRESHOLD_RULES,
  id: string = 'threshold-auditor',
): IConfidenceAuditor {
  return {
    id,
    skandha: 'vijnana',
    audit(context: AuditContext): ConfidenceAuditResult {
      for (const rule of rules) {
        if (rule.riskCategory !== undefined && rule.riskCategory !== context.riskCategory) continue;
        if (rule.gearMin !== undefined && context.routeResult.gear < rule.gearMin) continue;
        if (rule.gearMax !== undefined && context.routeResult.gear > rule.gearMax) continue;
        return { delta: rule.delta, reasoning: rule.reasoning };
      }
      return { delta: 0, reasoning: 'no matching rule: passthrough' };
    },
  };
}
```

### auto-mount [D3-R1, 20/24]

```typescript
// Replace PassthroughAuditor auto-mount with ThresholdAuditor:
const pluginAuditor = pluginLoader.getAuditor();
const effectiveAuditor = pluginAuditor ?? createThresholdAuditor();
```

### 蘊歸屬

ThresholdAuditor.skandha = 'vijnana'（識蘊），依據 D1-R6 功能分析原則：ThresholdAuditor 執行的是判斷/分別功能（riskCategory → delta 映射），屬於識蘊的粗分別。

### 測試 (~12)

| 測試 | 數量 | 焦點 |
|------|------|------|
| 規則匹配 | 5 | 各 RiskCategory 正確匹配 |
| 自訂規則 | 2 | 自訂規則矩陣覆蓋預設 |
| auto-mount | 2 | 無 plugin 時自動掛載 |
| 安全屬性 | 3 | destructive → delta ≤ 0 不變量 |

---

## Wave 3: Audit Trail JSONL Writer (~100 LOC)

### 新增檔案

| 檔案 | 說明 |
|------|------|
| `packages/core/src/observability/audit-trail-writer.ts` | JSONL Writer 實作 |

### 修改檔案

| 檔案 | 變更 |
|------|------|
| `packages/core/src/agents/agent-core.ts` | 初始化 AuditTrailWriter |
| `packages/sdk/src/types/agent.ts` | IAgentConfig 新增 auditTrail 配置 |

### 介面定義 [D6-R1, 24/24]

```typescript
export interface AuditTrailConfig {
  readonly filePath: string;
  readonly maxSizeBytes?: number;    // default: 10_000_000 (10MB)
  readonly maxFiles?: number;        // default: 5
  readonly enabled?: boolean;        // default: true
}

export interface AuditTrailEntry extends ConfidenceAuditLog {
  readonly timestamp: number;
  readonly agentId: string;
  readonly sessionId?: string;
  readonly version: 1;
}
```

### 工廠函數 [D2-R1, 23/24]

```typescript
export function createAuditTrailWriter(
  bus: EventBus,
  agentId: string,
  config: AuditTrailConfig,
): { start(): void; stop(): Promise<void> }
```

### 初始化 [agent-core.ts]

```typescript
// In start():
const auditTrailConfig = config.auditTrail ?? {
  filePath: `./audit-trail-${config.identity.id}.jsonl`
};
const auditTrailWriter = createAuditTrailWriter(bus, config.identity.id, auditTrailConfig);
auditTrailWriter.start();

// In stop():
await auditTrailWriter.stop();
```

### 旋轉策略

- 單一 JSONL 檔案，append-only
- 超過 maxSizeBytes 時，將現有檔案重命名為 `.1.jsonl`，最多保留 maxFiles 個歷史檔案
- 重啟後 append 到現有檔案

### 測試 (~10)

| 測試 | 數量 | 焦點 |
|------|------|------|
| JSONL 寫入 | 3 | 正確序列化、append |
| 旋轉 | 3 | 超限旋轉、歷史檔案保留 |
| 生命週期 | 2 | start/stop/flush |
| 錯誤處理 | 2 | 檔案不可寫、I/O 錯誤 |

---

## Wave 4: postRouteCheck v2 (~30 LOC, 可選)

**狀態**: 延後 (D-31-1)
**原因**: 連續延後四次（D-28-1 → D-29-4 → D-30-1 → D-31-1）
**條件**: 若 W1~W3 工作量允許且有餘裕

---

## WIENER 安全約束（不變）

| ID | 規則 | 執行位置 |
|----|------|---------|
| **C-1** | historicalConfidence = 僅原始 arbiter 信心度 | ManoAggregator push pre-clamp value |
| **C-2** | AuditContext 不含 previousAuditResult | 型別定義排除 |
| **C-3** | extras key 禁止 `audit:` 前綴 | isValidExtrasKey() case-insensitive check |
| **NEW** | destructive delta ≤ 0 | ManoAggregator post-clamp check (D1-R1) |

---

## LOC 估算

| 元件 | 生產 | 測試 |
|------|------|------|
| W1: AuditContext + ManoAggregator | ~120 | ~150 |
| W2: ThresholdAuditor | ~100 | ~120 |
| W3: Audit Trail Writer | ~100 | ~100 |
| W4: postRouteCheck v2 (可選) | ~30 | ~30 |
| **合計** | **~350** | **~400** |

---

## 依賴關係

```
W1 (AuditContext + ManoAggregator)
  ├─ W2 (ThresholdAuditor) — 依賴 W1 的 AuditContext 型別
  ├─ W3 (Audit Trail Writer) — 依賴 W1 的 audit:completed 事件
  └─ W4 (postRouteCheck v2, 可選) — 依賴 W1 的 route() 變更
```

---

## 里程碑

```
Plan30 ✓ (v0.30.0-alpha, 五層公式全面佈線, 215 LOC, 43 tests)
  ↓
Plan31 (v0.31.0-alpha, AuditContext + ThresholdAuditor + Audit Trail, ~350 LOC, ~37 tests)
  ↓
Plan32 (VasanaEngine / Skandha 軟約束 / WIENER 耦合)
```

---

*Plan31 研究基礎：Cycle 02-6 D2-R1~R5, D3-R5 + Cycle 02-7 D1-R1~D7-R1。所有決議經 24 位學者投票確認。*
