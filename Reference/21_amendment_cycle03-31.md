---
title: Reference/21 v0.2.1 → v0.2.2 Amendment (cycle 03-31; C-C8 NEW counter + §3.9 Phase 7 evidence-tier ladder canonical)
date: 2026-05-13
cycle: 03-31
authors: SCRIBE (chapter lead) + LINNAEUS (counter taxonomy) + R4 doc subagent
status: BINDING (ratified per cycle 03-31 R3 §5.5 D-§A6.5 BINDING + cycle 03-31 R3 §4 D-§A5a-f wording sweep ratifications; pending Master Ratification Batch 26 forward dispatch)
authority:
  - cycle 03-31 R3 §5.5 D-§A6.5 (NEW C-C8 counter BINDING; Phase 7 T4 cycle count = 1 inaugural per Stream 1 cell-1 tier-4 PASS)
  - cycle 03-31 R3 §3 D-§A4 (Reference/20 v3 BINDING — paired Reference cross-ref)
  - cycle 03-31 R3 §1.1 D-§roll_forward ρ-1 cell-individual reading BINDING (Reference/20 v3 §12 codify; this amendment §3.9 codifies the evidence-tier ladder hierarchy that anchors the C-C8 counter)
  - cycle 03-31 R3 §4 D-§A5a-f wording sweep ratifications (6 motions BINDING; this amendment forward-binds the sweep)
  - Master Ratification Batch 26 forward dispatch (cycle 03-31 R4 close target)
supersedes: none (additive amendment to Reference/21 v0.2.1; v0.2.1 substance preserved verbatim per MR-12 forward-only)
cross_refs:
  - openstarry_doc/Reference/21_Counter_Registry.md (v0.2.1 baseline; §9 + §3.8 carryover; this amendment adds §10 C-C8 + §3.9 evidence-tier ladder + §4.1 cycle 03-31 close snapshot update + §9.x amendment trigger ref)
  - openstarry_doc/Reference/20_Audit_Verdict_Format_Codification_v3.md (paired Reference; §11.4 tier-4 canonical + §12.4 LEIBNIZ N=2 observation window CLOSED status)
  - openstarry_doc/Reference/19_Audit_Methodology_Codification_v2.md (M1 BINDING; methodology evidence-tier anchor)
  - openstarry_doc/Reference/16_R4_Folder_Convention_Discipline_v3.md (G4-folder-3 + G5 sync rules)
  - claude research/research record/cycle03-31/R3/R3_decision_log.md §1.1 + §3 + §4 + §5.5 (cycle 03-31 R3 BINDING ratifications)
mr_zt_refs: [MR-2, MR-7, MR-11, MR-12, ZT-1, ZT-2, ZT-3, BG-1, BG-4]
---

# Reference/21 v0.2.1 → v0.2.2 Amendment (cycle 03-31)

## §1 Headline

This amendment adds **3 substance additions** to Reference/21 v0.2.1 per cycle 03-31 R3 BINDING ratifications:

1. **NEW C-C8 cardinality counter** (§10 below): Phase 7 T4 cycle count = 1 inaugural per cycle 03-31 Stream 1 cell-1 tier-4 PASS.
2. **NEW §3.9 Phase 7 evidence-tier ladder canonical** (4-tier: text-only / designed / inline-contract real-LLM / full-daemon real-LLM).
3. **§4.1 Counter snapshot update** to cycle 03-31 close baseline (including C-LEIBNIZ-N2-observation 2 → 0 transition + C-C8 inaugural value 1).

v0.2.1 §1-§9 + §3.1-§3.8 substance preserved verbatim per MR-12 forward-only. This amendment is **additive only**.

---

## §2 §10 NEW C-C8 Counter Definition

### §10 C-C8 Phase 7 T4 cycle count

| Field | Value |
|-------|-------|
| **ID** | C-C8 |
| **Name** | Phase 7 tier-4 (full-daemon real-LLM) cycle count |
| **Type** | Cardinality counter (cycles where ≥1 tier-4 verification ran successfully) |
| **Definition** | Cycles where Phase 7 tier-4 evidence-tier (full-daemon real-LLM via `pnpm install` + `apps/runner/bin.js daemon start`) PASS for ≥1 cell |
| **Source of truth** | Cycle R4 close per-cell `verdict.yaml` `evidence.tier4_full_daemon: PASS` attestation + Master_Confirmation Batch tier-4 evidence-tier section |
| **Update trigger** | Cycle R4 close + ≥1 cell tier-4 PASS attested → +1 |
| **Reset condition** | N/A (cardinality counter; forward-only per MR-12) |
| **Current value (cycle 03-31 close)** | **1** (inaugural per cycle 03-31 Stream 1 cell-1 tier-4 PASS; inline-replication caveat REMOVED) |
| **First established** | cycle 03-31 (per R3 §5.5 D-§A6.5 BINDING ratification 2026-05-13) |
| **Disambiguation** | 與 C-C7 (subagent permanent-mode round) 不同：本 counter 為 evidence-tier 而非 process mode；與 C-C4 (§75.X release-tag-level) 不同：本 counter 為 cycle-level tier-4 attestation；與 C-LEIBNIZ-N2-observation (§9) 不同：本 counter 為 forward-only cardinality 而非 state membership |

