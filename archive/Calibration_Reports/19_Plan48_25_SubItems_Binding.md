# Plan48 — 25 Atomic Acceptance Criteria (BINDING)

> **Front-matter**
> - **Cycle**: 03-12
> - **Date**: 2026-04-25
> - **Authors**: SYNTHESIST + ARCHIMEDES (MRB-11 joint drafters) + GUARDIAN (E-5 ASVS + NIST compliance) + HERACLITUS (W2-R13 runtime edge cases) + SCRIBE (sub-item ID discipline)
> - **Binding R3 decisions**: D-01(c) δ boundary 中間 SUSSMAN β (E-5 MUST on defense-in-depth grounds; 15 c / 5 b / 3 a) · D-14(c) E-5 dual-track UNANIMOUS (Plan48 primary + ENG-FAB v1.7 forward enforcement; 23/0) · D-17(a) HMAC rotation design-spec-only UNANIMOUS (23/0) · Plan48/49 split from R2 convergence (UNANIMOUS R2 recommendation, not R3-revisited) · MRB-11 21 sub-items + 3 MUST headings + 1 aggregate = 25 atomic criteria
> - **Status**: **BINDING** per R3 D-01(c) + D-14(c) compound; attached as binding annex to Master Ratification Request Batch 8a (`Master_Ratification_Plan48_Scope.md`)
> - **Scope**: Plan48 delivery (MUST scope) + corresponding dev attestations per Rule #75 + corresponding ENG-FAB v1.7 F-9 / F-10 audit
> - **Cross-refs**: `Research_Methodology/08_Rule_72_N10_SubClause_Draft.md` · `Research_Methodology/09_Rule_75_PreDelivery_Gate_Draft.md` · `Research_Methodology/10_ENG_FAB_v1.7_Candidate.md` · `deliver/O5_Plan48_scope_and_engineering_spec.md`

---

## §1 Purpose

Plan48 closes the remaining δ-2 audit-gap items (F-L3-SEC-2, F-L3-SEC-3, E-5 MUST elevation). This document enumerates the **binding acceptance criteria** in atomic form for dev-team to attest PASS and for research/test-team to audit independently.

### §1.1 Layout

**25 atomic acceptance criteria** = 3 MUST × sub-items (8 + 9 + 4) **+** 3 MUST-heading roll-ups **+** 1 aggregate delivery-level roll-up.

- **21 sub-items** = C48-M1a–h (8) + C48-M2a–i (9) + C48-M3a–d (4)
- **3 MUST-heading roll-ups** = C48-M1 / C48-M2 / C48-M3 each PASS iff all its sub-items PASS
- **1 aggregate** = Plan48 delivery PASS iff all 3 MUST-heading roll-ups PASS

**Per §5 R2 F-§5-R2-1 disambiguation** (R3 D-05): the count **25** is adopted as "sub-items + MUST headings + 1 roll-up" per SYNTHESIST + ARCHIMEDES MRB-11 §11.2 proposal.

### §1.2 Binding discipline

