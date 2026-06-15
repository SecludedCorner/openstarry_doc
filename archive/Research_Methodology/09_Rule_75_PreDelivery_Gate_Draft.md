# Rule #75 — Pre-Delivery Gate (DRAFT)

> **Front-matter**
> - **Cycle**: 03-12
> - **Date**: 2026-04-25
> - **Authors**: §1 R2 panel (DARWIN + SYNTHESIST + KNUTH) + ARCHIMEDES (LOC / delivery-report-structure impact)
> - **Strong support**: §1 R2 §5.3 (KNUTH: O(1) dev fix vs O(delivery→rollback) cost)
> - **Binding R3 decisions**: upgrade of CR-1 to binding ratification candidate (D-03-12-R3-14 / Batch 8c) + §5 R2 §12.2 generalisation from F-03-12-§5-R2-1 HIGH sub-item binding
> - **Status**: **CANDIDATE** — awaiting Master ratification via **Batch 8c** (bundled with Rule #72 N≥10 sub-clause + ENG-FAB v1.7)
> - **Effective date proposal**: Plan48 delivery (first post-ratification delivery). Plan47 retrospectively compliant without formal Rule #75 attestation — Plan47's K-3 5 MUST × 5 sub-check matrix provides equivalent coverage.

---

## §75.1 Purpose

Rule #75 complements **Rule #74 (L1' code + doc sync)** by adding a **dev-layer preventive block** at the Dev → Research transfer boundary. Where Rule #74 verifies post-delivery at the research side (O(delivery → research discovery → repair)), Rule #75 blocks pre-delivery at the dev side (O(dev self-check → fix)).

The cycle 03-11 G5-blindness lesson ("自宣 plan 滿足 ≠ 內容反映新裁決") and the six-cycle fabrication pattern (Plan36–41) both motivate a **process-level preventive control** that catches incomplete deliveries before they propagate into research verification or trigger F-6 (delivery-report completeness) violations. Rule #75 formalises this discipline as a dev-layer gate with an enforceable check-list.

---

## §75.2 Scope and trigger

### §75.2.1 Scope

Applies at **every Dev → Research transfer of a Plan delivery** — that is:
- After Dev completes Plan implementation.
- Before R → Dev transfer is initiated by Dev.
- Before Dev marks the Plan "ready for research review".

### §75.2.2 Trigger

**Automatic on every Plan delivery.** No exemption.

- **Plan47 grandfather clause**: Plan47 is assessed under the existing Rule #74 regime (Plan47's 5 MUST × 5 sub-check matrix provides equivalent coverage). Rule #75 applies **prospectively from Plan48 onward**.
- **No opt-out**: any Plan whose delivery report omits Rule #75 attestation is deemed incomplete; research-side G5 first-pass FAIL (see §75.5).

---

## §75.3 Check items (dev-team MUST attest before triggering R → Dev transfer)

### §75.3.1 MUST-item sub-check coverage

For every MUST item in the Plan delivery, dev-team MUST produce a **checkpoint acceptance-criteria document** listing **≥ 5 sub-checks per MUST**. This follows the Rule #74 L1' pattern demonstrated by Plan47's K-3: 5 MUST × 5 sub-check matrix (C47-K3-M1 through C47-K3-M5, each with 5 binding sub-checks, all PASS on 2026-04-19 dev verification).

### §75.3.2 Binding acceptance-criteria satisfaction

Dev-team MUST **explicitly reference and satisfy all prior-Master-ratified binding acceptance criteria** applicable to the Plan. Examples:
- **Plan47**: 5 MUST C47-K3-M1 through C47-K3-M5.
- **Plan48** (first Rule #75 application): **25 atomic acceptance criteria** = 3 MUST × sub-items (8 + 9 + 4) + 3 MUST headings + 1 roll-up (C48-M1a–h + C48-M2a–i + C48-M3a–d; see `Calibration_Reports/19_Plan48_25_SubItems_Binding.md`).
- **Plan49+**: binding acceptance criteria enumerated in the Plan's own spec + any carry-forward from prior Plans (per ENG-FAB F-8 carry-forward-must check).

### §75.3.3 ENG-FAB audit completion

ENG-FAB **v1.7** (or current ratified version) full audit PASS **attested in the delivery report, item-by-item** (no N/A unless explicitly justified). The new **F-10 (Pre-Delivery Gate Verified)** check (see `Research_Methodology/10_ENG_FAB_v1.7_Candidate.md`) forms the test-layer companion to Rule #75 — Rule #75 is the dev-layer block, F-10 is the research/test-layer verification (defense-in-depth; see §75.5).

### §75.3.4 δ-status explicit declaration

Dev-team MUST **explicitly state** whether the Plan:
- **Closes δ items** (δ DONE at delivery),
- **Carries δ forward** (list specific items + target Plan), OR
- Is **δ-neutral** (Plan does not touch δ scope).

Motivated by the **Plan46 → Plan47 K-3 carry-forward** learning (F-8 ENG-FAB item): a deferred Wave can silently miss a carry-forward-must SDK type without this explicit declaration. Rule #75.3.4 forces the declaration to be an affirmative statement, not a default-silent omission.

### §75.3.5 Runtime evidence completeness

For any MUST item requiring **runtime evidence** (e.g., W2-R13 runtime hookMap observation, canary positive-control verification, shutdown flush test, HMAC key clear-before-exit), dev-team MUST produce an **evidence artefact reference** (file path + line range) and attest the item passes the runtime check.

Example format:
```
C48-M1e (shutdown flush): runtime evidence = apps/runner/__tests__/shutdown-flush.test.ts:45-89
  Result: 1000 events written, 1000 events flushed, 0 loss, exit code 0
  Test run: 2026-04-DD HH:MM UTC, commit <git-rev>
```

---

## §75.4 Failure behavior

### §75.4.1 Dev-side failure

- **Any check item fails** → dev-team **blocks R → Dev transfer** (no dispatch to research inbox). Dev-team repairs and re-attests.
- **3 consecutive dev repairs fail** → Dev escalates to **Coordinator / Research joint intervention** (cross-team rescue session; analogous to the R3 Pareto-frontier empty-frontier escape in CR-PARETO, D-24a).

### §75.4.2 Research-side catch (defense-in-depth)

Research-team, upon receipt of a delivery, performs an **independent Rule #75 audit as the first step of L1–L4 verification**. Any Rule #75 item missed at the dev side and discovered at the research side = **Rule #75 failure logged**. Dev-team must remediate before research-team continues L1–L4.

### §75.4.3 No silent skip

Under no circumstances may a delivery proceed with a known missing Rule #75 check. A Plan that Dev believes is complete but that omits a check must first attest the check's non-applicability **with written rationale** in the delivery report; research-team's independent audit validates or rejects the non-applicability claim.

---

## §75.5 Interaction with existing rules

| Existing rule / process | Interaction | Relationship |
|-------------------------|-------------|--------------|
| **Rule #74 (L1' code + doc sync)** | Rule #75 is a **dev-layer block BEFORE** Rule #74 L1' verification at research side. Rule #75 ensures L1'-ready state; Rule #74 verifies it. | Complementary, no overlap |
| **Rule #63 (L4 assessment)** | Rule #75 occurs before L4 verification. L4 applies to research-side verification **after** R → Dev transfer. | Serial, no overlap |
| **ENG-FAB v1.7 F-10 (Pre-Delivery Gate Verified)** | F-10 is the **test-layer re-check** of Rule #75 (dev-layer gate). Dual-gate defense-in-depth. F-9 remains dev-layer doc-check; F-10 is test-layer gate verification. | Companion gate |
| **MR-10 (造假/錯誤/未完成補回)** | Rule #75 formalises the pre-delivery discipline that historically **reduced but did not eliminate** the fabrication pattern (Plan36–41, 6 consecutive cycles). Rule #75 is the process-level preventive control that MR-10 has long called for. | Structural upgrade |
| **ENG-FAB F-6 (delivery-report completeness)** | Rule #75 failure **automatically triggers** F-6 violation (delivery report is incomplete if pre-delivery gate attestation is absent). | Failure cascade |
| **SCRIBE G5 process gate** | Research-team G5 first-pass = Rule #75 attestation completeness check; any missing section → G5 FAIL, R1 cannot begin. | G5 extension |
| **SCRIBE G6.8** (new, see `Calibration_Reports/18_PathC_20Field_Monitoring_Format.md` §7) | Parallel SCRIBE-authority gate verifying Pre-Delivery Gate alignment across all cycle-end artefacts; SCRIBE authority, not Master route. | Parallel sister-gate |

---

## §75.6 Audit and enforcement

### §75.6.1 Delivery-report structure

Dev delivery report includes a new **Section 0: Pre-Delivery Gate Attestation** with §75.3.1 through §75.3.5 each checkmarked with evidence references. Example header:

```markdown
# Plan48 Delivery Report

## Section 0 — Rule #75 Pre-Delivery Gate Attestation (Dev)

- [x] §75.3.1 MUST-item sub-check coverage — see Appendix A (25 atomic acceptance criteria)
- [x] §75.3.2 Binding acceptance-criteria satisfaction — Plan48 25-sub-item table; Plan47 carry-forward: none
- [x] §75.3.3 ENG-FAB v1.7 audit PASS — see Appendix B (43-item item-by-item)
- [x] §75.3.4 δ-status — Plan48 closes δ (F-L3-SEC-2 + F-L3-SEC-3 + E-5 MUST)
- [x] §75.3.5 Runtime evidence — W2-R13 evidence artefact paths per sub-item (Appendix C)

Dev-team attestation: [signer], [timestamp]
```

### §75.6.2 SCRIBE compliance log

SCRIBE maintains a Rule #75 compliance log at `research record/<cycle>/discussions/SCRIBE_rule_75_log.md`. Each Plan delivery gets a row; the log is cross-referenced in R3 and R4 deliverables and in the Master Ratification Request package.

### §75.6.3 G5 process-gate extension

Research-team **G5 first-pass** adds a Rule #75 attestation completeness check. Any missing §75.3.x section → **G5 FAIL**; R1 cannot begin; coordinator alerts dev-team for immediate remediation.

---

## §75.7 Examples and non-examples

### §75.7.1 Example — Plan47 retrospective compliance (no formal Rule #75 attestation but equivalent coverage)

Plan47's delivery report (`engineering_delivery/cycle03-11_plan47/delivery_report.md`) includes:
- 5 MUST × 5 sub-check matrix → satisfies §75.3.1 (≥ 5 sub-checks per MUST).
- Explicit reference to Master 2026-04-18 binding constraints (HMAC + nonce + MR-6 Core zero + single-machine-only) → satisfies §75.3.2.
- ENG-FAB v1.6 42-item audit (35 PASS / 5 N/A / 1 INFO / 1 PARTIAL / 0 FAIL, all MUST PASS) → satisfies §75.3.3.
- δ partial carry-forward declared (F-L3-SEC-2/3 → Plan48) → satisfies §75.3.4.
- Runtime evidence via `pnpm test` 2613 passed + W2-R12 test_data → satisfies §75.3.5.

**Retrospective verdict**: Plan47 would PASS Rule #75 if it were in force at Plan47 delivery. Grandfather clause applies.

### §75.7.2 Non-example — hypothetical Plan-X with missing δ-status

A delivery that silently **carries** F-L3-SEC-2 forward without declaring it would fail §75.3.4. Even if F-L3-SEC-2 is not explicitly in the Plan's scope, failure to declare "δ-neutral" is itself a §75.3.4 violation. This prevents the six-cycle fabrication pattern's classic failure mode of phantom carry-forward.

### §75.7.3 Non-example — MUST without ≥ 5 sub-checks

A Plan whose MUST items each have only 2–3 sub-checks fails §75.3.1. Dev must expand the acceptance-criteria document before attestation; no short-circuit permitted.

---

## §75.8 Author attribution and voting history

- **Original proposal**: §1 R2 author panel (DARWIN + SYNTHESIST + KNUTH) — explicit §1:690–702 wording.
- **Refinement**: §5 R2 §12.2 (generalising F-03-12-§5-R2-1 HIGH sub-item binding finding into a permanent rule).
- **Strong support**: §1 R2 §5.3 KNUTH (Rule #75 dev-layer block vs F-10 test-layer catch: O(1) dev fix vs O(delivery → rollback) cost).
- **Drafted by**: §1 R2 author panel + ARCHIMEDES (LOC / delivery-report-structure impact attestation) + SYNTHESIST consolidation.

- **Expected R3 vote** (at drafting): ≥ 21/24 endorsement (dev-side efficiency + research-side preventive).
- **R3 ratified vote** (cycle 03-12 Day 5): upgraded from optional CR to binding ratification candidate; batched with Batch 8c.

## §75.9 Dissent

MR-11 dissent preservation:
- **Lighter-touch alternative considered** — annual "pre-delivery checklist" without formal rule. Rejected because a non-rule checklist lacks ENG-FAB F-6 / G5 FAIL cascade and cannot block transfer.
- **Heavier-enforcement alternative considered** — automated pre-flight CI gate with Dev repository hooks. Rejected because infrastructural scope exceeds Plan48's boundaries; may be revisited in a future Plan (Plan51+) as a Rule #75 v2 enhancement.

## §75.10 Status and ratification path

- **Status**: CANDIDATE — Batch 8c.
- **Interim guidance**: Plan48 dev delivery SHOULD attest Rule #75 as a forward-looking best practice even before Master ratification; this provides a no-risk pilot.
- **Effective date on ratification**: Plan48 delivery (first post-ratification).
- **Cross-refs**: `Research_Methodology/08_Rule_72_N10_SubClause_Draft.md` · `Research_Methodology/10_ENG_FAB_v1.7_Candidate.md` · `Calibration_Reports/19_Plan48_25_SubItems_Binding.md`.
