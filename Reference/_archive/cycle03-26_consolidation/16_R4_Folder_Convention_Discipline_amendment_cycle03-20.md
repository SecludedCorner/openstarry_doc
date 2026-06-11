# Reference/16 §3.1 BINDING Amendment — Cycle 03-20

**Status**: BINDING amendment proposal (cycle 03-20 R3 D-§4 ratified 23/0 UNANIMOUS; pending Master Ratification Batch 17 #3)
**Authority**: Master Ratification (Batch 17 dispatch 2026-05-02)
**Cycle**: 03-20 (consolidation; 4 防線 second enforce drill SUCCESS)
**Source**: cycle 03-20 R3 §5 + R2 §4 (VITRUVIUS + DARWIN + KNUTH amendments)
**Target document**: `Reference/16_R4_Folder_Convention_Discipline.md` §3.1

**Note**: This amendment patch DOES NOT modify the original Reference/16 binding text (per MR-12 forward-only); it is a forward-binding addendum that augments the original 4 防線 with 4 AMEND clauses ratified at cycle 03-20 R3. Coordinator G5 sync stage applies this amendment to canonical Reference/16 + creates Reference/16.tw amendment sibling per Rule #78 §78.5 BINDING-tier reflexive.

---

## §1 4 AMEND Items (cycle 03-20 R3 D-§4 23/0 UNANIMOUS each)

### AMEND-1: 防線 3 ≠ 防線 2 Backstop (HIGH)

**Original** (Reference/16 §3.1.3): G4-folder-3 加嚴 BINDING checks baseline 完整 + cycle-specific diff + CHANGELOG entry; 三項缺一即 fail.

**Amendment** (cycle 03-20 R3): **防線 3 (G4-folder-3 加嚴) is NOT a backstop for 防線 2 (R4 dispatch task explicit step)**. Upper-layer PASS does not validate lower-layer; **independent enforcement required** for each 防線. 防線 3 PASS via baseline 完整 + diff + CHANGELOG does NOT excuse 防線 2 silent skip (R4 dispatch task missing explicit step).

**Rationale** (VITRUVIUS R2 HIGH amendment): Architectural soundness requires non-overlapping defensive layers; conflating them creates false-positive PASS where 防線 2 is silently bypassed but 防線 3's surface check passes due to baseline + diff + CHANGELOG presence (irrespective of dispatch-task content).

### AMEND-2: 防線 4 Silent Skip Secondary Failure Mode (HIGH)

**Original** (Reference/16 §3.1.4): G5 BINDING canonical doc creation check; coordinator G5 stage 對照 ratified BINDING items vs canonical; 缺則自動 create at G5.

**Amendment** (cycle 03-20 R3): **防線 4 silent skip = secondary failure mode**. Coordinator G5 sync stage MUST emit **explicit non-empty enumeration** of "ratified BINDING items checked against canonical" even when result = "0 漏建". A blank/silent enumeration is a fail signal, not a PASS signal.

**Rationale** (VITRUVIUS R2 HIGH amendment): Symmetric to AMEND-1 — 防線 4 PASS requires explicit attestation, not absence of negative signal.

### AMEND-3: Mechanical Template Boilerplate (MED)

**Original** (Reference/16 §3): R4 SCRIBE + ARCHIMEDES 共筆責任明文 (responsibility matrix).

**Amendment** (cycle 03-20 R3): **Drill SUCCESS protection via mechanical template boilerplate** (verbatim header inclusion in R4 dispatch task description) NOT via "補強 culture" (too soft per DARWIN R2 MED). Specifically:

```
### Master directive 2026-05-01 防線 2 EXPLICIT STEP (本輪首次/續次 enforce)

> openstarry_doc/ 完整 baseline (≥ canonical N files) + cycle-specific BINDING 新 docs (per Batch M ratified items including [list]) MUST be present at R4 close; G4-folder-3 hard fail otherwise.
>
> R4 SCRIBE + ARCHIMEDES 共筆責任明文 (per Reference/16 §3 responsibility matrix).
```

Verbatim header inclusion is REQUIRED in every cycle's R4 dispatch task description, even when no new BINDING canonical doc is anticipated (template boilerplate provides invariance).

**Rationale** (DARWIN R2): "Convenience-driven mechanism erosion" anti-pattern (skipping verbatim header when content is "trivial") is the primary failure mode for mechanism erosion across cycles.

### AMEND-4: 防線 1 INVARIANT Strengthen (MED)

**Original** (Reference/16 §3.1.1): Coordinator pre-R4 baseline audit (operational); log baseline backfill action.

**Amendment** (cycle 03-20 R3): 防線 1 INVARIANT strengthen from `audit_log_nonempty` to:

```
audit_log_nonempty 
  ∧ enumerates_baseline_count_N
  ∧ enumerates_diff_state (added/modified/deleted files vs canonical)
```

**Rationale** (KNUTH R2 MED): Algorithmic rigor — "non-empty log" alone is insufficient; log must contain actionable state (baseline N count + diff state) for downstream 防線 enforcement.

---

## §2 4 CLARIFY Items (cycle 03-20 R3 D-§4 23/0 UNANIMOUS each)

### CLARIFY-1: 3 Anti-Pattern Naming (DARWIN R2)

Reference/16 §6 (anti-patterns) explicit names:
- **convenience-driven mechanism erosion**: skipping verbatim header / silent skip when content "trivial"
- **graceful degradation gone wrong**: lighter cycle treated as "no enforce needed" (vs invariance test)
- **structural completeness over content brevity**: defensive structure preserved even when content is concise

### CLARIFY-2: R3 Plan-Level Confirm vs R4 Final SUCCESS Boundary (DARWIN R2)

- **R3 (plan-level)**: drill SUCCESS confirmation (vote-based; informational receipt)
- **R4 (final SUCCESS)**: drill execution evidence + canonical doc creation + CHANGELOG + transfer + actual G4 gate PASS

R3 confirm ≠ R4 SUCCESS; both required.

### CLARIFY-3: 防線 3 `diff_present` Semantics (KNUTH R2)

- `diff_present := diff_log_exists ∧ enumerates_state` (NOT `nonempty` alone)
- `diff_log_exists`: cycle-specific diff folder structure present
- `enumerates_state`: explicit added/modified/deleted file list (not just folder existence)

### CLARIFY-4: Plan-Level Auditing vs Execution-Level Auditing

- **Plan-level**: Reference/16 §3 responsibility matrix (who is responsible)
- **Execution-level**: 4 防線 enforce mechanism (what is checked when)

Both layers required; conflating them obscures accountability.

---

## §3 Cycle 03-21+ Forward Carry-Forward

- This amendment patch + Reference/16.tw amendment sibling apply forward cycle 03-21+
- Reference/16 original text preserved (per MR-12 forward-only)
- Coordinator G5 sync stage updates canonical Reference/16 with this amendment patch + creates TW sibling
- 4 防線 enforce continues with strengthened amendments cycle 03-21 載重最重升級一輪 + cycle 03-22+

---

## §4 Compliance

| Constraint | Status |
|------------|:------:|
| MR-12 forward-only | ✅ amendment is forward addendum; original preserved |
| MR-11 dissent preservation | ✅ 0 dissent at this AMEND ratification (all 23/0 UNANIMOUS) |
| Rule #78 §78.5 TW sibling | ✅ Reference/16.tw amendment sibling required at coordinator G5 sync |
| F-15 v3 schema | ✅ |
| Master directive 2026-05-01 | ✅ extends 4 防線 binding |

---

*Reference/16 §3.1 BINDING Amendment — Cycle 03-20 R3 D-§4 23/0 UNANIMOUS — 2026-05-02*
*Master Ratification Batch 17 #3 dispatch ready*
*Drill = invariance test PASSED (cycle 03-19 重輪 first enforce + cycle 03-20 lighter second enforce)*
