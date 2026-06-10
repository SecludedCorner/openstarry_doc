---
title: Rule #78 — TW Translation Parity for Governance Docs (L1 High-Level Policy)
author: SCRIBE (#2) + LINNAEUS (#13) + ARCHIMEDES (#16) + KERNEL (#10) + GUARDIAN (#11) + SUNYATA (#0) + SYNTHESIST (#1)
date: 2026-04-27
cycle: 03-15
status: CANDIDATE (pending Master Ratification Batch 12 #1)
authority: research-team (R4 final draft); Master (ratification)
supersedes: null
cross_refs:
  - research record/cycle03-15/deliver/O1_TW_translation_final.md §2 + §11 + §13
  - research record/cycle03-15/R3/R3_decision_log.md §4.1 + §4.2 + §4.3 (D-§2-* binding)
  - research record/cycle03-15/deliver/Master_Ratification/Batch_12_Request.md §3.1 (Item #1)
  - research record/cycle03-15/openstarry_doc/Research_Methodology/16_F_15_v3_Third_Tier_Amendment.md (L3 sibling)
  - research record/cycle03-15/openstarry_doc/Reference/12_Cycle03-13_Backfill_Classification.md (Item #7 sibling)
  - research record/cycle03-14/openstarry_doc/Research_Methodology/13_F_15_Scope_Expansion_Governance_Docs.md (cycle 03-14 v2 baseline)
  - research input/master_letters/cycle03-15/Master_letter.md §二
binding_until: Master Ratification Batch 12 close
---

# Rule #78 — TW Translation Parity for Governance Docs (L1 High-Level Policy)

## 1. Status

**CANDIDATE** at cycle 03-15 R4 close. Upgrades to **BINDING** upon Master Ratification Batch 12 Item #1. Effective **forward-only** from cycle 03-15 R3 close onward; pre-cycle-03-15 canonical openstarry_doc/, Tenets, MR, ZT, Baseline Rules, Plan specs, ENG-FAB items are **grandfathered EN-only** per MR-12 既有不破壞.

**Landing pattern**: L1+L3 hybrid per D-§2-07 (14/4/3/2). This document is the **L1 high-level policy** half (Reference/, scope × strength × timing × responsibility × quality × retroactive × CI gate). The **L3 operational mechanism** half is `Research_Methodology/16_F_15_v3_Third_Tier_Amendment.md`. Master Ratification Batch 12 Item #1 dispatches both as one bundle.

**Rule cardinality**: new Rule #78 in Reference/. Per D-§2-07 anchor unification, **ENG-FAB v1.10 F-19 candidate NOT created**; ENG-FAB v1.8 = 48 binding preserved; v1.9 = 49 candidate F-16 SHOULD initial unchanged; v1.10 numbering NOT consumed.

## 2. R3 Provenance

- **D-§2-07 (A-1)** — TW landing point = **L1+L3 hybrid** — 14 votes (super-majority over L1-only 4 / L2-F-19 3 / L3-only 2 / 0 other). DSS-A1 4 dissent preserved verbatim §10.
- **D-§2-01 (A-2)** — scope tier = **by-ratification-status** (BINDING / CANDIDATE / KNOWLEDGE_ONLY / CONSULTATIVE / SCRIBE-internal) per VITRUVIUS R2-05 — 17/4/2; DSS-A2 6 dissent preserved §10.
- **D-§2-02 (B-1)** — strength = **static (b) governance MUST + canonical SHOULD + Plan/operational MAY** (18/5; DSS-B1 5 dissent §10) + temporal escalation YES (24h grace; 19/4; DSS-B1b 4 dissent §10).
- **D-§2-03** — timing × responsibility = DEFER R4 absorb (UNANIMOUS; resolves naturally from D-§2-01 × D-§2-02).
- **D-§2-04 (B-2)** — quality = glossary YES with epistemic prefix (verified/inferred/speculative; 21/2) + structural fidelity layer-bound (22/1) + dual-author NO (5/18; DSS-B2 5 dissent §10).
- **D-§2-05 (B-3)** — retroactive = **forward-only with scheduled audit per release-tag** (16/4/3; DSS-B3 7 dissent §10) + cycle 03-13 backfill = MR-10 retroactive precedent (one-off) (14/5/3/1; DSS-B3b 9 dissent §10).
- **D-§2-06 (C-1)** — CI gate = **multi-layer** (F-15 linter + pnpm build + doc gate + G4-folder + coordinator hook) UNANIMOUS 23/0.
- **D-§2-08 (D-2)** — framing = artefact-property primary + participation-right secondary (18/3/2; DSS-D2 3 dissent §10).
- **MRB-§2-01 RESOLVED** — cycle 03-13 backfill classification (see sibling doc 12).
- **MRB-§2-02 RESOLVED** — sibling-naming convention `<basename>.<lang>.md` codified (TURING + SCRIBE).

## 3. Incident Origin (Cycle 03-13)

The originating gap: cycle 03-13 `plugin-gear-arbiters.md` shipped to canonical EN-only without TW parity. Cycle 03-14 audit #93 retroactively backfilled the TW sibling under audit pressure. Cycle 03-15 §二 closes the structural lesson via Rule #78 + F-15 v3 amendment so future similar gaps are prevented forward.

The cycle 03-13 backfill is classified separately as **MR-10 retroactive precedent (one-off)** per `Reference/12_Cycle03-13_Backfill_Classification.md` (Batch 12 Item #7). The classification is **one-off**; future bilingual gaps follow Rule #78 §78.5 forward-only + scheduled audit per release-tag.

## 4. Binding Text

### 4.1 §78.1 — Scope (By-Ratification-Status)

> TW translation parity is required (or recommended, or optional) per the *by-ratification-status* tier classification of the doc instance:
>
> - **BINDING** tier (Tenets #1-#10, Master Rulings MR-1~MR-13, Zero Tolerance ZT-1/2/3, Baseline Rules #1-#78, ratified ENG-FAB items v1.X binding) — TW parity **MUST**.
> - **CANDIDATE** tier (canonical openstarry_doc/ ratified subdir, ratified Plan specs, ratified Master Ratification batches absorbed into canonical) — TW parity **SHOULD**.
> - **KNOWLEDGE_ONLY** tier (canonical openstarry_doc/ Reference/ research notes, calibration reports, technical specifications absorbed) — TW parity **SHOULD** (same as CANDIDATE; differential is in retroactive audit scheduling per §78.6).
> - **CONSULTATIVE** tier (Plan specs in-flight, draft Master Ratification batches, R-stage drafts upgraded for next-cycle reference) — TW parity **MAY**.
> - **SCRIBE-internal** tier (G0-G5 process gate logs, GP-coherence logs, R-stage working artefacts, side-bar discussions, retrospectives, daily logs, deliver/ cycle-internal artefacts) — TW parity **MAY**.

The tier classification is **by-ratification-status**, not by-location nor by-doc-class enumeration (per D-§2-01 17/4/2; DSS-A2 dissent preserved §10). The tier of a given doc may shift across cycles if its ratification status changes (e.g. CANDIDATE → BINDING upon Master Ratification; CONSULTATIVE → CANDIDATE upon R4 close); the parity requirement updates accordingly.

### 4.2 §78.2 — Strength (Static + Temporal Escalation)

> Strength is statically stratified per §78.1 tier (governance MUST / canonical SHOULD / Plan/operational MAY) per D-§2-02 (18/5; DSS-B1 5 dissent §10). A **temporal escalation** clause applies: a doc that lands EN-only in BINDING or CANDIDATE tier acquires a **24-hour grace window** before R3+1 cycle close to acquire TW parity; failure at R3+1 cycle close = TW parity **MUST** (governance) or **SHOULD** (canonical) violation per §78.7 CI gate (D-§2-02 19/4; DSS-B1b 4 dissent §10).

The escalation is **automated** via F-15 v3 linter timestamp comparison (EN sibling mtime vs TW sibling presence). No manual tracker required.

### 4.3 §78.3 — Timing × Responsibility

> Per scope tier (per §78.1) joint binding (per D-§2-03 DEFER R4 absorb; UNANIMOUS):
>
> - **BINDING tier**: same-PR-strict timing; primary author + SCRIBE/LINNAEUS reviewer (no dual-author per D-§2-04 5/18); MR-9 inheritance — if MUST cannot be met same-PR, the doc does not enter BINDING tier until parity met.
> - **CANDIDATE / KNOWLEDGE_ONLY tier**: 24h grace + R3+1 cycle close timing per §78.2 temporal escalation; primary author + SCRIBE post-audit at G4-folder gate.
> - **CONSULTATIVE tier**: cycle-end timing (R4 close); primary author best-effort + SCRIBE informational flag.
> - **SCRIBE-internal tier**: no timing constraint (MAY); primary author optional.

### 4.4 §78.4 — Quality (3 Sub-Clauses)

> Per D-§2-04 (3 sub-votes):
>
> - **Glossary YES with epistemic prefix** (21/2): every glossary entry SHALL carry a `verified:` / `inferred:` / `speculative:` prefix tag per ASANGA Yogācāra epistemic discipline. The prefix updates as community use settles the term. Initial v1 glossary lives in `tools/tw_glossary.yaml` per O1 §6.1 (~30 initial entries).
> - **Structural fidelity layer-bound** (22/1): TW SHALL preserve machine-checkable structure (heading hierarchy, table column count, code-block content, front-matter keys identical with values translated where appropriate, sibling-anchor stability for cross-references) MUST; human-checkable structure (bullet ordering, footnote anchor stability) SHOULD; un-checkable layers (sentence-level alignment, prose-paragraph parity) MAY (per ASANGA + TURING layer decomposition).
> - **Dual-author NO** (5/18; DSS-B2 5 dissent §10): primary author authors TW; SCRIBE/LINNAEUS performs reviewer role at G4-folder gate. Dual-author drafting NOT mandated; review-after-draft pattern adopted.

### 4.5 §78.5 — Retroactive Policy

> Per D-§2-05 (forward-only with scheduled audit per release-tag; 16/4/3; DSS-B3 7 dissent §10):
>
> - Rule #78 applies **forward-only** from cycle 03-15 onward. Existing canonical openstarry_doc/, Tenets, MR, ZT, Baseline Rules, Plan specs that landed pre-cycle-03-15 are **grandfathered EN-only** and not subject to retroactive TW parity mandate per MR-12 既有不破壞.
> - **Scheduled audit per release-tag**: at each release-tag boundary (per Rule #75 §75.X pnpm build cadence), SCRIBE performs a TW-parity sweep on the BINDING + CANDIDATE tier of the canonical corpus and flags any doc missing TW parity for owner attention. Owner may comply (author TW within R3+1 cycle close grace) or document a refusal-with-rationale (CONSULTATIVE downgrade or footnote per case).
> - **Cycle 03-13 `plugin-gear-arbiters.md` backfill classification** = "MR-10 retroactive precedent (one-off)" per D-§2-05 sub-vote (14/5/3/1; DSS-B3b 9 dissent §10). The cycle 03-14 audit #93 remediation is recognised as MR-10 retroactive precedent **for this single doc only**, not as a standing retroactive policy. Future similar gaps follow §78.5 forward-only + scheduled audit (b). See sibling `Reference/12_Cycle03-13_Backfill_Classification.md`.

### 4.6 §78.6 — CI Gate (Multi-Layer)

> Per D-§2-06 UNANIMOUS multi-layer specification:
>
> 1. **F-15 linter `tools/f15_check.py` extension** — at PR / commit time; checks sibling-file presence per §78.7 sibling-naming convention; per-tier rule application via config-driven dispatcher. Failure mode = block-PR for BINDING tier; warning for CANDIDATE / KNOWLEDGE_ONLY tier; informational for CONSULTATIVE / SCRIBE-internal.
> 2. **pnpm build integration** — at release-tag boundary (Rule #75 §75.X interplay); fails build if BINDING-tier TW parity gap detected. Hard fail = block-release.
> 3. **Doc gate** — at release-tag boundary; integrates F-15 linter output into release artefact; aligns with Rule #75 §75.X positional gate semantics.
> 4. **G4-folder gate** — at cycle R4 close (SCRIBE process gate); G4-folder-7 (new sub-check for cycle 03-15+) verifies TW parity of cycle's deliver/ outputs *for the tier they land in*.
> 5. **Coordinator hook** — at Suggestion → Canonical 4-mirror sync (cycle 03-14 invariant); coordinator subagent runs file-list diff per §78.7 sibling-naming convention and flags tier-violations. Hard fail for BINDING; warning for CANDIDATE.

Per WIENER closed-loop analysis, single-loop coverage leaves moment-of-failure gaps; multi-layer maintains closure across all moments (PR / release-tag / cycle-end / mirror-sync). Author-fatigue trade-off accepted per UNANIMOUS R3 vote.

### 4.7 §78.7 — Sibling-Naming Convention

> Per MRB-§2-02 (TURING + SCRIBE codified). The canonical sibling-naming pattern is `<basename>.<lang>.md`. Examples:
>
> - `plugin-gear-arbiters.md` (canonical EN — when language is implicit by project default; v0 / cycle 03-13-pre form).
> - `plugin-gear-arbiters.en.md` (canonical EN — when explicit `.en.` suffix used; **preferred for new docs landing under Rule #78 forward-only**).
> - `plugin-gear-arbiters.tw.md` (TW parity sibling).
>
> The `.en.` suffix is **recommended but not mandated** for backward compatibility with cycle 03-13-pre EN-only docs; the rule applies the parity check by *presence of `.tw.md` sibling* given the EN base file under either convention. Future cycles (03-20+) may bind `.en.` suffix as MUST after observation period (informational schedule per O1 §9.3).

### 4.8 §78.8 — Reflexive Application

> Rule #78 applies reflexively to its own definitional artefacts. Under §78.1 tier classification:
>
> - This Rule #78 itself, once Master-ratified, becomes BINDING tier → MUST acquire TW parity per §78.3 same-PR-strict timing in the cycle 03-15 R4 deliver/ + Master Ratification dispatch (MR-10 forward — to be authored cycle 03-15 R4 close → cycle 03-16 R0 24h grace).
> - The O1 deliver/ artefact is CYCLE_INTERNAL tier → MAY (no immediate TW parity required).
> - Rule #78 §78.X sub-clauses are bound to the parent Rule #78 status: BINDING upon Master ratification.

### 4.9 §78.9 — Master-Route Authority

> Rule #78 is **Master-route**, not SCRIBE-internal authority. Amendment requires R3 vote + Master Ratification per project standard rule-amendment process. SCRIBE-internal authority ENG-FAB-amendment items (G6.8/9/10 cycle 03-12; GP-coherence cycle 03-13; G4-folder-6 cycle 03-14; G4-folder-7 sub-check is Rule #78 §78.6 derived) remain SCRIBE-internal but the parent rule is Master-route.

### 4.10 §78.10 — MR-12 Forward-Only Reaffirmation

> This rule is **forward-only** per §78.5 + MR-12 既有不破壞. Existing artefacts pre-cycle-03-15 are not destabilised; they remain as authored. Scheduled audit (per §78.5) flags but does not coerce. The cycle 03-13 backfill is MR-10 one-off precedent (per §78.5 sub-vote classification), not a precedent for sweep-back of all ~225 canonical openstarry_doc/ files (per Pattern P4 cycle 03-14 R2 §11 DARWIN cross-cycle reference).

## 5. Interaction with MR-12 / MR-10

Rule #78 §78.5 + §78.10 jointly resolve the MR-10 vs MR-12 tension:
- **MR-12 priority**: forward-only is the default policy (no destabilisation of pre-cycle-03-15 artefacts).
- **MR-10 anchor**: scheduled audit + cycle 03-13 one-off classification recognise that retroactive correction *is* permissible when warranted; it is not a standing policy.
- Both Master Rulings honored simultaneously.

CV-§2-01..05 all UNANIMOUSLY reaffirmed at R3 close that no Rule / MR / ZT / Tenet is violated by Rule #78.

## 6. Cycle 03-13 Backfill Classification (One-Off)

The cycle 03-13 `plugin-gear-arbiters.md` TW backfill (executed at cycle 03-14 R4 audit #93 remediation) is classified **"MR-10 retroactive precedent (one-off)"** per D-§2-05 sub-vote (14/5/3/1).

The classification serves three purposes:
1. **Acknowledgement**: the cycle 03-14 audit #93 remediation was a **valid** governance action (not orphan precedent).
2. **MR-10 anchor**: the action exemplifies MR-10 (back-fill / retroactive correction) is permissible when a specific gap is identified and remediated; it does not require a forward-only rule.
3. **Boundary**: the one-off classification prevents the action from being read as establishing sweep-back of all ~225 canonical docs.

DSS-B3b 9 dissent (5 prefer MR-12 one-off framing; 3 prefer undecided-but-recorded; 1 prefer re-classification by new rule) preserved verbatim per MR-11 §10. Detail in sibling `Reference/12_Cycle03-13_Backfill_Classification.md`.

## 7. MR/ZT Compliance Audit

| Constraint | Status | Evidence |
|------------|:------:|----------|
| **MR-2 + MR-4** (Tenet 措辭不改) | PASS | Rule #78 does not propose Tenet wording change |
| **MR-5 hard** (Tenet #10 status no change) | PASS | §78.10 explicit: no Tenet #10 status change implied |
| **MR-6** (Core 零) | PASS | TW parity work is documentation tooling (linter ext ~130-230 LOC under `tools/`) |
| **MR-9** (no MUST WAIVE) | PASS | F-16 SHOULD initial unchanged; BINDING-tier MUST has no back-door waiver per §5.3 of O1 |
| **MR-10** (back-fill / retroactive) | PASS w/ classification | Cycle 03-13 backfill = "one-off precedent" §6 |
| **MR-11** (dissent preservation) | PASS | DSS-A1/A2/B1/B1b/B2/B3/B3b/D2 verbatim §10 |
| **MR-12** (既有不破壞 / forward-only) | PASS | §78.5 forward-only; §78.10 explicit reaffirmation |
| **MR-13** standby | PASS | Not invoked |
| **ZT-1** (governance preservation) | PASS | TW parity strengthens governance access for Mandarin-fluent readers |
| **ZT-2** (endpoint 10/0/0★) | PASS | 9/0/1★ ACTIVE preserved; endpoint unchanged |
| **ZT-3** (control-range triple-check) | PASS | No probability claim; no σ regime change; no V11 range change |
| **Rule #74 L1'** | Applicable | Rule #78 itself becomes L1'-applicable upon ratification |
| **Rule #75 §75.X** | Compatible | §78.5 scheduled audit at release-tag boundary integrates with §75.X pnpm build cadence |

## 8. ENG-FAB Numbering Note

- **ENG-FAB v1.8 = 48 items binding**: PRESERVED unchanged.
- **ENG-FAB v1.9 = 49 candidate F-16 SHOULD initial**: PRESERVED unchanged (per cycle 03-14 ratification; observation 1/2 cycle 03-15 informational only per Batch 12 Item #4).
- **ENG-FAB v1.10 candidate F-19**: **NOT created**. Per D-§3-05 + D-§2-07 anchor unification, the L2 ENG-FAB v1.10 F-19 candidate (3 votes in D-§2-07 14/4/3/2) was not selected; v1.10 numbering NOT consumed. Rule #78 + F-15 v3 covers TW + governance discipline jointly.

## 9. Implementation Forward (Post-Ratification)

Upon Master ratification at Batch 12 close:

- **Cycle 03-15 R4 close** (immediate): Rule #78 + F-15 v3 candidate landing in `cycle03-15/openstarry_doc/`. SCRIBE/coordinator G5 sync upgrades to BINDING in canonical.
- **Cycle 03-16 R0** (24h grace post-ratification): Rule #78 self-translates to TW per §78.8 reflexive (`Rule_78_TW_Translation_Parity.tw.md` sibling authored). `plugin-gear-arbiters.tw.md` re-quality-check per O1 §6.4 mitigation chain item 5.
- **Cycle 03-16 implementation** (Plan51 cycle): F-15 linter `tools/f15_check.py` extension implementation (~130-230 LOC) per F-15 v3 §3 sibling.
- **Cycle 03-16+ forward**: all new BINDING-tier docs acquire TW parity same-PR-strict; CANDIDATE / KNOWLEDGE_ONLY-tier docs acquire TW parity within 24h grace + R3+1 cycle close.
- **Cycle 03-17~03-19 observation**: `.en.` suffix adoption rate tracked in scheduled audit; glossary epistemic-prefix updates (`speculative:` → `inferred:` → `verified:` migration).
- **Cycle 03-20+ future bindings**: `.en.` suffix MUST binding considered if adoption ≥ 80%.

## 10. Dissent Preservation (per MR-11 verbatim)

Eight §2-related DSS entries preserved verbatim from R3 §7.1 dissent inventory.

### 10.1 DSS-A1 (D-§2-07 TW landing L1+L3 hybrid; 14/4/3/2)

**Minority**: NAGARJUNA + DARWIN + 2 others (4 votes preferred L2 ENG-FAB v1.10 F-19).

**Verbatim**: "L2 (ENG-FAB v1.10 F-19) provides cleaner工程化 + MUST/SHOULD/MAY 強度；L1+L3 hybrid 過於分散 governance."

### 10.2 DSS-A2 (D-§2-01 scope tier by-ratification-status; 17/4/2)

**Minority**: LINNAEUS + VITRUVIUS + 4 others (6 dissent: 4 by-location, 2 3-tier).

**Verbatim**: "By-location more architecturally stable；ratification-status 跨 cycle 變化大；3-tier 簡明."

### 10.3 DSS-B1 (D-§2-02 strength static b; 18/5)

**Minority**: 5 dissent.

**Verbatim**: "Static (c) governance + canonical MUST + 餘 SHOULD 更嚴；(b) 對 Plan/operational 過鬆."

### 10.4 DSS-B1b (D-§2-02 temporal escalation YES; 19/4)

**Minority**: 4 dissent.

**Verbatim**: "Temporal escalation 增加 coordination burden；MR-12 forward-only 已足保護."

### 10.5 DSS-B2 (D-§2-04 dual-author NO; 5/18)

**Minority**: 5 dual-author preferred.

**Verbatim**: "雙作者制度 reduces single-translator bias；漏譯防護 stronger; bilingual-jurisdiction precedent (R1 §10.bis (ii)) supports dual-author for constitutional layer."

### 10.6 DSS-B3 (D-§2-05 forward-only b; 16/4/3)

**Minority**: 7 dissent (4 strict-forward + 3 tier-graded retroactive + others).

**Verbatim**: "Scheduled-audit (a/c) 對歷史 docs 補回更積極；MR-10 spirit 更貼近. Strict-retroactive (c) tier-graded would close all governance gaps within 5 cycles. Forward-only is too permissive given the cycle 03-13 origin demonstrated systemic risk."

### 10.7 DSS-B3b (D-§2-05 cycle 03-13 backfill = MR-10 precedent; 14/5/3/1)

**Minority**: 9 dissent (5 prefer MR-12 one-off framing; 3 prefer undecided-but-recorded; 1 prefer re-classification by new rule).

**Verbatim**: "MR-12 one-off classification 較不會 set retroactive precedent；undecided-but-recorded 更保守."

### 10.8 DSS-D2 (D-§2-08 framing artefact + participation-right; 18/3/2)

**Minority**: 3 dissent (event-property concern).

**Verbatim**: "Event-property framing more accurate for governance-doc lifecycle (created / amended / superseded events)."

### 10.9 Aggregate

Total §2 dissent preserved: **8 entries** (DSS-A1 / DSS-A2 / DSS-B1 / DSS-B1b / DSS-B2 / DSS-B3 / DSS-B3b / DSS-D2). All preserved verbatim per MR-11. §2 contributes 8 of the cycle's 23 R3 dissent total (≥34% share — major chapter dissent share). MR-11 PASS.

---

*Rule #78 Candidate — Cycle 03-15 R4 Final Spec*
*Authors: SCRIBE (#2) + LINNAEUS (#13) + ARCHIMEDES (#16) + KERNEL (#10) + GUARDIAN (#11) + SUNYATA (#0) + SYNTHESIST (#1)*
*Status: CANDIDATE (pending Master Ratification Batch 12 #1)*
*Landing: L1 high-level policy half (Reference/); L3 operational mechanism half = `Research_Methodology/16_F_15_v3_Third_Tier_Amendment.md`*
*Sibling-naming convention codified: `<basename>.<lang>.md`*
*Compliance: MR-5 hard / MR-6 / MR-9 / MR-10 / MR-11 / MR-12 + ZT-1/2/3 全 PASS*