**Cross-references**:
- Reference/20 v3 §11.4 tier-4 canonical (FP-permissible when tier-3 + tier-4 PASS; same evidence-tier ladder category)
- Reference/21 v0.2.2 §3.9 (evidence-tier ladder canonical hierarchy)
- cycle 03-31 R3 §5.5 D-§A6.5 motion text + ratification vote
- Master_Confirmation Batch 26 forward dispatch entry (Item TBD post-R4 G4 close)

---

## §3 §3.9 Phase 7 Evidence-Tier Ladder Canonical (NEW)

### §3.9.1 4-tier evidence ladder (BINDING from cycle 03-31 R4 close)

Per cycle 03-31 R3 §5.5 D-§A6.5 BINDING ratification, the Phase 7 evidence-tier ladder canonical hierarchy is codified:

| Tier | Name | Method | Authority | First exercised |
|:----:|------|--------|-----------|-----------------|
| **Tier 1** | text-only | Static doc review + grep + manual spec read | Pre-Phase-7 baseline (W2-R36 deferral path per Master directive 2026-05-08-b) | cycle 02 era baseline |
| **Tier 2** | designed | Test harness designed (test code committed) but real-LLM execution NOT run; spec + implementation review only | Test team R-team co-design (cycle 03-25 era) | cycle 03-25 |
| **Tier 3** | inline-contract real-LLM | Harness mirrors plugin spawn pattern verbatim from src; real-LLM invoked; behavioural axis evidence captured at inline-contract granularity | cycle 03-25 R3 D-§3 Hybrid B+A activation; cycle 03-26 R3 D-§A.2 MR-7 audit-context precedent | cycle 03-30 (A1-1 + A1-3 + A1-5 FP per cycle 03-30 R3) |
| **Tier 4** | full-daemon real-LLM | Full daemon runtime via `pnpm install` + `apps/runner/bin.js daemon start`; multi-host harness option; complete behavioural attestation across all Tenet axes for cell scope | cycle 03-31 R3 §5.5 D-§A6.5 BINDING | **cycle 03-31** (Stream 1 cell-1 tier-4 PASS; C-C8 inaugural value 1) |

### §3.9.2 Tier promotion semantics

- **Tier 1 → Tier 2**: harness design + test code commit; no live LLM execution required
- **Tier 2 → Tier 3**: inline-contract real-LLM run; harness mirrors src spawn pattern; FP-permissible for behavioural Tenet axes per Reference/20 v3 §11.3
- **Tier 3 → Tier 4**: full-daemon real-LLM verified (`pnpm install` + daemon start); complete behavioural attestation; FP-permissible for all Tenet axes per Reference/20 v3 §11.4

**Tier inheritance** (per Reference/20 v3 §11.4): Tier-4 evidence supplements Tier-3 record; partial-tier-4 annotation belongs in `notes:` (NOT `caveats[]`; same category as tier-3 ladder annotation).

### §3.9.3 Cycle 03-31 inaugural tier-4 cycle attestation

Per cycle 03-31 R3 §5.5 D-§A6.5 BINDING:
- **Stream 1 cell-1**: tier-4 PASS (first ever Phase 7 tier-4 evidence-tier cycle)
- **Inline-replication caveat REMOVED** from active CP roster (tier-4 evidence supersedes the cycle 03-30 tier-3 inline-replication caveat)
- **C-C8 inaugural value = 1**
- **Forward-binding cycle 03-32+**: tier-4 evidence-tier becomes the default target for Phase 7 R-input cells; tier-3 inline-contract remains valid fallback per Reference/20 v3 §11.3 + §11.4 ladder discipline

---

## §4 §4.1 Cycle 03-31 Close Counter Snapshot

(Per Reference/21 v0.2.1 §4.1 per-cycle counter snapshot protocol; this update extends the existing protocol with cycle 03-31 close values.)

