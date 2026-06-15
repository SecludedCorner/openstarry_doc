# redaction_security_debt Schema v2 Candidate — Cycle 03-20 R4

**Status**: BINDING (cycle 03-20 R3 D-§3 ratified UNANIMOUS 23/0 8 sub-items + 1 super-majority 22/1; pending Master Ratification Batch 17 #2)
**Authority**: Master Ratification (Batch 17 dispatch 2026-05-02)
**Cycle**: 03-20 (consolidation; Reference/15 second concrete application)
**Source**: cycle 03-20 R3 D-§3 + R2 §3 (BABBAGE + LINNAEUS + RUSSELL)
**Target document**: `Calibration_Reports/redaction_security_debt.md` (cycle 03-19 ratified BINDING canonical) — Schema v2 update

---

## §1 Schema v2 Changes (compound 6-item codification)

### Change 1: 4-State Ordinal Sunset Progression (LINNAEUS R2; UNANIMOUS 23/0)

**Original** (cycle 03-19 ratified): 13-field schema; sunset via DEPRECATED label only.

**v2 Amendment**: Add 4-state ordinal sunset_state field (typed enum):

```
sunset_state: enum {
  PENDING_DEV     // scheduled commitment; retrofit pending
  RETROFIT_DONE   // retrofit complete; sunset clause not triggered
  DEPRECATED      // sunset clause triggered; replacement scheduled
  RETIRED         // replacement complete; entry retained for audit history
}
```

**Ordinal monotone-progression**: PENDING_DEV → RETROFIT_DONE → DEPRECATED → RETIRED (no skip; no reverse).

### Change 2: Typed-Enum sunset_state Field (BABBAGE R2; UNANIMOUS 23/0)

Add explicit field type declaration to schema. Field 14 of v2 = `sunset_state: enum`.

### Change 3: Sensitivity-Class Differential Redaction Documentation (LINNAEUS R2; UNANIMOUS 23/0)

Add sensitivity-class HIGH / MED / LOW differential redaction format documentation per category:

| Sensitivity | Categories | Redaction format |
|:-----------:|-----------|------------------|
| HIGH | retrieve / track-context / verify | `<redacted-volition-payload len:NN first0:>` (no hint) |
| MED | metadata / config / state | `<redacted-volition-payload len:NN first2:ab>` (2-char hint) |
| LOW | timestamp / source-ref / type | `<redacted-volition-payload len:NN first4:abcd>` (4-char hint default) |

### Change 4: Shared Helper Plan52 sourceContext SPOF Forward-Watch (RUSSELL R2; super-majority 22/1)

Add register column `helper_dependency` tracking Plan52 sourceContext shared `redact_volition_payload` helper. SPOF forward-watch enabled cycle 03-21+ Mesh.

**Dissent preserved per MR-11**:
- **DSS-CY20-§3-A** (DARWIN, 1 vote): "Helper SPOF acceptable for first-shipping; YAGNI; defer SPOF mitigation until actual incident triggers."

### Change 5: capability_orphan_status Attestation Standby (RUSSELL R2; UNANIMOUS 23/0)

Add register column `capability_orphan_status` (NULL until DEPRECATED entry). When sunset_state = DEPRECATED, populate with attestation:
- `IS_ORPHAN`: replacement plugin not yet identified → escalate to Master Ratification
- `HAS_REPLACEMENT`: replacement_plugin_id field populated → continue per replacement schedule

### Change 6: ENG-FAB v1.9 F-16 Cross-Cycle Schema-Discipline Link (BABBAGE R2; UNANIMOUS 23/0)

Cross-cycle linkage: register schema discipline ↔ F-16 StructuredError schema discipline. Both governed by ENG-FAB v1.9 framework. When F-16 升 MUST (future cycle 03-21+ R3), register schema field validation should reference F-16 6-field schema enforcement strength.

---

## §2 Schema v2 Updated Field List (15 fields)

| # | Field | Type | Description |
|:-:|-------|------|-------------|
| 1 | `plugin_id` | string | Plugin package identifier |
| 2 | `family` | string | Family group (provider / context / etc) |
| 3 | `sensitivity_class` | enum HIGH/MED/LOW | per category-aware redaction |
| 4 | `pre_format` | string | Format before retrofit |
| 5 | `post_format` | string | Format after retrofit |
| 6 | `retrofit_cycle` | string | Cycle when retrofit applied |
| 7 | `retrofit_pr` | string nullable | PR number / commit hash |
| 8 | `tri_party_audit` | enum TANENBAUM/KERNEL/GUARDIAN | Sign-off status |
| 9 | `f15_lint_status` | enum PASS/FAIL | F-15 v3 linter |
| 10 | `cross_os_ci` | enum PASS/FAIL | Linux + Windows |
| 11 | `w2_regression` | enum PASS/FAIL | W2-Rxx regression test |
| 12 | `dissent_inheritance` | string nullable | DSS preservation tracking |
| 13 | `notes` | string nullable | Free-form context |
| **14** | **`sunset_state`** | **enum** | **PENDING_DEV / RETROFIT_DONE / DEPRECATED / RETIRED (NEW v2)** |
| **15** | **`capability_orphan_status`** | **enum nullable** | **NULL / IS_ORPHAN / HAS_REPLACEMENT (NEW v2; populate at DEPRECATED)** |

**Plus**: `helper_dependency` column (NEW v2; SPOF tracking).

---

## §3 Migration from v1 to v2

Per MR-12 forward-only: v1 entries (cycle 03-19 ratified) preserved verbatim. v2 fields populated forward at cycle 03-20+ entries.

For cycle 03-19 baseline v1 entries:
- `sunset_state`: backfill = PENDING_DEV (all 18 IN-SCOPE plugins)
- `capability_orphan_status`: NULL (all)
- `helper_dependency`: backfill = "Plan52 sourceContext shared `redact_volition_payload`" (all 8 provider-* + 4 context-* via shared helper; others = "direct" inline)

---

## §4 Compliance

| Constraint | Status |
|------------|:------:|
| MR-12 forward-only | ✅ v1 preserved; v2 forward-binding |
| MR-11 dissent | ✅ DSS-CY20-§3-A preserved verbatim |
| Reference/15 second concrete application | ✅ |
| F-15 v3 schema | ✅ |
| ENG-FAB v1.9 link | ✅ cross-cycle schema-discipline |

---

*redaction_security_debt Schema v2 Candidate — Cycle 03-20 R4 — 2026-05-02*
*Master Ratification Batch 17 #2 dispatch ready*
*Coordinator G5 sync stage updates canonical redaction_security_debt.md with v2 schema + creates redaction_security_debt.tw.md sibling per Rule #78 §78.5*
