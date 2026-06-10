---
title: Plan51 — Zod Gate × 4-of-5 Modules 候選方案（cycle 03-16 首次出貨）
title_en_sibling: Plan51 — Zod Gate × 4-of-5 Modules Candidate (cycle 03-16 First-Shipping)
author: SCRIBE + LINNAEUS (TW translation backfill cycle 03-20)
date: 2026-05-02
cycle: 03-20 (TW backfill per D-§5 ratified + Batch 17 #4 APPROVED)
status: BINDING (TW sibling parity per Rule #78 §78.5 BINDING-tier reflexive)
authority: research-team R4 deliver (cycle 03-20)
supersedes: none
cross_refs:
  - openstarry_eco/share/openstarry_doc/Technical_Specifications/Plan51_Zod_Gate_Binding.md (EN sibling)
  - research record/cycle03-20/openstarry_doc/Reference/TW_backfill_list_cycle03-20.md
  - research record/cycle03-20/deliver/Master_Ratification/Batch_17_Request.md (Item #4)
hard_rule_restated: "TW sibling per Rule #78 §78.5 BINDING-tier reflexive same-PR forward backfill; MR-12 forward-only."
---

# Plan51 — Zod Gate × 4-of-5 Modules 候選方案（cycle 03-16 首次出貨）

## 1. 狀態

**CANDIDATE** 於 cycle 03-15 R4 close。經 Master Ratification Batch 12 Item #2 後升 **BINDING**。**首次出貨 cycle = cycle 03-16**，於 4-Plan stack 之後：Plan49 SHOULD landed → Plan50 σ_regime → Plan52 pushInput → Plan51 Zod gate。

**MR-12 forward-only honoured iff Plan51 spec 明確包含 v0.42 起每版 migration helpers**（≥3 prior versions）依 D-§5-G UNANIMOUS gate。

**Hard constraints 重申**：MR-2 / MR-4（Tenet 文字未動）/ MR-5 hard（Tenet #10 NC PENDING preserved）/ MR-6（Core 零 — 全 4 推薦 modules peripheral）/ MR-9（無 preemptive MUST F-16 binding）/ MR-11（4 dissent entries 逐字保留 §10）/ MR-12（forward-only 加 cross-version-skew migration helpers MUST per D-§5-G）/ ZT-1/2/3。

## 2. R3 來源

- **D-§5-A (A-4)** — 推薦 4-of-5 modules（17/6 aggregate）；per-module breakdown event-bus 19/3/1 / checkpoint-store 18/4/1 / WebSocket 21/2/0 / hook-registry 17/5/1 / **plugin-loader 9/11/3（無 super-majority — DEFERRED）**。DSS-A4 6 dissent §10。
- **D-§5-B (B-7)** — 第 5 module 身份 = Option α（5 modules 含 plugin-loader 為第 5）— 17/4/2；DSS-B7 4 dissent §10。（注意：count-axis Option α；implementation outcome 4-of-5 with plugin-loader DEFERRED 在 operational 上與 DSS-B7 偏好對齊。）
- **D-§5-C (B-8)** — rollout 順序 = **GUARDIAN-priority**（WebSocket → checkpoint-store → event-bus → hook-registry）UNANIMOUS 23/0。
- **D-§5-D (D-3)** — sunset = **sunset-by-execution** UNANIMOUS 23/0（active for 推薦 path；closes cycle 03-14 DSS-3 LEIBNIZ ossification concern）。
- **D-§5-E (C-4)** — hook-registry = **1 module + 2 schema artefacts**（DARWIN Strategy/Registry pattern）— 21/2；DSS-C4 2 dissent §10。
- **D-§5-F (A-5)** — vote granularity = B per-module 5 votes UNANIMOUS 23/0。
- **D-§5-G (B-9)** — **cross-version-skew migration helpers MUST**（HIGH per F-§5-R2-12）UNANIMOUS 23/0。
- **D-§5-H (D-4)** — schema-publication scope DEFER cycle 03-22+ Mesh / 03-23 API Runtime UNANIMOUS 23/0。
- **MRB-§5-01..03 RESOLVED**。
- **CV-§5-01..08** UNANIMOUS reaffirmed（Core 零 / Plan49 SCHEMA_DRIFT_MODE / F-16 SHOULD initial / Plan52 opaque sourceContext / Plan50 σ_regime non-interference / MR-12 forward-only / ZT-1/2/3 / Tenet #10）。

## 3. Binding Scope（Plan51 cycle 03-16 首次出貨）

### 3.1 4 Modules + 1 Shared Utility

| # | Module | LOC 中位 | LOC 範圍 | R3 票數 | Rollout 順序 |
|:-:|--------|:----------:|:---------:|:-------:|:-------------:|
| 1 | **WebSocket Zod gate** | 125 | 100-150 | 21/2/0 | **#1**（最高 GUARDIAN priority）|
| 2 | **checkpoint-store Zod gate**（含 cross-version-skew helpers MUST）| 110 | 85-140 | 18/4/1 | **#2**（disaster-recovery 保險）|
| 3 | **event-bus Zod gate**（CV-§5-05 σ_regime non-interference）| 110 | 85-135 | 19/3/1 | **#3**（Mesh forward-compat hub）|
| 4 | **hook-registry Zod gate**（1 module + 2 schema artefacts per D-§5-E）| 95 | 75-115 | 17/5/1 | **#4**（catches cycle 03-13 root-cause class）|
| 5 | **shared `ZodGateMiddleware` utility**（SUSSMAN strong support）| 40 | 30-50 | OQ-3 closed | shared — 跨全 4 modules 抽出 |

**Total factored**：~480 LOC 未 factored / **~405 LOC 中位** under DARWIN non-additive savings analysis（範圍 340-460 factored）。位於 Master letter §五 "300-500 LOC" sizing band 內 per Plan49 precedent（tests-in-band per F-§5-R2-09 VITRUVIUS）。

### 3.2 plugin-loader DEFERRED（cycle 03-17+）

**9/11/3 — 無 super-majority — DEFERRED**。依 D-§5-A breakdown：
- Plan51 entire 應 defer cycle 03-17+ post-AC-9（DSS-A4 cluster：LEIBNIZ + KNUTH + SUSSMAN + 3 others）
- 4-module commit 過早；event-bus + WebSocket 單獨已足
- Option β（4 modules 去掉 plugin-loader）較切合 Master letter "300-500" LOC band（DSS-B7 cluster 也引 LOC budget concern）
- plugin-loader 0-site greenfield 推遲合理

**Sustained MRB-06 deferral 附掛**：cycle 03-13 Plan49 spec MRB-06（plugin-loader 0-site 為 Plan50+ gap）trail SUSTAINED 但 cycle 03-15 R3 未 resolved。

**DEFERRED trail 的 sunset attachment**：soft anchor cycle 03-21（~6 cycles from cycle 03-15）為建議 re-evaluation floor。若 cycle 03-21 仍無 super-majority，MRB-06 應 explicit close-with-rationale 或 re-ratify 為 open architectural gap。Coordinator G5 obligation：跨 cycles 03-16/03-17/.../03-21 追蹤 plugin-loader DEFERRED trail。

### 3.3 Plan51 cycle 03-16 範圍外

- plugin-loader Zod gate（DEFERRED；cycle 03-17+ revisit per D-§5-A；AC-9 timing engagement）
- Schema-publication for client mirroring（D-§5-H DEFER cycle 03-22+；LEIBNIZ symmetric-fidelity foresight）
- Per-module SCHEMA_DRIFT_MODE override（OQ-6 KISS-defer；不在 Plan51 initial scope）
- 新 Rules / MR / ZT / ENG-FAB items（Master letter §七 #10；Plan51 inherits existing governance）

## 4. Module 規格（GUARDIAN-Priority Rollout 順序）

### 4.1 Module 1 — WebSocket Zod Gate（rollout #1）

**R3**：推薦 21/2/0。最高 GUARDIAN priority per D-§5-C UNANIMOUS。

**描述**：external-facing transport 供 agent-runner client 連線（browser dashboard、external orchestration、multi-process IPC stub）。1 schema-drift call-site existing per cycle 03-13 Plan49 §2.3。

**Schema artefact**：`WebSocketMessage<T>` = discriminated union over message-type（5-10 message types 由 Dev refinement）。

**Critical invariant per CV-§5-04 UNANIMOUS**（Plan52 opaque sourceContext non-interference）：若 message envelope carries 一個 sourceContext-typed field，WebSocket schema MUST 將該 field 標為 `z.unknown()`（或 `z.any()`）。Failure 違反 cycle 03-14 Plan52 ratified opacity invariant。

**Runtime guard**：
- **Inbound**：`safeParse` with mode fallback per Plan49 `SCHEMA_DRIFT_MODE`。**Initial mode = shadow** 至少 1 W2 round；隨後進展 `audited → strict`。
- **Outbound**：`parse`（assertion-style；we control the producer；LOW migration risk）。

**Migration risk**：HIGH for inbound（visible external behaviour change）+ LOW for outbound。Mitigation：shadow mode + Plan49 mode-progression pattern。

**LOC**：~125 中位（100-150）。4 module 中最大；反映 HIGH security-critical test ratio。

### 4.2 Module 2 — checkpoint-store Zod Gate（rollout #2；cross-version-skew helpers MUST）

**R3**：推薦 18/4/1，AND D-§5-G UNANIMOUS 23/0：cross-version-skew migration helpers MUST 包含。

**描述**：將 agent runtime state 持久化至 disk。2 schema-drift call-sites existing。Cross-version production data flowing per cycle 03-13 R3 StateTracker Path B。

**Schema artefact**：`CheckpointSchema<V>` parameterized by version tag。

**Cross-version-skew migration helpers（MUST per D-§5-G）**：
- **v0.42 起每版 migration helpers**（≥3 prior versions：v0.42 / v0.45 / v0.48 / v0.50）
- **Audited-mode 預設** for read-path；strict-mode 需 ≥1 W2 round zero-skew evidence
- **Graceful-degradation read-path**：read 失敗時 never block recovery on Zod gate failure
- **Explicit `from_version` → `to_version` migration matrix** in Plan51 spec doc
- **Audit emission**：`checkpoint_schema_violation` + `checkpoint_migration_applied` events 可區分
- **Cross-version round-trip test fixtures**：驗證 v0.42 checkpoint 在 v0.50 reader migration 後 cleanly 載入

**若無這些 helpers，MR-12 forward-only 有風險** — read-path enforcement 可能 break old checkpoints。F-§5-R2-12 HIGH severity。

**Runtime guard**：read = `safeParse` shadow → audited；write = `parse` assertion。

**Migration risk**：HIGH for cross-version-skew read sub-case + MEDIUM for in-process read + LOW for write path。

**LOC**：~110 中位（85-140；自 R1 ~90 上修以反映 mandatory cross-version-skew helpers）。

### 4.3 Module 3 — event-bus Zod Gate（rollout #3；σ_regime non-interference）

**R3**：推薦 19/3/1。

**描述**：in-process publish/subscribe substrate 承載 lifecycle events。3 schema-drift call-sites existing。

**Schema artefact**：`EventEnvelope<T>` typed wrapper + `EventBusSchemaRegistry` registry with register/lookup/validate API。

**Critical invariant per CV-§5-05**（Plan50 σ_regime non-interference）：σ-emission events 的 event-bus schema MUST 包含 `sigma_regime: z.enum(['composition_index', 'llm_variance', 'mixed'])` field。Plan51 honours Plan50 layer-orthogonality：structural validation at event envelope；numeric metric tag（sigma_regime）為 envelope 內的 payload field。

**Reflexive-case test fixture（per F-§5-R2-11 LEIBNIZ + WIENER NEW）**：emit 一個故意 malformed event；驗證 `event_bus_schema_violation` event 本身 cleanly validate（在 strict mode 下不被 suppressed）。

**Runtime guard**：audited mode 預設（subscribe + publish `safeParse`）。

**Migration risk**：MEDIUM。Mitigation：第一 cycle shadow mode，後 enforce。

**LOC**：~110 中位（85-135）。

### 4.4 Module 4 — hook-registry Zod Gate（rollout #4；1 module + 2 schema artefacts）

**R3**：推薦 17/5/1，AND D-§5-E binding（21/2）：1 module + 2 schema artefacts per DARWIN Strategy/Registry pattern。DSS-C4 2 dissent §10。

**描述**：追蹤哪些 plugins 已 register 哪些 lifecycle hooks。1 schema-drift call-site existing。Per cycle 03-13 root-cause investigation（`gear-arbiter-dynamic` plugin loader warning），hook-registry metadata fidelity 已是 known operational risk surface。

**Schema artefacts（DARWIN Strategy/Registry pattern split per D-§5-E）**：
- **`HookRegistration`** — Registry-pattern data structure schema（registration metadata、哪個 plugin declared 哪些 hooks、version）
- **`HookContract<I, O>`** — Strategy-pattern dispatch contract schema（per-event-type、dispatch registered handler with input/output contract）

**為何 1 module + 2 artefacts（不是 2 modules）**：registration 與 dispatch 為兩個 distinct lifecycle phases 共享一個 data structure。Forcing 進 separate modules 會 fragment registry without reducing abstraction-leak。Forcing 進 one schema artefact 會 conflate lifecycle phases。1-module-2-artefacts 折衷顧及兩面（D-§5-E 21/2；DSS-C4 2 minority §10）。

**Runtime guard**：
- Registration-time（`HookRegistration`）：`safeParse` strict from start。
- Dispatch-time（`HookContract<I, O>`）：`safeParse` audited mode initially。

**Migration risk**：LOW-MEDIUM。

**LOC**：~95 中位（75-115；自 R1 ~80 上修以反映 D-§5-E 的 2 distinct schema artefacts）。

## 5. Cross-Version-Skew Migration Helpers MUST（D-§5-G）

### 5.1 R3 binding decision

**D-§5-G UNANIMOUS 23/0**：cross-version-skew migration helpers MUST 包含於 Plan51 spec（HIGH per F-§5-R2-12）。

### 5.2 為何 MUST（非 SHOULD）

Cross-version-skew checkpoints 自 cycle 03-08（v0.42）起 in the wild 存在。Plan51 read-path Zod gate 在 v0.50+ enforced 將 reject 這些 checkpoints 除非提供 v0.42 → v0.50 migration path。若無 helpers，"reject on read" = silent recovery failure 對任何 upgrade runner with stale checkpoints 的 operator。**MR-12 forward-only 有風險** — Plan51 將 retroactively-break old checkpoints。R3 ratified MUST level（UNANIMOUS）正是要在 Master ratification 前 close 此 gap。

### 5.3 Required scope（Plan51 spec items per §4.2）

1. v0.42 起每版 migration helpers（≥3 prior versions）
2. Spec doc 中的 explicit `from_version` → `to_version` migration matrix
3. Read-path audited-mode 預設；strict-mode 需 ≥1 W2 round zero-skew evidence
4. Graceful-degradation read-path
5. 可區分的 audit emissions
6. Cross-version round-trip test fixtures

## 6. Non-Interference Invariants（Plan49 / Plan50 / Plan52）

### 6.1 Plan52 pushInput（cycle 03-14 ratified、live）

CV-§5-04 UNANIMOUS 於 R3 reaffirmed：
1. **Surface separation**：Plan52 plugin-public-input ε-surface vs Plan51 internal-peripheral module boundaries。**無重疊**。
2. **MR-6 Core 零跨兩者 preserved**：Plan52 ε-surface = 1 optional field；Plan51 Zod gates = 0 Core surface。
3. **Opaque sourceContext non-interference**：Plan52 的 opaque sourceContext **deliberately 不 Zod-validated** at Core level。**Plan51 WebSocket schema MUST 將任何 sourceContext-typed fields 視為 `z.unknown()` 或 `z.any()`** 以 preserve opacity（binding spec invariant）。
4. **Plan52 inherits Plan49 schema-drift policy**；Plan51 inherits 同樣的。無 conflict。

### 6.2 Plan50 σ_regime（cycle 03-14 ratified、50 LOC binding、atomicity MUST）

CV-§5-05 於 R3：
1. **Layer separation**：Plan50 SPC plugin-internal numeric layer vs Plan51 module structural-schema layer。Layer-orthogonal。
2. **σ-emission events on event-bus**：Plan51 之下 σ-events 的 Zod schema MUST 包含 `sigma_regime: z.enum(['composition_index', 'llm_variance', 'mixed'])` payload field（binding per CV-§5-05）。
3. **無 retroactive risk**：Plan50 forward-only；Plan51 forward-only。兩者 honour MR-12。
4. **Atomicity**：Plan51 應追求 **per-module migration atomicity**（在單一 PR 內；非 4-module mega-merge 而是 4 sequential per-module atomic merges）。

### 6.3 Plan49 schema-drift mode（cycle 03-13 ratified、live）

CV-§5-02 於 R3：
- Plan51 schemas 透過 `resolveSchemaDriftMode()`（Plan49 helper）dispatch。
- Plan51 audited-mode events 使用同一 `schema_drift_audit` event family。
- F-16 StructuredError integration extends，不取代 Plan49 emission shape。
- 單一 env var process-global per D-12（cycle 03-13 UNANIMOUS）；無 per-module fragmentation。

**Plan51 = Plan49 的 clean expansion**。

### 6.4 4-Plan stack 一致性

| Plan | Layer | Cycle live | 與 Plan51 互動 |
|------|-------|-----------|--------------------------|
| Plan49 | schema-drift mode dispatcher | 03-13 | Plan51 inherits SCHEMA_DRIFT_MODE；clean expansion |
| Plan50 | σ_regime SPC tag | 03-14 | Plan51 event-bus schema 包含 sigma_regime enum field |
| Plan52 | pushInput ε-surface | 03-14 | Plan51 WebSocket schema 將 sourceContext 視為 opaque |
| Plan51 | Zod gate × 4 modules + shared utility | 03-16 candidate | additive layer；forward-only；MR-12 honoured iff cross-version helpers |

## 7. Sunset-by-Execution（D-§5-D UNANIMOUS）

**D-§5-D UNANIMOUS 23/0**：推薦 sunset-by-execution。Active per D-§5-A 推薦 outcome（4-of-5 modules 在 cycle 03-16 implemented）。**6-cycle floor framing N/A for 推薦 path**。Closes cycle 03-14 DSS-3 LEIBNIZ ossification concern for 推薦 trail。

**plugin-loader DEFERRED trail** retains soft 6-cycle floor anchor（cycle 03-21 re-evaluation floor）per §3.2。

## 8. Implementation 順序

### 8.1 Cycle sequence

1. **Plan49 SHOULD landed**（cycle 03-13 live；SCHEMA_DRIFT_MODE 提供 dispatcher）
2. **Plan50 σ_regime**（cycle 03-14 live；sigma_regime enum field 由 event-bus schema consume）
3. **Plan52 pushInput**（cycle 03-14 live；opaque sourceContext invariant 由 WebSocket schema honour）
4. **Plan51 cycle 03-16 first-shipping**（this spec ratifies；GUARDIAN-priority rollout per D-§5-C）

### 8.2 Cycle 03-16 內 — Plan51 module sequence（per-module atomicity MUST）

| Stage | Module | Atomicity | 初始 mode |
|:----:|--------|-----------|--------------|
| #1 | WebSocket | 單一 PR | Shadow（inbound）+ assertion（outbound）|
| #2 | checkpoint-store | 單一 PR（含 cross-version-skew helpers）| Audited（read；never block recovery）+ strict（write）|
| #3 | event-bus | 單一 PR | Audited（matches Plan49 baseline）|
| #4 | hook-registry | 單一 PR（1 module + 2 schema artefacts）| Strict（registration）+ audited（dispatch）|

**Atomicity：per-module single-PR**（Plan50 atomicity MUST 風格）。每 PR 包含：
- Schema artefact(s)
- Validation hooks
- Migration helpers（where applicable）
- Test fixtures（positive + negative + edge-case + reflexive where applicable）
- Doc snippet（TW parity per Rule #78 §78.3 governance MUST + canonical SHOULD per D-§2-07）
- CHANGELOG entry

### 8.3 W2-R17 + W3 進展（cycle 03-22+ candidate）

inbound WebSocket 的 strict-mode rollout + checkpoint-store read-path 的 audited-strict transition 需要 **≥1 W2 round zero-skew evidence**。W2-R17 candidate per cycle 03-22 timeline。

## 9. Cycle 03-22+ Schema-Publication DEFERRED（D-§5-H）

**D-§5-H UNANIMOUS 23/0**：LEIBNIZ symmetric-fidelity Plan51 schema-publication scope DEFER cycle 03-22+ Mesh / 03-23 API Runtime。Cycle 03-15 = 無 schema-publication work。Plan51 cycle 03-16 first-shipping = runner-side only。Documentation cross-references cycle 03-22+ deferred work（Plan53 / Plan54 candidate；~10-20 LOC + doc work）。

## 10. Dissent Preservation（per MR-11 verbatim）

### 10.1 DSS-A4（D-§5-A Plan51 推薦 4-of-5 modules；vote 17/6）

**Minority**：6 dissent（LEIBNIZ + KNUTH + SUSSMAN + 3 others）。

**Verbatim**："Plan51 entire 應 defer cycle 03-17+ post-AC-9；4-module commit 過早；event-bus + WebSocket alone sufficient."

**R4 obligation**：explicit dissent slot — 此處逐字保留 per MR-11。Dissent 適用於 *entire* Plan51 scope。Master 可考慮是否將 cycle 03-16 的 4-module commitment 收緊至 event-bus + WebSocket only（2-module variant）；此將偏離 R3 ratified outcome 並需 Master ruling。

### 10.2 DSS-B7（D-§5-B 第 5 module 身份 Option α；vote 17/4/2）

**Minority**：4 dissent。

**Verbatim**："Option β（4 modules drop plugin-loader）better honours Master letter '300-500' LOC band；plugin-loader 0-site greenfield 推遲合理."

**R4 obligation**：footnote — 此處逐字保留 per MR-11。注意：implementation outcome（D-§5-A）落為 4-of-5 with plugin-loader DEFERRED，operationally 與 DSS-B7 中 Option β 偏好對齊。

### 10.3 DSS-C4（D-§5-E hook-registry 1 module + 2 schemas vs 2 modules；vote 21/2）

**Minority**：2 dissent。

**Verbatim**："2 modules cleaner separation；DARWIN pattern 過度抽象."

**R4 obligation**：footnote — 此處逐字保留 per MR-11。

### 10.4 DSS-3 carryover（cycle 03-14 LEIBNIZ Plan51 ossification）

**Verbatim**："Plan51 ossification concern if indefinite carry-forward without sunset clause."

**Status**：**resolved by R3**（D-§5-D 推薦 outcome 觸發 sunset-by-execution；推薦 path 不需 6-cycle floor）。**Closed by R3** for 推薦 trail。plugin-loader DEFERRED trail retains soft 6-cycle floor anchor per §3.2。

### 10.5 §5 dissent count summary

| Source D-item | Dissent count | Verbatim | Status |
|---------------|:-------------:|----------|--------|
| D-§5-A（Plan51 4-of-5 推薦）| 6（DSS-A4）| §10.1 | preserved |
| D-§5-B（Option α 5 modules）| 4（DSS-B7）| §10.2 | preserved |
| D-§5-C（GUARDIAN-priority rollout）| 0 | UNANIMOUS | — |
| D-§5-D（sunset-by-execution）| 0 | UNANIMOUS | — |
| D-§5-E（hook-registry 1+2）| 2（DSS-C4）| §10.3 | preserved |
| D-§5-F（per-module granularity）| 0 | UNANIMOUS | — |
| D-§5-G（cross-version-skew MUST）| 0 | UNANIMOUS | — |
| D-§5-H（schema-publication DEFER）| 0 | UNANIMOUS | — |
| **Chapter §5 internal** | **3 entries（12 votes）** | （DSS-A4 + DSS-B7 + DSS-C4）| preserved |
| **Carryover** | **1（DSS-3 → resolved D-§5-D）** | §10.4 | closed |

## 11. MR/ZT 合規 Audit

| Constraint | Status | Evidence |
|-----------|:------:|----------|
| **MR-2**（Tenet 文字未動）| PASS | 無 Tenet content modified |
| **MR-4**（ENG-FAB candidate 引用）| PASS | F-16 SHOULD initial 僅 cited；未 modified |
| **MR-5 hard**（Tenet #10 NC PENDING preserved）| PASS | 無 Tenet #10 status change |
| **MR-6（Core 零）** | PASS | 全 4 推薦 modules 位於 `apps/runner/src/<module>/` peripheral；CV-§5-01 UNANIMOUS |
| **MR-9**（無 preemptive MUST F-16 binding）| PASS | F-16 SHOULD initial inheritance；CV-§5-03 reaffirm |
| **MR-10 + MR-12**（forward-only with classification）| PASS w/ caveat | MR-12 forward-only honoured iff cross-version-skew migration helpers present（D-§5-G UNANIMOUS MUST）|
| **MR-11**（dissent preservation）| PASS | 4 dissent entries §10 verbatim |
| **MR-13** standby | PASS | §5 未 invoke |
| **ZT-1**（Tenet preservation）| PASS | CV-§5-07 UNANIMOUS |
| **ZT-2**（endpoint 10/0/0★）| PASS | Plan51 = governance/architectural、非 Phase 6 functional；endpoint unchanged |
| **ZT-3**（control-range triple-check）| PASS | 此 spec 中無 σ scope change |

---

*Plan51 Candidate Spec — Cycle 03-15 R4 Final*
*Authors：TANENBAUM (#20) + SUSSMAN (#22) + KERNEL (#10) + MESH (#4) + HERACLITUS (#15) + GUARDIAN (#11) + ARCHIMEDES (#16) + SYNTHESIST (#1)*
*Status：CANDIDATE（pending Master Ratification Batch 12 #2）*
*4-of-5 推薦：WebSocket / checkpoint-store / event-bus / hook-registry；plugin-loader DEFERRED cycle 03-17+*
*GUARDIAN-priority rollout（D-§5-C UNANIMOUS）；cross-version-skew helpers MUST（D-§5-G UNANIMOUS）*
*LOC factored ~405 中位（340-460 範圍）；位於 Master letter §五 "300-500" band 內 per Plan49 precedent*
*Non-interference：Plan49 + Plan50 + Plan52 4-Plan stack architecturally 一致*
*Dissent preserved：DSS-A4 (6) + DSS-B7 (4) + DSS-C4 (2) verbatim per MR-11；DSS-3 carryover closed by D-§5-D UNANIMOUS*
*Implementation 順序：Plan49 → Plan50 → Plan52 → Plan51（cycle 03-16 first-shipping）*
*Hard constraints：MR-5 hard ✓ / MR-6 ✓ / MR-9 ✓ / MR-11 ✓ / MR-12 ✓ + D-§5-G MUST close / ZT-1/2/3 ✓ / Tenet #10 untouched*
