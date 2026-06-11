# 69. M4a Dual-Track Design Rationale

## Document Metadata
- **Cycle**: 03-7
- **Contributors**: BABBAGE (formal verification), WIENER (control theory), PENROSE (quantum consciousness), RUSSELL (agent theory)
- **Related**: [63_W2_Phase2_Calibration_Architecture.md], [68_Phase2_Completion_Analysis.md], Rule #55, Rule #57, Rule #59

## 1. M4a Definition

M4a (shadow agreement rate) measures the degree to which the dynamic gear arbiter's shadow decisions align with the static arbiter's actual decisions. It is the primary metric for Phase 3 evaluation and the key gate for Phase 4 authority transfer.

## 2. Binary Track (Primary)

The binary track computes a simple agreement ratio per operation category:

```
M4a_binary[cat] = count(shadow_gear == actual_gear) / total[cat]
```

This produces a value in [0, 1] where 1.0 means perfect agreement. The binary track serves as the **primary** decision metric for phase transition gates.

## 3. Mean Deviation Track (Diagnostic)

The mean deviation track computes the average absolute difference between shadow and actual gear selections per category (Rule #59):

```
M4a_deviation[cat] = mean(|shadow_gear - actual_gear|) / total[cat]
```

This track is **diagnostic only** and does not gate phase transitions. It provides granularity that the binary track cannot capture.

## 4. Why Dual-Track

The dual-track design arose from BABBAGE's R2 cross-review challenge. The binary track treats all disagreements equally: a shadow decision of gear 4 vs actual gear 5 counts the same as gear 1 vs actual gear 5. This discards critical information about the **magnitude** of disagreement.

Consider two scenarios with M4a_binary = 0.60 (60% agreement):
- Scenario A: All disagreements are off-by-one (shadow=4 vs actual=5). Mean deviation = 0.40.
- Scenario B: All disagreements are maximum deviation (shadow=1 vs actual=5). Mean deviation = 1.60.

Both scenarios produce identical binary M4a, but represent fundamentally different arbiter behaviors. Scenario A suggests the dynamic arbiter has learned the general pattern but needs refinement. Scenario B suggests the dynamic arbiter is operating from an entirely different model.

## 5. Threshold Expectations (HYPOTHESES)

The following thresholds are labeled explicitly as **hypotheses**, not validated gates. They represent the research team's current best estimates, pending W2-R7 empirical validation:

| Category | Binary Threshold (hypothesis) | Rationale |
|----------|-------------------------------|-----------|
| informational | > 0.85 | High-frequency, predictable patterns, largest sample |
| read_only | > 0.80 | Moderate complexity, adequate sample size |
| state_modifying | 0.60 - 0.80 | Higher variance expected, smaller sample |
| destructive | 0.50 - 0.70 | Monitoring-only, anomaly detection (Rule #57) |

These thresholds will be validated or revised based on actual W2-R7 M4a data. Formal justification is deferred until empirical data is available.

## 6. Destructive Category: Monitoring-Only

Per Rule #55 (destructive operations are NEVER delegated), the destructive category M4a serves a fundamentally different purpose than other categories. For destructive operations, M4a is used exclusively for:

- **Anomaly detection**: Identifying when shadow decisions diverge significantly from actual decisions, which may indicate environmental changes or model drift.
- **Monitoring**: Tracking agreement trends over time without any authority transfer implications.

Rule #57 codifies this: destructive M4a is a monitoring metric, not a transition gate.

## 7. Perturbation Diagnostic

To assess M4a sensitivity and prevent overfitting to a stable environment, a perturbation diagnostic is defined: inject +/-1 count into a single category and observe M4a response. This tests whether M4a captures genuine decision quality or merely reflects environmental stability.

## 8. Critical Concerns

### 8.1 PENROSE Concern: Shared StateTracker Entanglement

The dynamic and static arbiters share the same StateTracker instance. PENROSE identified that this creates an entanglement: the dynamic arbiter's shadow decisions are conditioned on state that was shaped by the static arbiter's actual decisions. M4a may therefore measure **shared-state consistency** rather than independent decision quality. True independence would require the dynamic arbiter to maintain a separate state trajectory, which is architecturally infeasible in the current design.

### 8.2 RUSSELL Concern: Agreement vs Quality

RUSSELL identified a fundamental limitation: the dynamic arbiter lacks an independent utility function. It is being trained to replicate the static arbiter's behavior, not to optimize an independent objective. M4a therefore measures "agree with the status quo" rather than "make good decisions." This means high M4a proves non-inferiority (the dynamic arbiter is no worse than static) but cannot prove superiority.

## 9. Phase 3 Goal: Non-Inferiority

Given the concerns above, the Phase 3 research objective is explicitly scoped to **confirm non-inferiority**: demonstrate that the dynamic arbiter can match the static arbiter's decisions with acceptable accuracy. Proving superiority is deferred to Phase 4, where limited real-world delegation would provide ground-truth outcome data.

## 10. Design Implications

The dual-track architecture ensures that Phase 3 evaluation captures both the frequency of agreement (binary) and the severity of disagreement (deviation). This combination provides a more complete picture for the Phase 4 transition decision while acknowledging the inherent limitations of shadow-mode evaluation.

[Source: Cycle 03-7 R2 cross-review challenges, R3 D4 M4a debate]