```
### Cycle 03-31 Counter Snapshot (per Reference/21 v0.2.2)

# Streak counters (C-S*)
C-S1 = 19 (was 18; +1; 19 consecutive HEALTHY HISTORIC longest cycle 03-13~31)
C-S2 = 11 (was 10; +1; 6-mirror byte-identical sustained)
C-S3 = 11 (was 10; +1; amendment_cycle03-20 md5 preservation; md5 7ffd0f79... unchanged 11 cycles)
C-S4 = 10 (was 9; +1; F1/F2/F3 Popperian counter zero milestone 10-cycle; 30/30 zero candidate)
C-S5 = 10 (was 9; +1; 9-datapoint pool sustained)
C-S6 = 19 (was 18; +1; R3 0/2 correction rounds; same trajectory as C-S1)
C-S1-roll12 = 12/12 = 100% (rolling 12-cycle HEALTHY rate sustained)

# Cardinality counters (C-C*)
C-C1 = 13 (was 12; +1; 4 防線 N-th enforce; cycle 03-19~31)
C-C2 = 11 (was 10; +1; Reference/16 amendment N-th cycle apply)
C-C3 = 10 STRENGTHENED (was 9; +1; 5-point isolation v2 sustained; W2-R42 cycle 03-31)
C-C4 = 22 (was 21; +1; v0.57.7-alpha; Rule #75 §75.X N-th enforce canonical)
C-C4a = 16 (was 15; +1; §75.X per-cycle enforce)
C-C4b = 4 sustained (no spot-check change cycle 03-31)
C-C5 = 7 (was 6; +1; Rule #72 SPC re-cal N-th cycle application)
C-C6 = 13 (was 12; +1; /simplify N-th organic apply per Dev v0.57.7-alpha cycle 03-31)
C-C7 = 15 (was 14; +1; Subagent permanent-mode N-th round; cycle 03-15~31)
C-C8 = 1 NEW INAUGURAL (Phase 7 T4 cycle count; per cycle 03-31 R3 §5.5 D-§A6.5 BINDING; Stream 1 cell-1 tier-4 PASS)

# Cumulative counters (C-U*)
C-U1 = 171 (was 162; +9; GREEN cells aggregate = 19 × 9; HISTORIC longest)
C-U2 = 61 sustained (60 carryover + 0 NEW DSS-CY31; first 0-NEW-DSS R3 since cycle 03-24 per cycle 03-31 R3 32 sub-deliberations 100% UNANIMOUS HISTORIC)
C-U3 = 12 (was 11; +1; cycle 03-31 NEW Master directives ratified; subject to active sunset mechanism)
C-U4 = 348 (was 347; +1; Reference/20 v3 NEW canonical doc)

# State counters (C-T*)
C-T1 = 7/7 完工 sustained (Phase 6 completion preserved; HISTORIC)
C-T2 = 0 / 0 sustained (σ counter; D-14 deactivated)
C-T3 = 18 sustained (cycle 收尾 N 條; awaiting reform consolidate)
C-T4 = 7 sustained (Replay cache N-contributor; Phase 6 完工 final)

# Status enums (C-E*)
C-E1 = #1-#10 all COMPLIANT sustained (canonical Tenet status; README §十大核心宣言 sole authority)
C-E2 = 10/0/0★ FINAL substance preserved (endpoint; cycle 03-24 HISTORIC inheritance)
C-E3 = per-plan (Plan53 SUNSET / Plan55 DEFERRED FINAL / Plan60 BINDING; cycle 03-31 NO Plan status change)
C-E4 = SHOULD initial FINAL sustained (F-16 status)
C-E5 = DEACTIVATED sustained (D-14 trigger; cycle 03-20 W2-R24 baseline reset 11 cycles sustained)

# Sequence counters (C-Q*)
C-Q1 = 26 (was 25; +1; Master Ratification Batch 26 cycle 03-31 forward dispatch ~22-25 items)
C-Q2 = 42 (was 40; +2; W2-Rn round number; cycle 03-31 = W2-R42)
C-Q3 = 03-31 (closed); 03-32 in pipeline
C-Q4 = v0.57.7-alpha (cycle 03-31 final; doc-only ceremonial + R-AMEND batch + canonical Reference/20 v3 deploy)

# State counters — Phase 7 specific (§9 + §10)
C-LEIBNIZ-N2-observation = 0 cells (was 2 cells cycle 03-30; both p10_t6 + p13_t6 Option α activation transitioned FP cycle 03-31; observation window CLOSED; DSS-CY30-§A1.4-P2-α preserved per MR-11 as forward-watch historical anchor)
C-C8 = 1 INAUGURAL (Phase 7 T4 cycle count; per §10 above)

# Phase 7 closure progress
Tenet#6 closure = 3/46 sustained (cycle 03-30 baseline preserved)
Tenet#8 closure = 2/38 INAUGURAL (cycle 03-31 first-subset 2 NEW FP)
Phase 7 total = 7/94 (was 3/94 cycle 03-30; +4: 2 Tenet#8 FP + 2 Tenet#6 Option α transitions; HISTORIC JUMP per cycle 03-31 R3 §1.1)
```

---

## §5 §9.x Amendment Trigger Reference

