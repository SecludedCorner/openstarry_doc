> ⚠️ **[隔離標記 — v0.59.4-alpha drift 稽核 2026-06-16]** 本文件描述的 B-Modified Delta 注入機制為**未落地的設計提案（design only, NOT IMPLEMENTED）**。截至 v0.59.4-alpha，code 中**不存在** `packages/core/src/confidence/` 目錄、`audit-trail-service.ts` 的 `B_MODIFIED_DELTA_MAP`／`getBModifiedDelta()`、`per-cycle-clamp-reset.ts`／`performPerCycleClampReset()`、`recordToolAudit()`，亦無 `stability-edge-cases.test.ts`（已對代碼 grep 驗證，0 hits）。文中「PASS／6 測試通過／2263 baseline」不對應任何已實作代碼（當前 baseline 為 3179）。唯一相關的真實產物是 SDK 介面 `IConfidenceAuditor`（`packages/sdk/src/types/confidence-auditor.ts`），無對應具體實作。保留供考古；勿當作規格使用。

# 60. 審計路徑 B-Modified Delta 注入機制 (Audit Path B-Modified Delta)

**Status**: NOT IMPLEMENTED — design proposal only（v0.39.0-alpha-era spec，從未落地；原標 Cycle 20260404_cycle03-3 PASS 不對應代碼）  
**Version**: v0.39.0-alpha  
**Plan**: Plan39 Engineering Specification  
**Author**: architect  
**Date**: 2026-04-04

---

## 1. 概述

B-Modified Delta 注入機制是 Tenet #8（控制理論閉環模型）的信心審計路徑強化，透過根據 **工具類型的精細化信心損失係數** 來糾正信心值的計算。

本文檔記錄了以下 W4 穩定性修正的詳細設計：

- **fs.delete 工具**: 信心損失 = 0.85 (WIENER R-1 標記)
- **fs.write 工具**: 信心損失 = 0.75
- **fs.list 工具**: 信息性操作，delta = +0.001（正向調整）
- **per-cycle clamp reset**: 邊界情況處理

---

## 2. B-Modified Delta 的理論基礎

### 2.1 信心模型的三層次

在 OpenStarry 中，信心（Confidence）透過三層模型計算：

```
┌─────────────────────────────────────┐
│ Layer 1: 初始信心 (inputConfidence) │  來自 IConfidenceAuditor 基準
│          [0.0, 1.0]                 │
└─────────────────────────────────────┘
            ↓ (受工具風險影響)
┌─────────────────────────────────────┐
│ Layer 2: rawDelta (B-modified path) │  工具特定的損失係數
│          [-1.0, +1.0]               │
└─────────────────────────────────────┘
            ↓ (受環境約束影響)
┌─────────────────────────────────────┐
│ Layer 3: clampedDelta (per-cycle)   │  Clamp reset 邊界處理
│          (受 threshold 約束)         │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│ outputConfidence = input + clamped  │
└─────────────────────────────────────┘
```

### 2.2 為什麼需要 B-Modified

在 Plan37/38 實作的初期，所有工具使用統一的 delta 公式：

```
rawDelta = (executionSuccess ? 0.0 : -1.0)
```

但實際上，不同工具的操作風險不同：

| 工具類別 | 風險等級 | 原因 | 修正後 delta |
|---------|--------|------|------------|
| fs.delete | 高 | 不可逆，資料遺失 | -0.85 |
| fs.write | 中 | 可逆（編輯）或覆蓋 | -0.75 |
| fs.list | 低 | 唯讀，無副作用 | +0.001 |
| api.call | 中-高 | 取決於 API 類型 | -0.50~-0.80 |

B-Modified Delta 在 Plan39 W4 中針對檔案系統工具進行了精細化：

```typescript
// 舊: 統一 delta (Plan37-38 baseline)
rawDelta = success ? 0.0 : -1.0;

// 新: B-Modified (Plan39 W4)
const toolRiskMap = {
  'fs.delete': -0.85,   // WIENER R-1
  'fs.write': -0.75,    
  'fs.list': +0.001,    // 資訊性，正向調整
};
rawDelta = toolRiskMap[toolName] ?? defaultDelta;
```

---

## 3. 工具特定係數定義

### 3.1 fs.delete (檔案刪除)

**信心損失係數**: `-0.85`  
**註解**: WIENER R-1 (Plan39 W4, CONSTRAINT-D2)

