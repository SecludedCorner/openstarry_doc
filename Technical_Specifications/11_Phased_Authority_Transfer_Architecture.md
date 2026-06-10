# Phased Authority Transfer Architecture — Gear Arbiter

**版本**: 1.1 (Cycle 03-7, 2026-04-10)
**來源**: D2-Q5a/Q5b (03-6, 7-0 UNANIMOUS), D3-Q1/Q2 (03-7)
**規則**: Rule #55 (destructive never delegated), Rule #56 (phased transfer), Rule #57 (monitoring-only)

---

## 1. 背景

v0.41.0-alpha 引入 gear-arbiter-dynamic plugin (Plan41 W2, 149 LOC)，與既有 gear-arbiter-static 共存。兩者透過 IGearArbiter 介面實現，由 priority 排序決定執行順序。

### 雙重根因 (Cycle 03-6 發現)
- **RC-1**: CalibrationBridge payload extraction 靜默失敗 — dynamic arbiter 收不到 audit 事件
- **RC-2**: Priority deadlock — static (priority 10) 攔截所有決策，dynamic (priority 20) 無法累積觀測
- **結果**: P(observe mode exit) = 0.0 (PASCAL 證明)

---

## 2. 修復設計: Shadow Counting

**核心概念**: 解耦學習與決策。Dynamic arbiter 觀察 static arbiter 的決策結果，計入自己的狀態估計。

```
Static Arbiter (priority 10)  ──────────→  Decision
        │
        ├── audit event emitted
        │
Dynamic Arbiter (priority 20)  ←── shadow observation
        │
        └── state-tracker: per-category, per-event counting
```

- Static 保持權威
- Dynamic 在 shadow mode 下學習，不影響決策
- 當 per-category n ≥ 10 時，dynamic 退出 observe mode（但仍不覆蓋 static）

---

## 3. Phased Authority Transfer (Rule #56)

| Phase | 階段 | Dynamic 角色 | Static 角色 | 決策權 |
|-------|------|-------------|-------------|--------|
| **Phase 2** | Shadow Learning | 觀察 + 計數 + abstain | 全權決策 | Static 100% |
| **Phase 3** | Low-Risk Delegate | informational + read_only 決策 | state_mod + destructive 決策 | Shared |
| **Phase 4** | State-Mod Delegate | + state_modifying 決策 | destructive 決策 | Dynamic primary |
| **永久** | — | — | — | **Destructive: Static ONLY** |

### 轉換條件（待 Phase 3 設計時定義）
- Phase 2 → Phase 3: 至少 N 輪 shadow agreement rate ≥ 95%
- Phase 3 → Phase 4: 至少 M 輪 low-risk delegation 無異常
- Phase 4 不再進階: destructive 操作永遠由 static 控制

---

## 4. 永久約束: Rule #55

> **Destructive operations NEVER delegated to dynamic arbiter.**

理由 (D2-Q5b, 03-6):
- Destructive 操作不可逆（fs.delete, fs.write 覆蓋）
- 動態仲裁器基於統計學習，inherently 有不確定性
- 不確定性 + 不可逆 = 不可接受的風險
- Static arbiter 使用確定性規則，適合 destructive 決策

---

## 5. Plan42 修復範圍 (AC-CV5-FIX-1~7)

| AC | 驗收條件 |
|----|----------|
| FIX-1 | CalibrationBridge 正確提取 payload (RC-1 修復) |
| FIX-2 | Payload 提取失敗時記錄 error log (P0, 安全系統不可靜默) |
| FIX-3 | Shadow counting: per-event, per-category |
| FIX-4 | gear ≠ 1 hardcoding (F-latent-1 修復) |
| FIX-5 | Observe mode 在 n ≥ 10 時正確退出 |
| FIX-6 | 無 calibration regression (existing metrics stable) |
| FIX-7 | 新程式碼無新 attack surface |

---

## 6. 相關檔案 (v0.41.0-alpha)

