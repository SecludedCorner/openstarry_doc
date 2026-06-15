---
title: Reference/20 v3 — Audit Verdict Format Codification (Verdict-Shape Canonical Pattern + Strict Reading Codification)
date: 2026-05-13
cycle: 03-31
authors: SCRIBE (chapter lead) + KNUTH (algorithm precision; L2 / L6 / L9 / L10 lint formalism) + ARCHIMEDES (lint impl + tools/audit_verdict_lint.py v3 spec) + LINNAEUS (taxonomy hygiene; verdict-shape canonical pattern)
status: BINDING (RATIFIED cycle 03-31 R3 §3 D-§A4 5 sub-motions BINDING + ρ-1 cell-individual roll_forward strict reading per R3 §1.1 D-§roll_forward; §12.1 first-cycle CP wording mechanically corrected cycle 03-32 R3 §1 Option A ratification per ρ-1 substance precedence; pending Master Ratification Batch 26+27 forward dispatch)
authority:
  - cycle 03-25 R3 D-§13 (Reference/20 v1 BINDING; 22/1 super-majority; substance §2-§7 preserved verbatim through v2/v3)
  - cycle 03-26 R3 D-§A.2 (Reference/20 v2 BINDING; UNANIMOUS canonical Tenet re-anchor; §8 + §3 L8 + §9)
  - cycle 03-30 R3 §1.1 D-§A1.5-P2 Option B RELOCATE canonical pattern (UNANIMOUS 23/0; v3 §11 codify)
  - cycle 03-30 R3 §4.1 D-§tier3-caveat canonical pattern (UNANIMOUS 23/0; v3 §11 codify)
  - cycle 03-30 R3 §4.2 D-§roll_forward strict reading codification (UNANIMOUS 23/0; v3 §12 codify)
  - cycle 03-31 R3 §3 D-§A4 (5 sub-motions: #6 verdict-shape + #10 tier-3 caveat placement + #11 roll_forward strict reading + Reference/20 v3 BINDING ratify + audit_verdict_lint.py v3 L9 + L10 impl slot)
  - cycle 03-31 R3 §1.1 D-§roll_forward ρ-1 cell-individual strict reading (BINDING; v3 §12 codify)
  - cycle 03-32 R3 §1 Option A (R-AMEND-3 §12.1 first-cycle CP wording correction per ρ-1 substance precedence; mechanical drift correction per MR-12 forward-only)
  - Master Ratification Batch 26 forward dispatch (cycle 03-31 R4 close target)
  - Master Ratification Batch 27 forward dispatch (cycle 03-32 R3 §1 R-AMEND-3 routing)
supersedes: openstarry_doc/Reference/20_Audit_Verdict_Format_Codification_v2.md (substance §1-§10 preserved verbatim per MR-12; §11 + §12 NEW additive only)
cross_refs:
  - openstarry_doc/Reference/20_Audit_Verdict_Format_Codification_v2.md (full v2 canonical preserved verbatim in §3.1-§3.10)
  - openstarry_doc/Reference/19_Audit_Methodology_Codification_v2.md (paired Reference; cell_id enumeration + L1-L4 methodology authority)
  - openstarry_doc/Reference/16_R4_Folder_Convention_Discipline_v3.md (G4-folder-3 anchor)
  - openstarry_doc/Reference/21_Counter_Registry.md (C-LEIBNIZ-N2-observation §9 cross-ref; cycle-history index for L10)
  - openstarry_doc/Reference/22_Agent_Behavioral_Guidelines.md (BG-1 / BG-4 / BG-5 anchors)
  - claude research/research record/cycle03-30/R3/R3_decision_log.md §1.1 + §4.1 + §4.2 (3 amendment-trigger D-items UNANIMOUS 23/0 each)
  - claude research/research record/cycle03-30/R1/§A1-T6-§24-C1_provider-claude-cli/verdict.yaml (A1-1 canonical FP idiom reference exemplar; cited §11.1)
  - claude research/research record/cycle03-31/R3/R3_decision_log.md §1.1 + §3 + §4 (cycle 03-31 R3 BINDING ratifications)
mr_zt_refs: [MR-2, MR-7, MR-11, MR-12, ZT-1, ZT-2, ZT-3, Tenet#2 canonical, Tenet#7 canonical, Tenet#8 canonical, F-15 v3]
---

# Reference/20 v3 — Audit Verdict Format Codification

<!--
============================================================================
CYCLE 03-33 R4 SCOPE MARKS (top-of-file annotations; no §-body edits)
============================================================================

[CY33-NOTE-1] R-AMEND-3 §12.1 DEPLOYED (cycle 03-33)
  Reference/20 v3 §12.1 first-cycle CP wording correction (cycle 03-32 R3 §1
  Option A ratified per ρ-1 substance precedence) is DEPLOYED in canonical
  via Dev v0.57.9-alpha FIX-CY33-02 (cycle 03-33). §12.1 now reads
  "roll_forward_count := 0 regardless of verdict class" on first-cycle (no
  prior verdict.yaml at same cell_id), aligned with ρ-1 cell-individual
  semantic. tools/audit_verdict_lint.py v3 L10 implementation sync DEPLOYED
  via FIX-CY33-03. Master Ratification Batch 27 R-AMEND routing closed;
  cycle 03-33 R3 §1.1 Option A canonical cascade COMPLETE.

[CY33-NOTE-2] §6 amendment progression scope mark — N=3 empirical validation
  Cross-Tenet anchor pattern (provider FIRST cycle 03-31 + alaya SECOND
  cycle 03-32 + api-runtime THIRD cycle 03-33) graduates from N=2 ratified
  (cycle 03-32) to N=3 empirical validation (cycle 03-33). 3 distinct
  doctrinal substrates incl. NEW 識蘊 Vijnana structural-observability class
  (api-runtime; first non-consciousness-stream substrate). Pattern
  reproducibility 3/3 anchor plugins. Master Ratification Batch 28
  amendment ratification candidate (cycle 03-33 R4 close target). §6 body
  text NOT modified this cycle — substance amendment scope marked for v4
  forward authorship (per CY33-NOTE-4 below).

[CY33-NOTE-3] §11.5 amendment scope mark — plugin-cell vs conceptual-cell
  L2 gap material discipline disambiguation (per cycle 03-33 R3 §5.3
  D-§conceptual-cell-pattern). Capability-phase-6-coordinator established
  the conceptual-cell canonical precedent (R0-coined; non-plugin instance
  evaluated against Tenet#8). §11.5 body NOT modified this cycle —
  disambiguation amendment scope marked for v4 forward authorship.

[CY33-NOTE-4] Reference/20 v4 forward amendment scope marks
  Two NEW canonical patterns codified cycle 03-33 R3 require v4 authorship:
    - SIBLING substrate-invariance pair pattern (per R3 §5.2): replay-cache
      + alaya complementary consciousness-stream substrate; cross-Tenet
      anchor canonical pattern N=3 sub-extension.
    - Provider-class split-verdict (SV-β; per R3 §5.4): native API path
      PROMOTED to COMPLIANT (Option 2 messages[] system role) while CLI
      design boundary preserved as e2e 0/5 (not a regression; documented
      design boundary). Provider-class split-verdict canonical pattern
      v4 forward.
  Forward v4 authorship cycle 03-34+ per Master Ratification Batch 28
  amendment dispatch routing.

============================================================================
-->

## §1 Background — preserved verbatim from v2 §1

(Substance from v2 §1 preserved verbatim per MR-12 forward-only. v2 was BINDING per cycle 03-26 R3 D-§A.2; v3 extends additively with NEW §11 + §12 only. Original purpose: provide a canonical machine-parseable verdict shape for all cells produced under the 矩陣 sweep + Phase 7 R-input verification regime; resolve heterogeneous verdict prose into a uniform 4-tier + N/A taxonomy with structured caveats / scope / evidence / roll-forward fields.)

---

## §2 4-tier verdict taxonomy + N/A 5th lane — preserved verbatim from v2 §2

(Substance from v2 §2 preserved verbatim. Canonical 5-token verdict enum: `full_pass` / `conditional_pass` / `fail` / `fail_fix=<scope>` / `not_applicable`. **Verdict-shape discipline that emerged from §2 wording is now explicitly codified in NEW §11 below** — §2 wording "full_pass — no caveats; no roll-forward" is the textual basis for §11.1 FP rule.)

---

## §3 YAML schema (I-13.1..I-13.8) — preserved verbatim with lint table extension

(Substance from v2 §3 schema fields I-13.1..I-13.8 preserved verbatim: `cell_id`, `verdict`, `caveats`, `scope`, `evidence.*`, `reviewer.primary` / `reviewer.secondary`, `roll_forward_count`, `cycle`, `emit_source`, `canonical_tenet_axis`, `notes`.)

**Lint table extension** (additive only; L1-L8 from v2 unchanged; **L9 + L10 NEW in v3**):

| ID | Rule | Status |
|----|------|--------|
| L1 | `verdict` token in 5-token canonical enum | v1 sustained |
| L2 | `caveats` non-empty iff `verdict == conditional_pass` (biconditional) | v1 sustained; **clarified by v3 §11.6** |
| L3 | `scope` non-empty iff `verdict` matches `^fail_fix=` | v1 sustained |
| L4 | `evidence.*` ≥1 channel non-empty unless `not_applicable` | v1 sustained |
| L5 | `reviewer.secondary` non-empty if verdict ∈ {`conditional_pass`, `fail`, `fail_fix=*`} (R2-A) | v1 sustained |
| L6 | `roll_forward_count` ≤3 if `verdict == conditional_pass` (§5.2 Tier 4 N=3 BINDING) | v1 sustained |
| L7 | `cell_id` matches cycle R0 enumeration | v2 sustained |
| L8 | `canonical_tenet_axis` ∈ {`#1`..`#10`} per `README.md §十大核心宣言` authority; drift-list rejected | v2 sustained |
| **L9 NEW v3** | **Caveat-placement check**: for `verdict == full_pass`, `notes:` block MAY contain tier-3 / tier-4 forward-binding scope-expansion language; `caveats[]` MUST be empty. For `verdict == conditional_pass`, `caveats[]` MUST contain ≥1 material evidence-gap entry (NOT solely forward-binding doc-clarification). See NEW §11. | **NEW** |
| **L10 NEW v3** | **roll_forward_count strict reading**: reset to 0 ONLY when verdict transitions to `full_pass` / `fail` / `fail_fix=*` / `not_applicable` (vs prior cycle at same cell_id). Caveat-class-change while sustaining `conditional_pass` does NOT reset; increment by 1. Cell-individual reading per cycle 03-31 R3 §1.1 D-§roll_forward ρ-1 BINDING. See NEW §12. Lint validates against prior-cycle `verdict.yaml` at same `cell_id` via Reference/21 cycle-history index. | **NEW** |

---

## §4 Aggregation algorithm — preserved verbatim from v2 §4

(Substance from v2 §4.1 / §4.2 / §4.3 preserved verbatim. Mixed-PASS aggregation rules + Kahn topological sort + R2-D Conditional PASS roll-forward N=3 ceiling sustained.)

---

## §5 G-gate exception clause + endless-loop 4-tier hardening — preserved verbatim from v2 §5

(Substance from v2 §5.1 + §5.2 + Tier 1-4 table preserved verbatim. **§5.2 Tier 4 verbatim text "reset to 0 at full_pass / fail transition"** is the textual basis for NEW §12 codification — v3 §12 is the explicit codified strict reading of this §5.2 Tier 4 line.)

---

## §6 Per-cell evidence citation discipline — preserved verbatim from v2 §6

(Substance from v2 §6 preserved verbatim. Text-only L4 caveat discipline carried forward per Master directive 2026-05-08-b W2-R36 deferral; NEW §11 + §12 augment cross-cell placement discipline.)

---

## §7 DSS-CY25-§13-A LEIBNIZ verbatim — preserved verbatim from v2 §7

(Substance from v2 §7 preserved verbatim per MR-11 UNCHANGED. **DSS-CY25-§13-A LEIBNIZ** verbatim preserved. **LEIBNIZ N=2 sunset clause forward-looking observation continues** per cycle 03-30 R3 §1.2 D-§A1.4-P2 Option γ Hybrid 21/2 super-majority + **DSS-CY30-§A1.4-P2-α LEIBNIZ + KNUTH** dissent verbatim sustained per MR-11. Cycle 03-31 R3 observation window CLOSED per p10_t6 + p13_t6 cycle 03-30 → cycle 03-31 Option α activation transitions FP — see §12.4 cross-reference.)

---

## §8 Compliance — preserved verbatim with v3 addendum

(Substance from v2 §8 compliance table preserved verbatim. Addendum row added for NEW §11 + §12 anchors below.)

| Authority | Anchor in this doc |
|-----------|--------------------|
| **MR-7 BINDING** (G-gate exception) | §5.1 + §5.2 Tier-3 backstop |
| **MR-11** (DSS verbatim) | §7 LEIBNIZ DSS-CY25-§13-A + addendum: cycle 03-30 R3 §6.1 DSS-CY30-§A1.4-P2-α LEIBNIZ + KNUTH verbatim sustained |
| **MR-12** (既有不破壞 strict) | v1 / v2 cells retain original verdict prose; v3 §11 + §12 forward-only codification; v2 §1-§10 unchanged |
| **Canonical Tenet #2 一切皆插件** | Verdict cell granular plugin-level audit (per README.md §十大核心宣言 #2) |
| **Canonical Tenet #7 微內核與絕對純淨** | Verdict format = governance doc (per README.md §十大核心宣言 #7) |
| **Canonical Tenet #8 控制理論閉環模型** | Verdict aggregation feedback loop (per README.md §十大核心宣言 #8); cycle 03-31 first-subset 2/38 closed inaugural |
| **F-15 v3** governance docs IN-SCOPE | This doc = governance class; F-15 lint applies |
| **Master directive 2026-05-11 §3** drift-list flagging | §8 canonical re-anchor sustained |
| **Master directive 2026-05-08-b** W2-R36 deferral | §6 text-only L4 caveat discipline |
| **NEW v3 — cycle 03-31 R3 §3 D-§A4 + §1.1 D-§roll_forward** | §11 + §12 canonical-pattern codification |

---

## §9 Worked examples — preserved verbatim with v3 addendum cross-ref

(Substance from v2 §9.1-§9.4 preserved verbatim. NEW §11 + §12 cross-reference these examples; **A1-1 cycle 03-30 verdict.yaml is canonical real-LLM FP exemplar referenced from NEW §11.1**.)

---

## §10 Forward operational note — preserved verbatim

(Substance from v2 §10 preserved verbatim. Cycle 03-31 + onwards matrix sweep cells + Phase 7 R-input cells inherit v3 lint-at-close BINDING.)

---

## §11 NEW — Verdict-Shape Canonical Pattern (Items #6 + #10 consolidate)

**Authority**: cycle 03-30 R3 §1.1 D-§A1.5-P2 Option B RELOCATE UNANIMOUS 23/0 (Item #6) + cycle 03-30 R3 §4.1 D-§tier3-caveat canonical pattern UNANIMOUS 23/0 (Item #10) + cycle 03-31 R3 §3 D-§A4 BINDING consolidate. Both ratified under current Reference/20 v2 §2 + §3 L2 biconditional BINDING (no §2 / §3 substance amendment required; §11 is the cross-cell codification of how that biconditional is operationalised in practice).

### §11.1 Canonical pattern (BINDING from cycle 03-31 R4 close)

**FP cell** (`verdict: full_pass`):
- `caveats: []` empty MANDATORY (per v2 §2 wording "full_pass — no caveats; no roll-forward" + §3 L2 biconditional).
- Forward-binding doc-clarification language (e.g. "tier-3 evidence ladder", "tier-4 full-daemon deferred cycle 03-31+", "scope clarification F6 = 識蘊 per Plan59 §1") MUST be placed in `notes:` block.
- Forward-binding **scope expansion** (e.g. multi-mode evidence ladder; tier-3 vs tier-4 ladder annotation; Plan-cross-ref; Plan-numbering disambiguation; maxTurns>1 observability scope reduction annotation) is **NOT** an evidence-gap; it documents future expansion paths or doctrinal precision, not present-cycle audit deficits.

**CP cell** (`verdict: conditional_pass`):
- `caveats[]` MUST be non-empty (per v2 §3 L2 biconditional).
- Each entry MUST represent **material evidence-gap** sustained across the audit (e.g. multi-host harness gap, L2 plugin-local test gap, real-LLM evidence absent for behavioural axes).
- Forward-binding doc-clarification SHALL NOT be placed in `caveats[]`; place it in `notes:` block alongside the material evidence-gap entries.

**A1-1 cycle 03-30 verdict.yaml = canonical reference exemplar** (cite path: `claude research/research record/cycle03-30/R1/§A1-T6-§24-C1_provider-claude-cli/verdict.yaml`):
- `verdict: full_pass`
- `caveats: []` (empty per §11.1 FP rule)
- `notes: |` block contains: Phase 2 live-trace summary + 5-point isolation v2 STRENGTHENED attestation + **tier-3 evidence ladder annotation** (inline-contract real-LLM) + **tier-4 deferred-scope annotation** (full-daemon real-LLM deferred cycle 03-31+) + LEIBNIZ sunset clause observation + DSS-CY26-§A.3-3 cross-Tenet annotation + ε-surface Δ=0 sustained.
- A1-1 is the **cleanest FP idiom**: zero caveats[], rich notes[] forward-binding scope language.

### §11.2 Tier-3 vs Tier-4 vs material evidence-gap (decision tree)

To decide whether a piece of audit text belongs in `caveats[]` (FP-blocking) or `notes:` (forward-binding scope), apply:

```
IS the text describing a present-cycle deficit that, if absent, would have made the FP determination wrong?
  YES → caveats[] (FP-blocking → cell verdict MUST be conditional_pass)
  NO  → notes: block (forward-binding scope expansion OR doc-clarification)

Sub-decision for evidence-ladder annotation:
  Text describes tier-3 (inline-contract real-LLM) used in lieu of tier-4 (full-daemon real-LLM)?
    AND audit Tenet-axis is behavioural (Tenet#6 / Tenet#8)?
      AND tier-3 evidence is sufficient per §5.1 MR-7 audit-context precedent (cycle 03-26 R3 D-§A.2)?
        → notes: (forward-binding scope expansion; tier-4 fortifies but does not gate FP)
      ELSE → caveats[] (tier-4 IS the gating evidence; CP cell)

Sub-decision for multi-host gap (cross-Tenet adjacent):
  Text describes single-host evidence where multi-host harness is the methodological target?
    → caveats[] (material evidence-gap → CP cell; see A1-2 cycle 03-30 idiom; resolved cycle 03-31 Option α activation)

Sub-decision for L2 plugin-local test gap:
  Text describes absent test for a fix scope (e.g. FIX-A test suite scope expansion)?
    → caveats[] (material evidence-gap → CP cell; see A1-4 cycle 03-30 idiom; resolved cycle 03-31 Option α activation)
```

### §11.3 Tier-3 inline-contract caveat for behavioural axes — FP-permissible per §5.1 MR-7

The cycle 03-26 R3 D-§A.2 MR-7 audit-context precedent established that **tier-3 (inline-contract real-LLM) evidence is FP-permissible** for behavioural Tenet axes (Tenet#6 / Tenet#8) when:
- Architectural A-view PASSES UNCONDITIONAL, AND
- Behavioural B-view evidence is captured via inline-contract real-LLM (harness mirrors plugin spawn pattern verbatim from src), AND
- Tier-4 full-daemon real-LLM is deferred to a forward cycle as scope expansion (NOT remediation).

**§11.3 codifies**: under this condition, the tier-3 ladder annotation belongs in `notes:` (forward-binding scope; tier-4 fortifies later but does not gate present-cycle FP), **NOT** in `caveats[]`. A1-1 cycle 03-30 is the canonical instance. Cycle 03-31 Tenet#8 first-subset 2 FP cells (Stream 1 cell-1 tier-4 PASS + Stream 1 cell-1 inaugural tier-4 cycle) inherit this discipline.

### §11.4 Tier-4 full-daemon caveat — same category as Tier-3

If a future cycle attempts tier-4 (full-daemon real-LLM via `pnpm install` + `apps/runner/bin.js daemon start`) and succeeds, the tier-4 evidence supplements the existing tier-3 record. If tier-4 partially completes but does not fully verify, the **partial-tier-4 annotation also goes in `notes:`** (same category as tier-3 ladder annotation: forward-binding scope expansion, NOT evidence-gap). **Cycle 03-31 inaugural tier-4 cycle** (Stream 1 cell-1 PASS) — the inline-replication caveat REMOVED from active CP roster; tier-4 evidence-tier ladder canonical per Reference/21 §3.9.

### §11.5 Material evidence-gap → CP cell — examples

The following are **material evidence-gaps** (place in `caveats[]`; verdict = `conditional_pass`):

1. **Multi-host harness gap** (A1-2 cycle 03-30 distributed-alaya × Tenet#6): single-host evidence captured; multi-host harness is the methodological target for distributed-alaya substrate verification; gap sustained cycle 03-30 → resolved cycle 03-31 Option α activation transitions cell to FP per LEIBNIZ N=2 observation window CLOSED.
2. **L2 plugin-local test gap** (A1-4 cycle 03-30 guide-character-init × Tenet#6): FIX-A test suite addition scope sized ~40-80 LOC; sustained cycle 03-30 → resolved cycle 03-31 Option α activation transitions cell to FP per LEIBNIZ N=2 observation window CLOSED.
3. **Real-LLM evidence absent for behavioural axis** (cycle 03-26 worked example p24_t6 text-only baseline): text-only L4 evidence captured per W2-R36 text-only sustained; real-LLM Hybrid B+A deferred → CP cell (resolved cycle 03-30 A1-1 via tier-3 inline-contract real-LLM upgrade per cycle 03-25 R3 D-§3 Hybrid B+A activation; cell transitioned CP → FP).

### §11.6 L2 lint biconditional (BINDING from cycle 03-31 R4 close)

```
L2 lint rule (extended in v3 §3 lint table):
  caveats[] non-empty IFF verdict == conditional_pass
```

**§11.6 codifies**: cycle 03-31+ R1 subagent prompts MUST include an explicit **lint-at-close step** that:
- Validates `caveats[]` non-empty IFF verdict == `conditional_pass` for every `verdict.yaml` in the cell artifact folder.
- Validates forward-binding doc-clarification text is in `notes:` block (NOT in `caveats[]`).
- Validates material evidence-gap entries are in `caveats[]` (NOT in `notes:`).
- Failure mode → subagent self-correct before close; if escalation needed, route to R2 reviewer.

**Cycle 03-31 R0 §13.1 G0 Folder Spec Compliance Check** explicitly anchors this lint-at-close discipline (per R0 subagent_prompt_skeleton.md cycle 03-31 §0 Field 9 Report format). Cycle 03-31+ R1 subagent prompts MUST include this lint-at-close step.

### §11.7 Cross-cell A1-1 / A1-3 / A1-5 cycle 03-30 normalize attestation

Per cycle 03-30 R3 §1.1 Option B RELOCATE UNANIMOUS 23/0:
- **A1-1 p24_t6** (provider-claude-cli × Tenet#6): already canonical FP idiom; no relocation needed.
- **A1-3 p40_t6** (vedana-sensor-core × Tenet#6): RELOCATE 1 tier-3 caveat → notes; FP substance preserved; canonical FP idiom post-relocate.
- **A1-5 F6_t6** (api-runtime F6 × Tenet#6): RELOCATE 3 caveats (tier-3 + scope-clarification + Plan59 §6.3 3-field) → notes; FP substance preserved; canonical FP idiom post-relocate.

R4 close cycle 03-30 executed the actual `verdict.yaml` file relocation per R3 §1.1 forward-to-R4 directive. Cycle 03-31 R3 §3 D-§A4 ratifies the canonical pattern v3 codification.

---

## §12 NEW — roll_forward_count Strict Reading Codification (Item #11 + ρ-1 cell-individual)

**Authority**: cycle 03-30 R3 §4.2 D-§roll_forward UNANIMOUS 23/0 (Item #11) + cycle 03-31 R3 §1.1 D-§roll_forward ρ-1 cell-individual reading BINDING UNANIMOUS. Codifies strict reading of Reference/20 v2 §5.2 Tier 4 verbatim text "reset to 0 at full_pass / fail transition" with explicit cell-individual semantic.

### §12.1 Strict reading rule (BINDING from cycle 03-31 R4 close; first-cycle CP wording corrected per cycle 03-32 R3 §1 Option A ratification)

```
roll_forward_count strict reading (per-cell ρ-1 cell-individual; cycle 03-31 R3 §1.1 D-§roll_forward BINDING + cycle 03-32 R3 §1 Option A first-cycle CP correction):

  reset_trigger = verdict transitions to full_pass OR fail OR fail_fix=* OR not_applicable
  increment_trigger = verdict remains conditional_pass (regardless of caveat-class change)

  On reset_trigger:
    roll_forward_count := 0
  On increment_trigger:
    roll_forward_count := prior_cycle.roll_forward_count + 1
  On first cycle (no prior verdict.yaml at same cell_id):
    roll_forward_count := 0  (regardless of verdict class; ρ-1 substance precedence)

  Cell-individual reading (ρ-1):
    Each cell tracks its own roll_forward_count independently.
    NO aggregate / batch reset based on sibling-cell behaviour.
    NO carryover from supersedes-chain unless explicit cell_id match.
    Lint L10 validates against prior-cycle verdict.yaml at SAME cell_id only.

  Caveat-class-change does NOT trigger reset.
```

**First-cycle CP correction note** (cycle 03-32 R3 §1 Option A ratified): the prior v3 §12.1 wording stated `roll_forward_count := 1 if verdict == conditional_pass` on first cycle. Per ρ-1 substance precedence — `roll_forward_count` measures **prior-cycle carryover**, and a first-cycle cell has no prior cycle to roll forward from — the canonical first-cycle baseline is **0 regardless of verdict class**. The first increment to roll=1 (if the cell sustains CP) occurs at the **second** cycle of the same cell_id. The prior `1 if CP` wording was a mechanical drift introduced during v3 §12 drafting and is mechanically corrected here per MR-12 forward-only (no retroactive verdict re-classify; cycle 03-31 cell_id first-cycle entries already followed the substance precedence even where the wording was ambiguous).

**Key clarification** (per R3 §4.2 KNUTH discussion + cycle 03-31 R3 §1.1 ρ-1 BINDING): Phase 1 inconsistency on A1-2 + A1-4 (Phase 1 chose reset on caveat-class change → roll=1) was **non-compliant** with Reference/20 v2 §5.2 Tier 4 verbatim "reset to 0 at full_pass / fail transition". Phase 2 normalization (A1-2 + A1-4 roll=2 strict) is **mechanical correction per MR-12** (forward-only; Phase 1 preserved in `_phase1_synthetic/` subdirectory per cycle 03-30 R1 audit trail).

**ρ-1 cell-individual reading codification** (NEW cycle 03-31): the strict reading applies **per-cell-individually**; sibling-cell or batch-aggregate behaviour does NOT bear on any individual cell's roll_forward_count update. Cycle 03-31 R3 §1.1 D-§roll_forward Option ρ-1 ratified BINDING; v3 §12 codifies the cell-individual semantic explicit.

### §12.2 5-cell cycle 03-30 Phase 2 normalize = canonical reference exemplar

| Cell | Cycle 03-26 baseline | Cycle 03-30 Phase 1 (non-compliant) | Cycle 03-30 Phase 2 (strict) | Rationale |
|------|---|---|---|-----------|
| A1-1 p24_t6 | roll=1 (text-only CP baseline) | roll=2 (Phase 1 increment chosen) | **roll=0** | FP reset per §5.2 Tier 4 verbatim (CP→FP transition via tier-3 inline-contract real-LLM) |
| A1-2 p10_t6 | roll=1 (CP baseline) | roll=1 (Phase 1 reset-on-caveat-change; non-compliant) | **roll=2** | CP sustained per BG-1 honest reading; strict normalize = cycle 03-26 baseline 1 + cycle 03-30 = 2 (at LEIBNIZ N=2 ceiling) |
| A1-3 p40_t6 | (cycle 03-30 first cycle of cell-ID-schema; no prior) | roll=2 (Phase 1 increment chosen) | **roll=0** | FP per Option B RELOCATE; reset per §5.2 Tier 4 verbatim |
| A1-4 p13_t6 | roll=1 (CP baseline) | roll=1 (Phase 1 reset-on-caveat-change; non-compliant) | **roll=2** | CP sustained per BG-1 honest reading; strict normalize = cycle 03-26 baseline 1 + cycle 03-30 = 2 (at LEIBNIZ N=2 ceiling) |
| A1-5 F6_t6 | (cycle 03-30 first cycle of cell-ID-schema; no prior) | roll=1 (first cycle) | **roll=0** | FP per Option B RELOCATE; reset per §5.2 Tier 4 verbatim (cycle 03-26 cell-id schema-evolution; F6_t6 is new cell-id for the API-runtime F6 row formerly cycle 03-26 §B.6 row 1 baseline) |

**§12.2 codifies**: the 5-cell cycle 03-30 Phase 2 column = canonical reference exemplar; cycle 03-31+ R1 subagent prompts MUST normalize `roll_forward_count` per §12.1 strict reading from cycle start (not after Phase 1 lint-at-close iteration).

### §12.3 Interaction with §5.2 Tier 4 N=3 ceiling

§12.1 + §12.2 strict reading does **NOT** alter §5.2 Tier 4 N=3 ceiling. Under strict reading, A1-2 + A1-4 sat at roll=2 entering cycle 03-31 = **1 grace cycle remaining** under N=3 BINDING. Cycle 03-31 cycle resolution: both cells activated Option α (forced-escalate via cycle 03-30 fix-scope completion per Reference/21 §3.8 Option α flag conditions; multi-host harness + L2 test scope addition) → transitions FP per LEIBNIZ N=2 observation window CLOSED → roll=0 reset. N=3 ceiling never actually engaged at cell-individual level; observation window terminated by Option α transitions before N=3 reach.

### §12.4 Interaction with LEIBNIZ DSS-CY25-§13-A N=2 stricter proposal — observation window CLOSED

§12.1 + §12.2 strict reading aligned with **both** N=3 BINDING (majority) and N=2 stricter (LEIBNIZ + KNUTH DSS sustained). Under N=2 stricter, A1-2 + A1-4 at roll=2 = **0 grace cycles remaining** = AT ceiling (1 cycle grace already given).

**Observation window status (cycle 03-31 R3)**: LEIBNIZ N=2 sunset clause **NOT triggered** — cycle 03-31 R3 §5.5 D-§A6.5 confirms both p10_t6 + p13_t6 Option α activated cycle 03-31 → FP transition before N=2 sunset eligible. Observation window CLOSED. **DSS-CY30-§A1.4-P2-α LEIBNIZ + KNUTH preserved per MR-11** (cycle 03-30 R3 §6.1 verbatim sustained); window closure does NOT modify the DSS — preserves it as forward-watch historical anchor.

`C-LEIBNIZ-N2-observation` counter (Reference/21 §9): cycle 03-30 close = 2 cells; cycle 03-31 close = 0 cells (membership empty post-Option α transitions; observation window CLOSED). NEW `C-C8` counter (Reference/21 §10 v0.2.2 amendment): Phase 7 T4 cycle count = 1 inaugural per Stream 1 cell-1 tier-4 PASS.

### §12.5 L10 NEW lint rule reference

L10 lint rule (cf. §3 lint table extension) validates §12.1 strict reading against prior-cycle `verdict.yaml` at same `cell_id`. Implementation requires:
- `tools/audit_verdict_lint.py` v3 to maintain a cycle-history index of `verdict.yaml` at each `cell_id` (per Reference/21 v0.2.2 §3.5 cell-history attestation + ρ-1 cell-individual semantic per §12.1 above).
- Lint failure mode → subagent self-correct prior-cycle baseline reading; if cycle-history index ambiguous, escalate R2 reviewer.

---

## §13 R1 Prompt Skeleton Lint-at-Close Step (BINDING from cycle 03-31)

Per cycle 03-30 R3 §1.1 + §4.1 + §4.2 forward-to-R4 directive + cycle 03-31 R3 §3 D-§A4 BINDING, cycle 03-31+ R1 subagent prompts MUST include in Field 9 Report format:

```
Pre-close lint-at-close checklist (per Reference/20 v3 §11.6 + §12.5):
  [ ] caveats[] non-empty IFF verdict == conditional_pass (L2 + L9)
  [ ] notes: block contains forward-binding scope language (tier-3 ladder / tier-4 deferred / Plan-cross-ref) if applicable (L9)
  [ ] caveats[] entries are material evidence-gaps NOT forward-binding scope (L9 heuristic)
  [ ] roll_forward_count reset to 0 IFF verdict transitions to full_pass / fail / fail_fix=* / not_applicable (L10)
  [ ] roll_forward_count increments by 1 IFF verdict sustains conditional_pass (regardless of caveat-class change) (L10)
  [ ] cell-individual reading per §12.1 ρ-1 (no sibling-cell carryover) (L10)
  [ ] canonical_tenet_axis ∈ {#1..#10} per README.md authority (L8)
  [ ] cell_id matches cycle R0 enumeration (L7)
```

Subagent self-correct before close; if escalation needed, route to R2 reviewer; if R2 cannot resolve → R3 D-item.

---

## §14 `tools/audit_verdict_lint.py` v3 Implementation Spec

### §14.1 v2 → v3 lint extension scope

| Lint ID | Source | v3 status | Implementation |
|---------|--------|-----------|----------------|
| L1 | v1 | unchanged | existing v2 implementation sustained |
| L2 | v1 | **clarified** in v3 §11.6 codification | existing biconditional check sustained; v3 docstring updated to reference §11 canonical pattern |
| L3 | v1 | unchanged | sustained |
| L4 | v1 | unchanged | sustained |
| L5 | v1 | unchanged | sustained |
| L6 | v1 | unchanged | sustained |
| L7 | v2 | unchanged | sustained |
| L8 | v2 | unchanged | sustained |
| **L9 NEW v3** | v3 §11 + §3 | **NEW** | caveat-placement check; ~30-50 Python LOC; FP cell → `caveats:[]` empty MANDATORY + `notes:` block parsed for tier-3 / tier-4 forward-binding scope language patterns; CP cell → `caveats[]` non-empty with material-evidence-gap-or-equivalent text (heuristic check). |
| **L10 NEW v3** | v3 §12 + §3 | **NEW** | `roll_forward_count` strict reading check; ~40-60 Python LOC; reads prior-cycle `verdict.yaml` at same `cell_id` (from cycle-history index per Reference/21 v0.2.2); validates strict reset rule + ρ-1 cell-individual semantic. |

Total ARCHIMEDES implementation slot (cycle 03-31 R2 close target per cycle 03-26 R3 D-§A.2 precedent): **~70-110 NEW Python LOC** added to existing v2 `tools/audit_verdict_lint.py`. CI hook G4-folder-3 gate retains existing structure.

### §14.2 v2 → v3 transition timeline

| Phase | Cycle | Event |
|-------|-------|-------|
| R1 candidate | 03-31 R1 | §A4 chapter produced |
| R2 cross-review | 03-31 R2 | SYNTHESIST R2-A2A3A4 chair + DARWIN secondary |
| R3 ratify | 03-31 R3 | §3 D-§A4 5 sub-motions BINDING + §1.1 D-§roll_forward ρ-1 BINDING |
| R4 close | 03-31 R4 | Reference/20 v3 EN canonical doc creation per G4-folder-3 + G5-sync-5 BINDING canonical doc creation check |
| Master Ratification | post-R4 | Batch 26 forward dispatch |
| Canonical deploy | 03-31 G5 sync | 4-mirror byte-identical (suggestion / canonical / release v0.57.7-alpha / agent_dev) |
| TW sibling | 03-32 R-AMEND | TW translation per Rule #78 §78.5 same-PR coordinator G5 sync pattern (post-Master Ratification) |
| Cycle 03-32+ R1 | 03-32+ | inherit v3 lint-at-close BINDING |

### §14.3 Reference/20 v2 archive flagging

Per MR-12 既有不破壞 + cycle 03-26 R3 v1 archive precedent:
- Reference/20 v2 = DRIFT-FLAGGED archived at cycle 03-31 R4 G4-folder-3 close (post-Master Ratification Batch 26 deploy);
- v2 frontmatter `status:` field updated to `DRIFT-FLAGGED archived 2026-05-13 per Master Ratification Batch 26 (Reference/20 v3 supersedes)`;
- v2 substance §1-§10 unchanged (preserved verbatim in v3 §1-§10 cross-reference);
- Historical cells (cycle 03-26 R1 §B.6 worked examples + cycle 03-30 R1 5 A1 cells) NOT retro-fitted (per MR-12); they reference v2 explicitly via frontmatter `authority:` field.

---

## §15 Compliance Refs

| Authority | This doc compliance |
|-----------|---|
| **MR-2** Tenet wording unchanged | v2 §2 + §3 + §5.2 + §6 substance preserved verbatim in v3 §2-§10; only NEW §11 + §12 + §13 + §14 added |
| **MR-7** R3 ratification | v3 ratified per cycle 03-31 R3 §3 D-§A4 5 sub-motions BINDING + §1.1 D-§roll_forward ρ-1 BINDING |
| **MR-11** DSS verbatim | v2 §7 DSS-CY25-§13-A LEIBNIZ verbatim preserved in v3 §7; v3 §12.4 cross-references cycle 03-30 R3 §6.1 DSS-CY30-§A1.4-P2-α LEIBNIZ + KNUTH verbatim (NOT modified — only cross-ref + observation window CLOSED status update added) |
| **MR-12** 既有不破壞 strict | v2 §1-§10 NOT modified; v3 NEW §11 + §12 + §13 + §14 additive only; v2 archived (not deleted); cycle 03-26 + cycle 03-30 cells NOT retro-fitted |
| **ZT-1** 十大宣言 | v3 §8 compliance table re-anchors Tenet refs to canonical 10 Tenets per README.md; cycle 03-26 R3 D-§A.2 v2 baseline carried forward |
| **ZT-2** endpoint 10/0/0★ FINAL | v3 publishes under endpoint 10/0/0★ FINAL effective cycle 03-24 inheritance; no endpoint change |
| **ZT-3** control-range 加嚴 only | v3 only adds NEW §11 + §12 + §13 + §14 = control-range 加嚴 (lint-at-close + strict reading + cell-individual ρ-1); no relaxation |

---

*Reference/20 v3 — Audit Verdict Format Codification — RATIFIED cycle 03-31 R3 §3 D-§A4 5 sub-motions BINDING + §1.1 D-§roll_forward ρ-1 cell-individual reading BINDING — 2026-05-13*
*Items #6 (Option B RELOCATE) + #10 (Tier-3 caveat placement) + #11 (roll_forward_count strict reading) consolidate into NEW §11 + §12*
*v2 §1-§10 substance verbatim preserved per MR-12 forward-only; BG-4 incremental refinement bounded + reversible*
*L9 + L10 NEW lint rules extend `tools/audit_verdict_lint.py` v3 (~70-110 NEW Python LOC; ARCHIMEDES cycle 03-31 R2 close target)*
*A1-1 cycle 03-30 verdict.yaml = canonical reference exemplar (§11.1); 5-cell cycle 03-30 Phase 2 normalize = canonical reference exemplar (§12.2)*
*LEIBNIZ N=2 observation window CLOSED cycle 03-31 per p10_t6 + p13_t6 Option α activation transitions FP (DSS-CY30-§A1.4-P2-α preserved per MR-11)*
*Master Ratification Batch 26 forward dispatch target post-R4 close 2026-05-13*
