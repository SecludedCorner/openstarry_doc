---
title: Reference/20 v2 — Audit Verdict Format Codification (Canonical Tenet Axis Re-anchor)
date: 2026-05-11
cycle: 03-26
authors: SCRIBE (chapter lead) + KNUTH + ARCHIMEDES + LINNAEUS
status: BINDING (RATIFIED cycle 03-26 R3 D-§A.2 2026-05-11; was DRAFT R1 candidate; awaiting R3 D-§A.2 BINDING ratify)
authority: cycle 03-26 R1 §B; substance verbatim from Reference/20 v1 §3-§7 (cycle 03-25 R3 D-§13 22/1 super-majority BINDING); §compliance Tenet refs re-anchor canonical per Master directive 2026-05-11 §3
supersedes: openstarry_doc/Reference/20_Audit_Verdict_Format_Codification.md (v1; DRIFT-FLAGGED archived per Master directive 2026-05-11 §3)
cross_refs:
  - claude research/research record/cycle03-25/deliver/O7_A9_audit_verdict_format_final.md (v1 full deliver)
  - openstarry_doc/Reference/19_Audit_Methodology_Codification.md (paired M1; v2 in §A chapter)
  - openstarry_doc/README.md § 十大核心宣言 (canonical Tenet axis authority)
  - openstarry_doc/Reference/16_R4_Folder_Convention_Discipline_v2.md (G4-folder-3 anchor)
