# Plan59 — API Runtime BINDING 規格（TW sibling）

**Status**: BINDING（cycle 03-22 R3 D-§1 ratified 23/0 UNANIMOUS plugin form upfront + 20/3 super-majority replay cache 6-extend；Master Ratification Batch 19 #2 已 APPROVED 2026-05-04）
**Authority**: Master Ratification（Batch 19 dispatch 2026-05-04 / `deliver/Master_Ratification/Master_Confirmation.md` 12/12 APPROVED）
**Cycle**: 03-22（Phase 6 第六棒；6/7 functional landing）
**Release**: v0.56.0-alpha minor-bump
**Plan number**: 59
**Subject**: API Runtime — runtime 觀察 + 有界介入；plugin form upfront
**五蘊 對應**: 識蘊（Vijnana；了別 / observability / introspection）
**TW sibling 義務**: per Rule #78 §78.5 BINDING-tier reflexive same-PR；對應 EN 主檔 `Plan59_API_Runtime_Binding.md`

**Inheritance chain**: Plan52 pushInput → Plan54 AC-9 → Plan56 D-30-4 → Plan57 D-30-5（+ plugin form amendment cycle 03-21）→ Plan58 Mesh → **Plan59 API Runtime**（Phase 6 連續第六棒 functional landing）

---

## §1 背景 — Phase 6 6/7 + Vijnana 對應

API Runtime 在 OpenStarry 五蘊架構內提供 runtime 觀察 + 有界介入能力。**架構選擇**：plugin form upfront 落於 `openstarry_eco/agent_dev/openstarry_plugin/api-runtime/`，採 `createApiRuntimePlugin(manifest, factory(ctx))` factory pattern（per cycle 03-22 R3 D-§1-A 23/0 UNANIMOUS）。

**Vijnana =「了別」（discriminating awareness）** 在系統面對應：
- **觀察 (Observation)**：plugin runtime state introspection — read-only per-plugin state / handler frames / capability holdings / replay cache stats；
- **介入 (Intervention)**：bounded mutability surface（log-level / debug flag / soft tracing）— 不可動 plugin lifecycle / 不可動 ε-surface。

**前向綁定（本輪 NOT in scope）**:
- Mesh 路由 cross-plugin API runtime command（cycle 03-23+）；
- Plan60 Blackboard-Alaya 整合（single-direction Plan60 → Plan59 query；per cycle 03-22 R3 D-§1-Clarif C2）；
- Phase 7 batch elevation（Plan59 已是 plugin form；不在 elevation list）。

## §2 架構 — Plugin Form Upfront（G-only Instance）

**Plugin layer 位置**：`openstarry_eco/agent_dev/openstarry_plugin/api-runtime/`
**Factory pattern**：`createApiRuntimePlugin(manifest, factory(ctx))`，對齊 OpenStarry plugin convention。

**4-property R/S/C/G template**（cycle 03-21 D-§0-B AMEND-6 ratified）：Plan59 = **G-only instance**：
- R (Refactor)：N/A（無前置 form）
- S (SICP-canonical)：partial — black-box / minimal-interface 設計紀律仍適用
- C (Compatibility)：N/A（無前置 data/entries）
- G (Greenfield)：primary lens — new factory + manifest + container-plugin lifecycle 整合

**Phase 7 elevation 先驅範例 第三例**：cycle 03-21 Plan57 amendment（R-elevation 先驅）+ cycle 03-21 provider-claude-cli（G-greenfield edge plugin 先驅）+ cycle 03-22 Plan59（G-greenfield 核心 observability primitive 先驅）。

## §3 Plan52/54/56/57(plugin)/58/59 Isomorph 10 維度

| # | 維度 | Plan52 | Plan54 | Plan56 | Plan57 plugin | Plan58 | **Plan59** |
|:-:|------|--------|--------|--------|---------------|--------|-----------|
| 1 | Plugin layer | input | quota | volition queue | vasana-engine | mesh broker | **api-runtime** |
| 2 | Core surface delta | 0/0 | 0/0 | 0/0 | 0/0 | 0/0 | **0/0** |
| 3 | ε-surface 7-sub-check | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 4 | Tri-party MR-6 | (baseline) | TKG | TKG | TKG | TKG | **TKG（AND-condition）** |
| 5 | HMAC-SHA256 + nonce | yes | yes | yes | yes | yes | **yes** |
| 6 | Replay cache | 1-c `psh:` | 2-c `+ac9:` | 3-c `+mvq:` | （cycle 03-19 had `vsn:`；refactor 不變）| **5-c `+msh:`** | **6-c `+apr:`** |
| 7 | Cross-OS CI | baseline | extended | extended | extended | extended Tier δ | **extended Tier δ + api-runtime** |
| 8 | F-13/14/15 v3 reflexive | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 9 | TW sibling Rule #78 §78.5 | yes | yes | yes | yes | yes | **yes（即本檔）** |
| 10 | LOC ceiling | (baseline) | 600/400 | 600/400 | 800-1000/600 | 900/600 | **600-900 / 400-600** |

（TKG = TANENBAUM + KERNEL + GUARDIAN）

