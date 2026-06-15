---
title: ENG-FAB v1.9 Candidate — F-16 StructuredError Schema
author: GUARDIAN (#11 Security) + KERNEL (#10 OS error patterns) + ARCHIMEDES (#16 engineering) + ASANGA (#8 epistemic-status discipline) + SYNTHESIST (#1 aggregator)
date: 2026-04-25
cycle: 03-14
status: CANDIDATE (pending Master Ratification Batch 11 #1)
authority: research-team (R4 final draft); Master (ratification)
supersedes: null
cross_refs:
  - research record/cycle03-14/deliver/O4_R_input_5_candidates_final.md §2 (lines 47-217)
  - research record/cycle03-14/R3/R3_decision_log.md §4 (D-§4-01 + D-§4-02 + D-§4-03 + D-§4-04)
  - research record/cycle03-14/deliver/Master_Ratification/Batch_11_Request.md §3.1 (Item #1)
  - research record/cycle03-13/openstarry_doc/Research_Methodology/11_ENG_FAB_v1.8_Binding.md (v1.8 = 48 baseline)
binding_until: Master Ratification Batch 11 close + 2-cycle observation window (cycle 03-15 + 03-16)
---

# ENG-FAB v1.9 Candidate — F-16 StructuredError Schema

## 1. Status

**CANDIDATE** at cycle 03-14 R4 close. SHOULD initial upon Master ratification (Batch 11 Item #1); MUST advancement scheduled at cycle 03-17 Plan iff three observation metrics ≥ 80% over the 2-cycle window (cycles 03-15 + 03-16) per D-§4-03 (B-6) 20/3.

**ENG-FAB roster impact**: v1.8 = 48 items canonical PRESERVED; v1.9 = 49 items if Master ratifies F-16 increment. CV-§4-α (R3 cycle 03-14) reaffirmed UNANIMOUS — F-16 is a single-item increment, not an item-count revision of v1.8.

## 2. R3 Provenance

- **D-§4-01 (A-3)** UNANIMOUS 23/0 — 10-constant enum closed (8 base + Halt + Other; Madhyamaka-coherent fallback rule absorbed).
- **D-§4-04 (A-4)** UNANIMOUS 23/0 — Plan48 structured-log transport binding mandatory; F-16 is NOT a parallel emission channel.
- **D-§4-02 (B-5)** 19/2/2 — `likely_cause` prefix-discipline (`verified:` / `inferred:` / `speculation:` per ASANGA Yogācāra epistemic-status); DSS-5 absorbed verbatim §10.
- **D-§4-03 (B-6)** 20/3 — 2-cycle observation window with 80% threshold for SHOULD→MUST advancement; DSS-15 (WIENER + RUSSELL conservative 3-cycle) absorbed verbatim §10.

## 3. Binding Text

### 3.1 F-16 Schema — 6-Field StructuredError Record

JSON-serialisable, fixed field order, transport-bound to Plan48 structured-log channel:

```text
{
  "error":                  <enum; 10 constants — see §3.2>,
  "message":                <human-readable, single sentence, ≤ 200 chars>,
  "likely_cause":           <prefix-disciplined string — see §3.3; ≤ 200 chars>,
  "suggested_fix_location": <relative file path + optional line range, e.g.
                              "apps/runner/src/event-bus.ts#L142-L160";
                              if unknown, the literal string "unknown" REQUIRED>,
  "context":                <object; relevant identifiers, e.g.
                              { "plan_id": ..., "request_id": ..., "trace_id": ... }>,
  "trace_id":               <opaque correlation id; structured-log existing trace
                              id when present (single source of truth)>
}
```

**Transport binding (D-§4-04 UNANIMOUS)**: F-16 emissions MUST flow through the Plan48 structured-log channel as a payload type. F-16 is NOT a parallel emission channel. SCRIBE consults Plan48 structured-log existing payload taxonomy at first F-16 site introduction; if Plan48 already attaches a trace correlation id, F-16 defers to that field name.

### 3.2 Enumerated `error` Set — 10 Constants (D-§4-01 UNANIMOUS)

Closed enumeration; 8 base + Halt + Other:

| # | Constant | Semantic |
|:-:|----------|----------|
| 1 | `ValidationError` | Input failed schema/structural validation |
| 2 | `NotFound` | Requested resource/identifier not present |
| 3 | `Conflict` | State conflict (concurrent update, version mismatch) |
| 4 | `DependencyFailure` | Downstream dependency (HMAC verify, plugin load, etc.) failed |
| 5 | `TimeoutError` | Time-bounded operation exceeded budget |
| 6 | `PermissionDenied` | Authorisation/capability check failed |
| 7 | `InternalError` | Unanticipated runtime error (catch-all internal) |
| 8 | `SchemaDriftDetected` | Plan49 class — observed schema disagrees with declared |
| 9 | `Halt` | Cycle-halt-class event (sustained, not ephemeral; e.g., audit halt request) |
| 10 | `Other` | Genuinely-unclassified-at-emit-time; reader should not branch on it |

**Madhyamaka-coherent fallback rule** (NAGARJUNA caveat absorbed at R3 UNANIMOUS):

> Parsers operating on schema version `v_N` MUST treat any discriminator not in `v_N` as semantically equivalent to `Other`. This is operationally a graceful-degradation rule and engineering-aligned. The set is conventional, not essentialist (per NAGARJUNA Madhyamaka critique absorbed).

**MR-10 back-fill discipline**: future cycles may extend the enumeration via MR-10 back-fill rules; each back-fill ratifies the new constant in a Master Ratification batch.

### 3.3 `likely_cause` Prefix-Discipline (D-§4-02 19/2/2 — DSS-5 absorbed)

ASANGA Yogācāra critique surfaced an epistemic-status drift in the original `likely_cause` field: a downstream LLM agent reading the field treats the string as ground truth even when the author wrote it as speculation. Prefix-discipline addresses this:

The `likely_cause` field MUST begin with one of three controlled prefixes:

- **`verified: <cause statement>`** — author has confirmed the cause by reproduction (sampajāna; direct knowledge). Downstream consumers MAY act on this as ground truth.
- **`inferred: <cause statement>`** — author has strong evidence but no direct reproduction (anumāna; inference). Downstream consumers should weight as high-confidence but check before action.
- **`speculation: <cause statement>`** — author's best guess (kalpanā; constructed). Downstream consumers should NOT act on this; they should solicit clarification or attempt reproduction.

If the field is genuinely empty (no cause hypothesis), the literal string `"speculation: unknown"` is REQUIRED (parallel to the `suggested_fix_location: "unknown"` sentinel discipline).

**ATHENA practical concern absorbed**: in production LLM workflows, the prefix delivers the actual token-economy lever the AI-Agent-Toolkit rationale targeted. An LLM consumer that sees `speculation:` knows to ask for clarification rather than retry; an LLM that sees `verified:` retries deterministically.

### 3.4 Strength Path — SHOULD → MUST (D-§4-03 20/3 — DSS-15 absorbed)

Four-stage advancement:

1. **Plan50 / Plan52 dispatch (cycle 03-14 R4 close)** — F-16 introduced as **SHOULD**.
   - New code in `apps/runner` + new pushInput plugin SHOULD emit StructuredError on every reachable error path.
   - Existing `apps/runner` partial-pattern code is **NOT retrofitted** per MR-12 + CV-§4-β.

2. **2-cycle observation window** (cycles 03-15 + 03-16) — SCRIBE collects per-error compliance counts and field-completion rates from delivery_reports. Three observation metrics:
   - **adoption rate** — % of new error sites emitting conforming F-16. Target ≥ 80% over 2-cycle window (per RUSSELL agent-theory baseline; relaxed from R1's tentative 90% per second-cycle-maturity calibration).
   - **field completion rate** — per-field % populated. `suggested_fix_location: "unknown"` rate tripwire — if > 30% trends, surface for revision before MUST.
   - **prefix-discipline adoption rate** — % of `likely_cause` fields conforming to one of three prefixes.

3. **MUST advancement (cycle 03-17 Plan, typically Plan52+1)** — if all three metrics ≥ 80% over the 2-cycle window, F-16 advances to MUST. New code MUST emit conforming F-16 at every error path. Existing partial-pattern code remains exempt per MR-12.

4. **Revision branch** — if observation surfaces a structural problem (enumeration too narrow, `unknown` rate > 30% sustained, or prefix-discipline adoption < 60%), the candidate is revised before MUST advancement. Revision goes through R0-R4 again as a v1.9.1 amendment proposal.

### 3.5 Apply Scope (per MR-12 + CV-§4-β)

**In scope** (forward-only):
- `apps/runner` code authored from cycle 03-14 onward
- New plugins: pushInput Plan52; Plan50 σ_regime emissions; future plugins

**Out of scope**:
- Existing plugins (gear-arbiter-dynamic; Plan47 K-3; etc.) — NOT retrofitted
- Cycle 03-13 and earlier code paths preserved as-is

**Edge case**: when an existing plugin is **modified** in cycle 03-14+ for any reason (bug fix, feature addition), the modified function SHOULD adopt F-16 on its newly-touched error sites; surrounding unmodified code remains exempt. This is the standard MR-12 "modify-touch only" discipline.

### 3.6 Effort Estimate

- F-16 spec text in O4 / Plan-spec: ~80 lines (this section + Plan52 dispatch §pushInput error wrapping).
- Plan50 σ_regime first adoption: ~20 LOC error wrapping in σ_regime emission paths.
- Plan52 pushInput first adoption: ~30 LOC error wrapping in pushInput plugin entrypoints.
- F-16 audit logic in `audit_calc.py` extension (cycle 03-15): ~60 LOC delta.
- **Total LOC budget at MUST**: 100-150 incremental over cycles 03-14 + 03-15 + 03-16.

### 3.7 Conflict-Check vs ENG-FAB v1.8 (Final)

| ENG-FAB v1.8 item | Relation to F-16 | Verdict |
|-------------------|------------------|---------|
| F-9 (openstarry_doc completeness) | independent (covers docs, not error format) | no overlap |
| F-13 (Plugin Hook Dispatch Verified) | independent (covers hook registration, not error format) | no overlap |
| F-14 (External Resource Usage Baseline) | independent (covers billing observation) | no overlap |
| F-15 (Code-Path Verified Before Impact Assessment) | independent input-discipline (impact-assessment vs runtime error emission) | no cycle |
| Plan48 structured-log (C48-M1) | **complementary; F-16 is payload type ON structured-log** | binding overlap resolved by D-§4-04 transport mandate |

No item in v1.8 is rendered redundant; no MR requires WAIVE; no Rule conflicts with the SHOULD→MUST advancement path.

## 4. Worked Example Emissions

### 4.1 ValidationError in pushInput plugin (Plan52 first-emission case)

```text
{
  "error": "ValidationError",
  "message": "pushInput payload schema-drift detected at field 'sourceContext.sig'.",
  "likely_cause": "verified: caller's payload was generated against schema v1.0 but plugin runtime expects v1.1 (sig field added)",
  "suggested_fix_location": "packages/plugin-push-input/src/schema.ts#L42-L58",
  "context": {
    "plan_id": "Plan-52-pushInput",
    "request_id": "01HXXXXXX",
    "schema_observed_version": "1.0",
    "schema_expected_version": "1.1"
  },
  "trace_id": "01HZZZZZZ"
}
```

### 4.2 DependencyFailure (HMAC verification fail)

```text
{
  "error": "DependencyFailure",
  "message": "HMAC signature on parentAgentId failed verification.",
  "likely_cause": "inferred: signing key rotation likely occurred but verifier was not refreshed",
  "suggested_fix_location": "apps/runner/src/auth/hmac-verify.ts#L88-L95",
  "context": { "agent_id": "<redacted>", "key_id": "k-2026-04" },
  "trace_id": "01HZZZZZZ"
}
```

### 4.3 SchemaDriftDetected (Plan49 class)

```text
{
  "error": "SchemaDriftDetected",
  "message": "Plugin manifest declares hooks but runtime hookMap shows no registration.",
  "likely_cause": "speculation: registerHooks() not invoked at module load, OR hook name typo in manifest vs code",
  "suggested_fix_location": "unknown",
  "context": {
    "plugin_name": "gear-arbiter-dynamic",
    "manifest_hooks_declared": ["samskara", "vijnana"],
    "runtime_hooks_observed": []
  },
  "trace_id": "01HZZZZZZ"
}
```

### 4.4 Halt (cycle-halt-class event)

```text
{
  "error": "Halt",
  "message": "Audit-halt request received from coordinator-side investigation.",
  "likely_cause": "verified: coordinator task #49 W-round token-zero anomaly under investigation",
  "suggested_fix_location": "apps/runner/src/halt-handler.ts#L20-L45",
  "context": {
    "halt_reason": "audit_pending",
    "cycle": "03-13",
    "investigation_ticket": "task-49"
  },
  "trace_id": "01HZZZZZZ"
}
```

## 5. Observation Metrics — Detailed Definitions

For the 2-cycle observation window (cycles 03-15 + 03-16), SCRIBE collects:

### 5.1 Adoption Rate

```
adoption_rate = (count of F-16-conforming error emissions in cycle X new code)
              / (count of all error emissions in cycle X new code)
```

- Numerator: emissions that pass schema validation against §3.1.
- Denominator: emissions on code paths authored cycle 03-14+.
- Threshold: ≥ 80% per cycle; aggregate over 2-cycle window also ≥ 80%.

### 5.2 Field Completion Rate

```
field_completion_rate[field] = (count of F-16 emissions with non-default <field>)
                             / (count of all F-16 emissions)
```

- Per-field; reported for each of `error` / `message` / `likely_cause` / `suggested_fix_location` / `context` / `trace_id`.
- Tripwire: `suggested_fix_location: "unknown"` rate > 30% over 2-cycle window — surfaces for revision before MUST advancement.

### 5.3 Prefix-Discipline Adoption Rate

```
prefix_adoption_rate = (count of `likely_cause` fields beginning with valid prefix)
                     / (count of all F-16 emissions with `likely_cause` field present)
```

- Valid prefixes: `verified:` / `inferred:` / `speculation:` / `speculation: unknown` (sentinel).
- Threshold: ≥ 80% per cycle for advancement.

### 5.4 Computation responsibility

`audit_calc.py` extension (cycle 03-15 ~60 LOC delta) computes the three metrics on each cycle's R4 close. Output appended to `delivery_report.md §F16_observation_metrics` section by Dev; SCRIBE cross-checks and includes in O7 evidence chain.

## 6. Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 / MR-4 (Tenet wording unchanged) | ✅ PASS | No Tenet wording proposed |
| MR-5 hard (Tenet #10 status unchanged) | ✅ PASS | F-16 is plugin/runtime layer; no compliance status implication |
| MR-6 (Core 零) | ✅ PASS | Plugin/runtime emission only; no Core surface |
| MR-7 (timing audit) | ✅ PASS | SHOULD→MUST scheduled; 2-cycle observation discipline |
| MR-8 (quality first) | ✅ PASS | 80% threshold relaxation per second-cycle empirical maturity |
| MR-9 (no MUST WAIVE) | ✅ PASS | SHOULD→MUST is strengthening path, not waive |
| MR-10 (back-fill) | ✅ PASS | Enum extensible via future MR-10 back-fill |
| MR-11 (dissent preservation) | ✅ PASS | DSS-5 + DSS-15 verbatim §10 |
| MR-12 (既有 not retrofit) | ✅ PASS | Existing apps/runner partial-pattern code exempt |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | No 10-Tenet violation; endpoint unchanged; control-range unaffected |

### 6.3 FORBIDDEN-Phrasings List (Strength-Discipline Wording Sweep)

per cycle 03-14 R4 ratification（base 3 patterns；coordinator 補錄至 canonical 2026-04-27）+ cycle 03-15 Master Ratification Batch 12 Item #6（near-miss 擴展 3 patterns；UNANIMOUS 23/0）

以下措辭在 R0/R1/R2/R3/R4 任一階段或 deliver/results/openstarry_doc/todo/test_instructions 任一產出**禁用**（隱晦 advocate F-16 SHOULD→MUST 升級即違反 MR-9 強度 disciplined）：

**Base patterns (cycle 03-14 R4 ratified)**:
1. "F-16 is now MUST"
2. "should advance to MUST"
3. "is ready for MUST"

**Near-miss patterns (cycle 03-15 Batch 12 Item #6 ratified)**:
4. "trending toward MUST"
5. "MUST-ready signal positive"
6. "promotion path effectively confirmed"

**判定原則**：F-16 強度由 SCRIBE 排定 2-cycle observation table 達標後在 cycle 03-17 Plan dispatch 決定；任何在此之前以**陳述**或**暗示**方式 framing 「F-16 已 / 即 / 應 / 將 升 MUST」的措辭一律 SCRIBE wording-sweep reject 並回退作者修正。Strict reading；不容意會。

**範圍**：cycle 03-15+ forward only；既有 cycle 03-14 ratified text 不 retrofit per MR-12。

**Activation**：Master Ratification Batch 12 ratify 後即時生效（2026-04-27）。

## 7. Authority Notes

- **Authority route**: Master ratification (ENG-FAB increment v1.8 → v1.9 = 49 candidate).
- **Effective once Master ratifies**: SHOULD initial; MUST advancement contingent on 2-cycle observation pass.
- **2-cycle window administration**: SCRIBE schedules observation table emission at cycle 03-15 R4 close (1-cycle interim) and cycle 03-16 R4 close (2-cycle aggregate). cycle 03-17 Plan absorbs MUST decision.

## 8. Cross-References

- `research record/cycle03-14/deliver/O4_R_input_5_candidates_final.md §2` — F-16 narrative source
- `research record/cycle03-14/R3/R3_decision_log.md §4 D-§4-01..04` — R3 voting record
- `research record/cycle03-14/deliver/Master_Ratification/Batch_11_Request.md §3.1 Item #1` — ratification dispatch
- `research record/cycle03-13/openstarry_doc/Research_Methodology/11_ENG_FAB_v1.8_Binding.md` — v1.8 = 48 baseline (preserved)
- `research record/cycle03-14/deliver/O1_pushInput_Plan52_final.md §pushInput-error-wrapping` — Plan52 first-emission site
- `research record/cycle03-14/deliver/O2_Plan50_sigma_regime_final.md §F16-applicability` — Plan50 σ_regime applicability confirmation (D-§2-Q3 UNANIMOUS)

## 9. Authority Notes (Strength Discipline)

The SHOULD initial → MUST advancement design absorbs:

- **GUARDIAN security-first concern** — error format consistency is operational defense; SHOULD initial gives Dev runway, MUST consummates the discipline.
- **KERNEL OS-discipline** — error enum closure (10 constants) parallels Linux errno; the `Other` fallback parallels `EOTHER` graceful-degradation discipline.
- **ASANGA Yogācāra discipline** — prefix-discipline operationalises eight-consciousness epistemic-status awareness in machine-readable form.
- **WIENER conservatism** — 2-cycle minimum allows feedback-loop calibration; if metrics diverge from target by cycle 03-16, R0-R4 amendment cycle absorbs.

## 10. Dissent Slots — DSS-5 + DSS-15 Verbatim (per MR-11)

### 10.1 DSS-5 (D-§4-02, 4 minority votes — 2 reject + 2 sibling-field)

> NAGARJUNA + ASANGA philosophical objection to prefix-discipline:
>
> "Prefix-discipline reifies the verified / inferred / speculation distinction as if these were three discrete epistemic categories rather than a continuum (per Yogācāra cittamātra and Madhyamaka two-truth refinements). A downstream consumer that strictly branches on prefix may discard nuance — e.g., a `verified:` claim that is actually 'verified for one reproduction but with non-zero variance' is conflated with a 'verified-deterministic' claim. Prefer a separate `likely_cause_confidence` enumeration with explicit gradation (e.g., `low / medium / high / verified`) over inline prefix."
>
> Sibling-field alternative (2 votes, ATHENA + 1):
>
> "Inline prefix mixes data type (cause statement) with metadata (epistemic status). A separate `likely_cause_confidence` field is structurally cleaner and parser-simpler."
>
> R3 majority (19/2/2) preferred prefix-discipline on grounds:
> - Inline prefix is single-field self-contained; sibling-field requires two-field consistency check.
> - Three-prefix discipline is operationally adequate for LLM consumers (the actual target consumer per ATHENA practical concern); a 4-or-more gradation produces consumer confusion.
> - Madhyamaka two-truth concern absorbed via the conventional (saṃvṛti-satya) framing — the prefixes are operational tags, not metaphysical claims.

### 10.2 DSS-15 (D-§4-03, 3 minority votes — WIENER + RUSSELL + 1)

> WIENER + RUSSELL conservative 3-cycle preference:
>
> "Cybernetic feedback-loop calibration requires sufficient sample variance to distinguish stable adoption from cycle-1 enthusiasm artefact. A 3-cycle window with 90% threshold (cycles 03-15 + 03-16 + 03-17) provides higher signal-to-noise on adoption stability. The 2-cycle / 80% formulation risks promoting on a transient peak."
>
> R3 majority (20/3) preferred 2-cycle / 80%:
> - 2-cycle / 80% matches second-cycle empirical maturity per RUSSELL's own agent-theory baseline (which the dissent set aside in favor of cybernetic conservatism).
> - The 80% threshold relaxation FROM 90% is the substantive concession to the conservative concern — the threshold is NOT loosened beyond second-cycle maturity baseline.
> - Revision branch (§3.4 stage 4) absorbs the conservative concern: if 2-cycle data is anomalous, revision rather than MUST advancement.

Both dissent positions are preserved per MR-11; the SHOULD→MUST advancement path may be re-tightened to 3-cycle / 90% via R0-R4 amendment if cycle 03-15 data exhibits anomalous transient-peak signature.

---

*Cycle 03-14 R4 final — 2026-04-25*
*Authors: GUARDIAN (#11) + KERNEL (#10) + ARCHIMEDES (#16) + ASANGA (#8) + SYNTHESIST (#1)*
*Status: CANDIDATE (pending Master Ratification Batch 11 #1)*
*ENG-FAB v1.8 = 48 canonical PRESERVED; v1.9 = 49 if Master ratifies*