| 檔案 | 角色 |
|------|------|
| `openstarry_plugin/gear-arbiter-dynamic/src/index.ts` | Plugin factory (30 LOC) |
| `openstarry_plugin/gear-arbiter-dynamic/src/dynamic-arbiter.ts` | IGearArbiter impl (68 LOC) |
| `openstarry_plugin/gear-arbiter-dynamic/src/state-tracker.ts` | Per-instance state tracking (27 LOC) |
| `openstarry_plugin/gear-arbiter-dynamic/src/calibration-bridge.ts` | EventBus subscription (27 LOC) |
| `openstarry_plugin/gear-arbiter-static/` | Static arbiter (priority 10, authoritative) |

---

## 7. Destructive Category: Phase 3 Monitoring-Only Role (Rule #57, Cycle 03-7)

### 7.1 W2-R6 Destructive Data

| Metric | Value |
|--------|-------|
| Sample size (n) | 12 |
| Sigma | 0.0 |
| All delta values | -0.04675 (identical) |
| Operation type | fs.delete (single type) |

Every destructive operation produced exactly the same quality delta, reflecting that all destructive tests are fs.delete with identical pre/post conditions.

### 7.2 BABBAGE Formal Analysis: Degenerate Distribution

A sigma=0 distribution is a **point mass** (Dirac delta). Formally:
- **Entropy**: H(X) = 0 -- zero information beyond location
- **Inference**: Confidence intervals have width zero; parameter estimation is meaningless
- **Hypothesis testing**: Test statistics are undefined when variance is zero -- any test is vacuous
- **Decision theory**: A threshold on a point mass is either always or never triggered; neither provides discriminatory power

**Implication**: Any decision depending on destructive M4a data is formally vacuous -- it would appear evidence-based while providing no actual evidence.

### 7.3 Three-Fold Justification

**Rule #55: Never Delegated (Regulatory)**

Rule #55 states destructive operations are NEVER delegated to the dynamic arbiter. If delegation is impossible, no threshold can trigger or prevent it. The only remaining role for M4a is **observation**.

**Sigma=0: Non-Informative (Statistical)**

Even if Rule #55 were relaxed, the data provides no basis for decisions: cannot detect drift (no graduated signal), cannot set thresholds (requires non-zero spread), cannot compare rounds (point mass comparison yields one bit at best).

**Single Operation Type (Empirical)**

The test suite contains only fs.delete. This single type does not represent production diversity (resource cleanup, cache invalidation, config reset, etc.), produces identical deltas, and cannot distinguish good from bad arbiter decisions.

### 7.4 Rule #57 Specification

| Aspect | Specification |
|--------|--------------|
| Role | Monitoring-only |
| Actionable threshold | None (no gate, no minimum agreement rate) |
| Anomaly detection | Flag if agreement rate changes > 20pp between consecutive rounds |
| Label | Shadow decisions computed but labeled "non-informative" |
| Review trigger | Re-evaluate when destructive sigma > 0 for two consecutive W2 rounds |

The 20pp anomaly threshold is deliberately loose -- with a point mass, any observable change indicates a structural shift (likely test redesign), not arbiter behavior change.

### 7.5 SEC-002 Connection: Vacuous Guard Parallel

SEC-002 documented a vacuous guard: `if (negMean > 0)` -- always false because negative means are <= 0 by definition. The guard **appears** to protect but provides no actual protection.

Destructive M4a without Rule #57 exhibits the same pattern: it would **appear** to be a data-driven gate while providing no discriminatory power. Both share the flaw of "looks like evidence-based decision-making, but the evidence is vacuous."

MR-11 demands honesty about what the architectural choice requires. Pretending sigma=0 data informs decisions violates that honesty.

### 7.6 Phase 3 Integration and Future Enhancement

During Phase 3, destructive shadow decisions are computed (completeness), recorded (data preservation), labeled "non-informative" (Rule #57), and M4a is reported but non-actionable. This preserves data collection while preventing vacuous decision-making.

**Future enhancement path** (Plan44+): context-dependent delta modifiers, multiple destructive operation types (cleanup, cache invalidation), and pre-condition variation to increase diversity.

**Re-evaluation gate**: When destructive sigma > 0 for two consecutive W2 rounds, Rule #57 monitoring-only status should be reconsidered via R3 vote.

---

*Phased Authority Transfer Architecture v1.1*
*Rule #55 + Rule #56 + Rule #57 | Cycle 03-6 D2-Q5, Cycle 03-7 D3-Q1/Q2*
