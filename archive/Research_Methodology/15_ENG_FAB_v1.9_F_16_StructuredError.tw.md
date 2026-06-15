---
title: ENG-FAB v1.9 候選 — F-16 StructuredError Schema
title_en_sibling: ENG-FAB v1.9 Candidate — F-16 StructuredError Schema
author: SCRIBE + LINNAEUS (TW translation backfill cycle 03-20)
date: 2026-05-02
cycle: 03-20 (TW backfill per D-§5 ratified + Batch 17 #4 APPROVED)
status: CANDIDATE (TW sibling parity per Rule #78 §78.5 BINDING-tier reflexive; SHOULD 24h grace tier per F-15 v3 §3.2)
authority: research-team R4 deliver (cycle 03-20)
supersedes: none
cross_refs:
  - openstarry_eco/share/openstarry_doc/Research_Methodology/15_ENG_FAB_v1.9_F_16_StructuredError.md (EN sibling)
  - research record/cycle03-20/openstarry_doc/Reference/TW_backfill_list_cycle03-20.md
  - research record/cycle03-20/deliver/Master_Ratification/Batch_17_Request.md (Item #4)
hard_rule_restated: "TW sibling per Rule #78 §78.5 BINDING-tier reflexive same-PR forward backfill; MR-12 forward-only. CANDIDATE tier per OPT-B; 24h grace tier per F-15 v3 §3.2 SHOULD-tier discipline."
---

# ENG-FAB v1.9 候選 — F-16 StructuredError Schema

## 1. 狀態

**CANDIDATE** 於 cycle 03-14 R4 close。Master ratification（Batch 11 Item #1）後升 SHOULD initial；MUST advancement scheduled at cycle 03-17 Plan iff 三 observation metrics ≥ 80% over 2-cycle window（cycles 03-15 + 03-16）per D-§4-03 (B-6) 20/3。

**ENG-FAB roster impact**：v1.8 = 48 items canonical PRESERVED；v1.9 = 49 items if Master ratifies F-16 increment。CV-§4-α（R3 cycle 03-14）reaffirmed UNANIMOUS — F-16 為單一 item increment、非 v1.8 的 item-count revision。

## 2. R3 來源

- **D-§4-01 (A-3)** UNANIMOUS 23/0 — 10-constant enum closed（8 base + Halt + Other；Madhyamaka-coherent fallback rule absorbed）。
- **D-§4-04 (A-4)** UNANIMOUS 23/0 — Plan48 structured-log transport binding mandatory；F-16 不是 parallel emission channel。
- **D-§4-02 (B-5)** 19/2/2 — `likely_cause` prefix-discipline（`verified:` / `inferred:` / `speculation:` per ASANGA Yogācāra epistemic-status）；DSS-5 absorbed verbatim §10。
- **D-§4-03 (B-6)** 20/3 — 2-cycle observation window with 80% threshold for SHOULD→MUST advancement；DSS-15（WIENER + RUSSELL conservative 3-cycle）absorbed verbatim §10。

## 3. Binding Text

### 3.1 F-16 Schema — 6-Field StructuredError Record

JSON-serialisable、fixed field order、transport-bound 至 Plan48 structured-log channel：

```text
{
  "error":                  <enum; 10 constants — 見 §3.2>,
  "message":                <human-readable, single sentence, ≤ 200 chars>,
  "likely_cause":           <prefix-disciplined string — 見 §3.3; ≤ 200 chars>,
  "suggested_fix_location": <relative file path + optional line range, e.g.
                              "apps/runner/src/event-bus.ts#L142-L160";
                              if unknown, the literal string "unknown" REQUIRED>,
  "context":                <object; relevant identifiers, e.g.
                              { "plan_id": ..., "request_id": ..., "trace_id": ... }>,
  "trace_id":               <opaque correlation id; structured-log existing trace
                              id when present (single source of truth)>
}
```

**Transport binding（D-§4-04 UNANIMOUS）**：F-16 emissions MUST 透過 Plan48 structured-log channel 作為 payload type 流。F-16 不是 parallel emission channel。SCRIBE 在第一 F-16 site introduction 時 consult Plan48 structured-log existing payload taxonomy；若 Plan48 已附 trace correlation id，F-16 defers to 該 field name。

### 3.2 列舉 `error` Set — 10 Constants（D-§4-01 UNANIMOUS）

Closed enumeration；8 base + Halt + Other：

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

**Madhyamaka-coherent fallback rule**（NAGARJUNA caveat absorbed at R3 UNANIMOUS）：

> Parsers operating on schema version `v_N` MUST 將任何不在 `v_N` 中的 discriminator 視為 semantically equivalent to `Other`。此 operationally 為 graceful-degradation rule、與 engineering-aligned。Set 為 conventional（saṃvṛti-satya 世俗諦）、非 essentialist（per NAGARJUNA Madhyamaka critique absorbed）。

**MR-10 back-fill discipline**：未來 cycles 可透過 MR-10 back-fill rules 擴展 enumeration；每 back-fill 在 Master Ratification batch 中 ratifies 新 constant。

### 3.3 `likely_cause` Prefix-Discipline（D-§4-02 19/2/2 — DSS-5 absorbed）

ASANGA Yogācāra critique 浮現原 `likely_cause` field 中的 epistemic-status drift：downstream LLM agent 讀此 field 即使作者寫為 speculation 也 treat 為 ground truth。Prefix-discipline 解此問題：

`likely_cause` field MUST 以 三個 controlled prefixes 之一 開頭：

- **`verified: <cause statement>`** — 作者以 reproduction confirm（sampajāna；direct knowledge）。Downstream consumers MAY 以此為 ground truth 行動。
- **`inferred: <cause statement>`** — 作者有 strong evidence 但無 direct reproduction（anumāna；inference）。Downstream consumers 應 weight 為 high-confidence 但 action 前 check。
- **`speculation: <cause statement>`** — 作者最佳猜測（kalpanā；constructed）。Downstream consumers NOT 應 act on this；應 solicit clarification 或 attempt reproduction。

若 field genuinely empty（無 cause hypothesis），literal string `"speculation: unknown"` REQUIRED（parallel to `suggested_fix_location: "unknown"` sentinel discipline）。

**ATHENA practical concern absorbed**：在 production LLM workflows 中，prefix delivers AI-Agent-Toolkit rationale 鎖定的 actual token-economy lever。看到 `speculation:` 的 LLM consumer 知道 ask for clarification 而非 retry；看到 `verified:` 的 LLM deterministically retries。

### 3.4 Strength Path — SHOULD → MUST（D-§4-03 20/3 — DSS-15 absorbed）

Four-stage advancement：

1. **Plan50 / Plan52 dispatch（cycle 03-14 R4 close）** — F-16 introduced as **SHOULD**。
   - `apps/runner` 中新 code + 新 pushInput plugin SHOULD 在每 reachable error path emit StructuredError。
   - 既有 `apps/runner` partial-pattern code **不 retrofitted** per MR-12 + CV-§4-β。

2. **2-cycle observation window**（cycles 03-15 + 03-16）— SCRIBE 自 delivery_reports 蒐集 per-error compliance counts 與 field-completion rates。三 observation metrics：
   - **adoption rate** — 新 error sites emitting conforming F-16 之 %。Target ≥ 80% over 2-cycle window（per RUSSELL agent-theory baseline；自 R1 tentative 90% 依 second-cycle-maturity calibration relaxed）。
   - **field completion rate** — per-field 已填 %。`suggested_fix_location: "unknown"` rate tripwire — 若 > 30% trends，在 MUST 前 surface 給 revision。
   - **prefix-discipline adoption rate** — `likely_cause` fields conforming 三 prefixes 之一 之 %。

3. **MUST advancement（cycle 03-17 Plan、typically Plan52+1）** — 若三 metrics 在 2-cycle window 全 ≥ 80%，F-16 advances to MUST。新 code MUST 在每 error path emit conforming F-16。既有 partial-pattern code 仍 exempt per MR-12。

4. **Revision branch** — 若 observation surfaces structural problem（enumeration too narrow、`unknown` rate > 30% sustained、或 prefix-discipline adoption < 60%），candidate 在 MUST advancement 前 revised。Revision 走過 R0-R4 again 為 v1.9.1 amendment proposal。

### 3.5 Apply Scope（per MR-12 + CV-§4-β）

**In scope**（forward-only）：
- 自 cycle 03-14 起 authored 的 `apps/runner` code
- 新 plugins：pushInput Plan52；Plan50 σ_regime emissions；future plugins

**Out of scope**：
- 既有 plugins（gear-arbiter-dynamic；Plan47 K-3；等）— 不 retrofitted
- Cycle 03-13 與更早 code paths preserved as-is

**Edge case**：當既有 plugin 在 cycle 03-14+ 因任何理由（bug fix、feature addition）**modified**，被 modified 的 function SHOULD 在其 newly-touched error sites adopt F-16；周圍 unmodified code 仍 exempt。此為標準 MR-12 "modify-touch only" discipline。

### 3.6 Effort Estimate

- O4 / Plan-spec 中 F-16 spec text：~80 lines（本 section + Plan52 dispatch §pushInput error wrapping）。
- Plan50 σ_regime first adoption：σ_regime emission paths 中 ~20 LOC error wrapping。
- Plan52 pushInput first adoption：pushInput plugin entrypoints 中 ~30 LOC error wrapping。
- `audit_calc.py` extension（cycle 03-15）中 F-16 audit logic：~60 LOC delta。
- **MUST 時 Total LOC budget**：跨 cycles 03-14 + 03-15 + 03-16 incremental 100-150。

### 3.7 Conflict-Check vs ENG-FAB v1.8（Final）

| ENG-FAB v1.8 item | 與 F-16 關係 | Verdict |
|-------------------|------------------|---------|
| F-9（openstarry_doc completeness）| independent（covers docs、非 error format）| no overlap |
| F-13（Plugin Hook Dispatch Verified）| independent（covers hook registration、非 error format）| no overlap |
| F-14（External Resource Usage Baseline）| independent（covers billing observation）| no overlap |
| F-15（Code-Path Verified Before Impact Assessment）| independent input-discipline（impact-assessment vs runtime error emission）| no cycle |
| Plan48 structured-log（C48-M1）| **complementary；F-16 為 structured-log 上的 payload type** | binding overlap resolved by D-§4-04 transport mandate |

無 v1.8 item rendered redundant；無 MR requires WAIVE；無 Rule conflicts with SHOULD→MUST advancement path。

## 4. Worked Example Emissions

### 4.1 ValidationError in pushInput plugin（Plan52 first-emission case）

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

### 4.2 DependencyFailure（HMAC verification fail）

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

### 4.3 SchemaDriftDetected（Plan49 class）

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

### 4.4 Halt（cycle-halt-class event）

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

## 5. Observation Metrics — 詳細定義

對 2-cycle observation window（cycles 03-15 + 03-16），SCRIBE 蒐集：

### 5.1 Adoption Rate

```
adoption_rate = (count of F-16-conforming error emissions in cycle X new code)
              / (count of all error emissions in cycle X new code)
```

- Numerator：通過 §3.1 schema validation 的 emissions。
- Denominator：cycle 03-14+ authored code paths 上的 emissions。
- Threshold：每 cycle ≥ 80%；2-cycle window aggregate 也 ≥ 80%。

### 5.2 Field Completion Rate

```
field_completion_rate[field] = (count of F-16 emissions with non-default <field>)
                             / (count of all F-16 emissions)
```

- Per-field；對 `error` / `message` / `likely_cause` / `suggested_fix_location` / `context` / `trace_id` 各報告。
- Tripwire：`suggested_fix_location: "unknown"` rate > 30% over 2-cycle window — 在 MUST advancement 前 surfaces 給 revision。

### 5.3 Prefix-Discipline Adoption Rate

```
prefix_adoption_rate = (count of `likely_cause` fields beginning with valid prefix)
                     / (count of all F-16 emissions with `likely_cause` field present)
```

- Valid prefixes：`verified:` / `inferred:` / `speculation:` / `speculation: unknown`（sentinel）。
- Threshold：advancement 每 cycle ≥ 80%。

### 5.4 計算責任

`audit_calc.py` extension（cycle 03-15 ~60 LOC delta）在每 cycle R4 close 計算三 metrics。Output 由 Dev append 至 `delivery_report.md §F16_observation_metrics` section；SCRIBE cross-checks 並納入 O7 evidence chain。

## 6. Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 / MR-4（Tenet wording unchanged）| ✅ PASS | 無 Tenet wording proposed |
| MR-5 hard（Tenet #10 status unchanged）| ✅ PASS | F-16 為 plugin/runtime layer；無 compliance status implication |
| MR-6（Core 零）| ✅ PASS | Plugin/runtime emission only；無 Core surface |
| MR-7（timing audit）| ✅ PASS | SHOULD→MUST scheduled；2-cycle observation discipline |
| MR-8（quality first）| ✅ PASS | 80% threshold relaxation per second-cycle empirical maturity |
| MR-9（no MUST WAIVE）| ✅ PASS | SHOULD→MUST 為 strengthening path、非 waive |
| MR-10（back-fill）| ✅ PASS | Enum extensible via future MR-10 back-fill |
| MR-11（dissent preservation）| ✅ PASS | DSS-5 + DSS-15 verbatim §10 |
| MR-12（既有 not retrofit）| ✅ PASS | 既有 apps/runner partial-pattern code exempt |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | 無 10-Tenet violation；endpoint unchanged；control-range unaffected |

### 6.3 FORBIDDEN-Phrasings List（Strength-Discipline Wording Sweep）

per cycle 03-14 R4 ratification（base 3 patterns；coordinator 補錄至 canonical 2026-04-27）+ cycle 03-15 Master Ratification Batch 12 Item #6（near-miss 擴展 3 patterns；UNANIMOUS 23/0）

以下措辭在 R0/R1/R2/R3/R4 任一階段或 deliver/results/openstarry_doc/todo/test_instructions 任一產出**禁用**（隱晦 advocate F-16 SHOULD→MUST 升級即違反 MR-9 強度 disciplined）：

**Base patterns（cycle 03-14 R4 ratified）**：
1. "F-16 is now MUST"
2. "should advance to MUST"
3. "is ready for MUST"

**Near-miss patterns（cycle 03-15 Batch 12 Item #6 ratified）**：
4. "trending toward MUST"
5. "MUST-ready signal positive"
6. "promotion path effectively confirmed"

**判定原則**：F-16 強度由 SCRIBE 排定 2-cycle observation table 達標後在 cycle 03-17 Plan dispatch 決定；任何在此之前以**陳述**或**暗示**方式 framing 「F-16 已 / 即 / 應 / 將 升 MUST」的措辭一律 SCRIBE wording-sweep reject 並回退作者修正。Strict reading；不容意會。

**範圍**：cycle 03-15+ forward only；既有 cycle 03-14 ratified text 不 retrofit per MR-12。

**Activation**：Master Ratification Batch 12 ratify 後即時生效（2026-04-27）。

## 7. Authority Notes

- **Authority route**：Master ratification（ENG-FAB increment v1.8 → v1.9 = 49 candidate）。
- **Effective once Master ratifies**：SHOULD initial；MUST advancement contingent on 2-cycle observation pass。
- **2-cycle window administration**：SCRIBE 在 cycle 03-15 R4 close（1-cycle interim）與 cycle 03-16 R4 close（2-cycle aggregate）排定 observation table emission。Cycle 03-17 Plan absorbs MUST decision。

## 8. Cross-References

- `research record/cycle03-14/deliver/O4_R_input_5_candidates_final.md §2` — F-16 narrative source
- `research record/cycle03-14/R3/R3_decision_log.md §4 D-§4-01..04` — R3 voting record
- `research record/cycle03-14/deliver/Master_Ratification/Batch_11_Request.md §3.1 Item #1` — ratification dispatch
- `research record/cycle03-13/openstarry_doc/Research_Methodology/11_ENG_FAB_v1.8_Binding.md` — v1.8 = 48 baseline（preserved）
- `research record/cycle03-14/deliver/O1_pushInput_Plan52_final.md §pushInput-error-wrapping` — Plan52 first-emission site
- `research record/cycle03-14/deliver/O2_Plan50_sigma_regime_final.md §F16-applicability` — Plan50 σ_regime applicability confirmation（D-§2-Q3 UNANIMOUS）

## 9. Authority Notes（Strength Discipline）

SHOULD initial → MUST advancement design absorbs：

- **GUARDIAN security-first concern** — error format 一致性為 operational defense；SHOULD initial 給 Dev runway，MUST consummates discipline。
- **KERNEL OS-discipline** — error enum closure（10 constants）parallels Linux errno；`Other` fallback parallels `EOTHER` graceful-degradation discipline。
- **ASANGA Yogācāra discipline** — prefix-discipline operationalises 八識（vijñāna）epistemic-status awareness 為 machine-readable form。
- **WIENER conservatism** — 2-cycle minimum 允許 feedback-loop calibration；若 metrics 在 cycle 03-16 偏離 target，R0-R4 amendment cycle absorbs。

## 10. Dissent Slots — DSS-5 + DSS-15 Verbatim（per MR-11）

### 10.1 DSS-5（D-§4-02、4 minority votes — 2 reject + 2 sibling-field）

> NAGARJUNA + ASANGA philosophical objection to prefix-discipline：
>
> "Prefix-discipline reifies the verified / inferred / speculation distinction as if these were three discrete epistemic categories rather than a continuum (per Yogācāra cittamātra and Madhyamaka two-truth refinements). A downstream consumer that strictly branches on prefix may discard nuance — e.g., a `verified:` claim that is actually 'verified for one reproduction but with non-zero variance' is conflated with a 'verified-deterministic' claim. Prefer a separate `likely_cause_confidence` enumeration with explicit gradation (e.g., `low / medium / high / verified`) over inline prefix."
>
> Sibling-field alternative（2 votes、ATHENA + 1）：
>
> "Inline prefix mixes data type (cause statement) with metadata (epistemic status). A separate `likely_cause_confidence` field is structurally cleaner and parser-simpler."
>
> R3 majority（19/2/2）偏好 prefix-discipline 理由：
> - Inline prefix 為 single-field self-contained；sibling-field 需 two-field consistency check。
> - Three-prefix discipline 對 LLM consumers operationally 已足（per ATHENA practical concern 之 actual target consumer）；4-or-more gradation 產生 consumer confusion。
> - Madhyamaka 二諦（two-truth：saṃvṛti-satya 世俗諦 / paramārtha-satya 勝義諦）concern 透過 conventional（saṃvṛti-satya）framing 吸收 — prefixes 為 operational tags、非 metaphysical claims。

### 10.2 DSS-15（D-§4-03、3 minority votes — WIENER + RUSSELL + 1）

> WIENER + RUSSELL conservative 3-cycle 偏好：
>
> "Cybernetic feedback-loop calibration requires sufficient sample variance to distinguish stable adoption from cycle-1 enthusiasm artefact. A 3-cycle window with 90% threshold (cycles 03-15 + 03-16 + 03-17) provides higher signal-to-noise on adoption stability. The 2-cycle / 80% formulation risks promoting on a transient peak."
>
> R3 majority（20/3）偏好 2-cycle / 80%：
> - 2-cycle / 80% 對齊 RUSSELL 自身 agent-theory baseline 之 second-cycle empirical maturity（dissent set 為了 cybernetic conservatism 將其擱置）。
> - 自 90% 至 80% 的 threshold relaxation 為對 conservative concern 的 substantive concession — threshold NOT 鬆於 second-cycle maturity baseline。
> - Revision branch（§3.4 stage 4）absorbs conservative concern：若 2-cycle data anomalous，revision 而非 MUST advancement。

兩個 dissent positions 全 preserved per MR-11；若 cycle 03-15 data exhibits anomalous transient-peak signature，SHOULD→MUST advancement path 可透過 R0-R4 amendment 收緊至 3-cycle / 90%。

---

*Cycle 03-14 R4 final — 2026-04-25*
*Authors：GUARDIAN (#11) + KERNEL (#10) + ARCHIMEDES (#16) + ASANGA (#8) + SYNTHESIST (#1)*
*Status：CANDIDATE（pending Master Ratification Batch 11 #1）*
*ENG-FAB v1.8 = 48 canonical PRESERVED；v1.9 = 49 if Master ratifies*
*TW sibling backfill：cycle 03-20 D-§5 ratified + Batch 17 #4 APPROVED；CANDIDATE tier per OPT-B；24h grace per F-15 v3 §3.2 SHOULD-tier discipline*