Dev-team **cannot claim Plan48 complete** unless all 21 sub-items attested PASS with runtime evidence where required (per Rule #75 §75.3.5). Research-team independently audits via L1–L4 + ENG-FAB v1.7 F-9 / F-10.

---

## §2 MUST C48-M1 — F-L3-SEC-2 Structured Log (8 sub-items)

**Source**: §5 R1 §5-3.1.2 + §5 R2 §3.1 + MRB-11 §11.1.
**Target LOC**: 150–250 prod + ~50–80 test.
**Layer classification**: Runner / Plugin-adjacent (NOT Core; MR-6 preserved).
**Feature summary**: Self-built zero-external-dep structured-log writer with `{timestamp, level, event, payload}` JSON-line schema, level filter, ring-buffer back-pressure, SIGTERM/SIGINT synchronous flush.

| ID | Sub-item | Verification method | Acceptance criteria | Source | Layer |
|----|----------|---------------------|---------------------|--------|:-----:|
| **C48-M1a** | structured-log writer module exists (zero external dep, self-built) | Dev: file-presence + unit test. Research: L1 grep + Rule #74 doc-check. | `structured-log/index.ts` exists under runner/plugin-adjacent path; no external logging-library import | §5 R1 §5-3.1.2 | Runner |
| **C48-M1b** | writer emits JSON-line schema `{timestamp, level, event, payload}` | Dev: `it("emits schema")`. Research: L2 assertion; ENG-FAB v1.7 F-9(i) | Every emitted line is valid JSON matching the schema; schema keys present and typed | §5 R1 §5-3.1.2 + ENG-FAB F-9 | Runner |
| **C48-M1c** | Level filter (DEBUG / INFO / WARN / ERROR / FATAL) configurable via `LOG_LEVEL` env var | Dev: test per level. Research: L2 config test | `LOG_LEVEL=WARN` suppresses DEBUG + INFO lines; test suite covers all 5 levels | §5 R1 §5-3.1.2 | Runner |
| **C48-M1d** | Back-pressure: ring-buffer overflow → `W_AUDIT_OVERFLOW` structured warn emission | Dev: simulated overflow test. Research: L2 + HERACLITUS W2-R13 Edge 3 | Overflow scenario emits a `W_AUDIT_OVERFLOW` structured warn; no silent drop; test asserts emission | §5 R2 F-§5-R2-5 Edge 3 | Runner |
| **C48-M1e** | Shutdown flush: SIGTERM / SIGINT → in-memory buffer flushed to file, **zero entries lost** | Dev: shutdown test suite. Research: L2 + HERACLITUS W2-R13 Edge 1 + G6.10 proposed gate | Shutdown test: 1000 events queued → SIGTERM → file contains 1000 events, exit code 0 | §5 R2 F-§5-R2-5 Edge 1 + F-§5-R2-6 | Runner |
| **C48-M1f** | W2-R13 runtime evidence: ≥ 1 event logged per test scenario | Test: W2-R13 `todo_test_instructions` runtime check + log-file row count | Every W2-R13 scenario (canary 1–3, shadow agreement, perturbation) emits ≥ 1 structured-log line | §5 R2 F-§5-R2-5 + MRB-12 | Test |
| **C48-M1g** | L1' doc sync: `structured-log.md` documentation published; CHANGELOG entry; Doc 78 candidate | Research: Rule #74 L1' doc-check (5 sub-checks) | `structured-log/README.md` exists; CHANGELOG appends Plan48 structured-log entry; `Architecture_Documentation/78_*.md` candidate staged | Rule #74 existing | Doc |
| **C48-M1h** | ENG-FAB v1.7 F-9 new sub-checks PASS: (i) module+test, (ii) doc+CHANGELOG, (iii) runtime evidence | Research: ENG-FAB audit per F-9 extension | F-9 (i) file+test exist; (ii) README+CHANGELOG present; (iii) W2-R13 evidence artefact references in delivery report | §1 R2 CR-2 + §5 R2 §12.1 | Audit |

### §2.1 C48-M1 roll-up

**C48-M1 PASS iff all 8 sub-items PASS**. Failure of any sub-item → C48-M1 FAIL → Plan48 delivery blocked per Rule #75.

---

## §3 MUST C48-M2 — F-L3-SEC-3 Plugin Audit Sink (9 sub-items)

**Source**: §5 R1 §5-3.2 + §5 R2 §3.1 + MRB-11 §11.1.
**Target LOC**: 100–200 prod + ~40–60 test.
**Layer classification**: Runner (audit-sink subscriber; subscribes to event bus on runner side, NOT Core).
**Feature summary**: Audit-sink subscriber on runner side that subscribes to `capability_denied` + `ws_connection_denied` event types, deduplicates via `(timestamp, event_hash)` composite key, appends to JSONL audit-trail file.

| ID | Sub-item | Verification method | Acceptance criteria | Source | Layer |
|----|----------|---------------------|---------------------|--------|:-----:|
| **C48-M2a** | plugin emit → runner audit-sink subscription pattern (bus → sink) implemented | Dev: unit test on subscription. Research: L1 architecture diagram check | `audit-sink/index.ts` subscribes to bus at runner startup; test asserts subscription | §5 R1 §5-3.2.2 | Runner |
| **C48-M2b** | Deduplication / ordering via `(timestamp, event_hash)` composite key | Dev: test ordering. Research: L2 | Duplicate events (same timestamp + hash) are written once; ordering preserves event-timestamp | §5 R1 §5-3.2.2 | Runner |
| **C48-M2c** | `capability_denied` event type subscribed and logged | Dev: test case emitting `capability_denied`. Research: L2 | Synthetic `capability_denied` emission appears in audit-trail file | §5 R1 §5-3.2 | Runner |
| **C48-M2d** | `ws_connection_denied` event type subscribed and logged | Dev: test case emitting `ws_connection_denied`. Research: L2 | Synthetic `ws_connection_denied` emission appears in audit-trail file | §5 R1 §5-3.2 | Runner |
| **C48-M2e** | Audit-sink file path configurable via `AUDIT_SINK_PATH` env var; default `<data_dir>/audit-trail.jsonl` | Dev: config test with and without env var. Research: L2 | Env-var override honored; default resolves to `<data_dir>/audit-trail.jsonl` | §5 R1 §5-3.2 | Runner |
| **C48-M2f** | Back-pressure identical to C48-M1d (shared infra) | Dev: test. Research: L2 | Ring-buffer behavior identical to C48-M1d; overflow emits `W_AUDIT_OVERFLOW` | §5 R2 §3.1 shared infra | Runner |
| **C48-M2g** | Shutdown flush identical to C48-M1e (shared infra) | Dev: test. Research: L2 + G6.10 | Shutdown test: queued events flushed on SIGTERM/SIGINT; zero entries lost | §5 R2 F-§5-R2-6 | Runner |
| **C48-M2h** | W2-R13 runtime evidence: at least 1 `capability_denied` + 1 `ws_connection_denied` event captured | Test: W2-R13 runtime instrumented scenario | Audit-trail file contains ≥ 1 of each event type after W2-R13 run | §5 R2 F-§5-R2-5 + MRB-12 | Test |
| **C48-M2i** | L1' doc sync + CHANGELOG + ENG-FAB F-9 PASS (shared with C48-M1g/h) | Research: Rule #74 L1' + ENG-FAB F-9 | Audit-sink README exists; CHANGELOG entry; F-9 audit PASS | Rule #74 existing | Doc |

### §3.1 C48-M2 roll-up

**C48-M2 PASS iff all 9 sub-items PASS**. Failure of any sub-item → C48-M2 FAIL → Plan48 delivery blocked per Rule #75.

---

## §4 MUST C48-M3 — E-5 HMAC Key Cleanup (MUST elevation) (4 sub-items)

**Source**: §5 R1 §5-1.3 + §5 R2 C-5R1-04 ACCEPT + MRB-11 §11.1 + D-14(c) dual-track.
**Target LOC**: 60–120 prod + ~30–50 test.
**Layer classification**: Runner (HMAC key handling at runner layer only; Core untouched per MR-6).
**Feature summary**: E-5 closure pattern (HMAC key consumed and cleared from memory after use; closure-retained for shutdown-signing only) + ephemeral-key-source enforcement + OWASP ASVS V2.10.1 + NIST SP 800-57 Part 1 §8.2.2 compliance attestation.

| ID | Sub-item | Verification method | Acceptance criteria | Source | Layer |
|----|----------|---------------------|---------------------|--------|:-----:|
| **C48-M3a** | E-5 closure pattern: HMAC key consumed-and-cleared from memory after use; closure retained for shutdown-signing only | Dev: memory-inspection test (Node.js heap-dump assertion). Research: L2 | After startup consumption, HMAC key string is zeroed in env + any intermediate buffer; heap-dump shows no plaintext key | §5 R1 §5-3.3.2 + §5 R2 C-5R1-10 | Runner |
| **C48-M3b** | Ephemeral key-source enforcement: no key persisted to disk except in pre-agreed secure-store location | Dev: filesystem-scan test. Research: L1 grep + doc | Disk scan after run: no plaintext key in any temp / log / state file; only secure-store path allowed | §5 R1 §5-1.3 + GUARDIAN threat model | Runner |
| **C48-M3c** | OWASP ASVS V2.10.1 + NIST SP 800-57 Part 1 §8.2.2 compliance attested | Dev: standards-compliance doc `docs/hmac-compliance.md`. Research: doc review | `docs/hmac-compliance.md` maps each sub-item to the specific ASVS / NIST clause; signed by GUARDIAN | §5 R1 §5-1.3 面向 3 + §5 R2 C-5R1-03 refinement | Doc |
| **C48-M3d** | W2-R13 runtime evidence: ≥ 1 shutdown signing event with key clear-before-process-exit observed | Test: W2-R13 shutdown scenario + heap-dump snapshot | Shutdown sign + key-clear observed in same scenario; heap-dump post-clear shows no plaintext key | §5 R2 F-§5-R2-5 Edge 1 | Test |

### §4.1 C48-M3 roll-up

**C48-M3 PASS iff all 4 sub-items PASS**. Failure of any sub-item → C48-M3 FAIL → Plan48 delivery blocked per Rule #75.

### §4.2 E-5 MUST elevation rationale (D-14c)

- **OWASP ASVS V2.10.1**: "Verify that secrets are not kept in resident memory longer than necessary." Direct mapping to C48-M3a.
- **NIST SP 800-57 Part 1 §8.2.2**: Key-destruction requirements — "when a cryptographic key is no longer needed, it shall be destroyed." Direct mapping to C48-M3a + C48-M3d.
- **Dual-track**: Plan48 is the primary implementation track; `Research_Methodology/10_ENG_FAB_v1.7_Candidate.md` E-5 MUST (item B.5 E-5) is the forward-enforcement track.
- **δ closure**: C48-M3 completion closes the E-5 δ item; combined with C48-M1 (F-L3-SEC-2 closure) and C48-M2 (F-L3-SEC-3 closure), Plan48 delivery + W2-R13 runtime verification = **δ DONE** (subject to timing Candidate B: Plan48 close-out AND W2-R13 PASS).

---

## §5 Aggregate Roll-Up

**Plan48 delivery PASS iff all 3 MUST-heading roll-ups PASS**.

| MUST-heading | Sub-item count | Roll-up | Aggregate contribution |
|--------------|:--------------:|:-------:|:----------------------:|
| C48-M1 | 8 | PASS iff 8/8 | Required |
| C48-M2 | 9 | PASS iff 9/9 | Required |
| C48-M3 | 4 | PASS iff 4/4 | Required |
| **Plan48 aggregate** | 21 sub-items + 3 MUST headings + 1 aggregate | PASS iff 3/3 MUST-heading PASS | — |

---

## §6 Interactions with MR-6, Tenet #2/#7/#8, and Rule #75

### §6.1 MR-6 Core zero-policy audit

- **Layer placement**: every sub-item's implementation is on **runner / plugin-adjacent layer**, NOT Core. Core policy constants remain at **53** (canonical per KERNEL itemization); Plan48 adds **zero** Core policy constants.
- **Verification**: Research-team L1 audit runs `grep` over `packages/core/**` comparing Plan47 baseline vs Plan48; delta must be zero policy constants. Documented in Plan48 delivery report.

### §6.2 Tenet alignment

| Tenet | How Plan48 sub-items contribute |
|-------|----------------------------------|
| **#2 契約一致性** (contract consistency) | C48-M1b JSON-line schema + C48-M2b dedup ordering = deterministic contracts over log + audit event types |
| **#7 絕對純淨 δ 邊界** (absolute purity at δ boundary) | All three MUST items close δ-2 audit gap; MR-6 Core zero preserved |
| **#8 控制理論閉環** (control-theory closed-loop) | C48-M1e + C48-M2g shutdown flush + C48-M3a key clear = deterministic shutdown closure |

### §6.3 Rule #75 Pre-Delivery Gate integration

Per Rule #75 §75.3.2 (binding acceptance-criteria satisfaction), Plan48 dev delivery report MUST explicitly reference each of the 25 atomic criteria and attest PASS with runtime evidence where required. Dev failure to attest any criterion → R → Dev transfer blocked per §75.4.1; research-side F-10 audit catches any dev-missed item per §75.4.2.

### §6.4 ENG-FAB v1.7 alignment

- F-9 sub-check (i) module+test → every sub-item's unit-test file referenced.
- F-9 sub-check (ii) doc+CHANGELOG → C48-M1g, C48-M2i, C48-M3c doc items.
- F-9 sub-check (iii) runtime evidence → C48-M1f, C48-M2h, C48-M3d evidence items.
- F-10 Pre-Delivery Gate Verified → attestation of Rule #75 §75.3.1–§75.3.5 in delivery-report Section 0.

---

## §7 Status and Binding Annex

**Status**: **BINDING** per R3 D-01(c) + D-14(c) compound.

This document is the **binding annex** attached to Master Ratification Request **Batch 8a** (`Master_Ratification_Plan48_Scope.md`). Master ratification of Batch 8a = ratification of this table verbatim.

**Upon Master ratification**:
- Plan48 dev-team uses this table as the canonical acceptance-criteria contract.
- Plan48 delivery report Section 0 lists all 25 criteria with PASS/FAIL + evidence reference per criterion.
- Plan48 research-team independently audits each criterion.
- Plan48 test-team W2-R13 runtime executes the 5 runtime-evidence criteria (C48-M1f, C48-M2h, C48-M3d + shutdown cascade flush in C48-M1e + C48-M2g).

**Until Master ratification**: this table is the research-team's authoritative plan and shared with dev-team as a binding specification draft; dev-team is strongly encouraged to build against this table to minimise rework.

---

## §8 Dissent (MR-11 preservation)

- **D-01(c) δ boundary 中間 SUSSMAN β** — 15 (c) / 5 (b) / 3 (a); 8 non-chair dissents preserved. (b) 廣義 lens would have classified E-5 as part of δ proper; (a) 狹義 lens would have kept E-5 outside δ without MUST elevation. SUSSMAN β middle path adopted: δ wording unchanged (狹義 preserved), E-5 elevated MUST on independent defense-in-depth grounds (not as part of δ). This is the binding that makes E-5 MUST dual-track coherent with this 25-sub-item list. See `R3_decision_log_HIGH.md` §D-01 for 24-agent vote detail.
- **D-14(c) E-5 dual-track** — UNANIMOUS 23/0 (non-chair); no dissent. Plan48 primary implementation (25 sub-items below) + ENG-FAB v1.7 F-E-5 forward enforcement. See `R3_decision_log.md` §2.2.
- **Plan48/49 split** — not a R3 D item; converged at R2 (UNANIMOUS R2 recommendation via §5 R2 §10.1 + §13.2 MUST-only + SHOULD supplement pattern); R3 did not re-open. Rationale: S-1 (500-1000 LOC) guideline + MR-12 back-fill priority + MR-9 quality-first.
- **MRB-11 count (21 vs 25)** — R3 D-05 resolved by SYNTHESIST + ARCHIMEDES disambiguation: 21 sub-items + 3 MUST headings + 1 aggregate = 25 atomic acceptance criteria. See §1.1.

## §9 Author attribution

- **Primary drafters**: SYNTHESIST (#1) + ARCHIMEDES (#16) — MRB-11.
- **E-5 compliance review**: GUARDIAN (#11) — ASVS + NIST.
- **W2-R13 runtime edge-case review**: HERACLITUS (#15).
- **Sub-item ID discipline**: SCRIBE (#2).
- **Coordination**: SUNYATA (#0).
- **R3 votes** (ground truth `R3_decision_log.md §2.1/§2.2`): D-01(c) 15 (c) / 5 (b) / 3 (a); D-14(c) 23/0 UNANIMOUS; D-05 UNANIMOUS on MRB-11 disambiguation; D-17(a) HMAC rotation design-spec-only 23/0 UNANIMOUS.
