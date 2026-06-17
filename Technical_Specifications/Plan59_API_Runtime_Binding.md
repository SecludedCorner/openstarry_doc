# Plan59 — API Runtime BINDING Specification

> ✅ **[實作狀態 — v0.59.6（2026-06-16）] BINDING + SHIPPED。** 本規格不再是「pending」設計稿——api-runtime 插件已完整出貨並接線上線，與下方規格逐條相符：
> - **插件源碼**：`openstarry_plugin/api-runtime/src/{index,runtime,observe,invoke,state}.ts`。
>   - 唯讀 `observe`（`observe.ts:34-61` `createObserve`；冪等、無 replay 證據，符 §6.1）；
>   - 變動 `invoke`（`invoke.ts:95-149` `createInvoke`；HMAC-SHA256 驗章 `invoke.ts:49-60` + `apr:` replay cache `invoke.ts:122-126` + `INTERVENTION_KINDS` allow-list 守門 `invoke.ts:106-111`）；
>   - 雙路徑由 `runtime.ts:23-30` `IRuntime` 組合（`observe`/`invoke`/`register`/`replayCacheSize`），boundary invariant §6.2 成立（兩 method 簽名皆不引用 ε-surface envelope 欄位）。
> - **3 個可變狀態欄位**（log_level / debug_flag / soft_tracing）：`state.ts:20-24` `PluginRuntimeRecord` + `state.ts:73-80` `mutate()`；預設 info/false/false（`state.ts:26-30`）。replay_cache_size 為 introspection-only（非可變），符 §6.3「3 active + 1 negative-scope sentinel」。
> - **SDK 型別**：`packages/sdk/src/types/api-runtime.ts`（自 `packages/sdk/src/index.ts:239-240` re-export）——`API_RUNTIME_REPLAY_CACHE_PREFIX = 'apr:'`（`api-runtime.ts:30`，符 §4 第 6 contributor）+ `INTERVENTION_KINDS = [log_level, debug_flag, soft_tracing]`（`api-runtime.ts:33-37`）+ invoke/observe request/result schemas。
> - **測試**：`openstarry_plugin/api-runtime/__tests__/{invoke,observe,plugin,state}.test.ts` 共 **42 個 it()/test()**（16+8+11+7）。
> - **接線上線**：`configs/phase6-agent.json:30` 載入 `@openstarry-plugin/api-runtime`（2026-06-11 修復衝刺加入，冷啟動 smoke 有可證明的 activation path）；亦見 `assertion-coverage.json` 4 條測試檔登錄。
>
> 下方原規格（含「pending Master Ratification Batch 19 #2」等發佈前措辭）保留作歷史設計敘述；實作狀態以本牌為準。

**Status**: BINDING + **SHIPPED（v0.59.x）** — 插件已出貨並接線（見上方實作狀態牌）。〔原文：cycle 03-22 R3 D-§1 ratified 23/0 UNANIMOUS plugin upfront + 20/3 super-majority replay cache 6-extend; pending Master Ratification Batch 19 #2 — 此 pending 措辭已過時。〕
**Authority**: Master Ratification (Batch 19 dispatch 2026-05-04)
**Cycle**: 03-22 (Phase 6 第六棒; 6/7 functional landing)
**Release**: v0.56.0-alpha minor-bump
**Plan number**: 59
**Subject**: API Runtime — runtime observability + bounded intervention via plugin form upfront
**五蘊 correspondence**: 識蘊 (Vijnana; consciousness / observability / introspection)

**Inheritance chain**: Plan52 pushInput → Plan54 AC-9 → Plan56 D-30-4 → Plan57 D-30-5 (+ plugin form amendment cycle 03-21) → Plan58 Mesh → **Plan59 API Runtime** (6th consecutive Phase 6 functional landing)

---

## §1 Background — Phase 6 6/7 + Vijnana correspondence

API Runtime enables runtime observability + bounded intervention within OpenStarry's 五蘊 architecture. **Architectural choice**: plugin form upfront at `openstarry_eco/agent_dev/openstarry_plugin/api-runtime/` via `createApiRuntimePlugin(manifest, factory(ctx))` factory pattern (per cycle 03-22 R3 D-§1-A 23/0 UNANIMOUS).

**Vijnana = 「了別」 (discriminating awareness)** translates to:
- **觀察 (Observation)**: plugin runtime state introspection — read-only per-plugin state / handler frames / capability holdings / replay cache stats;
- **介入 (Intervention)**: bounded mutability surface (log-level / debug flag / soft tracing) — 不可動 plugin lifecycle / 不可動 ε-surface。