mr_zt_refs: [MR-2, MR-7, MR-11, MR-12, Tenet #2 canonical (一切皆插件), Tenet #8 canonical (控制理論閉環模型), F-15 v3]
---

# Reference/20 v2 — Audit Verdict Format Codification

**Status**: **BINDING** (was DRAFT R1 (R1 candidate; cycle 03-26 R3 D-§A.2 BINDING ratify target).
**Authority**: substance verbatim from v1 §3-§7 (cycle 03-25 R3 BINDING); §compliance section Tenet references re-anchor to canonical 10 Tenets per Master directive 2026-05-11.
**Effective**: cycle 03-26 R4 close (post-Master Ratification Batch 23).

---

## §1 Background

(verbatim from v1 §1 with cycle reference updated)

Cycle 03-26 inherits a **610-cell matrix sweep** (43 plugins × 10 canonical Tenets + 18 functions × 10 canonical Tenets; cell = `<chapter_id>_<canonical_tenet_axis>`; per cycle 03-26 R0 §6 + §7 enumeration). Each cell carries an audit verdict; aggregated row + column results feed the cycle 03-26 GP-coherence pre-gate and downstream G-gates.

Pre-cycle-03-25 verdicts were ad-hoc strings. v1 BINDING addressed two operational hazards: (a) machine-parseability blocked at thousands of verdict tokens; (b) verdict semantic ambiguity.

**v2 vs v1 delta**: substance §2 (taxonomy) + §3 (schema) + §4 (aggregation) + §5 (G-gate exception + endless-loop) + §6 (citation discipline) + §7 (DSS-CY25-§13-A LEIBNIZ verbatim) preserved verbatim. §8 compliance section Tenet references re-anchor to canonical 10 Tenets per README.md authority.

**Prerequisite**: cycle 03-26 矩陣 sweep cell verdict 依本格式 binding.

---

## §2 4-tier verdict taxonomy + N/A 5th lane (verbatim from v1 §2)

In descending strength:

1. **`full_pass`** — All canonical Tenet sub-checks GREEN; no caveats; no roll-forward.
2. **`conditional_pass`** — GREEN under explicit named caveats (`caveats:` field non-empty). Allowed **audit-context only** post-MR-7; **forbidden in G-gate context**. N=3 consecutive cycles roll-forward ceiling (per §5.2 Tier 4).
3. **`fail`** — One or more sub-checks RED; full re-audit required next cycle.
4. **`fail_fix=<scope>`** — RED with bounded fix scope (free-form ASCII identifier 1-128 chars). Next-cycle re-audit on `<scope>` only.

**N/A 5th lane**: `not_applicable` — cell × canonical Tenet axis combination does not apply. **Not a verdict**; lane-distinguished from the 4-tier ladder. Accounted in per-row `applicable_count` denominator only.

Strength order: `full_pass > conditional_pass > {fail, fail_fix=<scope>}`. The two FAIL forms are not strictly ordered (`fail_fix=*` operationally softer; semantically identical-or-weaker).

---

## §3 YAML schema (I-13.1..I-13.7) (verbatim from v1 §3)

```yaml
cell_id: <string>                    # I-13.1; per cycle 03-26 R0 enumeration (e.g. p01_t1 = api-runtime × Tenet#1)
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
roll_forward_count: <int>            # I-13.6; ≥0; ≤3 if conditional_pass per §5.2 Tier 4
cycle: <cycle-id>                    # e.g. "03-26"
emit_source: <enum>                  # ENG-FAB F-12 enum
canonical_tenet_axis: <#1..#10>      # NEW v2 field: explicit canonical Tenet number per README.md authority
notes: <string>                      # ≤500 chars
```

**v2 NEW field**: `canonical_tenet_axis` MUST be one of `#1` through `#10` per canonical README.md numbering. Drift-list Tenet references (微核心 / 誠實 / 接續斷裂 / etc.) NOT permitted in cycle 03-26+ verdict cells per Master directive 2026-05-11 §3.

**Lint** (`tools/audit_verdict_lint.py`; CI hook G4-folder-3 gate; cycle 03-26 implementation slot; v2 lint MUST include canonical_tenet_axis enum check):

| ID | Rule |
|----|------|
| L1 | `verdict` token in 5-token canonical enum |
| L2 | `caveats` non-empty iff `verdict == conditional_pass` |
| L3 | `scope` non-empty iff `verdict` matches `^fail_fix=` |
| L4 | `evidence.*` ≥1 channel non-empty unless `not_applicable` |
| L5 | `reviewer.secondary` non-empty if verdict ∈ {conditional_pass, fail, fail_fix=*} (R2-A) |
| L6 | `roll_forward_count` ≤3 if `verdict == conditional_pass` (§5.2 Tier 4) |
| L7 | `cell_id` matches cycle 03-26 R0 enumeration |
| **L8 NEW v2** | `canonical_tenet_axis` ∈ {`#1`..`#10`} per README.md authority; drift-list rejected |

**R2-E SHALL splits per method tier**: each evidence channel SHALL be tagged either `SHALL_AUTOMATIC` (lint-enforced) or `SHALL_REVIEWED` (human-gate via `reviewer.secondary`).

---

## §4 Aggregation algorithm (verbatim from v1 §4)

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

**Mixed PASS** distinguishes "row mostly green with one or two caveat cells" from blanket conditional_pass. Acceptable in audit-context; flagged for cycle-N+1 caveat closure.

### §4.2 Per-column aggregation (per canonical Tenet axis)

```
column.full_pass_pct = full_pass_count / applicable_count_in_column
column.conditional_pass_pct = conditional_pass_count / applicable_count
column.fail_pct = (fail_count + fail_fix_count) / applicable_count
column.not_applicable_count = not_applicable_count
```

Cycle 03-26 R3 GP-coherence pre-gate threshold candidate: each column `full_pass_pct ≥ 90%` (ratification deferred cycle 03-26 R3).

**Canonical Tenet #10 special**: column N/A count expected high (50-70%) per Reference/19 v2 §4.3 LINNAEUS taxonomy review (fractal social structure non-applicable for single-tool/non-multi-agent plugins). Column verdict applies aggregate to `applicable_count`, not raw cell count.

### §4.3 Kahn topological sort with cycle detection (verbatim from v1)

```
Step 1: build adjacency graph G(V=cells, E=cell.depends_on)
Step 2: compute in-degree per cell
Step 3: queue Q ← all in-degree-0 cells, sorted cell_id ASC (determinism)
Step 4: while Q non-empty:
        pop c; emit c
        for neighbour n: decrement n.in_degree; if 0, sorted-insert into Q
Step 5: if |output| < |V|:
        cycle_members ← V \ output       # explicit cell_ids enumeration
        emit error(cycle_members)
        halt aggregation; escalate Master MR-7 (§5 tier-3)
        else: return output
```

**Determinism**: deterministic by `cell_id ASC` tiebreak; two runs over same input MUST produce byte-identical aggregate (cycle 03-26 R3 sanity check).

---

## §5 G-gate exception clause + endless-loop 4-tier hardening (verbatim from v1 §5)

### §5.1 G-gate exception clause anchor (MR-7 BINDING)

- **Audit-context** (W-round audit / retrospective / cycle-mid compliance / cycle 03-26 矩陣 sweep): Conditional PASS allowed.
- **G-gate context** (G0/G1/G2/G3/G4): Conditional PASS NOT allowed; must resolve to `full_pass` / `fail` before G-gate dispatch, OR be excluded from gate scope, OR escalate Master MR-7.

### §5.2 Endless-loop hardening 4 tiers

| Tier | Mechanism | Source |
|------|-----------|--------|
| 1 | 3-iteration soft cap per audit pass (within single audit run) | v1 R1 §13.6 |
| 2 | Dependency closure (caveat-referenced cells must be enumerated and resolved) | v1 R1 §13.6 |
| 3 | Master MR-7 backstop (Kahn cycle detection OR R2-D ceiling exceeded) | v1 R1 §13.6 + R2-D |
| 4 | **Conditional PASS roll-forward N=3 consecutive cycles ceiling (R2-D BINDING)** | v1 R2 §13.4 |

**Tier 4 (R2-D)**: a cell may roll `conditional_pass` forward at most N=3 consecutive cycles. On the 4th cycle, MUST transition to `full_pass` (caveats resolved) or `fail` / `fail_fix=<scope>` (caveats unresolved); otherwise escalate Master MR-7. `roll_forward_count` field tracked in YAML; reset to 0 at full_pass / fail transition.

**LEIBNIZ DSS-CY25-§13-A**: N=2 stricter proposal preserved per MR-11; sunset clause forward-looking (cycle 03-26+ observation; if any Tier-4 escalation occurs, N=2 re-examined). Sustained verbatim per cycle 03-26 R3 deliberation.

---

## §6 Per-cell evidence citation discipline (verbatim from v1 §6)

Per I-13.7 + R2-B lint enforcement, **four citation channels** (no free-form):

1. `file_paths`: `<repo-relative-path>#<line>` or `<repo-relative-path>#<line>-<line>`
2. `test_ids`: test function name or file+name
3. `static_analysis_report_ids`: ENG-FAB F-15 / F-12 lint report IDs
4. `runtime_trace_ids`: W-round runtime traces / audit_calc state-(a) traces

**Empty-evidence rule**: cell with `verdict != not_applicable` AND all 4 channels empty fails L4 lint.

**Reviewer attribution (R2-A)**: For `conditional_pass`, `fail`, `fail_fix=<scope>`, `reviewer.secondary` MUST be non-original-author. For `full_pass` and `not_applicable`, secondary recommended but not required.

**R2-E method tier split**: each evidence channel SHALL be `SHALL_AUTOMATIC` (lint validates: file_paths exist, test_ids resolve) or `SHALL_REVIEWED` (reviewer.secondary attests). The two tiers may co-exist on one cell.

**Text-only L4 caveat (cycle 03-26 specific)**: when `runtime_trace_ids` evidence comes from text-only mode (provider-claude-cli sixth sustained), cell MUST emit `conditional_pass` with `caveats: ["text-only-mode L4 evidence; real-LLM Hybrid B+A defer cycle 04+"]` per Master directive 2026-05-08-b deferral semantics. Behavioral Tenets #6/#8 most affected.

---

## §7 DSS-CY25-§13-A — LEIBNIZ verbatim (per MR-11; preserved sustained) (verbatim from v1 §7)

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

**Operational footprint (cycle 03-26 R1 observation)**: cycle 03-26 = first cycle where matrix sweep verdicts populate; LEIBNIZ N=2 sunset clause observation window opens. Tier-4 escalation count tracked at C-S4-derivative new counter (proposed for Reference/21 v0.2 amendment if Tier-4 fires).

---

## §8 Compliance (v2 — re-anchored Tenet references to canonical 10 Tenets per README.md)

| Authority | Anchor in this doc |
|-----------|--------------------|
| **MR-7 BINDING** (G-gate exception clause) | §5.1 G-gate context retired sustained; §5.2 Tier-3 backstop |
| **MR-11** (DSS verbatim preservation) | §7 LEIBNIZ DSS-CY25-§13-A verbatim sustained |
| **MR-12** (既有不破壞 strict) | Cycles 03-13~25 audit reports retain v1 verdict prose; no retro-fit. Cycle 03-26 R4 onwards uses v2 format |
| **Canonical Tenet #2 一切皆插件 (per README.md §10-Tenets-2)** | Verdict cell granular plugin-level audit; per-plugin verdict against each canonical Tenet axis |
| **Canonical Tenet #7 微內核與絕對純淨 (per README.md §10-Tenets-7)** | Verdict format itself is governance doc, not core code — preserves Core 0-policy boundary |
| **Canonical Tenet #8 控制理論閉環模型 (per README.md §10-Tenets-8)** | Verdict aggregation feedback loop (per-row → per-column → overall audit verdict) embodies control-loop semantics |
| **F-15 v3** governance docs IN-SCOPE | This doc = governance class; F-15 lint applies; status field at top |
| **Master directive 2026-05-11 §3 drift-list flagging** | v2 explicitly re-anchors §8 Tenet references to canonical README.md authority; v1 drift-list (Tenet #2 = 誠實 etc.) NOT used in v2 |
| **Master directive 2026-05-08-b** W2-R36 provider scope deferral | §6 text-only L4 caveat discipline; cycle 03-26 cells emit `conditional_pass` with text-only-mode caveat for behavioral Tenets |

**Cross-refs**:
- `openstarry_doc/Reference/16_R4_Folder_Convention_Discipline_v2.md` — G4-folder-3 gate anchors verdict format compliance (v2 includes canonical_tenet_axis field check)
- `openstarry_doc/Reference/19_Audit_Methodology_Codification.md` (v2 in §A chapter) — paired methodology Reference; cell_id enumeration
- `openstarry_doc/Reference/21_Counter_Registry.md` — Counter ID discipline applies to verdict aggregate counters

---

## §9 Worked examples (cycle 03-26 canonical — updated v2 with `canonical_tenet_axis` field)

### §9.1 Single-cell `p24_t7` (full_pass; provider-claude-cli × Tenet#7 微內核)

```yaml
cell_id: p24_t7
verdict: full_pass
caveats: []
evidence:
  file_paths:
    - openstarry_eco/agent_dev/openstarry_plugin/provider-claude-cli/src/index.ts#1-50
    - openstarry_eco/agent_dev/openstarry_plugin/provider-claude-cli/manifest.json#1
  test_ids:
    - test_provider_claude_cli_purity_cycle03_26
  static_analysis_report_ids:
    - sa_03_26_microkernel_purity_p24
  runtime_trace_ids: []
reviewer:
  primary: GUARDIAN
  secondary: TANENBAUM
roll_forward_count: 0
cycle: 03-26
emit_source: provider_plugin
canonical_tenet_axis: "#7"
notes: provider-claude-cli plugin observes microkernel purity discipline; 0 Core dependency; manifest declares external plugin status; pnpm test:purity 0 violation.
```

### §9.2 Single-cell `p10_t10` (not_applicable; distributed-alaya × Tenet#10 fractal social structure)

```yaml
cell_id: p10_t10
verdict: not_applicable
caveats: []
evidence: {file_paths: [], test_ids: [], static_analysis_report_ids: [], runtime_trace_ids: []}
reviewer:
  primary: LINNAEUS
roll_forward_count: 0
cycle: 03-26
emit_source: capability_phase_6
canonical_tenet_axis: "#10"
notes: distributed-alaya is single-agent context (alaya seed-store per-agent process). Tenet #10 分形社會結構 (Fractal Social Structure; agent-of-agents pattern) non-applicable. LINNAEUS taxonomy profile substantiates: alaya layer maps to 阿賴耶識 within Tenet #6 八識俱轉 architecture, not fractal social structure aggregation. Per Reference/19 v2 §4.3 N/A discipline.
```

### §9.3 Single-cell `p24_t6` (conditional_pass; provider-claude-cli × Tenet#6 八識俱轉; text-only-mode caveat)

```yaml
cell_id: p24_t6
verdict: conditional_pass
caveats: ["text-only-mode L4 evidence; real-LLM Hybrid B+A defer cycle 04+ per Master directive 2026-05-08-b"]
evidence:
  file_paths:
    - openstarry_eco/agent_dev/openstarry_plugin/provider-claude-cli/src/runtime.ts#80-145
  test_ids:
    - test_provider_claude_cli_concurrent_invocation
  static_analysis_report_ids: []
  runtime_trace_ids:
    - trace_w2_r36_text_only_sustained_03_26
reviewer:
  primary: HERACLITUS
  secondary: ASANGA
roll_forward_count: 1
cycle: 03-26
emit_source: provider_plugin
canonical_tenet_axis: "#6"
notes: Concurrent consciousness stream observable in text-only mode trace; real-LLM verification deferred cycle 04+. Tier-4 ceiling roll_forward_count = 1 (first cycle conditional with caveat); resolution target cycle 04+ Hybrid B+A application.
```

### §9.4 Mixed PASS row aggregate (api-runtime row)

```yaml
row_id: p01_api_runtime
row_aggregate: mixed_pass
cells:
  - {cell_id: p01_t1, verdict: full_pass}      # Agent as OS Process
  - {cell_id: p01_t2, verdict: full_pass}      # Everything is a Plugin
  - {cell_id: p01_t3, verdict: full_pass}      # Five Aggregates
  - {cell_id: p01_t4, verdict: full_pass}      # Directory as Protocol
  - {cell_id: p01_t5, verdict: full_pass}      # Directory as Permission
  - {cell_id: p01_t6, verdict: conditional_pass, caveats: ["text-only-mode L4"]}  # Concurrent Consciousness
  - {cell_id: p01_t7, verdict: full_pass}      # Microkernel & Purity
  - {cell_id: p01_t8, verdict: conditional_pass, caveats: ["text-only-mode L4"]}  # Control-Theoretic Loop
  - {cell_id: p01_t9, verdict: full_pass}      # Pluggable Memory
  - {cell_id: p01_t10, verdict: not_applicable}  # Fractal Social Structure (single-runtime plugin)
applicable_count: 9
full_pass_count: 7
conditional_pass_count: 2
fail_count: 0
not_applicable_count: 1
```

---

## §10 Forward operational note

Cycle 03-26 R1 chapters §B.1~§B.10 inherit this Reference/20 v2 as canonical verdict format. CI hook `tools/audit_verdict_lint.py` v2 (extend L8 NEW field check) deploys at cycle 03-26 R2 close + R3 pre-gate. Non-conforming cells fail G4-folder-3 BINDING canonical doc creation gate.

Mixed PASS row class expected ~20-35% of total rows in cycle 03-26 sweep (text-only-mode caveat on behavioral Tenet cells across multiple plugin/function rows).

Tenet #10 column expected ~50-70% `not_applicable` per LINNAEUS taxonomy review (fractal social structure non-applicable for single-tool plugins).

---

## §11 Findings (R1 chapter §B)

| ID | Finding | Severity |
|----|---------|----------|
| F-CY26-§B-R1-01 | NEW field `canonical_tenet_axis` (`#1`..`#10`) introduced in v2 schema (§3) | INFO (schema addition) |
| F-CY26-§B-R1-02 | NEW lint rule L8 enforces canonical Tenet enum — drift-list rejected at G4-folder-3 gate | INFO |
| F-CY26-§B-R1-03 | §6 text-only L4 caveat discipline NEW codified (cycle 03-26-specific carry-forward; vending semantics for cycle 04+) | MED |
| F-CY26-§B-R1-04 | §8 compliance section Tenet refs re-anchored canonical (Tenet #2/#7/#8 instead of v1 drift list Tenet #2 誠實 / #8 證據>意見) | INFO |
| F-CY26-§B-R1-05 | §9 worked examples updated to use canonical Tenet refs + text-only-mode caveat pattern | INFO |
| F-CY26-§B-R1-06 | Mixed PASS row class expected ~20-35% prevalence (behavioral Tenet text-only caveat) | MED (cycle 04+ caveat closure scope candidate) |
| F-CY26-§B-R1-07 | Tenet #10 column expected ~50-70% N/A — LINNAEUS taxonomy review pre-gate required | MED |
| F-CY26-§B-R1-08 | DSS-CY25-§13-A LEIBNIZ N=2 sustained; cycle 03-26 first observation window | INFO |
| F-CY26-§B-R1-09 | `tools/audit_verdict_lint.py` v2 ARCHIMEDES implementation slot (extend L8 + canonical_tenet_axis check; ~150-220 Python LOC; cycle 03-26 R2 close target) | LOW |

**Total: 9 findings** (target 8-15; within band).

---

*Reference/20 v2 — Audit Verdict Format Codification (DRAFT R1; cycle 03-26)*
*Substance verbatim from v1 §2/§3/§4/§5/§6/§7; §8 compliance Tenet refs re-anchored canonical per Master directive 2026-05-11*
*NEW v2 field: `canonical_tenet_axis` (#1..#10) + NEW v2 lint L8; drift-list rejected at G4-folder-3 gate*
*5-token verdict enum + YAML schema I-13.1..I-13.8 + N=3 ceiling Tier-4 (DSS-CY25-§13-A LEIBNIZ N=2 sustained per MR-11)*
*Text-only L4 caveat discipline codified per Master directive 2026-05-08-b deferral semantics*
*Cycle 03-26 R3 D-§A.2 BINDING ratify target*
