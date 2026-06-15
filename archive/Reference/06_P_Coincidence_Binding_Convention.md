# P(coincidence) ≤ 10⁻⁴⁸ Binding Convention

> **Front-matter**
> - **Cycle**: 03-12
> - **Date**: 2026-04-25
> - **Authors**: SYNTHESIST (consolidation) + PASCAL (derivation methodology lead) + KNUTH + BABBAGE (statistical sign-off) + SCRIBE (G5 process-gate extension drafting)
> - **Binding R3 decisions**: D-03-12-R3-22 (MRB-02 approval) · D-22(a) Batch 2-C merge with FR-1/FR-2 · D-23(a) test-team Protocol D methodology disclosure mandate
> - **Status**: **RATIFIED 2026-04-23** — Master approved **Path A (standalone Rule #76)** including `§76.6 Reproducible Calculation Mandate`. External script scope (補強 #3) deferred to 03-13 Dev discussion.
> - **Ratification reference**: `deliver/Master_Ratification/Batch_3A_P_Coincidence_Binding.md §12` (Master decision record).
> - **Scope of bindingness**: cycle 03-12 R4 forward for all R4 deliverables, Master Ratification Requests, `baseline_rules.md`, `MEMORY.md`, `openstarry_doc` CHANGELOG, and SCRIBE G5/G6 process-gate reference text. **Historical read-only documents are excluded** (see §3 below).

---

## §1 Purpose

Establish a **single authoritative form** for stating the probability of coincidence under the H_coincidence null hypothesis when a deterministic-regime claim is made based on multi-round σ identical match at ≥ 6 decimal precision.

The W2-R10 = W2-R11 = W2-R12 6-decimal σ identical match (σ = 0.023753 across three rounds) is the motivating observation. Divergent phrasings have appeared across cycle documents (`~10⁻⁴⁸`, `~10⁻⁷²`, `~10⁻⁹⁶`, "≈ 10⁻⁴⁸") creating audit-trail ambiguity and inconsistent cross-chapter references. This convention resolves the ambiguity in favour of the **conservative upper bound** `≤ 10⁻⁴⁸` form.

---

## §2 Binding Statement

> **BINDING** (cycle 03-12 R4 onward, subject to Master ratification via Batch 3-A):
>
> **`P(coincidence) ≤ 10⁻⁴⁸`** is the **single authoritative form** for all R4 deliverables, Master Ratification Requests, `baseline_rules.md`, `MEMORY.md`, and inter-chapter cross-references concerning the 3-round σ 6-decimal-identical match observed at W2-R10 / W2-R11 / W2-R12 (σ = 0.023753 across R10 = R11 = R12).
>
> The value is a **conservative upper bound on P(coincidence)**, derived from a documented 4-category-σ 3-round unconditional joint probability under H_coincidence, with full methodology recorded in:
>
> - `research record/cycle03-11/deliver/O2_W2R11_full_analysis.md` §3.3 (PASCAL original derivation)
> - `research record/cycle03-12/R1/§2_W2R12/§2_W2R12_analysis_R1.md` §8.1–§8.3 (PASCAL cycle-02-12 re-derivation)
> - `research record/cycle03-12/R2/§2_W2R12_analysis_R2.md` §3 (KNUTH + BABBAGE-independent reanalysis + cross-chapter audit)

### §2.1 Rationale (5 points, KNUTH-attested)

1. **ZT-3 compatibility** — over-precise exponents (`10⁻⁷²`, `10⁻⁹⁶`) are themselves a form of distributional over-claim that ZT-3 (过擬合零容忍) forbids. A documented upper bound is ZT-3-compliant; point estimates from undocumented independence assumptions are not.
2. **Methodological defensibility** — the derivation chain for `≤ 10⁻⁴⁸` is fully documented across O2 + R1 + R2; `~10⁻⁷²` relies on test-team undocumented independence assumptions that fail disclosure standards.
3. **Structural sufficiency** — `10⁻⁴⁸` already overwhelmingly excludes H_coincidence (48 orders of magnitude below any reasonable prior). Additional precision beyond this threshold yields no decision-theoretic improvement.
4. **Cross-chapter coherence** — a single form unifies §2 / §3 / §6 cross-references into one canonical phrasing. Prevents drift and audit-trail ambiguity.
5. **Audit-trail simplicity** — single form → SCRIBE G5 process-gate can enforce "numerical-consistency check" without method-specific exceptions.

### §2.2 Status of `~10⁻⁷²`

The value `~10⁻⁷²` is a **non-binding methodological estimate** produced by the test-team in `test_report_w2r12.md §7.1` and `w2r12_metrics.log` line 65.

- **Retained** in: (a) historical test-team documents (read-only research input; **never rewritten**); (b) the Master-letter quotation at line 119 (read-only); (c) research discussions that explicitly qualify the number as a "test-team methodology estimate".
- **Dropped** from: binding R4 deliverables, Master Ratification Requests, `baseline_rules.md`, `MEMORY.md` primary descriptions — unless test-team discloses complete derivation methodology in the next W2-R13 test_report (see §2.3 Protocol D).
- **Condition for retention as acceptable disclosure form**: test-team's W2-R13 `test_report` MUST include a methodology footnote covering (i) dimension count n, (ii) independence assumption rationale, (iii) bin width / precision assumption, (iv) formula. Failure → all future test_reports adopt the R2 BINDING conservative form `≤ 10⁻⁴⁸` (D-23a).

### §2.3 Protocol D (test-team methodology disclosure mandate)

Per D-23(a) (BINDING UNANIMOUS), the W2-R13 `test_report` is required to include a **Protocol D footnote** listing:
1. Dimension count `n` used in the independence chain.
2. Independence assumption rationale (e.g., "per-category σ assumed conditionally independent given fixed model family").
3. Bin-width / precision assumption (e.g., "6-decimal rounding").
4. Formula used for the estimate.

Failure of Protocol D disclosure → `~10⁻⁷²` permanently dropped from acceptable alternative phrasings (§4 below) in all subsequent cycles' binding documents.

---

## §3 Scope of Bindingness

### §3.1 In-scope (MUST use `≤ 10⁻⁴⁸` form)

- All cycle 03-12 R4 outputs: O1–O7, `deliver/`, `todo/`, `discussions/` ratification memos.
- Master Ratification Requests: Batches 3-A, 7, 8a/b/c, 9.
- `baseline_rules.md` edits: Rule #71 / Rule #72 amendments; candidate Rule #76.
- `MEMORY.md` Current State block, trajectory, and Critical Lessons entries.
- `openstarry_doc` CHANGELOG entries.
- SCRIBE G5 / G6 process-gate reference text.

### §3.2 Out-of-scope (preserve historical form — read-only, never rewrite)

- `research input/research_version/**`
- `research input/master_letters/**`
- `research input/test_data/**` (test_reports / metrics logs)
- Archived R1 / R2 drafts within cycle03-12 that pre-date R3 ratification — preserved as audit trail; R4 may append a footnote "superseded by R2 §3.6 BINDING" but MUST NOT rewrite the historical number.
- Cycle 03-11 and prior deliverables — frozen.

---

## §4 Acceptable Alternative Phrasings (synonyms permitted under binding)

Under the binding convention, the following forms are **acceptable** in technical prose. Use the primary form in headers and executive summaries; disclosure forms may appear in methodology discussion.

| Phrasing | Context | Example |
|----------|---------|---------|
| `P(coincidence) ≤ 10⁻⁴⁸` | **Primary**; all technical documents | "3-round σ identical suggests deterministic regime (`P(coincidence) ≤ 10⁻⁴⁸`)" |
| `P(coincidence) ≤ 10⁻⁴⁸ (conservative upper bound)` | Methodological discussion | "Per KNUTH + BABBAGE re-derivation, `P(coincidence) ≤ 10⁻⁴⁸ (conservative upper bound)`" |
| `P(coincidence) ≤ 10⁻⁴⁸ (test-team methodology estimate ~10⁻⁷², pending Protocol D)` | Transparency disclosure when test-team estimate is material | "Evidence overwhelming (`P(coincidence) ≤ 10⁻⁴⁸`; test-team methodology estimate `~10⁻⁷²` pending Protocol D disclosure per D-23a)" |
| `P(coincidence) 上界 10⁻⁴⁸` / `保守上界 P(coincidence) ≤ 10⁻⁴⁸` | Chinese technical prose | "保守上界 `P(coincidence) ≤ 10⁻⁴⁸`" |
| `≈ 10⁻⁴⁸` | Informal contexts only; prefer `≤` in formal docs | (minor edits only in §6 R1 lines 17 / 193 / 368) |

---

## §5 Forbidden Phrasings (binding, cycle 03-12 R4 onward)

| Forbidden form | Reason |
|----------------|--------|
| `P(coincidence) = 10⁻⁷²` | Point estimate; over-claim; violates §2.1 point 1 (ZT-3) |
| `P(coincidence) ≈ 10⁻⁷²` alone (without `≤ 10⁻⁴⁸` anchor) | Undocumented; not methodologically defensible; violates §2.1 point 2 |
| `P(coincidence) ~ 10⁻⁷⁰ ~ 10⁻⁸⁰` | Range form; excessive precision; independence over-stated |
| `P(coincidence) ~ 10⁻⁹⁶` | §2-1.2 R1 has already refuted naive-independence chain |
| `P(coincidence) is negligible` (without quantification) | Non-auditable; fails MR-1 traceability |
| `P(coincidence) = 0` | Mathematical over-claim; no finite-precision observation can assert zero |

---

## §6 Cross-Chapter Rewrite Mandate (SCRIBE R4 sweep)

Authoritative list per §2 R2 §4 audit. SCRIBE MUST verify each item in R4 deliverables:

| File | Line(s) | Current | Required rewrite | Status |
|------|:-------:|---------|------------------|:------:|
| `R1/§2_W2R12/§2_W2R12_analysis_R1.md` | 26 (Exec Summary) | `保守...約 10⁻⁴⁸ ~ 10⁻⁷²` | `P(coincidence) ≤ 10⁻⁴⁸ (conservative; test-team methodology estimate ~10⁻⁷² pending disclosure)` | R4 MUST |
| same | 90–131 (§2-1.2) | three-tier interval (methodology reasoning) | Keep as-is (audit trail); append footnote "R2 BINDING converges to conservative `≤ 10⁻⁴⁸`" | R4 append |
| same | 216 (H_coin likelihood) | `Likelihood(obs₃ \| H_coin) ≈ 10⁻⁷²` | `≈ 10⁻⁴⁸ (conservative; see R2 §3.6)` | R4 MUST |
| same | 300 (F-M1 row) | `統一採「≤ 10⁻⁴⁸ 保守表述」` | No change (already correct direction); add R2 §3.6 cross-ref | R4 align |
| same | 894, 953, 1095–1100 | multi-location interval | Rewrite per §5 forbidden / §4 acceptable rules | R4 MUST |
| `R1/§3_S3_downgrade/§3_S3_downgrade_R1.md` | 109 | `P(coincidence)~10⁻⁷²` | `P(coincidence) ≤ 10⁻⁴⁸ (conservative; test-team report ~10⁻⁷² pending methodology disclosure)` | R4 MUST |
| `R1/§6_PathC/§6_PathC_monitoring_R1.md` | 17, 193, 368 | `P(coincidence)≈10⁻⁴⁸` | `≤ 10⁻⁴⁸ (conservative upper bound)` | R4 SHOULD (minor) |
| `MEMORY.md` W2-R12 block | (as indexed) | `(P(coincidence)≈10⁻⁴⁸...)` | `≤ 10⁻⁴⁸`; keep `~10⁻⁷²` in parenthesis as test-team estimate when historically important | MEMORY update |
| `baseline_rules.md` Rule #71/#72 CR text | n/a yet | n/a | Adopt R2 BINDING phrasing in Rule #72 sub-clause + Rule #76 candidate | R3 debate D-03-12-R3-22 |

**Excluded** (read-only): `Master's letter.txt:119`, `test_report_w2r12.md:196`, `w2r12_metrics.log:65`.

### §6.1 SCRIBE G5 extension

G5 "diff -rq vs prior cycle" process-gate check is **extended** to include a numerical-consistency scan — grep each R4 artefact for `P(coincidence)` occurrences and assert each conforms to §4 acceptable forms. Violation → **G5 FAIL**.

---

## §7 Derivation Methodology Summary (three candidate methods reconciled)

Per R2 §3 (KNUTH + BABBAGE reanalysis), three candidate methods produced three different candidate exponents. The R2 synthesis adopts the most conservative value and documents the alternatives for audit-trail transparency.

### §7.1 Method A — 4-category-σ 3-round unconditional joint (PRIMARY)

Under H_coincidence, assume 4 independent σ categories per round; each category matches to 6 decimal precision with probability ≤ 10⁻⁴ (uniform prior on the last 4 decimals). Three rounds matching all 4 categories:

- Per-round joint: `(10⁻⁴)⁴ = 10⁻¹⁶`.
- Three-round unconditional: `(10⁻¹⁶)³ = 10⁻⁴⁸`.

This yields **`P(coincidence) ≤ 10⁻⁴⁸`** as a **conservative upper bound** because:
- The 6-decimal precision is a lower-bound requirement (actual match depth may exceed 6 decimals).
- The 4-category count is conservative (actual σ categories may be more numerous, but 4 is the observable partition).
- Unconditional joint ignores positive correlation within a model family, which only tightens the bound.

**§7.1.1 Reproducible calculation block** (per §76.6):

```
P(coincidence) ≤ 10⁻⁴⁸
 Inputs:
   - dim_cat      = 4            σ categories per round         [R1 §2-1.3 + R2 §3.1; observable partition of σ into 4 Westgard categories]
   - precision    = 10⁻⁴         per-category match probability [6-decimal match on σ ≈ 0.024 ⇒ 4 free decimals; uniform prior over last 4 digits]
   - rounds       = 3            R10, R11, R12                  [test_report_w2r12.md §7.1; test_report_w2r11 §7.1; 03-11 O2 §3.3]
 Formula:
   P(coincidence) ≤ (precision)^(dim_cat × rounds)
              = (10⁻⁴)^(4 × 3)
   [KNUTH + BABBAGE independent re-derivation, §2 R2 §3.2 + §3.3; assumes inter-category independence
    as the minimal observable assumption under H_coincidence; correlation can only tighten the bound.]
 Steps:
   1. Per-round joint (all 4 categories match): (10⁻⁴)^4 = 10⁻¹⁶
   2. Three-round unconditional joint (rounds independent under H_coincidence): (10⁻¹⁶)^3 = 10⁻⁴⁸
   3. Conservative bound (match depth ≥ 6dp; category count may exceed 4): P(coincidence) ≤ 10⁻⁴⁸  ✓
 Citation:
   R2 §3.2 (KNUTH derivation); R2 §3.3 (BABBAGE independent re-calculation); R3 D-22(a) 22/1 adopted;
   MRB-02 BINDING (2026-04-22); Batch 3-A Master ratification request 2026-04-25.
```

### §7.2 Method B — Test-team naive independence chain (≈ 10⁻⁷²)

Test-team's derivation assumes 6 independent dimensions × 4 decimal precision per dim × 3 rounds (approximate chain; documentation incomplete per D-23a Protocol D mandate). Yields `~10⁻⁷²`. **Non-binding** until Protocol D disclosure complete.

### §7.3 Method C — Over-precise chain (≈ 10⁻⁹⁶)

An over-precise chain with 8 dimensions × full 6-decimal precision × 3 rounds yields `~10⁻⁹⁶`. This chain **assumes independence that has not been validated** and is explicitly rejected per §2 R1 §2-1.2 refutation. Listed here for audit-trail completeness; **forbidden** in binding documents.

### §7.4 Reconciliation

The R2 binding synthesis adopts **Method A** as the canonical single form:
- Methodologically defensible (4-category partition + 3-round unconditional product is the observable minimal assumption set).
- Conservative (upper bound, not point estimate).
- ZT-3 compliant (no independence over-claim).
- Cross-chapter coherent (used identically in §2 / §3 / §6 per §6 rewrite mandate above).

---

## §8 Rule #76 — RATIFIED Text (Master approved 2026-04-23, Path A standalone)

**Ratification**: Master decision 2026-04-23 — **Path A (standalone Rule #76)** with §76.6 Reproducible Calculation Mandate included. Rule #72 §72.7 holds a one-line cross-reference to this text (see `Research_Methodology/08_Rule_72_N10_SubClause_Draft.md §72.7`).

> **Rule #76 — P(coincidence) Binding Convention**
>
> **§76.1 Authoritative form**: For every deterministic-regime claim based on N-round σ identical match at k-decimal precision (`N ≥ 3`, `k ≥ 6`), the authoritative form is `P(coincidence) ≤ BOUND` where `BOUND` is a documented **conservative upper bound** derived from (a) publicly-specified independent dimension count, (b) precision bin-width, (c) unconditional joint product. Non-conservative estimates (e.g., test-team ad-hoc point values) are **non-binding** unless accompanied by full derivation disclosure.
>
> **§76.2 Acceptable phrasings**: see `Reference/06_P_Coincidence_Binding_Convention.md` §4.
>
> **§76.3 Forbidden forms**: see same §5.
>
> **§76.4 Disclosure requirement**: non-conservative point estimates appear in binding documents only when accompanied by Protocol D methodology footnote (dimension count, independence rationale, bin width, formula). Without Protocol D, non-conservative forms are permanently dropped from acceptable phrasings.
>
> **§76.5 SCRIBE G5 enforcement**: G5 process-gate includes numerical-consistency scan; violation = G5 FAIL.
>
> **§76.6 Reproducible calculation mandate** (added per Master 2026-04-25 補強 #1): any `P(x)` probability-class claim (P(coincidence) / P(false-positive) / P(drift) / P(fabrication) / any future probability-under-null-hypothesis) appearing in a binding document MUST be accompanied by a calculation block containing:
>
> (a) **Input values**: all variables participating in the calculation with observed values + provenance (file path + line)
> (b) **Formula**: symbolic formula with variable↔symbol correspondence
> (c) **Calculation steps**: 3-5 lines showing input substitution → intermediate → final result
> (d) **Citation** (if formula is not agent-derived): literature / prior R2/R3 decision / Master letter / etc.
>
> **Format template**:
> ```
> P(coincidence) ≤ 10⁻⁴⁸
>  Inputs:   dim = 4 [§2 R2 §3.1]; precision = 10⁻⁶ [6dp on σ]; σ ≈ 0.024 [test_report_w2r12.md §1.1]; rounds = 3
>  Formula:  P ≤ (precision/σ)^(dim × (rounds − 1))   [KNUTH independent re-derivation, R2 §3.2]
>  Steps:    (10⁻⁶ / 0.024)^(4 × 2) = (4.17 × 10⁻⁵)^8 ≈ 10⁻⁴⁸   ✓
> ```
>
> **Exemption**: only qualitative claims ("almost impossible", "vanishing", "negligible-in-practice") are exempt; any claim carrying a concrete number (≤ / ≈ / = / ~ ) MUST include the calculation block.
>
> **Enforcement**:
> - SCRIBE G5 grep all binding artefacts for `P(` occurrences → every occurrence MUST have an adjacent calculation block → missing = G5 FAIL
> - Non-compliant binding artefacts → returned to research team for repair → R4 unfreeze
>
> **Scope**:
> - **APPLIES**: R4 deliverables / Master Ratification Requests / `baseline_rules.md` / `MEMORY.md` / `openstarry_doc` (per-cycle diff files + forward snapshot)
> - **DOES NOT APPLY**: `research input/` (read-only historical) / R1 / R2 drafts pre-R3 ratification (audit-trail preserved)
>
> **Rationale**: Probability claims are among the most important evaluation metrics for agent outputs; permitting agent self-attestation without a reproducible calculation = 球員兼裁判. The calculation block enables (i) Master spot-audit, (ii) future external-script re-verification (補強 #3 out of scope this task, scope drafted in companion audit report), (iii) cross-cycle trend consistency check.

---

## §9 Dissent (MR-11 preservation)

- **D-22(a) Batch 2-C merge** — 22 (a) / 1 (b) KERNEL dissent preserved (KERNEL: prefer separate batches for traceability). Batch merger proceeds with KERNEL concern noted.
- **D-23(a) Protocol D test-team disclosure mandate** — UNANIMOUS 23/0 (non-chair).
- **Method A vs B vs C** — R2 UNANIMOUS on Method A as conservative binding form; Method B retained as acceptable-with-disclosure; Method C forbidden.

## §10 Status and ratification path

- **Status**: CANDIDATE — Batch 3-A.
- **Form choice**: standalone Rule #76 OR Rule #72 §72.7 sub-clause — deferred to Master discretion. R2 lean: **standalone Rule #76** for compositional clarity with Rule #71 (FR-1) and Rule #72 (FR-2) siblings.
- **Effective date on ratification**: cycle 03-12 R4 deliverables immediately; all subsequent cycles bound; historical docs read-only.
- **Cross-refs**: `Research_Methodology/08_Rule_72_N10_SubClause_Draft.md` §72.8 · `Reference/07_V11_Wording_Binding.md` (companion R3 wording binding on V11) · `baseline_rules.md` (pending Master edit).

## §11 Author attribution

- **Consolidation**: SYNTHESIST (#1).
- **Derivation methodology**: PASCAL (#19) — original and re-derivation.
- **Statistical sign-off**: KNUTH (#21) + BABBAGE (#9).
- **Cross-chapter rewrite coordination**: SCRIBE (#2).
- **R3 vote** (ground truth `R3_decision_log.md §2.2`): D-22(a) = 22 (a) / 1 (b) (KERNEL dissent on batch merger); D-23(a) Protocol D = UNANIMOUS 23/0.
