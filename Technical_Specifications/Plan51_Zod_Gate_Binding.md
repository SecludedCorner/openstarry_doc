---
title: Plan51 — Zod Gate × 4-of-5 Modules Candidate (cycle 03-16 First-Shipping)
author: TANENBAUM (#20) + SUSSMAN (#22) + KERNEL (#10) + MESH (#4) + HERACLITUS (#15) + GUARDIAN (#11) + ARCHIMEDES (#16) + SYNTHESIST (#1)
date: 2026-04-27
cycle: 03-15
status: CANDIDATE (pending Master Ratification Batch 12 #2)
authority: research-team (R4 final draft); Master (ratification); Dev (cycle 03-16 implementation)
supersedes: null (first Plan51 ratification)
language: en
cross_refs:
  - research record/cycle03-15/deliver/O4_Plan51_candidate_final.md (898 lines, R4 source)
  - research record/cycle03-15/R3/R3_decision_log.md §4 (D-§5-A/B/C/D/E/F/G/H + per-module §4.5)
  - research record/cycle03-15/deliver/Master_Ratification/Batch_12_Request.md §3.2 (Item #2)
  - research record/cycle03-13/openstarry_doc/Technical_Specifications/Plan49_MR6_Conditional_Gates.md (SCHEMA_DRIFT_MODE substrate)
  - research record/cycle03-14/openstarry_doc/Technical_Specifications/Plan50_Sigma_Regime_Binding.md (sigma_regime enum substrate)
  - research record/cycle03-14/openstarry_doc/Technical_Specifications/Plan52_pushInput_Binding.md (opaque sourceContext invariant)
  - research record/cycle03-13/deliver/O2_plan49_engineering_spec.md MRB-06 (plugin-loader 0-site origin)
binding_until: Master Ratification Batch 12 close
---

# Plan51 — Zod Gate × 4-of-5 Modules Candidate (cycle 03-16 First-Shipping)

## 1. Status

**CANDIDATE** at cycle 03-15 R4 close. Upgrades to **BINDING** upon Master Ratification Batch 12 Item #2. **First-shipping cycle = cycle 03-16**, after the 4-Plan stack: Plan49 SHOULD landed → Plan50 σ_regime → Plan52 pushInput → Plan51 Zod gate.

**MR-12 forward-only honoured iff Plan51 spec explicitly includes per-version migration helpers covering v0.42 onwards** (≥3 prior versions) per D-§5-G UNANIMOUS gate.

**Hard constraints restated**: MR-2 / MR-4 (Tenet text untouched) / MR-5 hard (Tenet #10 NC PENDING preserved) / MR-6 (Core 零 — all 4 推薦 modules peripheral) / MR-9 (no preemptive MUST F-16 binding) / MR-11 (4 dissent entries preserved verbatim §10) / MR-12 (forward-only with cross-version-skew migration helpers MUST per D-§5-G) / ZT-1/2/3.

## 2. R3 Provenance

- **D-§5-A (A-4)** — 推薦 4-of-5 modules (17/6 aggregate); per-module breakdown event-bus 19/3/1 / checkpoint-store 18/4/1 / WebSocket 21/2/0 / hook-registry 17/5/1 / **plugin-loader 9/11/3 (NO super-majority — DEFERRED)**. DSS-A4 6 dissent §10.
- **D-§5-B (B-7)** — 5th-module identity = Option α (5 modules incl plugin-loader as 5th) — 17/4/2; DSS-B7 4 dissent §10. (Note: count-axis Option α; implementation outcome 4-of-5 with plugin-loader DEFERRED operationally aligns with DSS-B7 preference.)
- **D-§5-C (B-8)** — rollout order = **GUARDIAN-priority** (WebSocket → checkpoint-store → event-bus → hook-registry) UNANIMOUS 23/0.
- **D-§5-D (D-3)** — sunset = **sunset-by-execution** UNANIMOUS 23/0 (active for 推薦 path; closes cycle 03-14 DSS-3 LEIBNIZ ossification concern).
- **D-§5-E (C-4)** — hook-registry = **1 module + 2 schema artefacts** (DARWIN Strategy/Registry pattern) — 21/2; DSS-C4 2 dissent §10.
- **D-§5-F (A-5)** — vote granularity = B per-module 5 votes UNANIMOUS 23/0.
- **D-§5-G (B-9)** — **cross-version-skew migration helpers MUST** (HIGH per F-§5-R2-12) UNANIMOUS 23/0.
- **D-§5-H (D-4)** — schema-publication scope DEFER cycle 03-22+ Mesh / 03-23 API Runtime UNANIMOUS 23/0.
- **MRB-§5-01..03 RESOLVED**.
- **CV-§5-01..08** UNANIMOUS reaffirmed (Core 零 / Plan49 SCHEMA_DRIFT_MODE / F-16 SHOULD initial / Plan52 opaque sourceContext / Plan50 σ_regime non-interference / MR-12 forward-only / ZT-1/2/3 / Tenet #10).

## 3. Binding Scope (Plan51 cycle 03-16 First-Shipping)

### 3.1 4 Modules + 1 Shared Utility

| # | Module | LOC median | LOC range | R3 vote | Rollout order |
|:-:|--------|:----------:|:---------:|:-------:|:-------------:|
| 1 | **WebSocket Zod gate** | 125 | 100-150 | 21/2/0 | **#1** (highest GUARDIAN priority) |
| 2 | **checkpoint-store Zod gate** (incl cross-version-skew helpers MUST) | 110 | 85-140 | 18/4/1 | **#2** (disaster-recovery insurance) |
| 3 | **event-bus Zod gate** (CV-§5-05 σ_regime non-interference) | 110 | 85-135 | 19/3/1 | **#3** (Mesh forward-compat hub) |
| 4 | **hook-registry Zod gate** (1 module + 2 schema artefacts per D-§5-E) | 95 | 75-115 | 17/5/1 | **#4** (catches cycle 03-13 root-cause class) |
| 5 | **shared `ZodGateMiddleware` utility** (SUSSMAN strong support) | 40 | 30-50 | OQ-3 closed | shared — extracted across all 4 modules |

**Total factored**: ~480 LOC unfactored / **~405 LOC median** under DARWIN non-additive savings analysis (range 340-460 factored). Within Master letter §五 "300-500 LOC" sizing band per Plan49 precedent (tests-in-band per F-§5-R2-09 VITRUVIUS).

### 3.2 plugin-loader DEFERRED (cycle 03-17+)

**9/11/3 — NO super-majority — DEFERRED**. Per D-§5-A breakdown:
- Plan51 entire 應 defer cycle 03-17+ post-AC-9 (DSS-A4 cluster: LEIBNIZ + KNUTH + SUSSMAN + 3 others)
- 4-module commit 過早；event-bus + WebSocket alone sufficient
- Option β (4 modules drop plugin-loader) better honours Master letter "300-500" LOC band (DSS-B7 cluster also cited LOC budget concern)
- plugin-loader 0-site greenfield 推遲合理

**Sustained MRB-06 deferral attached**: cycle 03-13 Plan49 spec MRB-06 (plugin-loader 0-site as Plan50+ gap) trail SUSTAINED but NOT resolved by cycle 03-15 R3.

**Sunset attachment for DEFERRED trail**: soft anchor cycle 03-21 (~6 cycles from cycle 03-15) is the recommended re-evaluation floor. If still no super-majority by cycle 03-21, MRB-06 should explicitly close-with-rationale or be re-ratified as an open architectural gap. Coordinator G5 obligation: track the plugin-loader DEFERRED trail across cycles 03-16/03-17/.../03-21.

### 3.3 Out of Plan51 cycle 03-16 Scope

- plugin-loader Zod gate (DEFERRED; cycle 03-17+ revisit per D-§5-A; AC-9 timing engagement)
- Schema-publication for client mirroring (D-§5-H DEFER cycle 03-22+; LEIBNIZ symmetric-fidelity foresight)
- Per-module SCHEMA_DRIFT_MODE override (OQ-6 KISS-defer; not in Plan51 initial scope)
- New Rules / MR / ZT / ENG-FAB items (Master letter §七 #10; Plan51 inherits existing governance)

## 4. Module Specs (GUARDIAN-Priority Rollout Order)

### 4.1 Module 1 — WebSocket Zod Gate (rollout #1)

**R3**: 推薦 21/2/0. Highest GUARDIAN priority per D-§5-C UNANIMOUS.

**Description**: external-facing transport for agent-runner client connectivity (browser dashboard, external orchestration, multi-process IPC stub). 1 schema-drift call-site existing per cycle 03-13 Plan49 §2.3.

**Schema artefact**: `WebSocketMessage<T>` = discriminated union over message-type (5-10 message types subject to Dev refinement).

**Critical invariant per CV-§5-04 UNANIMOUS** (Plan52 opaque sourceContext non-interference): if a message envelope carries a sourceContext-typed field, the WebSocket schema MUST mark that field as `z.unknown()` (or `z.any()`). Failure violates cycle 03-14 Plan52 ratified opacity invariant.

**Runtime guard**:
- **Inbound**: `safeParse` with mode fallback per Plan49 `SCHEMA_DRIFT_MODE`. **Initial mode = shadow** for ≥1 W2 round; then progress `audited → strict`.
- **Outbound**: `parse` (assertion-style; we control the producer; LOW migration risk).

**Migration risk**: HIGH for inbound (visible external behaviour change) + LOW for outbound. Mitigation: shadow mode + Plan49 mode-progression pattern.

**LOC**: ~125 median (100-150). Largest of the 4; reflects HIGH security-critical test ratio.

### 4.2 Module 2 — checkpoint-store Zod Gate (rollout #2; cross-version-skew helpers MUST)

**R3**: 推薦 18/4/1, AND D-§5-G UNANIMOUS 23/0: cross-version-skew migration helpers MUST be included.

**Description**: persists agent runtime state to disk. 2 schema-drift call-sites existing. Cross-version production data flowing per cycle 03-13 R3 StateTracker Path B.

**Schema artefact**: `CheckpointSchema<V>` parameterized by version tag.

**Cross-version-skew migration helpers (MUST per D-§5-G)**:
- **Per-version migration helpers covering v0.42 onwards** (≥3 prior versions: v0.42 / v0.45 / v0.48 / v0.50)
- **Audited-mode default** for read-path; strict-mode requires ≥1 W2 round of zero-skew evidence
- **Graceful-degradation read-path**: never block recovery on Zod gate failure
- **Explicit `from_version` → `to_version` migration matrix** in Plan51 spec doc
- **Audit emission**: `checkpoint_schema_violation` + `checkpoint_migration_applied` events distinguishable
- **Cross-version round-trip test fixtures**: verify v0.42 checkpoint loads cleanly under v0.50 reader after migration

**Without these helpers, MR-12 forward-only is at risk** — read-path enforcement could break old checkpoints. F-§5-R2-12 HIGH severity.

**Runtime guard**: read = `safeParse` shadow → audited; write = `parse` assertion.

**Migration risk**: HIGH for cross-version-skew read sub-case + MEDIUM for in-process read + LOW for write path.

**LOC**: ~110 median (85-140; revised up from R1 ~90 to reflect mandatory cross-version-skew helpers).

### 4.3 Module 3 — event-bus Zod Gate (rollout #3; σ_regime non-interference)

**R3**: 推薦 19/3/1.

**Description**: in-process publish/subscribe substrate carrying lifecycle events. 3 schema-drift call-sites existing.

**Schema artefact**: `EventEnvelope<T>` typed wrapper + `EventBusSchemaRegistry` registry with register/lookup/validate API.

**Critical invariant per CV-§5-05** (Plan50 σ_regime non-interference): event-bus schema for σ-emission events MUST include the `sigma_regime: z.enum(['composition_index', 'llm_variance', 'mixed'])` field. Plan51 honours Plan50 layer-orthogonality: structural validation at event envelope; numeric metric tag (sigma_regime) is a payload field within the envelope.

**Reflexive-case test fixture (per F-§5-R2-11 LEIBNIZ + WIENER NEW)**: emit an intentionally-malformed event; verify the `event_bus_schema_violation` event itself validates cleanly (does not get suppressed under strict mode).

**Runtime guard**: audited mode default (subscribe + publish `safeParse`).

**Migration risk**: MEDIUM. Mitigation: shadow mode for first cycle, then enforce.

**LOC**: ~110 median (85-135).

### 4.4 Module 4 — hook-registry Zod Gate (rollout #4; 1 module + 2 schema artefacts)

**R3**: 推薦 17/5/1, AND D-§5-E binding (21/2): 1 module + 2 schema artefacts per DARWIN Strategy/Registry pattern. DSS-C4 2 dissent §10.

**Description**: tracks which plugins have registered which lifecycle hooks. 1 schema-drift call-site existing. Per cycle 03-13 root-cause investigation (`gear-arbiter-dynamic` plugin loader warning), hook-registry metadata fidelity is already a known operational risk surface.

**Schema artefacts (DARWIN Strategy/Registry pattern split per D-§5-E)**:
- **`HookRegistration`** — Registry-pattern data structure schema (registration metadata, what plugin declared which hooks, version)
- **`HookContract<I, O>`** — Strategy-pattern dispatch contract schema (per-event-type, dispatch the registered handler with input/output contract)

**Why 1 module + 2 artefacts (not 2 modules)**: registration and dispatch are two distinct lifecycle phases sharing one data structure. Forcing them into separate modules would fragment the registry without reducing the abstraction-leak. Forcing them into one schema artefact would conflate lifecycle phases. The 1-module-2-artefacts compromise honours both concerns (D-§5-E 21/2; DSS-C4 2 minority §10).

**Runtime guard**:
- Registration-time (`HookRegistration`): `safeParse` strict from start.
- Dispatch-time (`HookContract<I, O>`): `safeParse` audited mode initially.

**Migration risk**: LOW-MEDIUM.

**LOC**: ~95 median (75-115; revised up from R1 ~80 to reflect 2 distinct schema artefacts per D-§5-E).

## 5. Cross-Version-Skew Migration Helpers MUST (D-§5-G)

### 5.1 R3 binding decision

**D-§5-G UNANIMOUS 23/0**: cross-version-skew migration helpers MUST be included in Plan51 spec (HIGH per F-§5-R2-12).

### 5.2 Why MUST (not SHOULD)

Cross-version-skew checkpoints exist in the wild from cycle 03-08 (v0.42) onwards. Plan51 read-path Zod gate enforced on v0.50+ would reject these unless explicit v0.42 → v0.50 migration path is provided. Without helpers, "reject on read" = silent recovery failure for any operator who upgrades runner with stale checkpoints. **MR-12 forward-only is at risk** — Plan51 would retroactively-break old checkpoints. R3 ratified MUST level (UNANIMOUS) precisely to close this gap before Master ratification.

### 5.3 Required scope (Plan51 spec items per §4.2)

1. Per-version migration helpers covering v0.42 onwards (≥3 prior versions)
2. Explicit `from_version` → `to_version` migration matrix in spec doc
3. Audited-mode default for read-path; strict-mode requires ≥1 W2 round of zero-skew evidence
4. Graceful-degradation read-path
5. Distinguishable audit emissions
6. Cross-version round-trip test fixtures

## 6. Non-Interference Invariants (Plan49 / Plan50 / Plan52)

### 6.1 Plan52 pushInput (cycle 03-14 ratified, live)

CV-§5-04 UNANIMOUS at R3 reaffirmed:
1. **Surface separation**: Plan52 plugin-public-input ε-surface vs Plan51 internal-peripheral module boundaries. **No overlap**.
2. **MR-6 Core 零 preserved across both**: Plan52 ε-surface = 1 optional field; Plan51 Zod gates = 0 Core surface.
3. **Opaque sourceContext non-interference**: Plan52's opaque sourceContext is **deliberately not Zod-validated** at the Core level. **Plan51 WebSocket schema MUST treat any sourceContext-typed fields as `z.unknown()` or `z.any()`** to preserve opacity (binding spec invariant).
4. **Plan52 inherits Plan49 schema-drift policy**; Plan51 inherits the same. No conflict.

### 6.2 Plan50 σ_regime (cycle 03-14 ratified, 50 LOC binding, atomicity MUST)

CV-§5-05 at R3:
1. **Layer separation**: Plan50 SPC plugin-internal numeric layer vs Plan51 module structural-schema layer. Layer-orthogonal.
2. **σ-emission events on event-bus**: Zod schema for σ-events under Plan51 MUST include `sigma_regime: z.enum(['composition_index', 'llm_variance', 'mixed'])` payload field (binding per CV-§5-05).
3. **No retroactive risk**: Plan50 forward-only; Plan51 forward-only. Both honour MR-12.
4. **Atomicity**: Plan51 should aim for **per-module migration atomicity** (within a single PR; not a 4-module mega-merge but 4 sequential per-module atomic merges).

### 6.3 Plan49 schema-drift mode (cycle 03-13 ratified, live)

CV-§5-02 at R3:
- Plan51 schemas dispatch through `resolveSchemaDriftMode()` (Plan49 helper).
- Plan51 audited-mode events use the same `schema_drift_audit` event family.
- F-16 StructuredError integration extends, does not replace, Plan49 emission shape.
- Single env var process-global per D-12 (cycle 03-13 UNANIMOUS); no per-module fragmentation.

**Plan51 = clean expansion of Plan49**.

### 6.4 4-Plan stack consistency

| Plan | Layer | Cycle live | Interaction with Plan51 |
|------|-------|-----------|--------------------------|
| Plan49 | schema-drift mode dispatcher | 03-13 | Plan51 inherits SCHEMA_DRIFT_MODE; clean expansion |
| Plan50 | σ_regime SPC tag | 03-14 | Plan51 event-bus schema includes sigma_regime enum field |
| Plan52 | pushInput ε-surface | 03-14 | Plan51 WebSocket schema treats sourceContext as opaque |
| Plan51 | Zod gate × 4 modules + shared utility | 03-16 candidate | additive layer; forward-only; MR-12 honoured iff cross-version helpers |

## 7. Sunset-by-Execution (D-§5-D UNANIMOUS)

**D-§5-D UNANIMOUS 23/0**: 推薦 sunset-by-execution. Active per D-§5-A 推薦 outcome (4-of-5 modules implemented in cycle 03-16). **6-cycle floor framing N/A for the 推薦 path**. Closes cycle 03-14 DSS-3 LEIBNIZ ossification concern for the 推薦 trail.

**plugin-loader DEFERRED trail** retains soft 6-cycle floor anchor (cycle 03-21 re-evaluation floor) per §3.2.

## 8. Implementation Order

### 8.1 Cycle sequence

1. **Plan49 SHOULD landed** (cycle 03-13 live; SCHEMA_DRIFT_MODE provides dispatcher)
2. **Plan50 σ_regime** (cycle 03-14 live; sigma_regime enum field consumed by event-bus schema)
3. **Plan52 pushInput** (cycle 03-14 live; opaque sourceContext invariant honoured by WebSocket schema)
4. **Plan51 cycle 03-16 first-shipping** (this spec ratifies; GUARDIAN-priority rollout per D-§5-C)

### 8.2 Within cycle 03-16 — Plan51 module sequence (per-module atomicity MUST)

| Stage | Module | Atomicity | Initial mode |
|:----:|--------|-----------|--------------|
| #1 | WebSocket | Single PR | Shadow (inbound) + assertion (outbound) |
| #2 | checkpoint-store | Single PR (incl cross-version-skew helpers) | Audited (read; never block recovery) + strict (write) |
| #3 | event-bus | Single PR | Audited (matches Plan49 baseline) |
| #4 | hook-registry | Single PR (1 module + 2 schema artefacts) | Strict (registration) + audited (dispatch) |

**Atomicity: per-module single-PR** (Plan50 atomicity MUST style). Each PR includes:
- Schema artefact(s)
- Validation hooks
- Migration helpers (where applicable)
- Test fixtures (positive + negative + edge-case + reflexive where applicable)
- Doc snippet (TW parity per Rule #78 §78.3 governance MUST + canonical SHOULD per D-§2-07)
- CHANGELOG entry

### 8.3 W2-R17 + W3 progression (cycle 03-22+ candidate)

Strict-mode rollout for inbound WebSocket + audited-strict transition for checkpoint-store read-path requires **≥1 W2 round of zero-skew evidence**. W2-R17 candidate per cycle 03-22 timeline.

## 9. Cycle 03-22+ Schema-Publication DEFERRED (D-§5-H)

**D-§5-H UNANIMOUS 23/0**: LEIBNIZ symmetric-fidelity Plan51 schema-publication scope DEFER cycle 03-22+ Mesh / 03-23 API Runtime. Cycle 03-15 = no schema-publication work. Plan51 cycle 03-16 first-shipping = runner-side only. Documentation cross-references the cycle 03-22+ deferred work (Plan53 / Plan54 candidate; ~10-20 LOC + doc work).

## 10. Dissent Preservation (per MR-11 verbatim)

### 10.1 DSS-A4 (D-§5-A Plan51 推薦 4-of-5 modules; vote 17/6)

**Minority**: 6 dissent (LEIBNIZ + KNUTH + SUSSMAN + 3 others).

**Verbatim**: "Plan51 entire 應 defer cycle 03-17+ post-AC-9；4-module commit 過早；event-bus + WebSocket alone sufficient."

**R4 obligation**: explicit dissent slot — preserved here verbatim per MR-11. The dissent applies to the *entire* Plan51 scope. Master may consider whether the 4-module commitment in cycle 03-16 should be tightened to event-bus + WebSocket only (a 2-module variant); this would deviate from R3 ratified outcome and require Master ruling.

### 10.2 DSS-B7 (D-§5-B 5th-module identity Option α; vote 17/4/2)

**Minority**: 4 dissent.

**Verbatim**: "Option β (4 modules drop plugin-loader) better honours Master letter '300-500' LOC band；plugin-loader 0-site greenfield 推遲合理."

**R4 obligation**: footnote — preserved here verbatim per MR-11. Note: the implementation outcome (D-§5-A) ended up 4-of-5 with plugin-loader DEFERRED, which aligns operationally with the Option β preference in DSS-B7.

### 10.3 DSS-C4 (D-§5-E hook-registry 1 module + 2 schemas vs 2 modules; vote 21/2)

**Minority**: 2 dissent.

**Verbatim**: "2 modules cleaner separation；DARWIN pattern 過度抽象."

**R4 obligation**: footnote — preserved here verbatim per MR-11.

### 10.4 DSS-3 carryover (cycle 03-14 LEIBNIZ Plan51 ossification)

**Verbatim**: "Plan51 ossification concern if indefinite carry-forward without sunset clause."

**Status**: **resolved by R3** (D-§5-D 推薦 outcome triggers sunset-by-execution; no 6-cycle floor needed for the 推薦 path). **Closed by R3** for the 推薦 trail. plugin-loader DEFERRED trail retains soft 6-cycle floor anchor per §3.2.

### 10.5 §5 dissent count summary

| Source D-item | Dissent count | Verbatim | Status |
|---------------|:-------------:|----------|--------|
| D-§5-A (Plan51 4-of-5 推薦) | 6 (DSS-A4) | §10.1 | preserved |
| D-§5-B (Option α 5 modules) | 4 (DSS-B7) | §10.2 | preserved |
| D-§5-C (GUARDIAN-priority rollout) | 0 | UNANIMOUS | — |
| D-§5-D (sunset-by-execution) | 0 | UNANIMOUS | — |
| D-§5-E (hook-registry 1+2) | 2 (DSS-C4) | §10.3 | preserved |
| D-§5-F (per-module granularity) | 0 | UNANIMOUS | — |
| D-§5-G (cross-version-skew MUST) | 0 | UNANIMOUS | — |
| D-§5-H (schema-publication DEFER) | 0 | UNANIMOUS | — |
| **Chapter §5 internal** | **3 entries (12 votes)** | (DSS-A4 + DSS-B7 + DSS-C4) | preserved |
| **Carryover** | **1 (DSS-3 → resolved D-§5-D)** | §10.4 | closed |

## 11. MR/ZT Compliance Audit

| Constraint | Status | Evidence |
|-----------|:------:|----------|
| **MR-2** (Tenet text untouched) | PASS | No Tenet content modified |
| **MR-4** (ENG-FAB candidate citation) | PASS | F-16 SHOULD initial cited only; not modified |
| **MR-5 hard** (Tenet #10 NC PENDING preserved) | PASS | No Tenet #10 status change |
| **MR-6 (Core 零)** | PASS | All 4 推薦 modules sit on `apps/runner/src/<module>/` peripheral; CV-§5-01 UNANIMOUS |
| **MR-9** (no preemptive MUST F-16 binding) | PASS | F-16 SHOULD initial inheritance; CV-§5-03 reaffirm |
| **MR-10 + MR-12** (forward-only with classification) | PASS w/ caveat | MR-12 forward-only honoured iff cross-version-skew migration helpers present (D-§5-G UNANIMOUS MUST) |
| **MR-11** (dissent preservation) | PASS | 4 dissent entries §10 verbatim |
| **MR-13** standby | PASS | Not invoked by §5 |
| **ZT-1** (Tenet preservation) | PASS | CV-§5-07 UNANIMOUS |
| **ZT-2** (endpoint 10/0/0★) | PASS | Plan51 = governance/architectural, not Phase 6 functional; endpoint unchanged |
| **ZT-3** (control-range triple-check) | PASS | No σ scope change in this spec |

---

*Plan51 Candidate Spec — Cycle 03-15 R4 Final*
*Authors: TANENBAUM (#20) + SUSSMAN (#22) + KERNEL (#10) + MESH (#4) + HERACLITUS (#15) + GUARDIAN (#11) + ARCHIMEDES (#16) + SYNTHESIST (#1)*
*Status: CANDIDATE (pending Master Ratification Batch 12 #2)*
*4-of-5 推薦: WebSocket / checkpoint-store / event-bus / hook-registry; plugin-loader DEFERRED cycle 03-17+*
*GUARDIAN-priority rollout (D-§5-C UNANIMOUS); cross-version-skew helpers MUST (D-§5-G UNANIMOUS)*
*LOC factored ~405 median (340-460 range); within Master letter §五 "300-500" band per Plan49 precedent*
*Non-interference: Plan49 + Plan50 + Plan52 4-Plan stack architecturally consistent*
*Dissent preserved: DSS-A4 (6) + DSS-B7 (4) + DSS-C4 (2) verbatim per MR-11; DSS-3 carryover closed by D-§5-D UNANIMOUS*
*Implementation order: Plan49 → Plan50 → Plan52 → Plan51 (cycle 03-16 first-shipping)*
*Hard constraints: MR-5 hard ✓ / MR-6 ✓ / MR-9 ✓ / MR-11 ✓ / MR-12 ✓ + D-§5-G MUST close / ZT-1/2/3 ✓ / Tenet #10 untouched*