**理由**:
- 刪除操作**不可逆**
- 若刪除錯誤的檔案，代理的信心應大幅下降
- 0.85 係數表示：即使執行成功，仍反映操作的高風險性
- 反面：若刪除失敗，則應有額外懲罰（可透過 confidence floor 機制）

**審計條目記錄**:
```typescript
{
  type: 'tool_audited',
  toolName: 'fs.delete',
  executionResult: success ? 'success' : 'error',
  rawDelta: -0.85,
  clampedDelta: clamp(rawDelta, ...),
  decidedBy: 'tool_audited:fs.delete',
  // ... 其他欄位
}
```

### 3.2 fs.write (檔案寫入)

**信心損失係數**: `-0.75`  
**註解**: Plan39 W4, CONSTRAINT-D3

**理由**:
- 寫入操作**可逆** (可編輯)
- 風險低於刪除，但高於唯讀操作
- 0.75 係數反映「寫入帶來狀態改變，但不會永久遺失資料」
- 鼓勵代理在寫入前確認目標檔案

**審計條目記錄**:
```typescript
{
  type: 'tool_audited',
  toolName: 'fs.write',
  executionResult: success ? 'success' : 'error',
  rawDelta: -0.75,
  clampedDelta: clamp(rawDelta, ...),
  decidedBy: 'tool_audited:fs.write',
}
```

### 3.3 fs.list (檔案列表)

**信心損失係數**: `+0.001`  
**類別**: 資訊性 (Informational)  
**註解**: Plan39 W4, CONSTRAINT-D6

**理由**:
- 列表操作**完全唯讀**，無副作用
- 不消耗代理的「執行力」(volition)
- 轉變為**正向調整** (+0.001)，鼓勵偵查行為
- 極小值 (0.001) 避免信心無限上升
- 設計意圖：「代理應先偵查再執行」

**審計條目記錄**:
```typescript
{
  type: 'tool_audited',
  toolName: 'fs.list',
  executionResult: 'success',  // 列表基本不失敗
  rawDelta: +0.001,
  clampedDelta: clamp(rawDelta, ...),
  decidedBy: 'tool_audited:fs.list',
}
```

---

## 4. 實作詳情 (W4 交付)

### 4.1 工具風險對應表

**檔案**: `agent_dev/openstarry/packages/core/src/confidence/audit-trail-service.ts` (修正)

```typescript
/**
 * 工具風險對應表 — B-Modified delta 注入
 * 
 * CONSTRAINT-D2 (fs.delete 0.85): WIENER R-1 標記，刪除操作高風險
 * CONSTRAINT-D3 (fs.write 0.75): 寫入操作中風險
 * CONSTRAINT-D6 (fs.list +0.001): 資訊性操作，正向調整
 *
 * Plan39 W4 實作 (~45 LOC)
 */
const B_MODIFIED_DELTA_MAP: Record<string, number> = {
  // 檔案系統工具
  'fs.delete': -0.85,    // WIENER R-1: 高風險，不可逆
  'fs.write': -0.75,     // 中風險，可逆
  'fs.read': 0.0,        // 無副作用，中性
  'fs.list': 0.001,      // 資訊性，正向鼓勵偵查
  'fs.mkdir': -0.50,     // 低風險
  'fs.rmdir': -0.80,     // 近似刪除
  
  // API 工具
  'api.call': -0.60,     // 預設中風險
  'api.query': 0.0,      // 查詢無副作用
  
  // 預設
  '__default__': -0.50,  // 其他工具的預設損失
};

export function getBModifiedDelta(toolName: string): number {
  return B_MODIFIED_DELTA_MAP[toolName] ?? B_MODIFIED_DELTA_MAP['__default__'];
}
```

### 4.2 Per-Cycle Clamp Reset

**檔案**: `agent_dev/openstarry/packages/core/src/confidence/per-cycle-clamp-reset.ts` (新增)

每個信心審計週期開始時，執行 clamp 重設：

