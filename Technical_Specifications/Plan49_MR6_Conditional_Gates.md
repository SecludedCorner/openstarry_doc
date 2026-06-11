# Plan49 — MR-6 Conditional Gates, Scope Knowledge for Dev

> **Front-matter**
> - **Cycle**: 03-13
> - **Date**: 2026-04-24
> - **Authors**: ARCHIMEDES (#16, Plan49 engineering spec owner) + SCRIBE (#2, scope discipline + dissent preservation) + SYNTHESIST (#1, aggregation) · KERNEL (#10) + TANENBAUM (#20) + GUARDIAN (#11) contributing MR-6 Core-import-surface review
> - **Binding R3 decisions**: **D-10 18/5** (StateTracker Path B default / 3-channel zero check; 5 dissent ATHENA/KERNEL/ARCHIMEDES/PASCAL/RUSSELL for Path A preserved) · **D-11 UNANIMOUS 23/0** (Task #50 absorption = O3 + O4 composite; WIENER dissent on O1 rename preserved separately via MR-11 D-01) · **D-12 UNANIMOUS 23/0** (schema-drift SHOULD per-site + process-global C49-M3g MESH) · **D-13 20/3** (C49-M5g MUST-unconditional; 3 dissent TANENBAUM/SUSSMAN/RUSSELL preserved) · **D-14 UNANIMOUS 23/0** (5 LOC-contingency options incl. (v) F-15 tooling scaffold) · **MRB-03/04/05/06/07 RESOLVED** (v1.7 baseline / LOC band / sub-item count / call-sites / H6-H8)
> - **Status**: **KNOWLEDGE ONLY — not ratified as spec**. Master Batch 10 **Item 9** "APPROVE Plan49 scope ratification with 2 conditional MR-6 gates" — the scope is Master-approved as a Plan (Dev executes), but this `openstarry_doc/` file is **consultative/knowledge** for canonical reference (per Master instruction "知會即可"). The authoritative Plan49 implementation spec lives at `research record/cycle03-13/deliver/O2_plan49_engineering_spec.md` + `research record/cycle03-13/todo/Plan49_dev_spec.md`.
> - **Scope boundary**: This document provides canonical-doc knowledge for Dev teams consuming `openstarry_doc/` as reference. It is NOT a ratified binding spec; the binding Plan49 spec is in `deliver/O2_plan49_engineering_spec.md` and `todo/Plan49_dev_spec.md`.
> - **Cross-refs**: `Research_Methodology/11_ENG_FAB_v1.8_Binding.md` (v1.7 freeze for Plan49; v1.8 for Plan50+) · `Reference/09_Rule_77_SPC_Pooled_Mode_Sigma_Regime.md` (Rule #72 FR-2 amendment batched via Batch 10 #8) · `Technical_Specifications/Plan50_Sigma_Regime_Binding.md` (Plan49 → Plan50 sequencing) · `research record/cycle03-13/deliver/O2_plan49_engineering_spec.md` (full engineering spec, 1033 lines) · `research record/cycle03-13/todo/Plan49_dev_spec.md` (Dev-facing spec, 323 lines) · `research record/cycle03-13/deliver/R3_decision_log.md §4.2 + §4.5` (D-10/11/12/13/14 + MRB-03 through 07)

---

## §1 Plan49 Scope Overview

### §1.1 LOC band and Wave structure (MRB-04 adopted)

Single band adopted per R3 MRB-04 resolution (R2 F-03-13-§2-R2-01 reconciliation):

- **Prod LOC**: 308–555.
- **Test LOC**: 170–290.
- **Combined**: 478–845.
- **S-1 compliance**: well within the S-1 500–1000 LOC single-Plan guidance band.
- **Waves**: 3-Wave W0 / W1 / W2 dependency structure per CV-15 UNANIMOUS.

### §1.2 Sub-item canonical count (MRB-05 adopted)

**36 sub-items** distributed across **7 scope families + 1 discipline family**, per R2 F-03-13-§2-R2-02 reconciliation (R1 28 sub-items + R2 additions C49-M4-iii / C49-M5g / C49-M6-c.3 / C49-M7e / C49-M8 family = 5 additions). The 36-count is canonical; 28-count is R1 pre-reconciliation.

### §1.3 ENG-FAB baseline (v1.7 freeze per CV-16)

Plan49 is assessed under **ENG-FAB v1.7 baseline** (42 items + F-10). Rationale (CV-16 UNANIMOUS):
- Avoids mid-cycle retrofit (v1.8 would require re-auditing Plan49 scope from scratch).
- Plan50+ adopts v1.8 per D-21 timing table.
- **Exception**: F-12 tooling track delivers `audit_calc.py` in Plan49 as a Dev deliverable (per D-21 UNANIMOUS + `Research_Methodology/11_ENG_FAB_v1.8_Binding.md §H`); F-15 Plan49 immediate (governs Plan49's research acceptance report per F-15 effective-immediate status).

---

## §2 MR-6 Conditional Gates (2)

Plan49 carries **2 conditional MR-6 gates** that must be resolved at delivery time per Master Batch 10 Item 9 acknowledgement.

### §2.1 C49-M4 — Purity-Flag Path iii Deferral

**Gate**: C49-M4 "Core purity verification, path iii deferral" is **conditional**. Plan49 delivers paths (i) and (ii) of the Core purity predicate; path (iii) is **deferred to Plan50+** per engineering spec with **documented rationale** in delivery report.

**MR-6 compliance**: Core zero-policy constraint preserved. Plan49 does **not** add Core policy constants; paths (i) and (ii) (Core import surface + static-rule-arbiter isolation) are maintained. Path (iii) (deeper Core-boundary audit under multi-agent scenarios) requires Phase 6+ architecture first and is appropriately deferred.

**Delivery check**: dev delivery report MUST explicitly declare "C49-M4 path (iii) deferred to Plan50+ per Plan49 spec §X.Y" with justification. Research verification audits the deferral rationale.

### §2.2 C49-M5 — Core-Import-Surface Delta

**Gate**: C49-M5 "Core-import-surface delta" is **conditional** on Plan49 W1 final implementation. The predicted Core-import delta is **zero policy additions** (Plan49 adds no policy constants to Core); the delta may include **transport-layer additions** (plumbing for σ_regime tagging per Plan50 anticipation + C49-M5g per §3 below).

**MR-6 compliance**: 53 Core policy constants baseline (KERNEL itemization) preserved; Plan49 adds **zero** policy constants. Transport additions are MR-6-compatible (plumbing without policy).

**Delivery check**: Research verifies `packages/core/**` post-Plan49 against baseline via `grep` — delta must be zero policy constants. Transport additions separately enumerated and confirmed non-policy.

---

## §3 StateTracker Path B Default (D-10 18/5)

### §3.1 R3 D-10 decision

R3 D-10 (18/5 — flipped from R1 Path A recommendation) adopts **StateTracker Path B default** with **3-channel zero check** requirement (R2 risk-asymmetry argument adopted).

### §3.2 Path B specification

**Path B** = StateTracker retains existing hooks; new hooks added in additive mode; hook removal is **opt-in per-plugin** (not default-remove).

**3-channel zero check** (R2 refinement): on StateTracker deactivation of any hook, verify all three of:
- **Channel 1 — hookMap zero**: `hookMap.size()` returns 0 for the deactivated hook.
- **Channel 2 — dispatcher zero**: no pending dispatch entries for the deactivated hook.
- **Channel 3 — plugin-local zero**: plugin-reported hook-registration tally matches core-observed tally (zero inconsistency).

All 3 channels must report zero before deactivation is confirmed complete; otherwise deactivation is rolled back and error reported.

### §3.3 Path A reserved for 3-channel-zero confirmed case

Path A (default-remove) is **reserved** only for scenarios where the 3-channel zero check has already passed externally (e.g., whole-plugin-unload at startup; not incremental hook changes).

### §3.4 Dissent preservation (D-10 5 dissenters, MR-11)

5 agents preferred **Path A default** (more aggressive cleanup, simpler semantics):
- **ATHENA (#5)** operational: Path A simpler for dev team to reason about; Path B adds runtime-check overhead.
- **KERNEL (#10)** microkernel discipline: Path A aligns with monotone hook-lifecycle model.
- **ARCHIMEDES (#16)** engineering: Path A reduces LOC ~20–40; Plan49 budget relief.
- **PASCAL (#19)** decision theory: Path A is the Bayesian-prior-risk-lower option — default-remove + rollback if error is simpler under low-probability-error priors.
- **RUSSELL (#23)** agent theory: Path A is more honest about state-transitions; Path B hides transitions behind multi-channel verification.

**Majority (18) rationale**: risk-asymmetry favors Path B — Path A's default-remove has asymmetric downside (unrecoverable state loss if channels 2/3 not yet zero); Path B's runtime overhead is predictable and bounded. R2 risk-asymmetry calculation adopted via R3 vote.

---

## §4 C49-M5g MUST-Unconditional (D-13 20/3)

### §4.1 R3 D-13 decision

R3 D-13 (20/3) adopts **C49-M5g MUST-unconditional** (NOT SHOULD-conditional per R1 v1.7-transition hedge).

**C49-M5g** = StateTracker Path B 3-channel zero check enforcement per Plan49 delivery. MUST means dev delivery fails PA if C49-M5g not implemented; unconditional means no transition-window exemption.

### §4.2 Rationale

Per §四 σ-deterministic finding (cycle 03-13 R1/R2/R3), FR-2 pooled-mode rationale that supported SHOULD-conditional has been dropped — the "v1.7 → v1.8 transition aggressive" concern was predicated on σ-stochastic assumptions that no longer hold. C49-M5g MUST-unconditional is therefore the correct binding.

### §4.3 Dissent preservation (D-13 3 dissenters, MR-11)

3 agents preferred SHOULD-conditional:
- **TANENBAUM (#20)** microkernel: SHOULD-conditional allows incremental rollout; MUST risks Plan49 ship-blocking on edge-case StateTracker scenarios.
- **SUSSMAN (#22)** structure: SHOULD-conditional preserves plugin-flexibility; MUST locks StateTracker interaction pattern.
- **RUSSELL (#23)** agent theory: SHOULD-conditional allows agent self-configuration; MUST imposes framework-level uniformity.

**Majority (20) rationale**: §四 finding makes SHOULD-conditional rationale obsolete; MUST-unconditional matches governance integrity. D-13 3 dissent entries preserved per MR-11.

---

## §5 Task #50 Absorption — O3 + O4 Composite (D-11 UNANIMOUS)

### §5.1 R3 D-11 decision

R3 D-11 UNANIMOUS adopts **O3 + O4 composite** as Plan49's resolution of Task #50 scope. `gear-arbiter-dynamic` identity issue is addressed via:

- **O3**: Plan49 W1 +8 LOC `riskCategory` field added to `audit:completed` event emission path (cat=`wiener_gear_*` instead of cat=`undefined`).
- **O4**: doc clarify — canonical `openstarry_doc/Architecture/PLUGIN_GEAR_ARBITERS.md` states the plugin's identity + priority rationale + by-design null-output contract.

### §5.2 Plan49 integration

- O3 is **Plan49 W1 BLOCKER for §八 audit_calc Batch 2-C** per D-03 UNANIMOUS. Plan49 W1 must land before audit_calc.py Batch 2-C can merge.
- O4 is a canonical doc artefact added via coordinator 7-step sync at cycle 03-13 R4 close + onward (Doc added to `openstarry_doc/Architecture/` per §5.3 below).
- New sub-item **C49-M7e** added (R2 addition per MRB-05) for O4 doc artefact.

### §5.3 O1 rename tabled to Plan50 (D-01 20/3, 3 dissent preserved)

Per D-01 20/3, the O1 rename (`gear-arbiter-dynamic` → `gear-arbiter-wiener`) is **tabled to Plan50**, not in-Plan49. Majority rationale: Plan50 bundling allows atomic rename + dual-name support + migration guide + deprecation-warning schedule. Minority (WIENER / LEIBNIZ / TANENBAUM per D-01 dissent preservation): immediate truth-in-naming preferred. Dissent preserved per `Research_Methodology/11_ENG_FAB_v1.8_Binding.md` + `research record/cycle03-13/deliver/O3_core_trio_final.md §A.6`.

Plan50 rename scope: see `Technical_Specifications/Plan50_Sigma_Regime_Binding.md` + `research record/cycle03-13/todo/Plan50_sigma_regime_spec.md §7` (may bundle with σ_regime or stand alone per Master decision).

---

## §6 Schema-Drift — 8 Call-Sites + Process-Global C49-M3g (MRB-06 + D-12)

### §6.1 MRB-06 8-call-site enumeration

Per TURING + §2 authors enumeration at W1 kickoff (MRB-06 RESOLVED):

| Call-site | Count | Location |
|-----------|:-----:|----------|
| event-bus check | 3 | bus subscribers |
| checkpoint-store | 2 | persist + restore |
| IAgentConfig | 1 | config-consume |
| plugin-loader | 0 (deferred) | — |
| WebSocket | 1 | WS-subscribe |
| hook-registry | 1 | register |
| **Total** | **8** | (R1 estimate "6+" confirmed as floor) |

Plan49 addresses all 8 call-sites via C49-M3a–h sub-items (per `deliver/O2_plan49_engineering_spec.md §3`).

### §6.2 Process-global C49-M3g (D-12 UNANIMOUS, MESH)

Per D-12 UNANIMOUS (MESH distributed-systems rationale), schema-drift is addressed via **SHOULD per-site + process-global C49-M3g**:

- **SHOULD per-site**: each of the 8 call-sites validates schema on receive.
- **C49-M3g (new sub-item per R2 addition)**: process-global schema-drift monitor observes schema fingerprint stability across W2 rounds; emits `schema_drift_detected` event if fingerprint shifts unexpectedly.

Rationale: per-site validation alone misses cross-call correlated drift; process-global monitor provides the distributed-systems safety net.

---

## §7 H6/H7/H8 Hypothesis Extension (MRB-07 RESOLVED, HERACLITUS + MESH)

Per MRB-07 RESOLVED, the plugin-install hypothesis list adds:

- **H6**: worker thread race (potential concurrent plugin-install race condition).
- **H7**: filesystem journaling delay (possible metadata propagation lag at install time).
- **H8**: npm hoist vs Plan49 nested-import (potential import-resolution conflict with Plan49 module structure).

Each hypothesis is enumerated in §2R1 §2.2.1 hypothesis list (per spec reference). Plan49 test suite includes assertions covering each hypothesis; Research L2 verification checks hypothesis-coverage.

---

## §8 F-15 Compliance Family C49-M8

Per R2 addition + MRB-05, Plan49 includes a **discipline family C49-M8** for F-15 compliance:

- **C49-M8** = F-15 self-attestation in Plan49 research acceptance report front-matter block.
- Plan49 research acceptance report (produced at Plan49 delivery-time cycle 03-14+) carries the F-15(a) through F-15(e) elements per `Research_Methodology/11_ENG_FAB_v1.8_Binding.md §G`.

### §8.1 F-15 (v) tooling scaffold (D-14 UNANIMOUS 5th option)

Per D-14 UNANIMOUS, LOC contingency options expanded from 4 → 5 with **option (v) F-15 tooling scaffold**: if Plan49 LOC band is exceeded, F-15 front-matter generation may be scaffolded (~5–10 LOC tooling) and the scaffolded tooling absorbed into the LOC band. Option (v) is one of 5 LOC contingencies; the specific choice is Dev discretion at implementation time.

---

## §9 v1.7 Baseline Assessment Freeze (CV-16 UNANIMOUS)

Plan49 is assessed under **ENG-FAB v1.7 (43 items) baseline** per CV-16 UNANIMOUS:
- v1.7 = 42 items (v1.6) + F-10 NEW (Pre-Delivery Gate Verified).
- v1.7 ratification via cycle 03-12 Batch 8c; baseline frozen for Plan49.
- Plan50+ adopts v1.8 per `Research_Methodology/11_ENG_FAB_v1.8_Binding.md §H` effective timing table.

### §9.1 F-12 tooling track (overrides freeze narrowly)

Per D-21 UNANIMOUS + F-12 dual-track: Dev delivers `audit_calc.py` as a **Plan49 tooling deliverable** even though F-12 delivery-audit enforcement starts at Plan50+. This narrow override ensures tooling is available for Research Tier 1 audits from 03-14 onward.

### §9.2 F-15 immediate (overrides freeze narrowly)

F-15 (Code-Path Verified Before Impact Assessment) is **immediate-effective** per D-21 + CV-16 + `Research_Methodology/11_ENG_FAB_v1.8_Binding.md §G`. Plan49 research acceptance report is subject to F-15 despite v1.7 freeze; Dev Plan49 implementation is not (F-15 governs research deliverables, not Dev deliveries).

---

## §10 Dissent Roll-Up (MR-11 preservation)

Per MR-11, Plan49-scope dissent entries preserved:

| D-ID | Dissent agents | Count | Rationale (abridged) | Preservation location |
|------|----------------|:-----:|----------------------|-----------------------|
| D-01 | WIENER / LEIBNIZ / TANENBAUM | 3 | In-Plan49 rename preferred; immediate truth-in-naming | §5.3 above + `O3_core_trio_final.md §A.6` |
| D-10 | ATHENA / KERNEL / ARCHIMEDES / PASCAL / RUSSELL | 5 | StateTracker Path A default preferred; simpler semantics + reduced overhead | §3.4 above |
| D-13 | TANENBAUM / SUSSMAN / RUSSELL | 3 | C49-M5g SHOULD-conditional preferred; v1.7→v1.8 transition less aggressive | §4.3 above |

**Plan49 direct dissent count**: 11 entries (3 + 5 + 3). Cross-refs: D-11 WIENER dissent on O1 preserved via D-01 (same 3 agents), so no double-counting.

Additional Plan49-tangent dissent (D-17 F-15(e) 3 / D-04 σ_regime 5 / etc.) is preserved in their respective chapter docs.

---

## §11 Relationship to Other Deliverables

### §11.1 Plan48 (cycle 03-12 post-delivery)

Plan48 (δ close-out / E-5 MUST / F-L3-SEC-2/3 / HMAC) shipped 2026-04-22 per cycle 03-12. W2-R13 GO verified 2026-04-23. Plan49 is the **next Plan in sequence** and does NOT revisit Plan48 scope.

### §11.2 Plan50 σ_regime (this cycle's follow-on)

Plan50 σ_regime (~50 LOC Option γ) is specced in `Technical_Specifications/Plan50_Sigma_Regime_Binding.md`. Plan50 is the follow-on Plan and bundles:
- σ_regime annotation per Rule #77 (Batch 10 Item 8).
- O1 rename (`gear-arbiter-dynamic` → `gear-arbiter-wiener`) per D-01 20/3.
- `pushInput` spec delivery per cycle 03-12 `Plan50_pushInput_CP4_Invariant.md`.

Plan49 before Plan50 per MR-12 back-fill priority (CV-14 UNANIMOUS).

### §11.3 `audit_calc.py` Batch 2-C (§八)

`audit_calc.py` Batch 2-C merge depends on Plan49 W1 per D-03 UNANIMOUS. The blocking direction is one-way: Plan49 W1 does NOT depend on audit_calc.

---

## §12 Status and Author Attribution

- **Status**: **KNOWLEDGE ONLY** in this `openstarry_doc/` canonical file. Master Batch 10 Item 9 APPROVE covers Plan49 scope as a Plan-ratification (execution authorisation), not this doc-knowledge file. The authoritative implementation spec lives at:
  - `research record/cycle03-13/deliver/O2_plan49_engineering_spec.md` (1033 lines full spec).
  - `research record/cycle03-13/todo/Plan49_dev_spec.md` (323 lines Dev-facing spec).
- **Effective (Plan execution)**: Plan49 opens at cycle 03-14 kick-off per Dev cycle planning.
- **Primary authorship**:
  - ARCHIMEDES (#16) — Plan49 engineering spec lead.
  - SCRIBE (#2) — scope discipline + MRB-03/04/05/06/07 resolution tracking.
  - SYNTHESIST (#1) — aggregation.
  - Contributing co-authors: KERNEL (#10), TANENBAUM (#20), GUARDIAN (#11), TURING (#17 — 8-call-site enumeration), HERACLITUS (#15 — H6/7/8), MESH (#4 — process-global C49-M3g).
- **Dissent agents preserved**:
  - D-01 (rename): WIENER (#12), LEIBNIZ (#14), TANENBAUM (#20).
  - D-10 (Path A preference): ATHENA (#5), KERNEL (#10), ARCHIMEDES (#16), PASCAL (#19), RUSSELL (#23).
  - D-13 (SHOULD-conditional): TANENBAUM (#20), SUSSMAN (#22), RUSSELL (#23).
- **R3 vote record** (ground truth `R3_decision_log.md §4.2 + §4.5`):
  - D-10 18/5 (Path B default).
  - D-11 UNANIMOUS (O3+O4 composite).
  - D-12 UNANIMOUS (SHOULD per-site + process-global C49-M3g).
  - D-13 20/3 (C49-M5g MUST-unconditional).
  - D-14 UNANIMOUS (5 LOC contingency options).
  - D-01 20/3 (O1 rename tabled to Plan50 — cross-ref).
  - MRB-03 RESOLVED (v1.7 baseline for Plan49).
  - MRB-04 RESOLVED (LOC band 308-555 prod + 170-290 test).
  - MRB-05 RESOLVED (36 sub-items).
  - MRB-06 RESOLVED (8 call-sites enumerated).
  - MRB-07 RESOLVED (H6/H7/H8 added).
  - CV-13/14/15/16 UNANIMOUS tacit (MR-6 Core-zero PASS / 7-family partition / 3-Wave / v1.7 freeze).

---

*Plan49 MR-6 Conditional Gates — Cycle 03-13 — 2026-04-24 R4 close*
*KNOWLEDGE ONLY for canonical `openstarry_doc/`; authoritative spec in `research record/cycle03-13/deliver/O2_plan49_engineering_spec.md`*
*Master Batch 10 Item 9 APPROVE Plan49 scope 2026-04-24*
*D-10/11/12/13/14 + MRB-03 through 07 resolved; 11 direct dissent preserved per MR-11*
