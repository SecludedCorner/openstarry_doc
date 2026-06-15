---
title: F-15 v3 — Third-Tier Amendment for TW Translation Parity (L3 Operational Mechanism)
author: SCRIBE (#2) + LINNAEUS (#13) + TURING (#17) + ARCHIMEDES (#16) + ASANGA (#8) + NAGARJUNA (#7) + SYNTHESIST (#1)
date: 2026-04-27
cycle: 03-15
status: CANDIDATE (pending Master Ratification Batch 12 #1)
authority: research-team (R4 final draft); Master (ratification)
supersedes: cycle 03-14 F-15 v2 (Research_Methodology/13_F_15_Scope_Expansion_Governance_Docs.md) — extended cumulatively, NOT replaced
language: en
cross_refs:
  - research record/cycle03-15/deliver/O1_TW_translation_final.md §3 + §4 + §10
  - research record/cycle03-15/R3/R3_decision_log.md §4.1 + §4.3 (D-§2-06 + D-§2-07 + D-§3-05)
  - research record/cycle03-15/openstarry_doc/Reference/11_Rule_78_TW_Translation.md (L1 sibling)
  - research record/cycle03-14/openstarry_doc/Research_Methodology/13_F_15_Scope_Expansion_Governance_Docs.md (v2 baseline, BINDING)
  - research record/cycle03-15/deliver/Master_Ratification/Batch_12_Request.md §3.1 (Item #1 bundle)
binding_until: Master Ratification Batch 12 close
---

# F-15 v3 — Third-Tier Amendment for TW Translation Parity (L3 Operational Mechanism)

## 1. Status

**CANDIDATE** at cycle 03-15 R4 close. Upgrades to **BINDING** upon Master Ratification Batch 12 Item #1 (bundled with Rule #78 L1 sibling). Effective **forward-only** from cycle 03-15 R3 close onward.

**Landing pattern**: L1+L3 hybrid per D-§2-07 (14/4/3/2). This document is the **L3 operational mechanism** half (Research_Methodology/, governance-doc front-matter parity-check + linter dispatcher mechanism). The **L1 high-level policy** half is `Reference/11_Rule_78_TW_Translation.md`. Master Ratification Batch 12 Item #1 dispatches both as one bundle.

**Cumulative scope**: F-15 v3 carries cumulative scope from v1 (cycle 03-14 ratified front-matter discipline) → v2 (cycle 03-14 ratified governance-doc scope expansion to 8 doc classes) → v3 (cycle 03-15 candidate TW parity tier). v1 + v2 functionality is PRESERVED unchanged; v3 is **additive** per MR-12 既有不破壞.

**ENG-FAB increment policy**: F-15 itself is a **governance-doc discipline bundle**, not a v1.X-numbered ENG-FAB item. Per cycle 03-14 ratification, F-15 amendment-not-increment progression keeps ENG-FAB v1.8 = 48 items canonical. v1.9 = 49 candidate F-16 unchanged. **v1.10 candidate F-19 NOT created** per D-§3-05 + D-§2-07 anchor unification.

## 2. R3 Provenance

- **D-§2-06 (C-1)** — CI gate = **multi-layer** (F-15 linter + pnpm build + doc gate + G4-folder + coordinator hook) UNANIMOUS 23/0.
- **D-§2-07 (A-1)** — TW landing point = L1+L3 hybrid (14 votes). F-15 v3 amendment = the L3 half.
- **D-§3-05 (B-6)** — Anchor framework = unified Rule #78 + F-15 third-layer (saves 1 candidate slot vs separate ENG-FAB v1.10 F-19) — 16/4/3; DSS-B6 4 dissent §10. **Conditional** on Master §3 outcome (A-trail covers §2+§3; B/C-trail covers §2 only).
- **D-§2-04** — quality discipline (glossary epistemic-prefix + structural fidelity + dual-author NO) feeds into linter checks per §3.
- **MRB-§2-02 RESOLVED** — sibling-naming convention `<basename>.<lang>.md` codified (TURING + SCRIBE).

## 3. Binding Text

### 3.1 F-15 v3 Three-Tier Bundle

> **F-15 v1 (cycle 03-14 ratified)** — Tier 1: governance-doc front-matter discipline. Front-matter language fields `title / author / date / cycle / status / authority / cross_refs` MUST present. Linter `tools/f15_check.py` enforces. **PRESERVED unchanged**.
>
> **F-15 v2 (cycle 03-14 ratified)** — Tier 2: governance-doc scope expansion. F-15 covers 8 doc classes: O-series / R-stage / Master_Ratification batches / SCRIBE process gates / Plan-spec dev-facing / Test instructions / todo_README / openstarry_doc canonical diff entries. Conditional-MUST gates per cycle 03-14 ratification text. **PRESERVED unchanged**.
>
> **F-15 v3 (cycle 03-15 NEW; this amendment)** — Tier 3: governance-doc TW translation parity check per Rule #78 §78.1 tier × §78.6 CI gate.

### 3.2 Tier 3 Five Sub-Mechanisms

> **Sub-mechanism 1 — Sibling presence check.**
> For each EN-side doc in §78.1 BINDING / CANDIDATE / KNOWLEDGE_ONLY tier (per `tools/f15_config.yaml` tier-config dispatcher), the linter asserts `<basename>.tw.md` exists in the same directory. Sibling-naming convention per Rule #78 §78.7 (codified MRB-§2-02). Failure mode per §78.6 layer-1: block-PR for BINDING; warning for CANDIDATE / KNOWLEDGE_ONLY; informational for CONSULTATIVE / SCRIBE-internal.

> **Sub-mechanism 2 — Front-matter language tag check.**
> TW sibling MUST have `language: tw` (or equivalent `language: zh-TW`) front-matter field, distinct from EN sibling's `language: en`. Cross-reference fields MUST identify the EN sibling explicitly (e.g., `cross_refs: - <basename>.md`). Front-matter schema enforced consistent with v1 baseline (title / author / date / cycle / status / authority / cross_refs).

> **Sub-mechanism 3 — Glossary epistemic-prefix check.**
> If a glossary file (`tools/tw_glossary.yaml` or similar) is present per Rule #78 §78.4 + O1 §6.1, every entry MUST have a `prefix:` field with value `verified` / `inferred` / `speculative`. Linter validates schema. Per ASANGA Yogācāra epistemic discipline. Absent or invalid prefix = block-PR for BINDING tier.

> **Sub-mechanism 4 — Structural-fidelity machine-checkable layer.**
> Linter compares heading hierarchy depth, table column counts, code-block presence between EN and TW siblings. Mismatch = warning for BINDING + CANDIDATE; informational for KNOWLEDGE_ONLY. Human-checkable layer (bullet ordering, footnote anchor stability, prose-paragraph-count parity) is reviewer-judgement at G4-folder gate; un-checkable layer (sentence-level alignment, idiomatic register match) is author latitude. Per ASANGA + TURING layer decomposition (Rule #78 §78.4 structural fidelity 22/1).

> **Sub-mechanism 5 — Per-tier dispatcher.**
> Linter accepts a config file (`tools/f15_config.yaml`) declaring which paths are which tier per Rule #78 §78.1 (BINDING / CANDIDATE / KNOWLEDGE_ONLY / CONSULTATIVE / SCRIBE-internal). Default config ships with the cycle 03-15 ratification; supports future tier refinement without re-coding.

### 3.3 NAGARJUNA Madhyamaka Coherence Note

Tier 3 is **bound by enumeration** (specific file paths in tier configs) rather than by the open-essence "all governance docs" reference. This neutralises the essentialism risk noted in R1 §10.3 + R2 F-01 (NAGARJUNA + ASANGA dissent on essentialist scope).

## 4. F-15 Linter Extension Spec

### 4.1 Scope Estimate

Per F-§2-R1-11 + R2 F-04 TURING code feasibility cross-check. Extension scope to `tools/f15_check.py`:

| Component | LOC range | Function |
|-----------|----------:|----------|
| Bare sibling-file presence check | 30-50 | For each EN-side doc in tier-config paths, assert `<basename>.tw.md` exists |
| Tier-graded scope dispatcher | 50-80 | Config-driven (`tools/f15_config.yaml`) dispatches per-tier rules per Rule #78 §78.1 |
| Glossary mechanism integration | 50-100 | If `tools/tw_glossary.yaml` present, validate schema (entry format with `prefix:` field) and optionally check term-consistency in TW siblings |
| **Aggregate** | **130-230** | **Within R1 §F-11 estimate** |

### 4.2 Implementation Guidance (Spec-Only; No Code in This Doc)

1. **Config-driven design**: tier definitions in external YAML, not hard-coded; supports future tier refinement.
2. **Sibling-naming flexibility**: linter accepts both `<basename>.md` (legacy) and `<basename>.en.md` (preferred) as EN canonical per Rule #78 §78.7.
3. **Failure-mode dispatcher**: per-tier failure mode (block / warning / informational) read from config; allows per-deployment customisation.
4. **Front-matter language tag check**: TW sibling MUST have `language: tw` (or `zh-TW`) front-matter; linter validates.
5. **Structural-fidelity machine-checkable check**: heading hierarchy depth + count, table column counts, code-block count compared between EN and TW siblings.
6. **Glossary epistemic-prefix validation**: every glossary entry must have `prefix: verified|inferred|speculative`; absent or invalid prefix = block-PR for BINDING tier.
7. **CI integration**: linter exits with code 0 (PASS) / 1 (warnings only) / 2 (block-level failure); CI consumes exit code per Rule #78 §78.6 layer 1.

### 4.3 Implementation Cycle

Scheduled for **cycle 03-16 (Plan51 implementation cycle)** per O1 §10 + F-§5-R2-15 coupling. Master Ratification Batch 12 #1 ratifies the policy + spec; Plan51 cycle 03-16 implements the linter extension.

## 5. Five-Tier Scope (Reflected from Rule #78 §78.1)

The linter dispatcher operates over Rule #78's five tiers:

| Tier | Strength | F-15 v3 layer 1 mode | Audit |
|------|----------|----------------------|-------|
| **BINDING** | MUST | block-PR | release-tag scheduled audit |
| **CANDIDATE** | SHOULD | warning | release-tag scheduled audit |
| **KNOWLEDGE_ONLY** | SHOULD | warning | release-tag scheduled audit (lower priority) |
| **CONSULTATIVE** | MAY | informational | informational (G4-folder gate) |
| **SCRIBE-internal** | MAY | informational | none |

Tier transitions (per Rule #78 §4.2 of O1 / §78.1 of L1 sibling):
- **CANDIDATE ↔ BINDING**: upgrade upon Master ratification; downgrade on supersession (forward-only respectful).
- **CONSULTATIVE → CANDIDATE**: upon ratification of draft into canonical artefact.
- **SCRIBE-internal → CANDIDATE**: rare; only if a SCRIBE-internal log is cited verbatim by a future Rule.

## 6. Strength Matrix (Reflected from Rule #78 §78.2 + §5)

| Strength axis | Governance | Canonical | Plan/operational |
|---------------|------------|-----------|------------------|
| F-15 v3 enforcement | **MUST** | **SHOULD** | **MAY** |
| Temporal escalation | 24h grace before R3+1 cycle close | 24h grace + R3+1 cycle close | n/a |
| Sub-mech 1 (sibling presence) | block-PR | warning | informational |
| Sub-mech 2 (lang tag) | block-PR | warning | informational |
| Sub-mech 3 (glossary prefix) | block-PR | warning | informational |
| Sub-mech 4 (struct fidelity) | machine MUST + human SHOULD warning | machine MUST + human SHOULD warning | machine MAY |
| Sub-mech 5 (tier dispatcher) | always-on | always-on | always-on |

## 7. Reflexive Application

Per F-§2-R1-10 + F-§2-R2-10 NAGARJUNA self-reference test: F-15 v3 itself follows the F-15 v3 schema. This document:

- Carries v1 front-matter discipline (title / author / date / cycle / status / authority / cross_refs ✓).
- Carries v2 scope (Research_Methodology/ subdir; CANDIDATE tier per §78.1).
- Carries v3 language tag (`language: en` ✓).
- Will acquire `16_F_15_v3_Third_Tier_Amendment.tw.md` sibling per Rule #78 §78.3 BINDING-tier same-PR-strict timing upon Master Ratification (cycle 03-16 R0 24h grace authoring).

## 8. Backward Compatibility (v1 + v2 Preserved)

F-15 v3 is **additive**:
- v1 front-matter discipline: PRESERVED unchanged. All cycle 03-14 ratified front-matter rules remain in force.
- v2 governance-doc scope expansion: PRESERVED unchanged. All 8 doc classes covered by v2 remain covered.
- v3 sub-mechanisms: ADDED layer atop v1+v2. Pre-cycle-03-15 docs grandfathered EN-only per Rule #78 §78.5 + MR-12; v3 sub-mechanisms apply to cycle 03-15+ docs forward.

Per R3 vote selection of L1+L3 hybrid over L1-only or L2-only: the policy lives in Rule #78 (clean concept) and the mechanism lives in F-15 amendment (infrastructure reuse). DSS-A1 4 dissent (preferred L2 ENG-FAB v1.10 F-19) preserved verbatim §10.

## 9. MR/ZT Compliance Audit

| Constraint | Status | Evidence |
|------------|:------:|----------|
| **MR-5 hard** (Tenet #10 status no change) | PASS | F-15 v3 is doc-tooling layer; orthogonal to Tenet #10 |
| **MR-6** (Core 零) | PASS | Linter extension lives under `tools/`; not Core |
| **MR-9** (no MUST WAIVE) | PASS | F-16 SHOULD initial unchanged; F-15 v3 strength stratified per Rule #78 §78.2 |
| **MR-10** (back-fill / retroactive) | PASS | Cycle 03-13 backfill = one-off precedent (sibling Reference doc 12) |
| **MR-11** (dissent preservation) | PASS | DSS-A1 / DSS-B6 preserved verbatim §10 |
| **MR-12** (既有不破壞 / forward-only) | PASS | Pre-cycle-03-15 grandfathered; v1+v2 preserved unchanged |
| **MR-13** standby | PASS | Not invoked |
| **ZT-1 / ZT-2 / ZT-3** | PASS | No Tenet rewrite; endpoint 10/0/0★ unchanged; control-range untouched |
| **ENG-FAB v1.8 = 48** | Preserved | F-15 amendment-not-increment per cycle 03-14 precedent |
| **ENG-FAB v1.9 = 49 F-16 SHOULD** | Preserved | F-16 strength unchanged (Batch 12 Item #4 informational only) |
| **v1.10 F-19** | NOT created | Per D-§3-05 + D-§2-07 anchor unification |

## 10. Dissent Preservation (per MR-11 verbatim)

### 10.1 DSS-A1 (D-§2-07 TW landing L1+L3 hybrid; 14/4/3/2)

**Minority**: NAGARJUNA + DARWIN + 2 others (4 votes preferred L2 ENG-FAB v1.10 F-19).

**Verbatim**: "L2 (ENG-FAB v1.10 F-19) provides cleaner工程化 + MUST/SHOULD/MAY 強度；L1+L3 hybrid 過於分散 governance."

**R4 obligation**: explicit dissent slot in this doc per MR-11. Note: this dissent applies to the L1+L3 hybrid landing decision. The R3 majority (14) outweighed but the dissent is preserved as a substantive minority position.

### 10.2 DSS-B6 (D-§3-05 anchor unified Rule #78 + F-15 third-layer; 16/4/3)

**Minority**: 4 dissent.

**Verbatim**: "Separate ENG-FAB v1.10 F-19 entry would provide cleaner ENG-FAB-side visibility for the trial-graduation governance integration; anchor unification 過於 saving 1 candidate slot 而失 ENG-FAB-side discoverability."

**R4 obligation**: footnote in this doc per MR-11. Note: D-§3-05 is **conditional** on Master §3 (A)/(B)/(C) outcome — if Master selects (A): Rule #78 + F-15 v3 covers both §2 + §3 graduation governance; if Master selects (B) or (C): F-15 v3 covers §2 only. Either way, F-15 v3 amendment proceeds.

### 10.3 Aggregate

§3-tagged dissent (DSS-B6) is preserved separately from §2-tagged dissent (DSS-A1 also relevant here as L3 vehicle choice). Other §2 dissent entries (DSS-A2 / DSS-B1 / DSS-B1b / DSS-B2 / DSS-B3 / DSS-B3b / DSS-D2) are scope-relevant to the L1 sibling Rule #78 doc and are preserved verbatim there.

---

*F-15 v3 Candidate — Cycle 03-15 R4 Final Spec*
*Authors: SCRIBE (#2) + LINNAEUS (#13) + TURING (#17) + ARCHIMEDES (#16) + ASANGA (#8) + NAGARJUNA (#7) + SYNTHESIST (#1)*
*Status: CANDIDATE (pending Master Ratification Batch 12 #1)*
*Landing: L3 operational mechanism half (Research_Methodology/); L1 high-level policy half = `Reference/11_Rule_78_TW_Translation.md`*
*Three-tier bundle: v1 front-matter (cycle 03-14) + v2 scope expansion (cycle 03-14) + v3 TW parity (cycle 03-15 candidate)*
*F-15 amendment-not-increment: ENG-FAB v1.8 = 48 PRESERVED; v1.10 F-19 NOT created*
*Compliance: MR-5 hard / MR-6 / MR-9 / MR-10 / MR-11 / MR-12 + ZT-1/2/3 全 PASS*