**Forward-binding (NOT in scope this cycle)**:
- Mesh-routed cross-plugin API runtime command (cycle 03-23+);
- Plan60 Blackboard-Alaya integration (single-direction Plan60 → Plan59 query; per cycle 03-22 R3 D-§1-Clarif C2);
- Phase 7 batch elevation (Plan59 already plugin form; not in elevation list)。

## §2 Architecture — Plugin Form Upfront (G-only Instance)

**Plugin layer location**: `openstarry_eco/agent_dev/openstarry_plugin/api-runtime/`
**Factory pattern**: `createApiRuntimePlugin(manifest, factory(ctx))` per OpenStarry plugin convention.

**4-property R/S/C/G template** (cycle 03-21 D-§0-B AMEND-6 ratified): Plan59 = **G-only instance**:
- R (Refactor): N/A (no prior form)
- S (SICP-canonical): partial — black-box / minimal-interface design discipline applies
- C (Compatibility): N/A (no prior data/entries)
- G (Greenfield): primary lens — new factory + manifest + container-plugin lifecycle integration

**Phase 7 elevation 先驅範例 第三例**: cycle 03-21 Plan57 amendment (R-elevation) + cycle 03-21 provider-claude-cli (G-greenfield edge plugin) + cycle 03-22 Plan59 (G-greenfield 核心 observability primitive)。

## §3 Plan52/54/56/57(plugin)/58/59 Isomorph 10-Dimension

| # | Dimension | Plan52 | Plan54 | Plan56 | Plan57 plugin | Plan58 | **Plan59** |
|:-:|-----------|--------|--------|--------|---------------|--------|-----------|
| 1 | Plugin layer | input | quota | volition queue | vasana-engine | mesh broker | **api-runtime** |
| 2 | Core surface delta | 0/0 | 0/0 | 0/0 | 0/0 | 0/0 | **0/0** |
| 3 | ε-surface 7-sub-check | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 4 | Tri-party MR-6 | (baseline) | TKG | TKG | TKG | TKG | **TKG (AND-condition)** |
| 5 | HMAC-SHA256 + nonce | yes | yes | yes | yes | yes | **yes** |
| 6 | Replay cache | 1-c `psh:` | 2-c `+ac9:` | 3-c `+mvq:` | (cycle 03-19 had `vsn:`; refactor unchanged) | **5-c `+msh:`** | **6-c `+apr:`** |
| 7 | Cross-OS CI | baseline | extended | extended | extended | extended Tier δ | **extended Tier δ + api-runtime** |
| 8 | F-13/14/15 v3 reflexive | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 9 | TW sibling Rule #78 §78.5 | yes | yes | yes | yes | yes | **yes** |
| 10 | LOC ceiling | (baseline) | 600/400 | 600/400 | 800-1000/600 | 900/600 | **600-900 / 400-600** |

(TKG = TANENBAUM + KERNEL + GUARDIAN)

## §4 Replay Cache 6-Contributor `apr:` Prefix

| # | Plugin | Prefix |
|:-:|--------|:------:|
| 1 | Plan52 pushInput | `psh:` |
| 2 | Plan54 AC-9 | `ac9:` |
| 3 | Plan56 D-30-4 | `mvq:` |
| 4 | Plan57 D-30-5 (plugin) | `vsn:` |
| 5 | Plan58 Mesh | `msh:` |
| **6** | **Plan59 API Runtime** | **`apr:`** |

**R2-C strict bounding (5-item AND-condition same-PR; cycle 03-22 D-§1-R2-C 20/3)**:
1. `apr:` prefix structure verbatim 同 `msh:` (3-char-lowercase + colon-suffix);
2. nonce length verbatim cycle 03-21 setting (DSS-CY21-§1-B + DSS-CY22-§1-B KERNEL N=8 hex preferred preserved per MR-11);
3. replay cache table contributor 6 row 同 PR 必擴;
4. prefix-collision audit script 同 PR 必擴 to 6-contributor table;
5. F-13/14/15 v3 schema lint 同 PR run 6-contributor reflexive PASS。

任一缺漏 = Rule #75 §75.X 9th-enforced gate 阻 v0.56.0-alpha tag。

**Forensic complexity**: O(N choose 2) bounded by Phase 6 strict 7-list anchor — N=6 → 15 pairs; cycle 03-23 Plan60 N=7 → 21 pairs (small constant; not 通膨 in unbounded sense)。

## §5 ε-surface 0-delta + 7-sub-check + Tri-party MR-6 AND-Condition

### §5.1 ε-surface 0-delta strict (MR-6 鐵律)

**0 fields, 0 const** vs Plan52 baseline. Plugin-internal namespace (`PluginRuntimeState` enum / `HandlerFrame` struct / `ReplayStats` struct) 不 leak to ε-surface envelope; ε-surface envelope (`capability_holdings` / `parent_agent_id` / `nonce` / `signature`) 不暴露 plugin-internal types。