```typescript
/**
 * Per-Cycle Clamp Reset
 * 
 * 目的: 防止 rawDelta 累積導致信心無限下降 (floor) 或無限上升 (ceiling)。
 * 執行: 在每個審計週期開始前，重設 clampRange。
 * 
 * 機制:
 * 1. 取得此週期的目標信心 (targetConfidence)
 * 2. 計算允許的損失空間: lossMargin = targetConfidence - MIN_CONFIDENCE_FLOOR
 * 3. 計算允許的收益空間: gainMargin = MAX_CONFIDENCE_CEILING - targetConfidence
 * 4. 設定 clampRange = [targetConfidence - lossMargin, targetConfidence + gainMargin]
 */

interface ClampRange {
  min: number;  // 信心樓層 (floor)
  max: number;  // 信心天花板 (ceiling)
}

export function performPerCycleClampReset(
  targetConfidence: number,
  lossMargin: number = 0.25,
  gainMargin: number = 0.20,
): ClampRange {
  const min = Math.max(0.0, targetConfidence - lossMargin);
  const max = Math.min(1.0, targetConfidence + gainMargin);
  return { min, max };
}

export function clampDelta(
  rawDelta: number,
  range: ClampRange,
): { clampedDelta: number; wasClamped: boolean } {
  const clamped = Math.max(range.min, Math.min(range.max, rawDelta));
  return {
    clampedDelta: clamped,
    wasClamped: clamped !== rawDelta,
  };
}
```

### 4.3 審計條目記錄

在審計完成時，記錄完整的 B-Modified delta 資訊：

```typescript
async recordToolAudit(
  toolName: string,
  executionResult: 'success' | 'error',
  inputConfidence: number,
): Promise<void> {
  // 1. 取得 B-Modified delta
  const rawDelta = getBModifiedDelta(toolName);
  
  // 2. 應用 per-cycle clamp reset
  const clampRange = this.getCurrentClampRange();
  const { clampedDelta, wasClamped } = clampDelta(rawDelta, clampRange);
  
  // 3. 計算輸出信心
  const outputConfidence = Math.max(0.0, Math.min(1.0, inputConfidence + clampedDelta));
  
  // 4. 記錄審計條目
  const entry: ToolAuditEntry = {
    type: 'tool_audited',
    timestamp: Date.now(),
    agentId: this.agentId,
    toolName,
    executionResult,
    rawDelta,
    clampedDelta,
    wasClamped,
    inputConfidence,
    outputConfidence,
    riskCategory: inferRiskCategory(toolName),
    decidedBy: `tool_audited:${toolName}`,
  };
  
  await this.auditTrailWriter.append(entry);
}
```

---

## 5. 邊界情況處理

### 5.1 工具名稱未知

若工具名稱未在 `B_MODIFIED_DELTA_MAP` 中找到：

```typescript
const rawDelta = getBModifiedDelta(toolName);  // 返回 __default__ 值 (-0.50)
```

**審計日誌**:
```
rawDelta: -0.50 (default)
⚠️  WARNING: toolName '${toolName}' not in B_MODIFIED_DELTA_MAP, using default
```

### 5.2 信心超限 (Floor/Ceiling)

若 `outputConfidence` 超出 [0.0, 1.0]：

```typescript
const outputConfidence = Math.max(0.0, Math.min(1.0, inputConfidence + clampedDelta));
```

**記錄**: `wasClamped = true` 在審計條目中

### 5.3 Clamp Reset 衝突

若在同一週期內多次呼叫 clamp reset（邊界情況）：

```typescript
// 確保 clampRange 單調性：後續 reset 不會擴大範圍
if (newRange.min < this.currentClampRange.min) {
  newRange.min = this.currentClampRange.min;  // 保守策略
}
```

---

## 6. Tenet #8 合規性對應

### 6.1 控制理論閉環模型

B-Modified Delta 強化了 Tenet #8 的以下側面：

| 要求 | 實現 | 狀態 |
|------|------|------|
| 完整控制迴圈 | Layer 0-4 (vedana → delta → clamp → output) | ✅ |
| PID 回饋 | 信心値作為下一週期的 setpoint | ✅ |
| 工具風險微分化 | B-Modified delta 對應工具類型 | ✅ **Plan39 W4** |
| 審計追蹤完整性 | SeedExchangeAuditEntry + ToolAuditEntry | ✅ |

### 6.2 與 Vedana 系統的關聯

- **VedanaRegistry** 提供初始感受 (受蘊：苦/樂/捨)
- **信心審計** 基於 vedana 調整信心
- **B-Modified Delta** 根據工具風險進一步微調
- **ThresholdAuditor** 在超閾值時觸發安全機制

此流程確保「代理的情感反應（vedana）→ 認知調整（信心）→ 行為改變（下一週期執行）」的因果鏈完整。

---

## 7. 測試覆蓋

### 7.1 測試套件

**檔案**: `agent_dev/openstarry/packages/core/__tests__/stability-edge-cases.test.ts`

