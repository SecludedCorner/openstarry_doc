# Phase 2 Calibration Monitoring Framework

**版本**: v2.0 (updated Cycle 03-7)
**狀態**: ACTIVE (Phase 2 COMPLETE per Cycle 03-7 R1; Phase 3 prerequisites pending)
**基線**: W2-R5 (v0.41.0-alpha, 2026-04-08)

---

## 1. Phase 2 Baseline (LOCKED)

| Parameter | Value | Source |
|-----------|-------|--------|
| DELTA_SCALING_FACTOR | **0.055** | Plan40 W0, D2-Q4 (03-5) |
| Cycle sigma (F2) | **0.02216** | W2-R5, D3-Q3 (03-6) |
| F1/F2 ratio | **0.99x** | W2-R5 |
| Clamp rate | **0.0%** | W2-R5 |
| info mean | +0.000437 | W2-R5 |
| read_only mean | +0.01490 | W2-R5 |
| state_mod mean | -0.03232 | W2-R5 |
| destructive mean | -0.04675 | W2-R5 |
| neg freq | 24.1% (WARNING observed) | W2-R5 |

---

## 2. Monitoring Metrics

### Original Metrics (D5-Q2, 03-5)

| M | Metric | WARNING | FAIL |
|---|--------|---------|------|
| M1 | F2 sigma ratio (Rn/baseline) | [0.7x, 1.5x] (Rule #61, effective R7) | 0.25x-3.0x |
| M2 | F1/F2 | > 2.0x | > 3.0x |
| M3 | Clamp rate | > 5% | > 15% |
| M4 | neg freq | <10% or >25% | <5% or >40% |
| M5 | Category sign violation | any | immediate re-cal |

### New Metrics (D3-Q4, 03-6)

| M | Metric | Purpose | Source |
|---|--------|---------|--------|
| M2a | Per-category observation count | CV-5 observe mode progress visibility | D3-Q4 (03-6) |
| M4a | Shadow agreement rate | Dynamic vs static arbiter alignment | D3-Q4 (03-6) |
| M6 | Per-category arbiter state | observe/active state at run end | D3-Q4 (03-6) |

---

## 3. Re-calibration Triggers

| T | Trigger | Condition | Action |
|---|---------|-----------|--------|
| T1 | sigma ratio | out of [0.7, 1.5] (Rule #61, effective R7) | Investigation + possible W2-R(n+1) |
| T2 | F1/F2 | > 3.0x | Investigation (correlation group: {M2, M3}) |
| T3 | Clamp rate | > 15% | Investigation (correlation group: {M1, M4}) |
| T4 | SC/CV FAIL | any (excluding CV-5 until fixed) | Immediate re-calibration |

**Correlation groups** (D5-Q3, 03-5): {M2,M3}, {M1,M4}, {M5} — correlated triggers share root cause investigation.

---

## 4. CV-6 Two-Run Smoothing (Rule #51 amended, D3-Q2, 03-6)

- Single run neg freq ≥ 25%: **WARNING** (record, do not block)
- Two consecutive runs neg freq ≥ 25%: **FAIL** (trigger T4)
- Rationale: P(single false positive) ≈ 9%, P(two consecutive) ≈ 0.8%

---

## 5. Six-Round Trajectory

| Metric | R1 (v0.38) | R2 (v0.39) | R3 (hotfix) | R4 (v0.40) | R5 (v0.41) | R6 (v0.42) |
|--------|:---:|:---:|:---:|:---:|:---:|:---:|
| Verdict | GO | IF | GO | GO | GO | **GO** |
| sigma | 0.002 | 0.023 | 0.027 | 0.020 | 0.022 | **0.021** |
| F1/F2 | 1.0x | 14.2x | 13.2x | 1.0x | 0.99x | **—** |
| Clamp | 0% | 32.2% | 29.9% | 0% | 0% | **0%** |
| neg freq | 3.2% | 3.4% | 19.7% | 19.8% | 24.1% | **20.8%** |
| Phase | P1 | P1 | P1 | P1 LOCKED | P2 | **P2 COMPLETE** |

---

## 6. CV-5 Dynamic Arbiter Status

**Current**: Observe mode (permanent under current design — RC-1 + RC-2)
**Fix**: Plan42 W2 (payload extraction + shadow counting)
**Architecture**: Phased authority transfer (Rule #56)
- Phase 2: Shadow learning (current)
- Phase 3: Delegate low-risk categories
- Phase 4: Delegate state_modifying
- **Permanent**: Destructive NEVER delegated (Rule #55)

---

## 7. Phase 2 Completion (Cycle 03-7)

Phase 2 (Shadow Learning / observe-and-count mode) is confirmed **COMPLETE** as of Cycle 03-7 R1. During Phase 2, the dynamic gear arbiter received all cognitive loop inputs and maintained per-category invocation counters and quality metrics, but produced no active decisions. The static arbiter retained full authority.

### 7.1 W2-R6 Evidence

W2-R6 was the first 0-error 50-cycle execution. Per-category observation counts:

| Category | Count (n) | vs MIN_N (10) | Margin |
|----------|-----------|---------------|--------|
| informational | 68 | PASS | +58 |
| read_only | 31 | PASS | +21 |
| state_modifying | 14 | PASS | +4 |
| destructive | 12 | PASS | +2 |

All four categories exceed MIN_N=10. The destructive category (n=12) is the statistical bottleneck, exceeding the minimum by only 2 observations.

### 7.2 Three-Agent Independent Confirmation

Phase 2 completion was independently confirmed by three agents from distinct disciplinary perspectives:

- **WIENER** (control theory): System identification phase complete. Sufficient excitation data exists across all four categories for model parameter estimation. The plant model is observable.
- **RUSSELL** (agent theory): The exploration phase has yielded adequate coverage. The exploration-to-exploitation transition is justified by the coverage metrics. Continued exploration yields diminishing information gain.
- **BABBAGE** (formal verification): The counting invariant (M2a >= MIN_N for all categories) is satisfied. The formal precondition for Phase 3 entry is met.

### 7.3 Conclusion

Phase 2 is COMPLETE. All three completion criteria are met simultaneously:
1. **M2a**: All per-category counts >= MIN_N (10)
2. **M6**: All categories ACTIVE eligible
3. **Baseline stability**: Sigma locked at R5 provisional (sigma=0.02216)

---

## 8. Graduated Readiness Assessment

Per-category readiness for Phase 3 varies based on sample size and behavioral stability:

| Category | Readiness | Rationale |
|----------|-----------|-----------|
| informational | **HIGH** | n=68, rich behavioral data, high-frequency category |
| read_only | **MODERATE-HIGH** | n=31, adequate sample, stable patterns |
| state_modifying | **MODERATE** | n=14, minimal surplus over MIN_N, limited diversity |
| destructive | **LOW** | n=12, near-minimum sample; Rule #55 = never delegated |

Note: The destructive category's LOW readiness is academic -- Rule #55 permanently excludes it from delegation at any phase.

---

## 9. Phase 3 Prerequisites

Transitioning from Phase 2 to Phase 3 (Shadow Decisions) requires four prerequisites:

| ID | Prerequisite | Description |
|----|-------------|-------------|
| T1 | Shadow decision computation | Dynamic arbiter computes gear selections without executing them |
| T2 | M4a baseline establishment | 3 consecutive W2 rounds (R7, R8, R9) to establish M4a baseline |
| T3 | AC-P3-GATE | Formal acceptance criteria gate for Phase 3 entry |
| T4 | COND-3 + COND-4 | Options object refactoring + config exposure (implementation prerequisites) |

Phase 3 definition: The dynamic arbiter **computes** gear selections for each cognitive loop invocation but does **not execute** them. The static arbiter continues to make all actual decisions. Shadow decisions are recorded and compared against actual decisions to compute M4a (shadow agreement rate).

---

## 10. M4a Status

M4a (shadow agreement rate) is **N/A** during Phase 2 because the dynamic arbiter does not compute shadow decisions in this phase. M4a = N/A is a **Phase 3 indicator**, not a Phase 2 blocker. It becomes meaningful only when shadow decision computation is activated (prerequisite T1 above).

---

*Phase 2 Calibration Monitoring Framework v2.0*
*Activated: Cycle 03-5 D2-Q5 | Baseline: W2-R5 | Phase 2 COMPLETE: Cycle 03-7 | Updated: Cycle 03-7*