### §5.2 7-sub-check ci_check (verbatim cycle 03-17 D-§1-R2-B)

1. ε-surface envelope schema unchanged (0 field add)
2. ε-surface envelope const unchanged (0 enum add)
3. nonce field type / range unchanged
4. HMAC algorithm / key derivation path unchanged
5. parentAgentId / capability_holdings array semantics unchanged
6. Replay cache prefix scheme structured (`apr:` no collision)
7. Plugin-internal introspection schema **strictly OUT** of ε-surface boundary

**Design-stage**: 6/7 + 1 resolved at D-§1-B → 7/7 PASS. **Runtime-stage**: ✅ **SHIPPED**（v0.59.x）——`implementation_locus = 'openstarry_plugin/api-runtime/'` 已落地；7-sub-check 的 ε-surface 0-delta 由 `packages/sdk/src/types/api-runtime.ts` 不含任何 envelope 欄位確認，`apr:` 無碰撞由 `API_RUNTIME_REPLAY_CACHE_PREFIX`（`api-runtime.ts:30`）+ 6-contributor `__tests__` 覆蓋。〔原文：「pending Dev」已過時。〕

### §5.3 Tri-party MR-6 AND-condition (R3 D-§1-R2-B 23/0)

任一 sub-attestation FAIL = R3 阻; 不可由「一方代驗」化約。
- **TANENBAUM** (plan-level boundary): plugin manifest 不 declare Core extension; lifecycle 不 register Core hook;
- **KERNEL** (runtime ε-surface envelope): byte-level 比對 implementation-side 7-sub-check 7/7 PASS;
- **GUARDIAN** (security threat-model): intervention surface bounded; HMAC/replay 防 forge token; 三維路徑分清 (read-only / mutating / cross-plugin-routing)。

## §6 Plugin context.invoke/observe Boundary + Intervention Two-Branch

### §6.1 File-level separation (R3 D-§1-R2-D 23/0)

- `src/invoke.ts` — mutating intervention path (HMAC + replay cache `apr:` attestation);
- `src/observe.ts` — read-only introspection path (idempotent semantic; no replay cache attestation needed)。

### §6.2 Boundary invariant (R3 D-§1-R2-E 23/0)

`IRuntime.*` method signatures 不引用 ε-surface envelope fields; ε-surface envelope schema 不暴露 plugin-internal types。Static-analysis grep verifiable; KERNEL R2 sub-check #7 = set-disjointness predicate (Yes/No decidable)。

### §6.3 Intervention bounded enumeration (3 active + 1 negative-scope sentinel; R3 D-§1-Clarif C3 23/0;cycle 03-31 R-AMEND-5 clarified per Batch 26 Item #13)

The mutability surface contains exactly **3 active intervention fields** + **1 explicit negative-scope sentinel** (rejected at runtime per `INTERVENTION_KINDS` enum):

1. **log-level toggle** (active): per-plugin level ∈ {info, warn, error, debug}; 0 ε-surface delta;
2. **debug flag toggle** (active): per-plugin boolean; 0 ε-surface delta;
3. **soft tracing on/off** (active): per-plugin boolean; non-persistent; 0 ε-surface delta;
4. **(NEGATIVE-SCOPE sentinel; NOT a 4th field)** any other intervention category requires R-input + R3 vote (runtime rejects 4th-field requests per `__tests__/plugin.test.ts INTERVENTION_KINDS` enum)。

**Cross-reference (ApiRuntimeObserveResult schema)**: 3 mutable fields (log_level / debug_flag / soft_tracing) **+ replay_cache_size top-level** (introspection-only, not mutable). The "4-row tuple" historic title was misleading;the canonical mutability surface = 3 active fields. Verified verbatim against `openstarry_plugin/api-runtime/src/observe.ts:10-11` + `__tests__/plugin.test.ts:109 INTERVENTION_KINDS` + `__tests__/state.test.ts:10 newRecord()`.

### §6.4 Two-branch design (R3 D-§1-R2-F 23/0)

- **Option B path (ratified)**: 走 `apr:` prefix replay cache attestation;
- **Option A fallback path (preserved per MR-11)**: idempotent semantic + (a) globally unique within plugin lifetime (b) monotonically ordered (c) cryptographically bound — NOT activated this cycle。

## §7 LOC Trajectory + Cross-OS CI + TW Sibling

### §7.1 LOC trajectory

| Checkpoint | When | prod | test |
|-----------|------|------|------|
| CP-1 R4 close | 2026-05-04 | indicative ~700 / band 600-900 | indicative ~480 / band 400-600 |
| CP-2 Dev mid | 2026-05-05~06 | drift > 10% reassess | drift > 10% reassess |
| CP-3 Pre-tag v0.56.0-alpha | 2026-05-07 | hard ≤ 900 | hard ≤ 600 |

