# CV-5 Observe Mode Investigation Report

**版本**: 1.0 (Cycle 03-6, 2026-04-09)
**優先級**: P0
**狀態**: Root cause confirmed, fix designed (Plan42 W2)
**來源**: R1 TURING + KNUTH, R2 Group B convergence, D2 (03-6, 12 decisions, 7-0)

---

## 1. 問題描述

W2-R5 (v0.41.0-alpha, 50 cycles, 108 audit entries) 中，gear-arbiter-dynamic 全程停留在 observe mode：
- M1 (gear switch count) = 0
- M2 (observe mode exit) = None
- 所有 108 個決策由 static arbiter 做出

預期行為：informational category 有 52 個 audit entries，遠超 n ≥ 10 門檻，應觸發 observe mode exit。

---

## 2. 雙重根因

### RC-1: Payload Extraction Bug (HIGH)

**位置**: `calibration-bridge.ts`

CalibrationBridge 訂閱 `audit:tool_audited` EventBus 事件，期望 `event.payload.clampedDelta` 作為直接屬性。但實際 EventBus payload 結構不同（可能有額外包裝層）。

```typescript
// calibration-bridge.ts (simplified)
eventBus.on('audit:tool_audited', (event) => {
  const p = event.payload;
  if (typeof p.clampedDelta === 'number') {  // ← always false
    stateTracker.recordDelta(category, p.clampedDelta);
  }
  // silent skip when clampedDelta is undefined
});
```

`typeof` guard 靜默跳過 → `recordDelta()` 從未被呼叫 → `deltas.length` 永遠為 0。

### RC-2: Priority Deadlock (HIGH)

**位置**: Plugin priority scheme

| Arbiter | Priority | 行為 |
|---------|----------|------|
| gear-arbiter-static | 10 (higher) | 先執行，做出決策 |
| gear-arbiter-dynamic | 20 (lower) | 後執行，但 static 已決策 |

Static arbiter 攔截所有決策，dynamic arbiter 無法透過自己的決策路徑累積觀測。即使修好 RC-1（payload），dynamic 仍然只能 abstain（因為 static 已經決定了）。

### PASCAL 證明

P(observe mode exit) = **exactly 0.0** under current design.

兩個獨立阻斷點，任一個都足以阻止退出。修好 RC-1 但不修 RC-2 → 仍然卡住。修好 RC-2 但不修 RC-1 → 仍然卡住。必須同時修復。

---

## 3. 修復設計: Shadow Counting

**核心**: 解耦學習與決策 (KNUTH Option 3, D2-Q2a)

Dynamic arbiter 不需要自己做決策才能學習。它可以觀察 static arbiter 的決策結果，計入自己的狀態估計。

| 修復 | 內容 | LOC |
|------|------|-----|
| FIX-1 | 修正 payload extraction 路徑 | ~3 |
| FIX-2 | Payload 失敗時記 error log (P0) | ~3 |
| FIX-3 | Shadow counting (observe static's decisions) | ~8-12 |
| FIX-4 | gear=1 hardcoding → 動態 gear 值 | ~2 |
| **Total** | | **~16-20** |

---

## 4. 驗收條件 (AC-CV5-FIX-1~7)

| AC | 條件 | 驗證方法 |
|----|------|----------|
| FIX-1 | Payload 正確提取 | Unit test + integration test |
| FIX-2 | 失敗時 error log | Log 存在性檢查 |
| FIX-3 | Per-event per-category counting | W2-R6 M2a > 0 |
| FIX-4 | gear 值動態 | Unit test |
| FIX-5 | n ≥ 10 時退出 observe | W2-R6 M6 shows active state |
| FIX-6 | 無 calibration regression | W2-R6 sigma/F1F2/clamp stable |
| FIX-7 | 無新 attack surface | GUARDIAN 審查 |

---

## 5. 影響評估

- **W2-R5 結果有效**: Dynamic arbiter 在 observe mode 下不影響任何決策，所有 calibration metrics 完全由 static arbiter 產生。W2-R5 GO verdict 不受影響。
- **Phase 2 baseline 有效**: 基線參數基於 static arbiter 決策，不受 dynamic 影響。
- **Plan42 W2 為 P0**: 必須修復才能進入 Phase 3 (delegate authority)。

---

*CV-5 Observe Mode Investigation v1.0*
*Cycle 03-6 D2 (12 decisions, 7-0 UNANIMOUS)*
