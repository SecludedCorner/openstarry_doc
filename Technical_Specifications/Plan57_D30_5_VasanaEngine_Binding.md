# Plan57 — D-30-5 VasanaEngine BINDING Specification

**Status**: BINDING (cycle 03-19 R3 D-§1 ratified 22/1 super-majority; pending Master Ratification Batch 16 #1)
**Authority**: Master Ratification (Batch 16 dispatch 2026-05-01)
**Cycle**: 03-19 (Phase 6 第四棒; 4/7 functional landing target)
**Release**: v0.54.0-alpha minor-bump
**Plan number**: 57
**Subject**: D-30-5 VasanaEngine — vasanā (習氣) deposit-only passive-observer plugin

**Inheritance chain**: Plan52 pushInput (cycle 03-14) → Plan54 AC-9 (cycle 03-17) → Plan56 D-30-4 multi-volition (cycle 03-18) → **Plan57 D-30-5 VasanaEngine (cycle 03-19)**

---

## §1 Background

D-30-5 VasanaEngine implements **vasanā** (Sanskrit: 習氣, "habit-energy traces") as a **passive-observer deposit log** within the OpenStarry五蘊 architecture. Per唯識八識 mapping, vasanā corresponds to seed-impressions (bīja) deposited in 阿賴耶識 (8th consciousness) by past karma-actions (volitions, perceptions, etc.).

**Architectural relationship to D-30-4 multi-volition (Plan56)**:
- D-30-4 (Plan56) = 現行 (manifest activity) — multi-stream concurrent volition queue
- D-30-5 (Plan57) = 種子化痕跡 (seed-trace deposit) — passive observer recording impressions

**MR-6 鐵律 compliance**: Plan57 is plugin-layer ONLY; ε-surface 0-delta vs Plan52 baseline (0 fields added, 0 const added).

---

## §2 Architecture: Option C dual-track passive-observer deposit log

Per cycle 03-19 R3 D-§1 ratified 22/1 super-majority (DSS-CY19-§1-A LEIBNIZ Option C+ stub variant minority preserved):

### §2.1 Plan52/54/56 isomorph 10-dimension verbatim

| Dimension | Plan52 | Plan54 | Plan56 | **Plan57** |
|-----------|--------|--------|--------|------------|
| Plugin layer | sourceContext path | sub-agent spawn | volition queue | **vasanā deposit log** |
| Core surface delta | 0 fields/const | 0 fields/const | 0 fields/const | **0 fields/const** |
| ε-surface check | 7-sub-check ci_check | 7-sub-check ci_check | 7-sub-check ci_check | **7-sub-check ci_check (inheritance)** |
| Tri-party MR-6 audit | TANENBAUM+KERNEL+GUARDIAN | TANENBAUM+KERNEL+GUARDIAN | TANENBAUM+KERNEL+GUARDIAN | **TANENBAUM+KERNEL+GUARDIAN** |
| HMAC-SHA256 + nonce | yes | yes | yes | **yes** |
| Replay cache | 1-contributor | 2-contributor | 3-contributor | **4-contributor** |
| Cross-OS CI matrix | Linux+Windows | Linux+Windows | Linux+Windows + Tier δ | **Linux+Windows + Tier δ extension** |
| F-13/F-14/F-15 v3 reflexive | PASS | PASS | PASS | **PASS** |
| TW sibling Rule #78 BINDING-tier | yes | yes | yes | **yes** |
| LOC ceiling | (varies) | 800/500 | 900/600 | **900/600 (same as Plan56)** |

### §2.2 Dual-track architecture

**Track 1 — Deposit-only (THIS cycle 03-19 IN-SCOPE)**:
- Append-only `vasana_log[]` per agent instance
- Each entry: `{volition_id, deposit_time_utc, content_redacted, hmac_signature, nonce, prev_hash}`
- HMAC-chain integrity (each entry's prev_hash = SHA-256(prev_entry))
- Boot-time + runtime append-only re-verification (D-§1-R2-E)
- Refuse-to-start on integrity violation

**Track 2 — Read-API (DEFERRED to Plan60 Blackboard-Alaya consumer)**:
- NOT IMPLEMENTED this cycle
- Reserved interface stub (LEIBNIZ DSS-CY19-§1-A Option C+ minority preference)
- Forward-only commitment per D-§1-clarify-A (MR-12)

**Yogācāra discipline foreclosure modes** (5/5 preserved):
1. No central registry (vasanā deposit per agent, not global)
2. No vasanā-as-entity (always conditioned bīja, never svabhāva)
3. No retroactive modification (append-only)
4. No current-emit coupling (passive observer; deposit ≠ active influence)
5. No ε-surface extension (plugin-layer only)

### §2.3 SICP-canonical Black-box public API minimality

Per SUSSMAN R2 cross-review §2.4: minimal public API surface (4 methods only):
- `deposit(volition_id, content, secret_key)` — append entry; return entry hash
- `verify_chain(start_idx?, end_idx?)` — runtime integrity check
- `count()` — entry count (no content access)
- `latest_hash()` — for external attestation

Track 2 read-API (`peek`, `iterate`, `query_by_volition_id`) DEFERRED.

---

## §3 ε-surface 0-delta Strict Equality (7-sub-check inheritance)

Verbatim inheritance from cycle 03-17 D-§1-R2-B + cycle 03-18 D-§1 carry-forward:

1. No new file in `core/`
2. No new exported symbol from `core/index.ts`
3. No new interface field in `core/types.ts`
4. No new const/enum value in `core/constants.ts`
5. No new lexical token (extended 13-token corpus per cycle 03-17 KERNEL §2.3)
6. No new Core → plugin imports
7. No new transitive type re-exports

`ci_check_core_surface_freeze` 7-sub-check conjunction enforced. Frozen baseline at v0.53.0-alpha boundary; re-attestation at v0.54.0-alpha (D-§1-R2-C carry-forward).

---

## §4 Tri-party MR-6 Sign-off

Per cycle 03-17 D-§1-R2-A inheritance:
- **TANENBAUM** (Plan-spec authority) — architectural soundness sign-off
- **KERNEL** (Core surface authority) — ε-surface 0-delta verification
- **GUARDIAN** (security authority) — HMAC + nonce + replay cache + redaction format

Markdown 4-column table + Git GPG-signed commit per persona; persisted to `delivery_report.md` § "Tri-Party MR-6 Sign-Off — Plan57".

---

## §5 Replay Cache 4-contributor Structured Prefix Table

Per D-§1-R2-A 23/0 UNANIMOUS:

| Contributor | Plugin | Prefix | Topology |
|-------------|--------|--------|----------|
| 1 | Plan52 pushInput | `psh:` | single-process |
| 2 | Plan54 AC-9 | `ac9:` | single-process; multi-process opt-in |
| 3 | Plan56 D-30-4 | `mvq:` | single-process; multi-process opt-in |
| **4** | **Plan57 D-30-5** | **`vsn:`** | **single-process; multi-process opt-in** |

Refuse-to-start on topology mismatch; refuse-to-start on prefix collision.

---

## §6 Deposit Redaction Format Codification

Verbatim cycle 03-18 D-§1-R2-B inheritance (with KERNEL DSS-CY19-§1-C N=8 hex preference preserved as carryover from DSS-CY18-02):

**Redaction format**: `<redacted-vasana-deposit len:NN first4:abcd>`
- `NN` = original payload byte length (decimal)
- `abcd` = first 4 alphanumeric characters of payload (forensic hint)

**Category-aware sensitivity**:
- HIGH-sensitivity vasanā categories (intent / preference / aversion): tightest redaction
- MED categories (action-trace / observation): standard redaction
- LOW categories (timestamp / source-ref): minimal redaction

---

## §7 Windows Fallback HMAC-chain Compensating Control

Per D-§1-R2-C 23/0 UNANIMOUS:

When Windows platform atomic deposit log primitives differ from POSIX (`O_APPEND` semantics; `fsync` behavior), the HMAC-chain provides **equivalent integrity guarantee** via cryptographic linkage. No POSIX-specific atomic-append assumptions in the spec.

---

## §8 Plan52 6-invariant Boundary Attestation Explicit

Per D-§1-R2-D 23/0 UNANIMOUS (cycle 03-18 inheritance):

1. No new emit-time hook
2. No new envelope schema variant
3. No new dispatch ordering invariant
4. No new sourceContext metadata field outside `Record<string, unknown>` open extension
5. No new InputEvent timestamp tolerance contract
6. No new emit-recipient-set semantics

Plan57 deposits flow through Plan52 sourceContext metadata path; **0 invariant violation**.

---

## §9 Boot+Runtime Append-only Re-verification

Per D-§1-R2-E 22/1 (KERNEL DSS-CY19-§1-B deposit backend memory default minority preserved):

**Boot-time**:
- Verify deposit log integrity (hash-chain validation from log[0] → log[N])
- Refuse-to-start on integrity violation
- Default backend = file-based (KERNEL minority preferred memory; not blocking)

**Runtime**:
- Re-verify on every deposit emit (incremental hash check)
- Refuse to deposit on chain-corruption detection

---

## §10 Forward-only Commitment + Reflexive MR-6 Counting

### §10.1 Forward-only commitment (D-§1-clarify-A 23/0)

Plan60 Blackboard-Alaya read-API extension (Phase 6 7th棒) MUST be forward-only addition to Plan57. **No retrofit of Plan57 deposit-only design**. Per MR-12 forward-only.

### §10.2 Reflexive MR-6 鐵律 counting (D-§1-clarify-B 23/0)

Plan57 = **3rd consecutive Phase 6 functional landing maintaining MR-6 鐵律**:
- Plan52 (cycle 03-14) → 1st
- Plan54 (cycle 03-17) → 2nd
- Plan56 (cycle 03-18) → not consecutive (different baton; γ retrofit also applies)
- Plan57 (cycle 03-19) → **3rd consecutive functional landing per Phase 6 7-list anchor**

---

## §11 LOC Trajectory + Same-Cycle Implementation

| Checkpoint | When | prod | test |
|-----------|------|------|------|
| **CP-1** Pre-Dev kickoff (R4 close 2026-05-01) | NOW | indicative ~580 / ceiling 900 | indicative ~410 / ceiling 600 |
| **CP-2** Dev mid-cycle | 2026-05-02~03 | drift > 10% reassess | drift > 10% reassess |
| **CP-3** Pre-tag v0.54.0-alpha | 2026-05-04 | hard ≤ 900 (waiver if < 10% overshoot) | hard ≤ 600 (waiver if < 10% overshoot) |
| **CP-4** Post-implementation | post-shipping | Dev fills `delivery_report.md` actual | Dev fills actual |

**Buffer at CP-1**: prod 36% / test 32% (well below ceiling).

---

## §12 Dissent Preservation per MR-11

3 NEW DSS-CY19 entries verbatim preserved:

- **DSS-CY19-§1-A** (LEIBNIZ, 1 vote): Option C+ stub variant — add minimal read-API stub at deposit-only layer for future Plan60 consumer; cycle 03-19+ extension preserved
- **DSS-CY19-§1-B** (KERNEL, 1 vote): Deposit backend default = memory (in-process) preferred over file-default for first-shipping
- **DSS-CY19-§1-C** (KERNEL, 1 vote): N=8 hex preferred for forensic balance; carryover from DSS-CY18-02

All preserved per MR-11 UNCONDITIONAL.

---

## §13 Compliance Audit

| Constraint | Status |
|------------|:------:|
| MR-5 hard (Tenet #10 NC PENDING) | ✅ PASS |
| MR-6 鐵律 (plugin-layer only) | ✅ PASS |
| MR-9 (no MUST WAIVE) | ✅ PASS |
| MR-11 dissent preservation | ✅ PASS (3 NEW DSS-CY19) |
| MR-12 forward-only | ✅ PASS (D-§1-clarify-A) |
| ZT-1/2/3 | ✅ PASS |
| Rule #74/75/76/77/78 | ✅ PASS (Rule #75 §75.X 6th-enforced; Rule #78 TW sibling) |
| ENG-FAB v1.8 = 48 canonical | ✅ preserved |
| ENG-FAB v1.9 = 49 CANDIDATE per OPT-B | ✅ maintained |
| F-15 v3 reflexive PASS | ✅ |
| Phase 6 strict 7-list anchor | ✅ Plan57 = 4th of 7 |

---

*Plan57 D-30-5 VasanaEngine BINDING Specification — cycle 03-19 R3 D-§1 ratified 22/1 — 2026-05-01*
*Master Ratification Batch 16 #1 dispatch ready*
*Inheritance: Plan52 → Plan54 → Plan56 → Plan57 (4/7 Phase 6 functional)*
*Status: BINDING; pending Master Ratification*