**Buffer at CP-1**: prod ~22% / test ~20% (same profile as Plan58)。

### §7.2 Cross-OS CI

Linux + Windows primary + Tier δ extension (Plan51 reserved + Plan53 reserved + Plan55 sunset 不在 active matrix)。

### §7.3 TW sibling

`Plan59_API_Runtime_Binding.md` (EN) + `Plan59_API_Runtime_Binding.tw.md` (TW) — same-PR mandate per Rule #78 §78.5 BINDING-tier reflexive。ATHENA quality 4/4 PASS exemplar pattern。

## §8 Forward + Dissent Preservation

### §8.1 Forward constraints

- Mesh-routed API runtime command: cycle 03-23+
- Plan60 Blackboard-Alaya integration: single-direction Plan60 → Plan59 query (Plan60 自身 mutation 走 Plan60 ε-surface envelope path with `bbk:` prefix if Plan60 +1)
- Phase 7 batch elevation: Plan59 not in elevation list (already plugin form)
- 5-property R/S/C/G/I template (I = Isolation 5th): cycle 03-25 Master 端決 (per cycle 03-22 R3 D-§7 Option γ 23/0)

### §8.2 4 NEW DSS-CY22-§1 verbatim (per MR-11)

- **DSS-CY22-§1-A** (LEIBNIZ): Mesh aggregation-mode + minimal-routing should precede replay cache topology extension; defer `apr:` to cycle 03-23+ post-Plan60.
- **DSS-CY22-§1-B** (KERNEL, YES with reservation): N=8 hex preferred for forensic balance; YES on 6-extend with N=8 hex form.
- **DSS-CY22-§1-C** (ASANGA): Aggregation-mode via Mesh routing should precede; 6-extend acceptable但建議 deferred to post-Plan60 cycle 03-23+.
- **DSS-CY22-§1-D** (RUSSELL): Provider abstraction layer hardening should ship before further plugin layer extensions; tactically acceptable but strategically dependent on provider abstraction maturity.

### §8.3 3 carryover DSS-CY21-§1 (per MR-11)

DSS-CY21-§1-A (LEIBNIZ Gossip B'), DSS-CY21-§1-B (KERNEL N=8 hex), DSS-CY21-§1-D (LEIBNIZ aggregation defer Phase 7) preserved verbatim.

## §9 Compliance

| Constraint | Status |
|------------|:------:|
| MR-5 hard / MR-6 鐵律 / MR-9 / MR-11 / MR-12 | ✅ PASS |
| MR-13 standby (cycle 03-13 Batch 11 #9; mechanical auto re-escalate fallback) | ✅ standby |
| ZT-1 / ZT-2 (10/0/0★ unchanged; 9/0/1★ ACTIVE preserved) / ZT-3 | ✅ PASS |
| ENG-FAB v1.8 = 48 canonical (v1.9 candidate retired per cycle 03-21) | ✅ PASS |
| Rule #74 / #75 §75.X (9th-enforced for v0.56.0-alpha) / #76 §76.6+§76.7 / #77 / #78 §78.5 | ✅ PASS |
| F-13/14/15 v3 reflexive | ✅ PASS |
| Phase 6 strict 7-list anchor | ✅ Plan59 = 6/7 |
| Master directive 2026-05-01 4 防線 fourth enforce | ✅ R0 + R4 dispatch (本檔 = 防線 2 BINDING canonical doc creation) |
| Master directive 2026-05-02 σ R3 downgrade / 2026-05-03 5-point isolation v2 | ✅ binding 持續 |
| F-16 / FORBIDDEN-phrasings / chair-rule | ✅ ALL RETIRED (per cycle 03-21 binary final terminal; cycle 03-22 D-§4 confirmed) |
| Tenet #10 | ⏸ NC PENDING (MR-5 hard; Phase 6 6/7 in flight; cycle 03-23 Plan60 7/7 終局解鎖路徑) |

---

*Plan59 API Runtime BINDING Specification — cycle 03-22 R3 D-§1 ratified 23/0 UNANIMOUS plugin upfront + 20/3 super-majority replay cache 6-extend — 2026-05-04*
*✅ SHIPPED（v0.59.x）：插件已出貨並接線（`openstarry_plugin/api-runtime/`、SDK `types/api-runtime.ts`、42 測試、`configs/phase6-agent.json` 載入）。〔原文「Master Ratification Batch 19 #2 dispatch ready / v0.56.0-alpha minor-bump release trigger」為發佈前措辭，已過時。〕*
*Inheritance: Plan52 → Plan54 → Plan56 → Plan57 (+ amendment) → Plan58 → **Plan59 (6/7 Phase 6 functional landing; 識蘊 Vijnana)***