## §4 Replay Cache 6-Contributor `apr:` Prefix

| # | Plugin | Prefix |
|:-:|--------|:------:|
| 1 | Plan52 pushInput | `psh:` |
| 2 | Plan54 AC-9 | `ac9:` |
| 3 | Plan56 D-30-4 | `mvq:` |
| 4 | Plan57 D-30-5（plugin） | `vsn:` |
| 5 | Plan58 Mesh | `msh:` |
| **6** | **Plan59 API Runtime** | **`apr:`** |

**R2-C strict bounding 5-item AND-condition same-PR（cycle 03-22 D-§1-R2-C 20/3）**：
1. `apr:` prefix 結構 verbatim 同 `msh:`（3-char-lowercase + colon-suffix）；
2. nonce 長度 verbatim cycle 03-21 設定（DSS-CY21-§1-B + DSS-CY22-§1-B KERNEL N=8 hex preferred 依 MR-11 保留）；
3. replay cache table contributor 6 row 同 PR 必擴；
4. prefix-collision audit script 同 PR 必擴 to 6-contributor table；
5. F-13/14/15 v3 schema lint 同 PR run 6-contributor reflexive PASS。

任一缺漏 = Rule #75 §75.X 9th-enforced gate 阻 v0.56.0-alpha tag。

**Forensic complexity**：O(N choose 2) 由 Phase 6 strict 7-list anchor 設上界 — N=6 → 15 pairs；cycle 03-23 Plan60 N=7 → 21 pairs（小常數；不 unbounded 通膨）。

## §5 ε-surface 0-delta + 7-sub-check + Tri-party MR-6 AND-Condition

### §5.1 ε-surface 0-delta strict（MR-6 鐵律）

**0 fields, 0 const** vs Plan52 baseline。Plugin-internal namespace（`PluginRuntimeState` enum / `HandlerFrame` struct / `ReplayStats` struct）不 leak 至 ε-surface envelope；ε-surface envelope（`capability_holdings` / `parent_agent_id` / `nonce` / `signature`）不暴露 plugin-internal types。

### §5.2 7-sub-check ci_check（cycle 03-17 D-§1-R2-B 逐字繼承）

1. ε-surface envelope schema 不變（0 field add）
2. ε-surface envelope const 不變（0 enum add）
3. nonce field type / range 不變
4. HMAC algorithm / key derivation path 不變
5. parentAgentId / capability_holdings array 語意不變
6. Replay cache prefix scheme structured（`apr:` 不衝突）
7. Plugin-internal introspection schema **嚴格 OUT** of ε-surface boundary

**設計階段**：6/7 + 1 由 D-§1-B 解決 → 7/7 PASS。**執行階段**：pending Dev（ci_check at `implementation_locus = 'openstarry_plugin/api-runtime/'`；delivery_report 附錄附 log path + entry hash）。

### §5.3 Tri-party MR-6 AND-condition（R3 D-§1-R2-B 23/0）

任一 sub-attestation FAIL = R3 阻；不可由「一方代驗」化約。
- **TANENBAUM**（plan-level boundary）：plugin manifest 不 declare Core extension；lifecycle 不 register Core hook；
- **KERNEL**（runtime ε-surface envelope）：byte-level 比對 implementation-side 7-sub-check 7/7 PASS；
- **GUARDIAN**（security threat-model）：intervention surface bounded；HMAC/replay 防 forge token；三維路徑分清（read-only / mutating / cross-plugin-routing）。

## §6 Plugin context.invoke/observe Boundary + Intervention Two-Branch

### §6.1 File-level separation（R3 D-§1-R2-D 23/0）

- `src/invoke.ts` — mutating intervention path（HMAC + replay cache `apr:` attestation）；
- `src/observe.ts` — read-only introspection path（idempotent 語意；無需 replay cache attestation）。

### §6.2 Boundary invariant（R3 D-§1-R2-E 23/0）

`IRuntime.*` method signatures 不引用 ε-surface envelope fields；ε-surface envelope schema 不暴露 plugin-internal types。Static-analysis grep verifiable；KERNEL R2 sub-check #7 = set-disjointness predicate（Yes/No 可決定）。

### §6.3 Intervention bounded enumeration（4-row tuple；R3 D-§1-Clarif C3 23/0）

1. **log-level toggle**：per-plugin level ∈ {info, warn, error, debug}；0 ε-surface delta；
2. **debug flag toggle**：per-plugin boolean；0 ε-surface delta；
3. **soft tracing on/off**：per-plugin boolean；non-persistent；0 ε-surface delta；
4. **（NOT in scope）** 其他 intervention 類別需 R-input + R3 vote。

### §6.4 Two-branch design（R3 D-§1-R2-F 23/0）

- **Option B path（ratified）**：走 `apr:` prefix replay cache attestation；
- **Option A fallback path（per MR-11 preserved）**：idempotent semantic +（a）plugin lifetime 內 globally unique（b）monotonically ordered（c）cryptographically bound — 本輪 NOT activated。

## §7 LOC 軌跡 + Cross-OS CI + TW Sibling

### §7.1 LOC 軌跡

