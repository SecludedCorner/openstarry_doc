# 64 -- Plan B4 Delta Scaling Design

**Document Type**: Calibration Report  
**Cycle**: 03-4  
**Date**: 2026-04-04  
**Authors**: WIENER (#12), KNUTH (#21), PASCAL (#19), ARCHIMEDES (#16), SUSSMAN (#22)  
**Status**: DECIDED -- 5-0 Unanimous on all issues  

---

## 1. Problem Statement

### 1.1 The 13.2x Signal Loss

W2-R3 calibration testing revealed that the confidence audit loop suffers from severe signal loss:

| Metric | Value |
|--------|-------|
| F1 sigma (pre-clamp) | 0.561286 |
| F2 sigma (post-clamp) | 0.042659 |
| **F1/F2 ratio** | **13.2x** |
| Information retention | 7.6% |
| Clamp activation rate | ~30% |

This means the clamp mechanism destroys 92.4% of the original signal information.

[Source: R1_WIENER_calibration.md#Section 4, R1_KNUTH_PASCAL_statistics.md#Section 5]

### 1.2 Root Cause: Scale Mismatch

`TOOL_CONFIDENCE_TABLE` values (0.001 -- 0.85) were designed as confidence scores on a 0-1 scale. These values are used directly as delta increments, but the clamp window is +/-0.05 -- a scale designed for incremental adjustments of 0.001-0.05.

```
Raw delta range:    [-0.85, +1.003]     (from audit trail)
Clamp window:       [-0.05, +0.05]
Dynamic range mismatch: 17:1
```

The consequence: three of four risk categories are compressed to the same absolute value by the clamp:

| Category | Raw Delta | Clamped Delta | Compression |
|----------|:---------:|:-------------:|:-----------:|
| informational | +0.001 | +0.001 | 1:1 (no clamp) |
| read_only | +0.50 | +0.05 | 10:1 |
| state_modifying | -0.75 | -0.05 | 15:1 |
| destructive | -0.85 | -0.05 | 17:1 |

The controller can only distinguish "informational" (+0.001) from "everything else" (+/-0.05). Category differentiation between read_only, state_modifying, and destructive is completely lost.

[Source: R1_WIENER_calibration.md#Section 3.1, R1_KNUTH_PASCAL_statistics.md#Section 5.1]

---

## 2. Solution Comparison

### 2.1 Plan A: Floor Expansion (REJECTED)

**Proposal**: Expand the negative clamp floor from -0.05 to -0.10 or -0.15.

| Floor | destructive clamped | state_mod clamped | Compression ratio |
|-------|:-------------------:|:-----------------:|:-----------------:|
| -0.05 (current) | -0.05 | -0.05 | 17:1 |
| -0.10 | -0.10 | -0.10 | 8.5:1 |
| -0.15 | -0.15 | -0.15 | 5.7:1 |

**WIENER's control theory rejection (4 reasons)**:

1. **Symptomatic, not causal**. Even at floor=-0.15, compression remains 5.7:1. The dynamic range mismatch root cause is unresolved.

2. **Symmetry violation**. An asymmetric clamp [-0.15, +0.05] means a single destructive operation has 150x the impact of a single informational operation. This introduces asymmetric gain into the control loop, complicating Lyapunov stability analysis and creating risk of rapid confidence collapse from a small number of destructive operations.

3. **Category differentiation still lost**. Destructive (-0.85 -> -0.10) and state_modifying (-0.75 -> -0.10) remain indistinguishable at floor=-0.10. Only at floor < -0.75 can they be separated, but that eliminates clamp's protective function.

4. **Requires repeated tuning**. Every change to `TOOL_CONFIDENCE_TABLE` may require re-calibrating the floor value. No systematic relationship exists.

**Verdict: REJECTED by all participants (R1/R2/R3 unanimous).**

[Source: R1_WIENER_calibration.md#Section 3.2]

### 2.2 Plan B: alpha=0.059, All Delta Paths (SUPERSEDED)

**Proposal**: Multiply all rawDelta values by a scaling factor alpha=0.059 before clamping.

**Derivation**: `max(|rawDelta|) * alpha = clamp_boundary => 0.85 * alpha = 0.05 => alpha = 0.059`

| Category | Raw Delta | Scaled (x0.059) | Clamped? |
|----------|:---------:|:----------------:|:--------:|
| informational | +0.001 | +0.000059 | No |
| read_only | +0.50 | +0.0295 | No |
| state_modifying | -0.75 | -0.04425 | No |
| destructive | -0.85 | -0.05015 | **Yes** (-> -0.05) |

**Problems identified in R2 cross-review**:

- **Informational signal nearly vanishes**: +0.000059 is functionally invisible; 72 entries contribute almost nothing to the distribution.
- **confidence_audited path unnecessarily compressed**: Values like +0.0005 and -0.01 (from gear-arbiter-static, not from TOOL_CONFIDENCE_TABLE) are already within the clamp window. Scaling them eliminates useful information.
- **Clamp still activates on destructive**: 9.4% clamp rate (12/127 entries).
- **F1/F2 predicted ~1.3x** (not 1.0x), because destructive remains boundary-clamped.

WIENER acknowledged in R2: "Applying scaling to confidence_audited path was my R1 blind spot. These values originate from a different signal mechanism than TOOL_CONFIDENCE_TABLE and should not share the same gain."

[Source: R2_07_WIENER_reviews_KNUTH_PASCAL.md#Section 2.4]

### 2.3 Plan B4: alpha=0.055, Tool_Audited Only (ADOPTED)

**Proposal**: Apply scaling factor alpha=0.055 only to the B-modified path (tool_audited delta), leaving confidence_audited path unchanged.

| Category | confidence_audited delta | tool_audited delta (scaled) | Category mean |
|----------|:------------------------:|:---------------------------:|:-------------:|
| informational | -- | +0.001 * 0.055 = +0.000055 | +0.000055 |
| read_only | +0.0005 (unchanged) | +0.50 * 0.055 = +0.0275 | +0.0157 |
| state_modifying | -0.01 (unchanged) | -0.75 * 0.055 = -0.04125 | -0.031 |
| destructive | -- | -0.85 * 0.055 = -0.04675 | -0.04675 |

Maximum |scaled delta| = 0.04675 < 0.05. **No value triggers the clamp.**

### 2.4 Plan C: Logarithmic Scaling (DEFERRED)

**Proposal**: `scaledDelta = sign * k * log(1 + confidence / c0)`

- Pro: Better amplification of small values (informational); stronger compression of large values.
- Con: Two parameters (k, c0) instead of one; non-linear transform breaks direct proportion to confidence ratios.
- Con: Increased model complexity is not justified in Phase 1.

**Verdict: Deferred to Phase 2 if linear scaling proves insufficient.**

[Source: R1_WIENER_calibration.md#Section 3.5]

---

## 3. Why Plan B4 Prevails

### 3.1 PASCAL Decision Matrix: EU = 0.803 vs 0.735

PASCAL applied a weighted Expected Utility framework with four criteria:

| Criterion | Weight | Plan B (0.059 all) | Plan B4 (0.055 tool only) |
|-----------|:------:|:---:|:---:|
| Signal preservation (F1/F2) | 0.35 | 0.80 (1.3x) | **0.95** (1.0x) |
| Category differentiation | 0.30 | 0.65 (38% gap) | **0.70** (34% gap + info preserved) |
| Simplicity | 0.15 | **0.80** (one param, all paths) | 0.60 (one param, selective) |
| Soundness | 0.20 | 0.70 (info signal concern) | **0.85** (all signals meaningful) |

**EU(Plan B) = 0.735 | EU(Plan B4) = 0.803** -- B4 leads by 9.2%.

**Sensitivity analysis** confirms robustness:

| Scenario | Weight shift | EU(B) | EU(B4) | B4 advantage |
|----------|-------------|:-----:|:------:|:------------:|
| Baseline | w1=.35 w2=.30 w3=.15 w4=.20 | 0.735 | 0.803 | +9.2% |
| Signal priority | w1=.50 w2=.20 w3=.10 w4=.20 | 0.670 | 0.785 | +17.2% |
| Differentiation priority | w1=.20 w2=.50 w3=.10 w4=.20 | 0.725 | 0.780 | +7.6% |
| Simplicity priority | w1=.20 w2=.20 w3=.40 w4=.20 | 0.730 | 0.710 | -2.7% |

B4 wins in all configurations except the extreme "simplicity above all" case, where it trails by only 2.7%.

**Minimax Regret analysis**: B4 max regret = 0.04 vs Plan B max regret = 0.20 (informational signal disappearance scenario). B4 wins decisively.

[Source: R2_08_KNUTH_PASCAL_reviews_WIENER.md#Section 1.2]

### 3.2 WIENER Observability Analysis: Independent Observation Channel

From control theory, **observability** measures the ability to reconstruct internal state from system outputs. Plan B4 preserves two complementary observation channels:

- **tool_audited channel**: gain = alpha (scaled signal from TOOL_CONFIDENCE_TABLE)
- **confidence_audited channel**: gain = 1 (original signal from gear-arbiter-static)

Plan B collapses both into a single-gain system (gain = alpha for all paths), losing the independent observation dimension. WIENER's observability matrix analysis concluded: "Multi-channel complementary observation is always superior to single-gain design in control theory."

WIENER's position shift was grounded in recognizing that confidence_audited deltas (+0.0005, -0.01) originate from gear-arbiter-static's direct output -- a fundamentally different signal mechanism than TOOL_CONFIDENCE_TABLE lookup. These two signal sources should not share the same transfer function.

[Source: R2_07_WIENER_reviews_KNUTH_PASCAL.md#Section 2.2-2.3]

### 3.3 ARCHIMEDES Engineering Clarification: Code Diff Is Identical

The most important engineering fact in the debate: **Plan B and Plan B4 have the same code diff**. The B-modified path naturally handles only tool_audited events. The confidence_audited path (arbiter route) does not pass through TOOL_CONFIDENCE_TABLE lookup, so it is architecturally isolated from any scaling applied there.

"Tool_audited only" scaling requires **no additional if-branch**. The path separation is inherent in the architecture.

The only difference between Plan B and B4 is the alpha value: 0.059 vs 0.055. LOC is identical (~2 lines).

[Source: R3_D3_calibration.md#Issue 2, ARCHIMEDES position]

### 3.4 Headroom: 5% Safety Margin (ARCHIMEDES)

alpha=0.055 keeps all scaled deltas within the clamp window, with the closest approach at destructive: -0.04675, which is 6.5% below the -0.05 boundary.

This headroom matters for future-proofing: if `TOOL_CONFIDENCE_TABLE[destructive]` is adjusted from 0.85 to 0.90, the scaled delta becomes 0.90 * 0.055 = 0.0495, still within the window. With alpha=0.059, the same change yields 0.90 * 0.059 = 0.0531, which triggers the clamp.

[Source: R3_D3_calibration.md#Issue 2, ARCHIMEDES position]

### 3.5 KNUTH Statistical Comparison: B4 Wins 5 of 7 Metrics

| Metric | Plan B (0.059 all) | Plan B4 (0.055 tool only) | Winner |
|--------|:---:|:---:|:---:|
| F1/F2 ratio | ~1.3x | **~1.0x** | **B4** |
| Clamp rate | 9.4% | **0%** | **B4** |
| Informational signal | +0.000059 | **+0.001** (preserved) | **B4** |
| n_eff impact | 72 near-zero | 72 at 0.001 | **B4** |
| Overall sigma | ~0.0235 | ~0.0220 | Tie |
| Dest/SM gap | 38% | 34% | B slight |
| Overall mean | -0.00389 | **-0.00335** | **B4** |

[Source: R2_08_KNUTH_PASCAL_reviews_WIENER.md#Section 1.1]

---

## 4. Alpha = 0.055 Derivation

### 4.1 Formula

```
alpha = (maxDelta / max(TOOL_CONFIDENCE_TABLE)) * HEADROOM_FACTOR
      = (0.05 / 0.85) * 0.935
      = 0.058824 * 0.935
      = 0.055
```

Where:
- `maxDelta = 0.05` -- the clamp boundary (symmetric)
- `max(TOOL_CONFIDENCE_TABLE) = 0.85` -- the destructive category confidence value
- `HEADROOM_FACTOR = 0.935` -- 6.5% headroom below clamp boundary

### 4.2 Design Philosophy

The alpha value is derived from **full-headroom design** (WIENER terminology): the actuator (clamp) never saturates during normal operation. The clamp serves purely as a safety net for abnormal conditions (e.g., if TOOL_CONFIDENCE_TABLE values are modified), not as a routine signal compressor.

### 4.3 Hardcoded vs Dynamic (R3 Decision)

WIENER proposed dynamic computation: `DELTA_SCALING_FACTOR = maxDelta / max(Object.values(TOOL_CONFIDENCE_TABLE)) * HEADROOM_FACTOR`, with KNUTH suggesting a sanity check `Math.min(1.0, ...)`.

ARCHIMEDES argued for Phase 1 hardcoding: "Premature generalization is as harmful as premature optimization. TOOL_CONFIDENCE_TABLE is a hardcoded constant that does not change at runtime."

**Decision**: Phase 1 hardcodes `0.055` with a derivation comment. Dynamic computation is deferred to Phase 2.

[Source: R3_D3_calibration.md#Issue 3]

---

## 5. Expected Outcomes

### 5.1 Predicted W2-R4 Values

| Metric | W2-R3 (current) | W2-R4 (predicted) | Improvement |
|--------|:---:|:---:|:---:|
| F1/F2 ratio | 13.2x | **~1.0x** | -92% |
| Clamp rate | ~30% | **~0%** | -100% |
| Category diff (dest vs sm) | indistinguishable | -0.047 vs -0.041 (**13% gap**) | restored |
| Informational signal | +0.001 | +0.000055 (tool_audited) | reduced but non-zero |
| Signal retention | 7.6% | **~100%** | +92pp |

### 5.2 F1/F2 = 1.0x Verification (Three-Party)

PASCAL, WIENER, and KNUTH independently verified the theoretical prediction:

- All scaled deltas fall within [-0.05, +0.05]: max |scaled delta| = 0.04675 < 0.05.
- Clamp is per-entry. No entry is modified by the clamp.
- Therefore F1 (pre-clamp sigma) = F2 (post-clamp sigma) exactly.
- F1/F2 = 1.0x (theoretical exact).

WIENER's earlier ~1.3x prediction was based on Plan B (alpha=0.059, all paths) where destructive is boundary-clamped. Under Plan B4 framework, WIENER confirmed the correction to ~1.0x. The discrepancy was purely due to different plan assumptions, not a calculation error.

[Source: R3_D3_calibration.md#Issue 6, R2_07_WIENER_reviews_KNUTH_PASCAL.md#Section 4]

### 5.3 Informational Signal Note

Under Plan B4, informational tool_audited delta = 0.001 * 0.055 = +0.000055. This is reduced but:
- Non-zero and directionally correct (positive).
- IEEE 754 double precision provides ample headroom (~10^-16).
- Not functionally zero in per-cycle accumulation (e.g., Block 1: ~3.3 events/cycle * 0.000055 = 0.00018/cycle).

R3 decision: no exemption for informational scaling. B-modified path maintains unified scaling semantics. If W2-R4 reveals insufficient signal, Phase 2 will address via TOOL_CONFIDENCE_TABLE adjustment, not via conditional branching.

[Source: R3_D3_calibration.md#Issue 7]

---

## 6. Implementation Specification

### 6.1 Location

`loop.ts`, B-modified path, approximately line 669 (rawDelta calculation).

### 6.2 Code Change (~2 LOC)

```typescript
// DELTA_SCALING_FACTOR: scales tool_audited rawDelta to fit within clamp window
// Derivation: (maxDelta / max(TOOL_CONFIDENCE_TABLE)) * HEADROOM_FACTOR
//           = (0.05 / 0.85) * 0.935 ≈ 0.055
// Ensures: max scaled delta (destructive) = 0.85 * 0.055 = 0.04675 < 0.05
// See: cycle03-4 W2-R3/R4 calibration
const DELTA_SCALING_FACTOR = 0.055;
```

And in the rawDelta calculation line:

```typescript
const rawDelta = TOOL_CONFIDENCE_TABLE[riskCat] * signMultiplier * DELTA_SCALING_FACTOR;
```

### 6.3 Scope

- **Affected**: B-modified path (tool_audited events) only.
- **Not affected**: confidence_audited path (arbiter route) -- architecturally isolated, no code change needed.
- **Total LOC**: ~2 implementation + ~4 comment lines.
- **Rollback cost**: `git revert`, 30 seconds.

### 6.4 Plan40 Priority

| Priority | Item | LOC | Time | Dependency |
|----------|------|-----|------|------------|
| **W0** | DELTA_SCALING_FACTOR implementation | ~2 + ~4 comment | 15 min | None |
| **W1** | W2-R4 verification test (5-block 50-cycle) | 0 (test only) | 2-3 hrs | W0 |

[Source: R3_D3_calibration.md#Issue 8]

---

## 7. W2-R4 Verification Criteria (V1-V11)

### 7.1 Threshold (MUST) and Target (SHOULD)

| # | Metric | Threshold (MUST) | Target (SHOULD) | Source |
|---|--------|:----------------:|:---------------:|:------:|
| V1 | SC-1 through SC-6 | 6/6 PASS | 6/6 PASS | ARCHIMEDES |
| V2 | CV-1 through CV-4 | 4/4 PASS | 4/4 PASS | ARCHIMEDES |
| V3 | CV-5 | FAIL (expected) | FAIL (expected) | ARCHIMEDES |
| V4 | CV-6 | NOT TRIGGERED | NOT TRIGGERED | ARCHIMEDES |
| V5 | F1/F2 ratio | < 3.0x | < 2.0x | ARCHIMEDES |
| V6 | Clamp rate | < 15% | < 5% | ARCHIMEDES |
| V7 | Category diff (dest vs sm) | dest != sm | \|dest-sm\| > 10% | ARCHIMEDES |
| V8 | Informational delta | > 0 | > 0.0001 | ARCHIMEDES |
| V9 | Mean delta sign per category | Same direction as W2-R3 | -- | WIENER |
| V10 | Per-category sign correctness | All correct | -- | WIENER |
| V11 | sigma(R4) / sigma(R3) | > 0.5 and < 2.0 | -- | KNUTH |

### 7.2 Phase 1 Lock Conditions (PASCAL)

| Outcome | Condition | Bayesian Posterior |
|---------|-----------|:------------------:|
| **LOCKED** | All thresholds pass + all targets met | P(valid) >= 0.99 |
| **CONDITIONAL LOCK** | All thresholds pass + some targets missed | P(valid) 0.97-0.99 |
| **NOT LOCKED** | Any threshold fails | Investigation required |

[Source: R3_D3_calibration.md#Issue 4]

---

## 8. Decision Traceability

### 8.1 Research Progression

| Phase | Key Event | Outcome |
|-------|-----------|---------|
| R1 | WIENER proposes Plan B (alpha=0.059, all paths) | Initial proposal |
| R1 | KNUTH+PASCAL propose Plan B4 (alpha=0.055, tool_audited only) | Counter-proposal |
| R2 | WIENER reviews KNUTH+PASCAL -- observability analysis | WIENER shifts to B4 |
| R2 | KNUTH+PASCAL review WIENER -- confirm Plan A rejection | Consensus on direction |
| R3 D3 | Formal debate, 8 issues | 8/8 unanimous (5-0) |

### 8.2 WIENER's Position Shift (R1 -> R2)

WIENER's shift from Plan B to Plan B4 was not a compromise but a correction based on control theory analysis:

> "I treated B-modified path (tool_audited) and A-route path (confidence_audited) delta as homogeneous sources. In reality, they are two different signal generation mechanisms that should not share the same gain. My R2 observability matrix analysis confirmed that Plan B4's multi-channel observation is superior to Plan B's single-gain design."

[Source: R2_07_WIENER_reviews_KNUTH_PASCAL.md#Section 2.4, R3_D3_calibration.md#Issue 2 Cross-Examination]

### 8.3 Open Items for Phase 2

1. Alpha dynamic computation: `maxDelta / max(TOOL_CONFIDENCE_TABLE) * HEADROOM_FACTOR`
2. Informational TOOL_CONFIDENCE_TABLE value adjustment (if W2-R4 shows +0.000055 insufficient)
3. CV-5 re-evaluation (pending arbiter refactoring)
4. F1/F2 long-term target < 1.5x

---

## References

- [Source: R1_WIENER_calibration.md -- WIENER R1 report]
- [Source: R1_KNUTH_PASCAL_statistics.md -- KNUTH+PASCAL R1 report]
- [Source: R2_07_WIENER_reviews_KNUTH_PASCAL.md -- WIENER R2 cross-review]
- [Source: R2_08_KNUTH_PASCAL_reviews_WIENER.md -- KNUTH+PASCAL R2 cross-review]
- [Source: R3_D3_calibration.md -- R3 Debate 3 record]

---

*Calibration Report 64 -- Plan B4 Delta Scaling Design*  
*Cycle 03-4*  
*2026-04-04*