This amendment (v0.2.1 → v0.2.2) was triggered by:
- **cycle 03-31 R3 §5.5 D-§A6.5 BINDING**: NEW C-C8 counter + Phase 7 evidence-tier ladder canonical
- **cycle 03-31 R3 §1.1 D-§roll_forward ρ-1 BINDING**: roll_forward strict reading codified Reference/20 v3 §12 (paired Reference cross-ref)
- **cycle 03-31 R3 §3 D-§A4 BINDING**: Reference/20 v3 deploy (paired Reference; this amendment §3.9 evidence-tier ladder canonical anchors tier-4 attestation per Reference/20 v3 §11.4)
- **cycle 03-31 R3 §4 D-§A5a-f BINDING**: 6 R-AMEND wording sweep motions (D-§A5a Plan-numbering + D-§A5b manas SDK + D-§A5c F6 scope + D-§A5d maxTurns + D-§A5e Plan59 §6.3 title + D-§A5f Plan28 volition Plan-citation extension) — these flow forward via paired Plan/Reference docs; this amendment reaffirms forward-binding alignment

---

## §6 Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 (Tenet wording unchanged) | ✅ PASS | This amendment = counter taxonomy + evidence-tier ladder canonical; no Tenet wording mutation |
| MR-5 (Tenet #10 status unchanged) | ✅ PASS | COMPLIANT preserved (cycle 03-24 HISTORIC inheritance) |
| MR-7 (R3 ratification) | ✅ PASS | cycle 03-31 R3 §5.5 D-§A6.5 BINDING + §1.1 D-§roll_forward ρ-1 BINDING + §3 D-§A4 BINDING + §4 D-§A5a-f BINDING |
| MR-11 (dissent preservation) | ✅ PASS | 0 NEW DSS cycle 03-31 (HISTORIC 32 sub-deliberations 100% UNANIMOUS); DSS-CY30-§A1.4-P2-α LEIBNIZ + KNUTH carryover sustained as forward-watch historical anchor per observation window CLOSED |
| MR-12 (forward-only) | ✅ PASS | v0.2.1 §1-§9 + §3.1-§3.8 substance preserved verbatim; this amendment adds §10 + §3.9 + §4.1 update only |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | No Tenet rewrite; endpoint 10/0/0★ FINAL preserved; control-range 加嚴 only (NEW counter + NEW tier-ladder canonical adds granular tracking, never relaxes) |
| BG-1 (honest disclosure) | ✅ PASS | C-LEIBNIZ-N2-observation 2 → 0 transition explicitly documented; DSS preservation explicit |
| BG-4 (incremental refinement bounded + reversible) | ✅ PASS | NEW additions scoped tight (C-C8 single row + §3.9 4-tier table + §4.1 snapshot update); reversible via amendment archive |

---

## §7 Coordinator G5 Sync Note

This amendment file lives at suggestion location:
- `claude research/research record/cycle03-31/openstarry_doc/Reference/21_amendment_cycle03-31.md`

Coordinator G5 post-R4 sync target paths:
- `openstarry_eco/share/openstarry_doc/Reference/21_amendment_cycle03-31.md` (canonical authority)
- `openstarry_eco/agent_dev/openstarry_doc/Reference/21_amendment_cycle03-31.md` (Dev sync target)
- `openstarry_eco/release/cycle03-31_v0.57.7-alpha/openstarry_doc/Reference/21_amendment_cycle03-31.md` (per release snapshot)
- `test research/cycle03-31/codebase_v0.57.7-alpha/openstarry_doc/Reference/21_amendment_cycle03-31.md` (test codebase)
- `claude research/research input/research_version/cycle03-31_v0.57.7-alpha/openstarry_doc/Reference/21_amendment_cycle03-31.md` (R-team input snapshot)
- `test research/cycle03-31/research_reference/Reference/21_amendment_cycle03-31.md` (R-team reference flat copy)

**4-mirror byte-identical verification** at G5 sync; **G5-sync-5 BINDING canonical doc creation check** (Master directive 2026-05-01 防線 4) applies to this amendment.

**TW sibling** (`21_amendment_cycle03-31.tw.md`): deferred to cycle 03-32 R-AMEND per Rule #78 §78.5 same-PR coordinator G5 sync pattern (post-Master Ratification Batch 26 deploy).

---

*Reference/21 v0.2.1 → v0.2.2 Amendment — cycle 03-31 R3 §5.5 D-§A6.5 BINDING + §1.1 D-§roll_forward ρ-1 BINDING + §3 D-§A4 BINDING + §4 D-§A5a-f BINDING — 2026-05-13*
*3 substance additions: §10 C-C8 NEW cardinality counter + §3.9 Phase 7 evidence-tier ladder canonical + §4.1 cycle 03-31 close snapshot update*
*v0.2.1 §1-§9 + §3.1-§3.8 substance preserved verbatim per MR-12 forward-only*
*Master Ratification Batch 26 forward dispatch target post-R4 close 2026-05-13*
