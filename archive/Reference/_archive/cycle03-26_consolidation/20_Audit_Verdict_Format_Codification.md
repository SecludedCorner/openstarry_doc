---
title: Reference/20 — Audit Verdict Format Codification
status: BINDING (cycle 03-25 R3 D-§13 22/1 super-majority; pending Master Ratification Batch 22 #7)
authority: SCRIBE-internal authority + R3 ratified
date: 2026-05-02
cycle: 03-25
authors: SCRIBE + KNUTH + ARCHIMEDES + LINNAEUS
supersedes: (none — first canonical codification)
cross_refs:
  - research record/cycle03-25/deliver/O7_A9_audit_verdict_format_final.md (full-form twin)
  - research record/cycle03-25/R3/decision_log.md DSS-CY25-§13-A
  - openstarry_doc/Reference/16_R4_Folder_Convention_Discipline.md (G4-folder-3 anchor)
mr_zt_refs: [MR-7, MR-11, MR-12, Tenet #2, Tenet #8, F-15 v3]
---

# Reference/20 — Audit Verdict Format Codification

**Status**: BINDING — cycle 03-25 R3 §1.2 D-§13 super-majority 22/1 (LEIBNIZ DSS-CY25-§13-A preserved verbatim per MR-11). Pending Master Ratification Batch 22 #7.

---

## §1 Background

Cycle 03-26 inherits a **530-680 cell matrix sweep** (cell = `<chapter_id>_<tenet_axis>`; D-§12 cycle 03-25 governs enumeration). Each cell carries an audit verdict; aggregated row + column results feed the cycle 03-26 GP-coherence pre-gate and downstream G-gates.

Pre-cycle-03-25 verdicts were ad-hoc strings (`PASS`, `PASS UNCONDITIONAL`, `CONDITIONAL PASS`, `FAIL`, bare prose). Two operational hazards: (a) machine-parseability blocked at ~5,300-6,800 verdict tokens (3-4 person-weeks manual; 6-8 hours linted); (b) verdict semantic ambiguity (cycle 03-21 R3 §6 audit_calc state-(a) demonstrated 3 false-positive cells with unresolved cross-axis dependencies).

**Prerequisite**: cycle 03-26 矩陣 sweep cell verdict 依本格式 binding.

---

## §2 4-tier verdict taxonomy + N/A 5th lane

In descending strength:

1. **`full_pass`** — All Tenet sub-checks GREEN; no caveats; no roll-forward.
2. **`conditional_pass`** — GREEN under explicit named caveats (`caveats:` field non-empty). Allowed **audit-context only** post-MR-7; **forbidden in G-gate context**. N=3 consecutive cycles roll-forward ceiling (R2-D §5).
3. **`fail`** — One or more sub-checks RED; full re-audit required next cycle.
4. **`fail_fix=<scope>`** — RED with bounded fix scope (free-form ASCII identifier 1-128 chars). Next-cycle re-audit on `<scope>` only.

**N/A 5th lane**: `not_applicable` — cell × Tenet axis combination does not apply. **Not a verdict**; lane-distinguished from the 4-tier ladder. Accounted in per-row `applicable_count` denominator only.

Strength order: `full_pass > conditional_pass > {fail, fail_fix=<scope>}`. The two FAIL forms are not strictly ordered (`fail_fix=*` operationally softer; semantically identical-or-weaker).

---

## §3 YAML schema (I-13.1..I-13.7)

```yaml
cell_id: <string>                    # I-13.1; per D-§12 cycle 03-25 enumeration
verdict: <token>                     # full_pass | conditional_pass | fail | fail_fix=<scope> | not_applicable
caveats: [<string>, ...]             # I-13.2; required iff verdict == conditional_pass
scope: <string>                      # I-13.3; required iff verdict matches ^fail_fix=
evidence:                            # I-13.4 + I-13.7; ≥1 channel non-empty unless not_applicable
  file_paths: [<path#line>, ...]
  test_ids: [<test-id>, ...]
  static_analysis_report_ids: [<id>, ...]
  runtime_trace_ids: [<id>, ...]
reviewer:
  primary: <agent-id>                # I-13.5; original author / audit primary
  secondary: <agent-id>              # R2-A non-original-author 2nd reviewer; required for {conditional_pass, fail, fail_fix=*}
roll_forward_count: <int>            # I-13.6; ≥0; ≤3 if conditional_pass per R2-D
cycle: <cycle-id>                    # e.g. "03-25"
emit_source: <enum>                  # ENG-FAB F-12 enum (cycle 03-19 D-§2 332 sub-check binding)
notes: <string>                      # ≤500 chars
```

**Lint** (`tools/audit_verdict_lint.py`; CI hook G4-folder-3 gate; cycle 03-26 implementation slot):

| ID | Rule |
|----|------|
| L1 | `verdict` token in 5-token canonical enum |
| L2 | `caveats` non-empty iff `verdict == conditional_pass` |
| L3 | `scope` non-empty iff `verdict` matches `^fail_fix=` |
| L4 | `evidence.*` ≥1 channel non-empty unless `not_applicable` |
| L5 | `reviewer.secondary` non-empty if verdict ∈ {conditional_pass, fail, fail_fix=*} (R2-A) |
| L6 | `roll_forward_count` ≤3 if `verdict == conditional_pass` (R2-D) |
| L7 | `cell_id` matches D-§12 enumeration |

**R2-E SHALL splits per method tier**: each evidence channel SHALL be tagged either `SHALL_AUTOMATIC` (lint-enforced) or `SHALL_REVIEWED` (human-gate via `reviewer.secondary`).

---

## §4 Aggregation algorithm

### §4.1 Per-row aggregation (Mixed PASS rule)

```
A = applicable_count = |{cells where verdict != not_applicable}|
F = fail_count + fail_fix_count
C = conditional_pass_count
P = full_pass_count

if F > 0:                       row_aggregate = fail
elif C > 0 and P > 0:           row_aggregate = mixed_pass
elif C > 0 and P == 0:          row_aggregate = conditional_pass
elif P == A:                    row_aggregate = full_pass
elif A == 0:                    row_aggregate = not_applicable
else:                           row_aggregate = error  # unreachable
```

**Mixed PASS** is novel to this codification: distinguishes "row mostly green with one or two caveat cells" from blanket conditional_pass. Acceptable in audit-context; flagged for cycle-N+1 caveat closure.

### §4.2 Per-column aggregation (per Tenet axis)

```
column.full_pass_pct = full_pass_count / applicable_count_in_column
column.conditional_pass_pct = conditional_pass_count / applicable_count
column.fail_pct = (fail_count + fail_fix_count) / applicable_count
column.not_applicable_count = not_applicable_count
```

Cycle 03-26 R3 GP-coherence pre-gate threshold candidate: each column `full_pass_pct ≥ 90%` (ratification deferred cycle 03-26 R3).

### §4.3 Kahn topological sort with cycle detection (R2-C)

```
Step 1: build adjacency graph G(V=cells, E=cell.depends_on)
Step 2: compute in-degree per cell
Step 3: queue Q ← all in-degree-0 cells, sorted cell_id ASC (R2-C determinism)
Step 4: while Q non-empty:
        pop c; emit c
        for neighbour n: decrement n.in_degree; if 0, sorted-insert into Q
Step 5: if |output| < |V|:
        cycle_members ← V \ output       # R2-C explicit cell_ids enumeration
        emit error(cycle_members)
        halt aggregation; escalate Master MR-7 (§5 tier-3)
        else: return output
```

**Determinism**: deterministic by `cell_id ASC` tiebreak; two runs over same input MUST produce byte-identical aggregate (cycle 03-26 R3 sanity check).

---

## §5 G-gate exception clause + endless-loop 4-tier hardening

### §5.1 G-gate exception clause anchor (MR-7 BINDING)

- **Audit-context** (W-round audit / retrospective / cycle-mid compliance / cycle 03-26 矩陣 sweep): Conditional PASS allowed.
- **G-gate context** (G0/G1/G2/G3/G4): Conditional PASS NOT allowed; must resolve to `full_pass` / `fail` before G-gate dispatch, OR be excluded from gate scope, OR escalate Master MR-7.

### §5.2 Endless-loop hardening 4 tiers

| Tier | Mechanism | Source |
|------|-----------|--------|
| 1 | 3-iteration soft cap per audit pass (within single audit run) | R1 §13.6 |
| 2 | Dependency closure (caveat-referenced cells must be enumerated and resolved) | R1 §13.6 |
| 3 | Master MR-7 backstop (Kahn cycle detection OR R2-D ceiling exceeded) | R1 §13.6 + R2-D |
| 4 | **Conditional PASS roll-forward N=3 consecutive cycles ceiling (R2-D BINDING)** | R2 §13.4 |

**Tier 4 (R2-D)**: a cell may roll `conditional_pass` forward at most N=3 consecutive cycles. On the 4th cycle, MUST transition to `full_pass` (caveats resolved) or `fail` / `fail_fix=<scope>` (caveats unresolved); otherwise escalate Master MR-7. `roll_forward_count` field tracked in YAML; reset to 0 at full_pass / fail transition.

R2-D **closes the perpetual roll-forward loophole** (cycle 03-23 W2-R30 5-cycle drift incident).

---

## §6 Per-cell evidence citation discipline

Per I-13.7 + R2-B lint enforcement, **four citation channels** (no free-form):

1. `file_paths`: `<repo-relative-path>#<line>` or `<repo-relative-path>#<line>-<line>`
2. `test_ids`: test function name or file+name
3. `static_analysis_report_ids`: ENG-FAB F-15 / F-12 lint report IDs
4. `runtime_trace_ids`: W-round runtime traces / audit_calc state-(a) traces

**Empty-evidence rule**: cell with `verdict != not_applicable` AND all 4 channels empty fails L4 lint.

**Reviewer attribution (R2-A)**: For `conditional_pass`, `fail`, `fail_fix=<scope>`, `reviewer.secondary` MUST be non-original-author (cycle 03-13 R2 reviewer non-overlap principle extended to verdict assignment). For `full_pass` and `not_applicable`, secondary recommended but not required.

**R2-E method tier split**: each evidence channel SHALL be `SHALL_AUTOMATIC` (lint validates: file_paths exist, test_ids resolve) or `SHALL_REVIEWED` (reviewer.secondary attests). The two tiers may co-exist on one cell.

---

## §7 DSS-CY25-§13-A — LEIBNIZ verbatim (per MR-11)

**Author**: LEIBNIZ. **Cycle**: 03-25 R3 §1.2 D-§13. **Position**: against majority N=3 ceiling; preferring N=2 stricter.

**Verbatim text** (preserved per MR-11):

> 我反對 R2-D N=3 ceiling。我主張 N=2。
>
> 理由四點：
>
> (1) **連續 3 cycle 的 conditional_pass = ~3 個月實時。** 以本專案 cycle 約 1 月節奏推算，N=3 等於允許某 cell 在 caveat 狀態漂浮 3 個月之久才被強制 resolve。3 個月是 OpenStarry Phase 6 的整個 sprint window，cycle 03-15~17 的全部 Phase 6 七功能 implement 期間。**caveat 漂浮一個 sprint window = 不可接受**。
>
> (2) **N=3 等於提供 2 cycle 的 grace；但實務上「合理複雜跨 cycle 解決」需要 1 cycle grace 即可。** 若 1 cycle 內無法 resolve，往往是 caveat 描述不精確或 dependency 未識別 — 這時應被迫 transition fail 或 fail_fix=<scope>，而非繼續 roll forward。**N=2 = 1 cycle grace = 充分**。
>
> (3) **N=3 設計留下 perpetual-loophole 邊際案例**：若一 cell 每隔 3 cycle reset 為 full_pass 一次（哪怕只有一個極微小的 caveat 暫消），然後又重新進入 conditional_pass，roll_forward_count 重新計數，理論上仍可永久漂浮。N=3 的 buffer 給此類惡意 / 半惡意循環提供空間。**N=2 + 嚴格 reset 規則 (cell 必須真正 full_pass at least 2 cycles before re-entering conditional_pass) = 真正關閉 loophole**。
>
> (4) **R2-D 立論依 cycle 03-23 W2-R30 5 cycle 漂浮事件。** 但該事件之所以發生，正因為當時 N 沒有上限。N=2 vs N=3 的選擇是 — 我們對「合理複雜跨 cycle resolution」的 tolerance window 設多寬。我主張 N=2 = 「1 cycle grace 已給；第 2 cycle 必須斷」。
>
> 我接受 22/1 super-majority 的程序合法性。N=3 ceiling BINDING。但我請求此 dissent 永久保留 per MR-11，並請求 cycle 03-26+ 觀察期內若再有 cell 觸發 N=3 ceiling escalation（即 R2-D 4th tier），重審 N=2 的可行性。

**Operational footprint**: §5.2 Tier 4 codifies N=3 ceiling. Cycle 03-26+ observation: if any cell triggers Tier-4 escalation, LEIBNIZ N=2 proposal re-examined per MR-11 minority sunset clause forward-looking.

---

## §8 Compliance

| Authority | Anchor in this doc |
|-----------|--------------------|
| **MR-7 BINDING** (G-gate exception clause) | §5.1 G-gate context retired sustained 4 cycles; §5.2 Tier-3 backstop |
| **MR-11** (DSS verbatim preservation) | §7 LEIBNIZ DSS-CY25-§13-A verbatim |
| **MR-12** (既有不破壞 strict) | Cycles 03-13~24 audit reports retain legacy verdict prose; no retro-fit. Cycle 03-25 R4 onwards uses Reference/20 format |
| **Tenet #2** (Form-Truth ground) | Verdict = ground projection; cannot float; grounded in §6 evidence + reviewer attestation |
| **Tenet #8** (No-Doctrine-By-Stealth) | Reference/20 published canonical at cycle 03-25 R4 close; format public, machine-parseable, lint-enforceable |
| **F-15 v3** governance docs IN-SCOPE | This doc = governance class; F-15 lint applies; status field at top |

**Cross-refs**:
- `Reference/16_R4_Folder_Convention_Discipline.md` — G4-folder-3 gate anchors verdict format compliance
- `Reference/19_*` (cycle 03-25 D-§12 sister doc; cell_id enumeration)
- `research record/cycle03-25/deliver/O7_A9_audit_verdict_format_final.md` — full-form twin (worked examples + extended commentary)

---

## §9 Worked example (canonical reference)

### §9.1 Single-cell `c037_t4` (FAIL fix=<scope>)

```yaml
cell_id: c037_t4
verdict: fail_fix=plugin-loader-row-2
caveats: []
scope: plugin-loader-row-2
evidence:
  file_paths:
    - openstarry_eco/openstarry_core/src/loader.ts#142
    - openstarry_eco/openstarry_core/src/loader.ts#158
  test_ids:
    - test_loader_row2_persist_cycle03_25
  static_analysis_report_ids:
    - sa_03_25_loader_row2
  runtime_trace_ids: []
reviewer:
  primary: ARCHIMEDES
  secondary: KERNEL
roll_forward_count: 0
cycle: 03-25
emit_source: plugin_loader
notes: Row 2 capability-declaration emit on plugin init violates Tenet #4 invariance.
```

### §9.2 Mixed PASS row aggregate (chapter c042)

```yaml
row_id: c042
row_aggregate: mixed_pass
cells:
  - {cell_id: c042_t1, verdict: full_pass}
  - {cell_id: c042_t2, verdict: full_pass}
  - {cell_id: c042_t3, verdict: conditional_pass, caveats: ["awaits Plan60 Blackboard"]}
  - {cell_id: c042_t4, verdict: full_pass}
  - {cell_id: c042_t5, verdict: not_applicable}
applicable_count: 4
full_pass_count: 3
conditional_pass_count: 1
fail_count: 0
not_applicable_count: 1
```

---

## §10 Forward operational note

Cycle 03-26 R0 orientation inherits this Reference/20 as canonical verdict format. CI hook `tools/audit_verdict_lint.py` (ARCHIMEDES O3 plan-spec implementation slot, ~150-220 Python LOC) deploys at cycle 03-26 R1 close + R2 close + R3 pre-gate. Non-conforming cells fail G4-folder-3 BINDING canonical doc creation gate.

Mixed PASS row class (§4.1) is novel to this codification; expected ~15-25% of total rows in the cycle 03-26 sweep. Tenet #10 column expected ~30-50% `full_pass_pct` until Phase 6 closure (per MR-5 NC PENDING hard cap).

---

**END Reference/20.**