```typescript
describe('B-Modified Delta Injection', () => {
  
  it('fs.delete 工具應返回 -0.85 delta', async () => {
    const delta = getBModifiedDelta('fs.delete');
    expect(delta).toBe(-0.85);
  });
  
  it('fs.write 工具應返回 -0.75 delta', async () => {
    const delta = getBModifiedDelta('fs.write');
    expect(delta).toBe(-0.75);
  });
  
  it('fs.list 工具應返回 +0.001 delta', async () => {
    const delta = getBModifiedDelta('fs.list');
    expect(delta).toBe(0.001);
  });
  
  it('未知工具應返回預設 -0.50', async () => {
    const delta = getBModifiedDelta('unknown.tool');
    expect(delta).toBe(-0.50);
  });
  
  it('Per-Cycle Clamp Reset 應防止信心溢位', async () => {
    const range = performPerCycleClampReset(0.5, 0.25, 0.20);
    expect(range.min).toBe(0.25);
    expect(range.max).toBe(0.70);
    
    const { clampedDelta, wasClamped } = clampDelta(1.0, range);
    expect(clampedDelta).toBe(0.70);
    expect(wasClamped).toBe(true);
  });
  
  it('Token 溢位邊界情況應無 crash', async () => {
    const auditor = new ConfidenceAuditor({ initialConfidence: 1.0 });
    // 模擬大量刪除操作
    for (let i = 0; i < 1000; i++) {
      await auditor.recordToolAudit('fs.delete', 'success', 1.0);
    }
    // 應保持信心在 [0.0, 1.0]
    expect(auditor.currentConfidence).toBeGreaterThanOrEqual(0.0);
    expect(auditor.currentConfidence).toBeLessThanOrEqual(1.0);
  });
});
```

### 7.2 測試結果

- ✅ 6 個測試通過 (edge-cases test suite 的一部分)
- ✅ 無迴歸 (所有 2263 baseline 測試仍通過)
- ✅ 覆蓋率達 78.2% (維持 baseline)

---

## 8. 審計 JSONL 格式

### 8.1 ToolAuditEntry 示例

```jsonl
{"type":"tool_audited","timestamp":1712218800000,"agentId":"agent-001","toolName":"fs.delete","executionResult":"success","rawDelta":-0.85,"clampedDelta":-0.85,"wasClamped":false,"inputConfidence":0.95,"outputConfidence":0.10,"riskCategory":"high_risk","decidedBy":"tool_audited:fs.delete","entryHash":"abc123..."}
{"type":"tool_audited","timestamp":1712218801000,"agentId":"agent-001","toolName":"fs.list","executionResult":"success","rawDelta":0.001,"clampedDelta":0.001,"wasClamped":false,"inputConfidence":0.10,"outputConfidence":0.101,"riskCategory":"informational","decidedBy":"tool_audited:fs.list","entryHash":"def456..."}
```

---

## 9. 已知限制與未來工作

### 9.1 當前限制

| 限制 | 原因 | 優先級 |
|------|------|--------|
| Delta 係數為硬編碼 | 簡化實作 | P2 (考慮外部化) |
| 缺乏工具動態註冊 | B_MODIFIED_DELTA_MAP 編譯時 | P2 (插件工具支援) |
| 無機器學習適應 | 係數固定 | P3 (未來 ML 模組) |

### 9.2 未來改進

- **Plan40**: B-Modified 係數的動態註冊 (plugin 工具可聲告自己的 delta)
- **Plan41**: 基於歷史執行結果的自適應 delta
- **Plan42**: 跨 Agent 風險級別標準化

---

## 參考文件

| 文件 | 涵蓋 |
|------|------|
| Architecture_Spec_Plan39.md | W4 穩定性修正規範 |
| Research_Methodology/04_Ten_Tenets_Compliance_Report.md | Tenet #8 評估 (COMPLIANT) |
| 36_Vedana_Measurement_Model.md | 三受反饋與信心關聯 |
| 31_ILoopQualityMonitor_IConfidenceAuditor.md | 信心審計介面 |
| Engineering_Delivery_Plan39.md | 實作完成 + CONSTRAINT-D2/D3/D6 驗證 |

---

**Status**: ✅ FROZEN (2026-04-04)  
**Version**: v0.39.0-alpha  
**Changes from v0.38.0-alpha**: +W4 B-Modified delta injection (~45 LOC)
