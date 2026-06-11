---
title: Plan52 — pushInput Candidate B Binding Engineering Spec
author: TANENBAUM (#20 microkernel) + KERNEL (#10 OS) + GUARDIAN (#11 security) + ARCHIMEDES (#16 engineering) + SUNYATA (#0 chair) + SYNTHESIST (#1 aggregator)
date: 2026-04-25
cycle: 03-14
status: BINDING + SHIPPED (v0.50.0-alpha, cycle 03-14; ratification confirmed via TW sibling parity record cycle 03-20, but EN write-back was never performed — header corrected 2026-06-11 repair audit)
authority: research-team (R4 final draft); Master (ratification)
supersedes:
  - research record/cycle03-12/deliver/O4_Plan50_pushInput_R4_engineering_spec.md (cycle 03-12 spec; pushInput renumbered Plan50 → Plan52 per cycle 03-14 R3 D-§1-01 Option B)
cross_refs:
  - research record/cycle03-14/deliver/O1_pushInput_Plan52_final.md (816 lines authoritative source)
  - research record/cycle03-14/R1/§1_pushInput/§1_pushInput_R1.md (830 lines)
  - research record/cycle03-14/R2/§1_pushInput/§1_pushInput_R2.md (852 lines)
  - research record/cycle03-14/R3/R3_decision_log.md §1 + §4 + §7 (10 D-items + 4 MRB)
  - research record/cycle03-14/deliver/Master_Ratification/Batch_11_Request.md §3.6 (Item #6)
  - research record/cycle03-13/openstarry_doc/Technical_Specifications/Plan50_pushInput_CP4_Invariant.md (cycle 03-12 CP-4 invariant; preserved)
binding_until: Master Ratification Batch 11 close
---

# Plan52 — pushInput Candidate B Binding Engineering Spec

## 1. Status

**CANDIDATE** at cycle 03-14 R4 close. Upgrades to **BINDING (Plan-level engineering spec)** if Master ratifies Batch 11 Item #6. Plan52 is the **first Phase 6 functional implementation** of the pushInput source-authentication architecture.

> **[2026-06-11 repair audit]** Code shipped in v0.50.0-alpha (cycle 03-14) and later plans cite Plan52 as the shipped baseline; the CANDIDATE→BINDING status upgrade was never written back to this document. Status corrected to **SHIPPED**.

**Architecture preservation**: the architecture (Candidate B + B' deferred + CP-1/2/3/4 + CR-SCK + CR-PARETO) is preserved verbatim from the cycle 03-12 R4 spec, which Plan52 supersedes for **plan-numbering only**. Editorial-only renumbering is permitted per Master 通則 ("符合十大宣言與規則就可以直接做了"; numbering is operational labelling, not Tenet/MR/ZT/structural).

## 2. R3 Provenance (10 D-items + 4 MRB)

- **D-§1-01 (A-1)** 22/1/0 — Plan number Option B (cycle 03-12 Plan50 → Plan52); DSS-3 LEIBNIZ Plan51 sunset minority preserved §10.
- **D-§1-02 (A-2)** 18/5/0 — H1 B-only Plan52 + Plan53 reserved; DSS-1 LEIBNIZ + RUSSELL multi-agent symmetry preserved §10.
- **D-§1-03 (B-1)** 22/1/0 — HMAC-SHA256 default + threat-model amendment; DSS-13 1-vote Ed25519 default for PQ migration preserved §10.
- **D-§1-04 (B-2)** UNANIMOUS 23/0 — tri-party MR-6 audit + automated CI defense-in-depth.
- **D-§1-05 (B-3)** 21/2/0 — deepFreeze recursive immutability; DSS-14 KNUTH + ATHENA shallow-freeze perf concern preserved §10.
- **D-§1-06 (B-4)** UNANIMOUS 23/0 — nonce TTL ≥ key-rotation overlap MUST.
- **D-§1-07 (C-1)** UNANIMOUS 23/0 — F-14 call-graph depth N=3.
- **D-§1-08 (C-2)** UNANIMOUS 23/0 — cross-OS CI matrix mandatory (Linux + Windows).
- **D-§1-09 (C-3)** UNANIMOUS 23/0 — local-CLI tokenSig MAY (process-local UID 已足).
- **D-§1-10 (D-1)** UNANIMOUS 23/0 — property tests Plan53 graduation slot.
- **MRB-§1-01..04** RESOLVED at R3.
- **CV-§1-01..13** reaffirmed (architecture invariants from cycle 03-12).

## 3. Plan Number Resolution (R3 D-§1-01 Option B)

R3 ratified Option B at cycle 03-14 R3 close: **pushInput → Plan52**; cycle 03-13 Plan50 stays bound to σ_regime in-place revise (per cycle 03-13 Master Ratification Batch 10 #8); **Plan51 reserved as gap-buffer**. The single dissent vote (LEIBNIZ minority) preferred Option A (collapse gap-buffer to Plan51); preserved at DSS-3 §10.

**Plan number canonical mapping** (forward references):

| Plan | Subject | Cycle ratified | Status |
|------|---------|----------------|--------|
| Plan49 | Plan49 MR-6 conditional gates | cycle 03-13 Batch 10 #9 | EXECUTED (cycle 03-14 Dev) |
| Plan50 | σ_regime in-place revise | cycle 03-13 Batch 10 #8 | BINDING (refined cycle 03-14 Batch 11 #7) |
| Plan51 | reserved gap-buffer | cycle 03-14 R3 | RESERVED |
| **Plan52** | **pushInput Candidate B** | **cycle 03-14 R3 / Batch 11 #6** | **CANDIDATE → BINDING upon ratification** |
| Plan53 | reserved B' deferred path | cycle 03-14 R3 D-§1-02 | RESERVED |

**Editorial errata propagation** (per F-03-14-§1-R2-01): SCRIBE propagates a one-line errata note to all cycle 03-12 references to "Plan50 = pushInput", logged in `discussions/cycle03-14/SCRIBE_plan52_errata_propagation_log.md` and verified at G4-folder-3 / G5 sync.

## 4. Architecture Summary (Candidate B)

### 4.1 ε-Surface (verbatim from cycle 03-12 D-09(b))

Core surface delta for Plan52 implement-round:

```
Core surface delta (Plan52 implement-round):
  + 1 optional field:    InputEvent.sourceContext?: Record<string, unknown>
  + 1 Zod passthrough:   z.record(z.unknown()).optional()
Core surface NOT added:
  - IAuthenticator interface           (would be MR-6 FAIL)
  - SourceIdentity type                (would be MR-6 FAIL)
  - verify(): boolean method           (would be MR-6 FAIL + Tenet #7 violation)
```

ε-surface = 1 optional field, 0 policy constants. CV-§1-01..02 reaffirm.

### 4.2 Transport Plugin Attests; sourceContext Opaque Passthrough

The transport plugin (e.g., HTTP / IPC / pushInput pull-mode) is responsible for source authentication. Core does NOT inspect or interpret `sourceContext`; it is opaque passthrough. The plugin attests to `sourceContext` correctness; Core stores and forwards.

This preserves Tenet #7 (絕對純淨) — Core does NOT contain authentication policy code; authentication is plugin-delegated (Tenet #2 一切皆插件).

### 4.3 CP-1 / CP-2 / CP-3 / CP-4 Invariants

(Preserved verbatim from cycle 03-12 spec; canonical reference: `cycle03-13/openstarry_doc/Technical_Specifications/Plan50_pushInput_CP4_Invariant.md` — file is preserved as cycle 03-13 baseline mirror despite the renumbering, since CP-4 invariant text is architecture-stable.)

- **CP-1** — sourceContext is plugin-attested, never Core-validated
- **CP-2** — Core trust boundary terminates at plugin entry
- **CP-3** — sourceContext shape is plugin's contract; Core has no shape opinion
- **CP-4** — ASANGA-authored + TANENBAUM-COMPATIBLE; zero Core cost (sourceContext immutable downstream of plugin attestation; deepFreeze enforces)

### 4.4 CR-SCK + CR-PARETO

- **CR-SCK** — SDK-exported `RecommendedSourceContextKeys` enumeration for plugin authors (preserved verbatim from cycle 03-12).
- **CR-PARETO** — three-tier red-line taxonomy + standing R3 procedure for sourceContext key promotion (preserved verbatim from cycle 03-12).

## 5. Signed-Token Specification (D-§1-03 22/1)

### 5.1 HMAC-SHA256 SHOULD Default

R3 ratified HMAC-SHA256 as the SHOULD default for pushInput plugin-attested signed tokens:

- **Algorithm**: HMAC-SHA256 (FIPS 140-2/3 approved).
- **Key length**: ≥ 256 bits.
- **Token format**: `<nonce>:<timestamp>:<HMAC-SHA256-base64-of-payload>`.
- **Verification**: plugin verifies HMAC; computes constant-time comparison.

Threat-model amendment (D-§1-03 absorption):

- **Replay attack**: nonce + timestamp tracked in plugin-side replay cache; nonce TTL constraint per §5.3.
- **Timing attack**: constant-time HMAC comparison.
- **Key compromise**: key rotation per existing HMAC compliance baseline (cycle 03-12 ratified).

### 5.2 Ed25519 Documented Alternative

Per DSS-13 (1-vote Ed25519 default for post-quantum migration), Ed25519 is documented as an acceptable alternative for plugins that elect post-quantum migration readiness:

- **Algorithm**: Ed25519 (signature, not HMAC) — public-key signature.
- **Key length**: 256 bits (curve definition).
- **Migration path**: plugins MAY choose Ed25519 from initial implementation; HMAC is the recommended default for cycle 03-14+ but not exclusive.
- **Cross-plugin compatibility**: plugins MUST declare the algorithm in their manifest; Core does NOT impose a single algorithm.

### 5.3 Nonce TTL ≥ Key-Rotation Overlap MUST (D-§1-06 UNANIMOUS)

The nonce TTL (time-to-live in plugin-side replay cache) MUST be ≥ the key-rotation overlap window (the period during which old and new keys are both valid during rotation). This prevents a stale-nonce-acceptance vulnerability during key rotation.

- Default key-rotation overlap: 24h (per cycle 03-12 HMAC compliance baseline).
- Default nonce TTL: 24h ≤ TTL ≤ 7d (operational discretion within the constraint).
- Plugin manifest documents both values; CI verifies invariant.

### 5.4 Local-CLI tokenSig MAY (D-§1-09 UNANIMOUS)

For plugins running in local-CLI mode (e.g., Dev-side test harness), tokenSig generation is **MAY** (not MUST). Process-local UID is sufficient for local trust (the OS process boundary itself is the trust delimiter). This concession reduces local-Dev friction; production / cross-process modes retain MUST.

## 6. Tri-Party MR-6 Audit + Automated CI (D-§1-04 UNANIMOUS)

### 6.1 Tri-Party Human Audit

Three personas must MR-6 audit Plan52 implementation Pull Request before merge:

- **TANENBAUM (#20)** — microkernel discipline; verifies ε-surface = 1 optional field; no Core policy constants added.
- **KERNEL (#10)** — OS / process-discipline; verifies trust boundary semantics.
- **GUARDIAN (#11)** — security; verifies threat model coverage; verifies HMAC implementation correctness.

Audit results recorded as 3 PR review comments (one per persona) plus a single CI status check that aggregates the three.

### 6.2 Automated CI Defense-in-Depth

In addition to tri-party human audit, CI runs:

- **MR-6 surface diff**: enumerates Core (`packages/core/*`) surface changes; FAILs if any new policy-class addition (interface / type / method) is detected.
- **F-14 call-graph audit**: depth N=3 per D-§1-07; verifies no Core-side `sourceContext` interpretation.
- **F-15 impact assessment**: code-path verified before assessment per ENG-FAB v1.8 baseline.
- **HMAC test vectors**: standard FIPS test vectors against implementation.

### 6.3 Why Both?

Tri-party human audit catches semantic / architectural drift; automated CI catches mechanical / surface drift. Both are required because each catches a different drift class. Defense-in-depth.

## 7. deepFreeze Recursive Immutability (D-§1-05 21/2)

`sourceContext`, once received from the plugin, MUST be deepFrozen (recursive `Object.freeze`) before being forwarded into Core event flow. This enforces CP-4 (sourceContext immutable downstream of plugin attestation; zero Core cost since freeze is one-time at boundary).

DSS-14 (KNUTH + ATHENA shallow-freeze perf concern) absorbed: the perf cost is bounded by sourceContext shape (typically < 10 keys); deepFreeze is a one-time O(n) walk at plugin boundary, not per-event hot-path. Profiling expected to show negligible overhead; if Plan52 implementation surfaces unexpected hot-path cost, fall-back to shallow-freeze + downstream defensive-copy is a documented amendment route.

## 8. F-14 Call-Graph Depth N=3 (D-§1-07 UNANIMOUS)

CI runs an F-14 call-graph audit on the merged Plan52 PR with depth N=3:

- **Depth 1**: direct callers of `sourceContext` field accessor in Core.
- **Depth 2**: callers of depth-1 functions.
- **Depth 3**: callers of depth-2 functions.

The audit reports any function in depth ≤ 3 that branches on `sourceContext` content; expected count = 0 (Core does not interpret sourceContext; it is opaque passthrough). Any non-zero finding is a MR-6 violation triggering review.

## 9. Cross-OS CI Matrix (D-§1-08 UNANIMOUS)

Plan52 CI MUST run on:

- **Linux** (Ubuntu LTS or equivalent; matches production typical deployment)
- **Windows** (Windows Server 2022 or equivalent; matches Dev workstation typical environment)

Both runners must pass `pnpm install --frozen-lockfile && pnpm build && pnpm test`. Cross-OS CI catches:

- Path-separator bugs (Windows `\` vs Linux `/`)
- Line-ending bugs (CRLF vs LF in test fixtures)
- Process-creation differences (Windows fork semantics vs Linux fork/exec)

This complements Rule #75 §75.X release-tag pnpm build (Batch 11 #4) at the per-PR level.

## 10. W2-R15 Sanity Spec (HERACLITUS 3 Sub-Conditions)

Test team to apply 3 HERACLITUS runtime-dynamics edge-case sub-conditions to W2-R15 sanity check:

1. **Plugin-load-time signature verification edge case** — verify plugin cannot reach Core if signature fails at load time.
2. **Concurrent pushInput event ordering** — verify Core preserves ordering across concurrent plugin emissions.
3. **Key rotation in-flight** — verify in-flight events accepted under old key during rotation overlap window; verify rejection after overlap window expires.

Detailed test instructions in `research record/cycle03-14/test_instructions/W2-R15_test_instructions.md` (HERACLITUS authored).

## 11. LOC Budget

- **Production code**: 200-400 LOC (estimated)
  - HMAC verify: ~80 LOC
  - Nonce cache: ~60 LOC
  - sourceContext deepFreeze: ~30 LOC
  - Schema validation: ~40 LOC
  - Plugin manifest extension: ~20 LOC
  - Error handling (F-16 SHOULD per Batch 11 #1): ~30 LOC
- **Test code**: 100-200 LOC
  - HMAC test vectors: ~50 LOC
  - Nonce replay test: ~30 LOC
  - Cross-OS CI matrix duplication factor: ×2 effective coverage
  - Threat model assertions: ~40 LOC
- **Plan-spec doc**: this file + Dev-facing concise spec in `cycle03-14/todo/Plan52_pushInput_dev_spec.md`

## 12. Implementation Order

Per cycle 03-14 R4 final dependency analysis:

1. **Plan49 SHOULD landed** (cycle 03-13 R4 ratified; Dev cycle 03-14 execution complete) — provides Pre-Delivery Gate baseline for Plan52.
2. **Plan50 σ_regime in-place** (cycle 03-13 binding + cycle 03-14 Batch 11 #7 atomicity refinement) — provides σ_regime emission discipline; Plan52 pushInput emissions tagged `composition_index` regime per CV-§1-08.
3. **Plan52 pushInput** — first Phase 6 functional implementation; this spec.
4. **Plan53 (reserved B' deferred path)** — future cycle if multi-agent symmetry need surfaces (DSS-1 absorbed via reservation).

## 13. Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 / MR-4 (Tenet wording unchanged) | ✅ PASS | No Tenet wording proposed |
| MR-5 hard (Tenet #10 status unchanged) | ✅ PASS | Plan52 advances 1 of 7 Phase 6 functions only; endpoint 10/0/0★ unchanged |
| MR-6 (Core 零) | ✅ PASS | ε-surface = 1 optional field, 0 policy constants; tri-party MR-6 audit + automated CI defense-in-depth |
| MR-7 (audit at right moment) | ✅ PASS | Pre-Delivery Gate + release-tag (Rule #75 §75.X per Batch 11 #4) |
| MR-8 (quality first) | ✅ PASS | Tri-party + automated CI; cross-OS matrix |
| MR-9 (no MUST WAIVE) | ✅ PASS | All MUST elements honoured |
| MR-10 (back-fill) | ✅ PASS | Plan53 reserved for B' multi-agent symmetry; DSS preservation |
| MR-11 (dissent preservation) | ✅ PASS | DSS-1 + DSS-3 + DSS-13 + DSS-14 verbatim §10 |
| MR-12 (既有 not retrofit) | ✅ PASS | Plan49 already shipped not modified; cycle 03-12 Plan50 spec carries editorial errata only |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | No 10-Tenet violation; endpoint unchanged; control-range unaffected |
| Rule #74 L1' | ✅ PASS | code + doc completeness |
| Rule #75 (incl. §75.X amendment per Batch 11 #4) | ✅ PASS | Pre-Delivery Gate + release-tag pnpm build |
| Rule #76 §76.6 | ✅ PASS | Reproducible Calc Mandate honoured |
| Rule #77 (cycle 03-13 ratified) | ✅ PASS | σ_regime conjunct preserved; Plan52 emissions are `composition_index` regime |

## 14. Authority Notes

- **Authority route**: Master ratification (Plan-level engineering spec).
- **Coordinator G5 sync**: upon Master ratification, coordinator syncs spec into canonical `Technical_Specifications/Plan52.md` (per Batch 11 Request §7 mapping).
- **Dev acceptance**: Dev cycle 03-15 kickoff absorbs Plan52 spec; expected Plan52 implementation in cycle 03-15 + 03-16 (LOC budget 200-400 prod + 100-200 test span 1-2 cycles).

## 15. Cross-References

- `research record/cycle03-14/deliver/O1_pushInput_Plan52_final.md` — full 816-line authoritative engineering spec source
- `research record/cycle03-14/R3/R3_decision_log.md §1 + §4 + §7` — 10 D-items + 4 MRB + dissent
- `research record/cycle03-14/deliver/Master_Ratification/Batch_11_Request.md §3.6` — Item #6 dispatch
- `research record/cycle03-13/openstarry_doc/Technical_Specifications/Plan50_pushInput_CP4_Invariant.md` — CP-4 invariant baseline (cycle 03-12 ratified; preserved by file name despite renumbering for historical traceability)
- `research record/cycle03-14/test_instructions/W2-R15_test_instructions.md` — W2-R15 test plan
- `research record/cycle03-14/todo/Plan52_pushInput_dev_spec.md` — Dev-facing concise spec (G4-folder-4)

## 16. Dissent Slots — DSS-1 + DSS-3 + DSS-13 + DSS-14 Verbatim (per MR-11)

### 16.1 DSS-1 (D-§1-02, 5 minority votes — LEIBNIZ + RUSSELL + 3)

> Multi-agent symmetry preference for B + B' simultaneous implementation:
>
> "From multi-agent coordination perspective (LEIBNIZ #14) and AI-agent theory perspective (RUSSELL #23), B + B' simultaneous implementation provides multi-agent symmetry — a parent agent and a child agent both push input through the same architecture without one being privileged. The H1 B-only path (Plan52) defers B' to Plan53 reservation, which means cycle 03-14+ cycles operating exclusively on B may calcify the architecture around single-agent assumptions, making future B' integration harder."
>
> R3 majority (18/5) preferred B-only Plan52 + Plan53 reservation on grounds:
> - Plan52 LOC budget is tight (200-400 prod); B + B' simultaneous would push toward 500+ LOC and trigger S-1 boundary review.
> - Plan53 reservation explicitly preserves B' route; calcification concern mitigated by reservation.
> - Multi-agent symmetry is genuine but is a Phase 7 multi-IVolition concern; cycle 03-14 Phase 6 first-step needn't tackle multi-agent symmetry now.

### 16.2 DSS-3 (D-§1-01, 1 minority vote — LEIBNIZ)

> LEIBNIZ Plan51 sunset preference:
>
> "Plan51 reservation as 'gap-buffer' is structurally an empty placeholder. Reserved plan numbers historically calcify into permanent placeholders that never resolve. Recommend an explicit 6-month sunset clause on Plan51 — if Plan51 is not assigned a subject by cycle 03-20, the reservation lapses and Plan51 number is reclaimable for any future Plan."
>
> R3 majority (22/1) preferred Plan51 reservation without sunset:
> - Gap-buffer pattern has historical precedent (cycle 03-7 Plan-32.5 emergency-insert).
> - Sunset clause adds a future-Master-decision burden without immediate benefit.
> - DSS-3 is preserved; future cycle adjudication may revisit if Plan51 indeed lingers.

### 16.3 DSS-13 (D-§1-03, 1 minority vote)

> Single dissent in favour of Ed25519 default for post-quantum migration:
>
> "HMAC-SHA256 is currently FIPS-approved but not post-quantum-safe. Ed25519 is also not PQ-safe in the strict sense (Shor's algorithm threatens both EdDSA and HMAC), but the migration path from Ed25519 to a PQ-safe signature (e.g., Dilithium / SPHINCS+) is cleaner since Ed25519 is already a public-key signature scheme. HMAC is symmetric; future PQ migration requires replacing the entire authentication primitive class. Recommend Ed25519 default to position for PQ-readiness."
>
> R3 majority (22/1) preferred HMAC-SHA256 default:
> - HMAC-SHA256 is FIPS-140-2/3 approved; deployment-ready in production environments with FIPS compliance requirements.
> - Ed25519 is documented as acceptable alternative; plugins may elect Ed25519 from initial implementation.
> - PQ migration is a Phase 8+ concern; cycle 03-14 first-Phase-6-function need not preempt.

### 16.4 DSS-14 (D-§1-05, 2 minority votes — KNUTH + ATHENA)

> Shallow-freeze performance concern:
>
> "deepFreeze recursive walk is O(n) where n = total key count in sourceContext object graph. For deeply-nested or large sourceContext payloads, recursion may surface as hot-path overhead. Recommend shallow-freeze (top-level only) + downstream defensive-copy discipline as alternative; this preserves CP-4 immutability semantically while avoiding recursive walk cost."
>
> R3 majority (21/2) preferred deepFreeze recursive:
> - sourceContext shape is plugin-controlled; typical shapes are small (< 10 keys, < 3 levels deep); recursive cost is negligible in practice.
> - deepFreeze is one-time at plugin boundary, not per-event hot-path.
> - If Plan52 implementation profiling surfaces unexpected hot-path cost, fall-back to shallow-freeze + defensive-copy is a documented amendment route — preserved here as DSS-14.

All four dissent positions documented per MR-11; majority position adopted with implementation-level mitigation maps where relevant.

---

*Cycle 03-14 R4 final — 2026-04-25*
*Authors: TANENBAUM (#20) + KERNEL (#10) + GUARDIAN (#11) + ARCHIMEDES (#16) + SUNYATA (#0) + SYNTHESIST (#1)*
*Status: CANDIDATE (pending Master Ratification Batch 11 #6)*
*Plan number resolution: cycle 03-12 Plan50 → Plan52 per cycle 03-14 R3 D-§1-01 Option B*