| Checkpoint | 時機 | prod | test |
|-----------|------|------|------|
| CP-1 R4 close | 2026-05-04 | indicative ~700 / band 600-900 | indicative ~480 / band 400-600 |
| CP-2 Dev mid | 2026-05-05~06 | 漂移 > 10% 重評 | 漂移 > 10% 重評 |
| CP-3 Pre-tag v0.56.0-alpha | 2026-05-07 | hard ≤ 900 | hard ≤ 600 |

**CP-1 緩衝**：prod ~22% / test ~20%（同 Plan58 profile）。

### §7.2 Cross-OS CI

Linux + Windows primary + Tier δ extension（Plan51 reserved + Plan53 reserved + Plan55 sunset 不在 active matrix）。

### §7.3 TW sibling

`Plan59_API_Runtime_Binding.md`（EN）+ `Plan59_API_Runtime_Binding.tw.md`（TW；即本檔）— same-PR mandate per Rule #78 §78.5 BINDING-tier reflexive。ATHENA quality 4/4 PASS exemplar pattern 沿用。

## §8 前向 + 異議保留

### §8.1 前向約束

- Mesh 路由 API runtime command：cycle 03-23+
- Plan60 Blackboard-Alaya 整合：single-direction Plan60 → Plan59 query（Plan60 自身 mutation 走 Plan60 ε-surface envelope path with `bbk:` prefix if Plan60 +1）
- Phase 7 batch elevation：Plan59 不在 elevation list（已是 plugin form）
- 5-property R/S/C/G/I template（I = Isolation 5th）：cycle 03-25 Master 端決（per cycle 03-22 R3 D-§7 Option γ 23/0）

### §8.2 4 NEW DSS-CY22-§1 verbatim（per MR-11）

- **DSS-CY22-§1-A**（LEIBNIZ）：Mesh aggregation-mode + minimal-routing 應先於 replay cache topology extension；defer `apr:` to cycle 03-23+ post-Plan60。
- **DSS-CY22-§1-B**（KERNEL，YES with reservation）：N=8 hex 在鑑識平衡上更佳；YES 6-extend with N=8 hex form。
- **DSS-CY22-§1-C**（ASANGA）：Aggregation-mode via Mesh routing 應先；6-extend acceptable但建議 deferred to post-Plan60 cycle 03-23+。
- **DSS-CY22-§1-D**（RUSSELL）：Provider abstraction layer hardening 應先於進一步 plugin layer extensions；tactically acceptable but strategically dependent on provider abstraction maturity。

### §8.3 3 carryover DSS-CY21-§1（per MR-11）

DSS-CY21-§1-A（LEIBNIZ Gossip B'）、DSS-CY21-§1-B（KERNEL N=8 hex）、DSS-CY21-§1-D（LEIBNIZ aggregation defer Phase 7）逐字保留。

## §9 合規

| Constraint | Status |
|------------|:------:|
| MR-5 hard / MR-6 鐵律 / MR-9 / MR-11 / MR-12 | ✅ PASS |
| MR-13 standby（cycle 03-13 Batch 11 #9；mechanical auto re-escalate fallback） | ✅ standby |
| ZT-1 / ZT-2（10/0/0★ unchanged；9/0/1★ ACTIVE preserved） / ZT-3 | ✅ PASS |
| ENG-FAB v1.8 = 48 canonical（v1.9 candidate retired per cycle 03-21） | ✅ PASS |
| Rule #74 / #75 §75.X（v0.56.0-alpha 第 9 次 enforce） / #76 §76.6+§76.7 / #77 / #78 §78.5 | ✅ PASS |
| F-13/14/15 v3 reflexive | ✅ PASS |
| Phase 6 strict 7-list anchor | ✅ Plan59 = 6/7 |
| Master directive 2026-05-01 4 防線 fourth enforce | ✅ R0 + R4 dispatch（EN 主檔 = 防線 2 BINDING canonical doc creation；本檔 = 防線 2 + Rule #78 §78.5 補完） |
| Master directive 2026-05-02 σ R3 downgrade / 2026-05-03 5-point isolation v2 | ✅ binding 持續 |
| F-16 / FORBIDDEN-phrasings / chair-rule | ✅ ALL RETIRED（per cycle 03-21 binary final terminal；cycle 03-22 D-§4 confirmed） |
| Tenet #10 | ⏸ NC PENDING（MR-5 hard；Phase 6 6/7 in flight；cycle 03-23 Plan60 7/7 終局解鎖路徑） |

---

*Plan59 API Runtime BINDING 規格（TW sibling）— cycle 03-22 R3 D-§1 ratified 23/0 UNANIMOUS plugin upfront + 20/3 super-majority replay cache 6-extend — 2026-05-04*
*Master Ratification Batch 19 #2 dispatch ready / Master_Confirmation 12/12 APPROVED / v0.56.0-alpha minor-bump release trigger*
*Inheritance：Plan52 → Plan54 → Plan56 → Plan57（+ amendment）→ Plan58 → **Plan59（6/7 Phase 6 functional landing；識蘊 Vijnana）***
*EN 主檔同步：`Plan59_API_Runtime_Binding.md`*
