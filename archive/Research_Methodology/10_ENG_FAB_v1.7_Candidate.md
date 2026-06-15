# ENG-FAB v1.6 → v1.7 Candidate (Anti-Fabrication Audit Checklist)

> **Front-matter**
> - **Cycle**: 03-12
> - **Date**: 2026-04-25
> - **Authors**: TURING + ARCHIMEDES + SYNTHESIST (framework drafting) · GUARDIAN + ASANGA + TANENBAUM (F-10 + F-9 sub-check review) · SCRIBE (item numbering + audit-trail consistency)
> - **Binding R3 decisions**: D-14(c) E-5 dual-track UNANIMOUS 23/0 (Plan48 primary + ENG-FAB v1.7 forward enforcement) · D-17(a) HMAC key rotation design-spec-only UNANIMOUS 23/0 (F-8 carry-forward) · D-18(a) SCRIBE-authority G6.8/G6.9/G6.10 UNANIMOUS 23/0 (F-10 proposal originated in §5 R2 §12.1 CR-§5-R2-F10-EngFab, integrated here as ENG-FAB v1.7 item)
> - **Status**: **CANDIDATE** — awaiting Master ratification via **Batch 8c** (bundled with Rule #72 N≥10 + Rule #75 Pre-Delivery Gate)
> - **Effective date proposal**: Plan48 delivery (first post-ratification delivery); Plan47 assessed under v1.6 (35 PASS / 5 N/A / 1 INFO / 1 PARTIAL / 0 FAIL, all MUST PASS, as observed 2026-04-19).
> - **Supersedes**: ENG-FAB v1.6 (42 items, ratified 2026-04-18).

---

## Section A — v1.6 → v1.7 Diff Summary

### A.1 Item-count change

| Version | Items | MUST | SHOULD | INFO | Notes |
|---------|:-----:|:----:|:------:|:----:|-------|
| v1.6 | 42 | 38 | 3 | 1 | E-5 SHOULD; F-8 carry-forward-must; F-9 NEW (Rule #74 L1' doc check) |
| **v1.7** | **43** | **40** | **2** | **1** | **E-5 SHOULD → MUST** (D-14c); **F-10 NEW** (Pre-Delivery Gate Verified); **F-9 extended** with 3 sub-checks |

### A.2 Principal changes

1. **E-5 elevated SHOULD → MUST** (D-14c dual-track per §5 R2 §12.1 + §1 R2 §5.3). Rationale: OWASP ASVS V2.10.1 + NIST SP 800-57 Part 1 §8.2.2 compliance + δ-2 audit-gap complete closure. Plan48 C48-M3a–d is the primary implementation track; ENG-FAB v1.7 F-E-5 is the forward-enforcement track.
2. **F-9 extended with 3 sub-checks** (from O1 §5 Rule #74 L1' first-application learning). Sub-checks (i) module+test, (ii) doc+CHANGELOG, (iii) runtime evidence.
3. **F-10 NEW — Pre-Delivery Gate Verified** — test-layer companion to Rule #75 (dev-layer block). Defense-in-depth dual-gate.
4. **Numbering**: items E-5 and F-10 receive MUST level; all other items retain v1.6 levels.
5. **Documentation**: each change includes "v1.6 → v1.7 migration" inline note for audit-trail transparency (per MRB-07 drafting-note precedent).

### A.3 Item-count arithmetic

- v1.6 = 42 items.
- v1.7 adds F-10 → 43 items.
- Elevation of E-5 SHOULD → MUST does **not** change count; only level. E-5 was an INFO-marked item for reference; it is now a binding MUST with four binding sub-items (C48-M3a–d in Plan48 + SHOULD continuity in ENG-FAB audit).

---

## Section B — Full Item List (v1.7)

### B.1 A-series (Architecture hygiene) — 9 items, all MUST

| ID | Item | Level | Change | Sub-checks |
|----|------|:-----:|:------:|------------|
| A-1 | Core surface opacity (no policy constants) | MUST | unchanged | Static grep of `packages/core/**` |
| A-2 | SDK types declared in `@openstarry/sdk`, not Core | MUST | unchanged | `find packages/core -name "*.ts" | xargs grep -l "export.*type"` = 0 policy types |
| A-3 | Plugin manifests well-formed | MUST | unchanged | schema-match per plugin |
| A-4 | Event-bus purity (no in-Core policy decisions) | MUST | unchanged | CP-2 verification |
| A-5 | Configuration-layer purity (no fabrication via config) | MUST | v1.3+ retained | H section check (fab pattern v7 prediction) |
| A-6 | Plugin-lifecycle checkpoint/restore symmetry | MUST | v1.5+ | K-3 wire-in verification |
| A-7 | Shadow-mode isolation from production path | MUST | unchanged | — |
| A-8 | Framework hook SDK type declarations | MUST | v1.5 | Plan46 class-level; K-3 extension |
| A-9 | Integration-test presence for each production feature | MUST | v1.2 | CF-1 root cause — wave-level granularity check |

### B.2 B-series (Build hygiene) — 6 items, all MUST

| ID | Item | Level | Notes |
|----|------|:-----:|-------|
| B-1 | Monorepo build clean | MUST | `pnpm build` exit 0 |
| B-2 | Lint + type-check green | MUST | — |
| B-3 | No unused imports / dead code | MUST | — |
| B-4 | Dep-tree audit (no security advisories) | MUST | — |
| B-5 | Test suite green (non-flaky) | MUST | `pnpm test` exit 0 |
| B-6 | Plugin isolation (no cross-plugin leakage) | MUST | — |

### B.3 C-series (Contract verification) — 4 items, all MUST

| ID | Item | Level | Notes |
|----|------|:-----:|-------|
| C-1 | InputEvent schema contract | MUST | — |
| C-2 | Plugin activation contract | MUST | — |
| C-3 | Event emission contract | MUST | — |
| C-4 | Shutdown contract (flush + clear) | MUST | W2-R13 runtime cross-check |

### B.4 D-series (Documentation) — 4 items, 3 MUST + 1 SHOULD

| ID | Item | Level | Notes |
|----|------|:-----:|-------|
| D-1 | README up-to-date per Plan | MUST | — |
| D-2 | CHANGELOG entry present | MUST | v1.3 |
| D-3 | Architecture-Documentation file per significant feature | MUST | Rule #74 L1' (c) |
| D-4 | Developer examples (tutorial update) | SHOULD | — |

### B.5 E-series (Cleanup / key-material) — 5 items, 4 MUST + 1 SHOULD

| ID | Item | Level | Change from v1.6 | Notes |
|----|------|:-----:|:----------------:|-------|
| E-1 | Temp-file / tmp-dir cleanup on exit | MUST | unchanged | — |
| E-2 | Process-local state clear on stop() | MUST | unchanged | — |
| E-3 | Env-var non-leakage in logs | MUST | unchanged | — |
| E-4 | Plugin-state persistence discipline | MUST | unchanged | — |
| **E-5** | **HMAC key cleanup (consumed + cleared from memory after use; closure-retained for shutdown signing only)** | **MUST (v1.7)** | **SHOULD → MUST (D-14c)** | **OWASP ASVS V2.10.1 + NIST SP 800-57 Part 1 §8.2.2; Plan48 C48-M3a–d primary** |

### B.6 F-series (Fabrication-pattern detectors) — 10 items, all MUST

| ID | Item | Level | Change from v1.6 | Notes |
|----|------|:-----:|:----------------:|-------|
| F-1 | Phantom-production detector (function body exists in Plan scope) | MUST | v1.0 | fab variant 1 |
| F-2 | Phantom-test detector (test file contents non-empty + assertions present) | MUST | v1.1 | fab variant 2 |
| F-3 | Empty test-snapshot detector | MUST | v1.2 | fab variant 3 |
| F-4 | Phantom-integration detector (integration test exercises full path) | MUST | v1.2 | fab variant 4 |
| F-5 | Configuration-layer fab detector (H section) | MUST | v1.3 | fab variant 7 |
| F-6 | Delivery-report completeness | MUST | v1.4 | Rule #75 failure cascade |
| F-7 | Post-delivery-fix classification (Rule #62 Tier 1/2/3) | MUST | v1.4 | Plan44 baseline |
| F-8 | Carry-forward-must check (deferred wave SDK type presence) | MUST | v1.5 | Plan46→Plan47 K-3 learning; CF-1 |
| F-9 | Rule #74 L1' doc-check (5 L1' facets + 3 new sub-checks in v1.7) | MUST | **extended** | See §C below |
| **F-10** | **Pre-Delivery Gate Verified** (test-layer re-check of Rule #75 dev-layer gate) | **MUST (v1.7)** | **NEW** | See §D below |

### B.7 G-series (Governance / process hygiene) — 2 items, 1 MUST + 1 SHOULD

| ID | Item | Level | Notes |
|----|------|:-----:|-------|
| G-1 | Dev delivery report vs research verification alignment | MUST | — |
| G-2 | Research-team TURING super-verification alignment | SHOULD | — |

### B.8 H-series (Configuration-layer audit) — 3 items, all MUST

| ID | Item | Level | Notes |
|----|------|:-----:|-------|
| H-1 | Config-override correctness (no silent skip) | MUST | v1.3 |
| H-2 | Config-schema Zod validation completeness | MUST | v1.3 |
| H-3 | Config-default audit (fallback to safe default) | MUST | v1.3 |

**Total v1.7**: 9 A + 6 B + 4 C + 4 D + 5 E + 10 F + 2 G + 3 H = **43 items** (40 MUST + 2 SHOULD + 1 INFO).

---

## Section C — F-9 Extension (v1.7, 3 New Sub-Checks)

F-9 is the Rule #74 L1' doc-check item introduced in ENG-FAB v1.6. v1.7 extends F-9 with three explicit sub-checks derived from the Plan47 first-application learning (O1 §5) and Plan48 25-sub-item binding acceptance (see `Calibration_Reports/19_Plan48_25_SubItems_Binding.md`).

### C.1 F-9 sub-check (i) — Module + test presence

For each MUST item's corresponding code module, audit:
1. Source file exists at declared path (L1 grep + file listing).
2. At least one test file references the module (L2 assertion).
3. Test file contains ≥ 1 non-trivial assertion on the module (not a stub).

### C.2 F-9 sub-check (ii) — Doc + CHANGELOG presence

For each MUST item, audit:
1. Architecture-Documentation file exists for the feature (Rule #74 L1' (c)).
2. CHANGELOG entry exists and references the MUST item ID (Rule #74 L1' (d)).
3. README or `docs/` inline doc covers user-facing behavior (Rule #74 L1' (c) extension).

### C.3 F-9 sub-check (iii) — Runtime evidence presence

For each MUST item requiring runtime evidence (shutdown flush, canary positive control, hookMap observation, key-clear-before-exit), audit:
1. Runtime evidence artefact exists (test run log, trace file, hookMap snapshot).
2. Evidence artefact references the MUST item ID in its header.
3. Evidence artefact is reproducible (test command + expected output documented).

### C.4 CP-4 integration (Plan50 prospective)

For plugins emitting `InputEvent` with agent-identity semantics beyond transport-identity, F-9 includes an additional CP-4 sub-check per MRB-06 §6.4:

> "Plugin author documents whether `source` string carries agent-identity semantics beyond transport-identity; if so, MUST cross-reference `sourceContext.parentAgentId` in the plugin README or JSDoc."

This CP-4 sub-check applies prospectively on the Plan50 pushInput implementation round.

---

## Section D — F-10 NEW — Pre-Delivery Gate Verified

F-10 is the test-layer companion to Rule #75 (dev-layer block). Defense-in-depth dual-gate: Rule #75 blocks pre-delivery at dev side; F-10 verifies gate-passage at research/test side post-delivery.

### D.1 F-10 check criteria

Per Plan delivery, research/test-team MUST audit:

1. **§75.3.1 attested** — delivery report Section 0 present with MUST-item sub-check coverage (≥ 5 sub-checks per MUST).
2. **§75.3.2 attested** — binding acceptance criteria explicitly referenced and satisfied (Plan-specific: Plan47 K-3 M1–M5; Plan48 25 sub-items; Plan49+ per spec).
3. **§75.3.3 attested** — ENG-FAB v1.7 43-item audit PASS item-by-item (no N/A unless justified).
4. **§75.3.4 attested** — δ-status explicit declaration (closes / carries-forward / δ-neutral).
5. **§75.3.5 attested** — runtime evidence artefact references present (file path + line range).

### D.2 F-10 failure behavior

Any F-10 check failure → **research/test-team logs Rule #75 violation** and remediation request; dev-team repairs attestation; research verification pauses until remediation.

### D.3 F-10 vs F-9 distinction

- **F-9** = Rule #74 L1' doc-check; applies to **every MUST item** individually (module + doc + runtime).
- **F-10** = Rule #75 attestation completeness; applies to **the delivery report as a whole** (Section 0 present, all §75.3.x sections populated).

Defense-in-depth rationale: F-9 catches missing doc for a specific MUST; F-10 catches a missing pre-delivery attestation across the whole Plan. Different granularities, complementary detection.

---

## Section E — E-5 MUST Elevation Justification (D-14c)

### E.1 Regulatory basis

- **OWASP ASVS V2.10.1**: "Verify that secrets are not kept in resident memory longer than necessary." Direct mapping to E-5 closure pattern (HMAC key consumed and cleared from memory after use; closure retained for shutdown signing only).
- **NIST SP 800-57 Part 1 §8.2.2**: Key-destruction requirements — "when a cryptographic key is no longer needed, it shall be destroyed." E-5 post-signing clear-from-memory is the implementation.

### E.2 Dual-track (D-14c)

Per R3 D-14(c) — adopted as both:
- **Plan48 primary track**: C48-M3a (closure pattern), C48-M3b (ephemeral source enforcement), C48-M3c (ASVS + NIST compliance attestation), C48-M3d (W2-R13 runtime evidence for key clear-before-exit).
- **ENG-FAB v1.7 forward-enforcement track**: E-5 MUST elevation applies to all future Plans; any Plan using HMAC keys MUST attest E-5 compliance in its delivery report.

### E.3 δ closure semantics

E-5 MUST completion is the final δ-2 (δ-2 = F-L3-SEC-2 + F-L3-SEC-3 + E-5) closure gate. Plan48 closing all three δ items = **δ DONE** unconditionally. The 9/0/1 PENDING† → 9/0/1★ auto-promotion requires: α DONE (Plan47 K-3, 2026-04-19) + β DONE (W2-R12 canary, 2026-04-20) + γ DONE (Rule #74 + ENG-FAB v1.6 ratified + Plan47 PASS, 2026-04-19) + **δ DONE (Plan48 + W2-R13 per timing Candidate B)**.

Per **MR-5 hard constraint** (see `MEMORY.md` Master Permanent Rulings): promotion 9/0/1† → 9/0/1★ is a *transitional* milestone; Tenet #10 status remains NON-COMPLIANT PENDING per MR-5 regardless of δ closure. Final endpoint is **10/0/0★** per ZT-2 (Phase 6 completion; Plan48–Plan55 roadmap).

---

## Section F — Migration Guide (v1.6 → v1.7)

### F.1 For Dev team

1. Adopt E-5 as MUST for any HMAC-key-handling code. Plan48 C48-M3a–d satisfies this for the current cycle.
2. Adopt F-10 in delivery-report Section 0 (Rule #75 attestation). See `Research_Methodology/09_Rule_75_PreDelivery_Gate_Draft.md` §75.6.1 for the template.
3. Extend F-9 compliance to include the 3 new sub-checks (§C above) per MUST item.

### F.2 For Research team

1. L1–L4 verification adds F-10 (Pre-Delivery Gate) as the first L1 audit step. If F-10 FAIL, research pauses L2–L4 and requests remediation.
2. F-9 audit extended to 3 sub-checks per MUST; TURING super-verification includes sub-check evidence review.
3. E-5 MUST audit baseline — prior Plans (Plan47 and earlier) assessed under v1.6 (E-5 SHOULD); Plan48 onward under v1.7 MUST.

### F.3 For Test team

1. W2-R13 test_instructions include runtime evidence artefact path table (per-MUST; see `Calibration_Reports/19_Plan48_25_SubItems_Binding.md` and `todo_test_instructions.md`).
2. Canary positive control + shadow-agreement scenarios retained unchanged.
3. F-10 test-side verification = Rule #75 attestation scan on receipt of every dev delivery.

### F.4 For SCRIBE

1. G5 extension — Rule #75 attestation completeness check (see Rule #75 §75.5 / §75.6.3).
2. G6.8 (new) — SCRIBE-authority Pre-Delivery Gate alignment audit across cycle-end artefacts.
3. ENG-FAB v1.7 item-by-item tracking in `SCRIBE_eng_fab_v1_7_log.md`.

---

## Section G — Dissent, Ratification Path, Status

### G.1 Dissent (MR-11 preservation)

- **E-5 SHOULD retention** — R2 stage considered v1.6 SHOULD retention for transition smoothness; R3 D-14(c) UNANIMOUS 23/0 adopted dual-track MUST elevation; no R3 dissent recorded. The SHOULD-retention path is preserved here as historical rationale only (not a minority R3 vote). Majority rationale: dual-track (Plan48 primary + ENG-FAB v1.7 forward) resolves transition concern; MUST elevation necessary for δ closure + ZT-2 endpoint alignment.
- **F-9 3 sub-checks alternative** — one-sub-check consolidation considered; rejected because three distinct failure modes (missing module, missing doc, missing runtime evidence) warrant separable sub-checks for targeted remediation.
- **F-10 vs Rule #75 overlap concern** — one agent raised overlap between F-10 and Rule #75; addressed via §D.3 granularity distinction (F-9 = per-MUST, F-10 = per-delivery-report).

### G.2 Ratification path

- **Batch**: 8c (Rules + ENG-FAB).
- **Master Ratification Request**: `Master_Ratification_ENG_FAB_v1_7.md` (consolidated with Rule #72 N≥10 + Rule #75 Pre-Delivery Gate per §5 R2 §13.3).
- **Effective date**: Plan48 delivery (post-ratification).
- **Retrospective assessment**: Plan47 assessed under v1.6 (frozen).

### G.3 Status

**CANDIDATE** — awaiting Master ratification via Batch 8c.

Until ratified, v1.6 remains the binding checklist; v1.7 governs research-team internal forward-looking planning and Plan48 delivery-report Section 0 pilot attestation (voluntary pilot for no-risk forward compatibility).

## Section H — Author attribution and R3 vote

- **Framework drafters**: TURING (#17) + ARCHIMEDES (#16) + SYNTHESIST (#1).
- **F-10 sub-check review**: GUARDIAN (#11) + ASANGA (#8) + TANENBAUM (#20).
- **Item numbering + audit-trail**: SCRIBE (#2).
- **R3 vote** (ground truth `R3_decision_log.md §2.2`): D-14(c) E-5 dual-track UNANIMOUS 23/0; D-17(a) HMAC rotation design-spec-only UNANIMOUS 23/0; D-18(a) SCRIBE-authority G6.8/G6.9/G6.10 UNANIMOUS 23/0. F-10 is not a standalone R3 D item — it is an ENG-FAB v1.7 item introduced here, proposed in §5 R2 §12.1 CR-§5-R2-F10-EngFab, aligned with D-18(a) SCRIBE G6.8 Pre-Delivery Gate authority. (D-19(a) 23/0 is the §3 Day 4-6 Coordinator Daily Check 6-item checklist, unrelated to F-10.)
- **Consolidation**: SYNTHESIST + SUNYATA (#0).
