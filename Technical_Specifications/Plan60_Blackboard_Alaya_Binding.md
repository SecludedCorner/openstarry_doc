# Plan60 — Blackboard-Alaya BINDING Specification

**Status**: BINDING (cycle 03-23 R3 D-§1 ratified D-§1-A 22/1 super-majority + D-§1-B 23/0 UNANIMOUS; pending Master Ratification Batch 20 #2)
**Authority**: Master Ratification (Batch 20 dispatch 2026-05-05)
**Cycle**: 03-23 (Phase 6 第七棒; **7/7 完工** ✅)
**Release**: v0.57.0-alpha minor-bump
**Plan number**: 60
**Subject**: Blackboard-Alaya — 共用記憶 + 種子儲存 via existing-plugin-spec-upgrade
**五蘊 對應**: 識蘊 第八識 (阿賴耶識; Vijnana 第八; 一切種子 storage layer)

**Inheritance chain**: Plan52 → Plan54 → Plan56 → Plan57 (+ amendment) → Plan58 → Plan59 → **Plan60** (Phase 6 7/7 ✅ 完工 final functional landing)

---

## §1 Background — Phase 6 7/7 完工 + 阿賴耶識 對應

Phase 6 strict 7-list anchor 第七棒 functional landing → **Phase 6 7/7 完工**。對應五蘊 識蘊 第八識 (阿賴耶識; 一切種子 / 記憶 / 習氣 / 業報 storage layer);OpenStarry 實作層為 Blackboard 共用記憶 + Alaya 種子儲存。

**Phase 6 完工 達成本輪** → cycle 03-24 endpoint 10/0/0★ ratification + Tenet #10 升 COMPLIANT 條件齊備。

## §2 Architecture — Option A Reuse `openstarry_plugin/distributed-alaya/` (per D-§1-A 22/1 Super-Majority)

**Plugin layer**: `openstarry_eco/agent_dev/openstarry_plugin/distributed-alaya/` (既有 plugin reuse + Phase 6 spec 升級 forward addendum per MR-12 既有不破壞)。

既有 plugin baseline ~500 LOC verified production code reuse: BijaStore + seed-signature + vector clock + SEC-002 + late-joiner snapshot。

Spec name "Plan60 Blackboard-Alaya" 對齊 Phase 6 strict 7-list anchor; implementation locus filename "distributed-alaya" 為既有路徑 (per D-§1-Clarif C2 23/0 UNANIMOUS naming reconciliation)。

**Form-pattern matrix 第四範例 candidate**: "existing-plugin-spec-upgrade" (cycle 03-25 master 端決 codification timing; DSS-CY23-§1-A DARWIN Type 4 candidate carryover preserved per MR-11)。

**Dissent**: DSS-CY23-§1-B LEIBNIZ Option B 新建 alternative preserved verbatim per MR-11。

## §3 Plan52~Plan60 Isomorph 11-Dimension

| # | 維度 | Plan60 |
|:-:|------|--------|
| 1 | Plugin layer | distributed-alaya |
| 2 | Core surface delta | 0/0 |
| 3 | ε-surface 7-sub-check | PASS (7/7 design-stage) |
| 4 | Tri-party MR-6 | TKG (7-sub-attest AND-condition) |
| 5 | HMAC-SHA256 + nonce | yes |
| 6 | Replay cache | 7-c `+aly:` |
| 7 | Cross-OS CI | Tier δ + alaya = 8 emit-source × 2 OS = 16 cells |
| 8 | F-13/14/15 v3 reflexive | PASS |
| 9 | TW sibling Rule #78 §78.5 | yes (deferred coordinator G5 sync stage) |
| 10 | LOC ceiling | Option A reuse: prod 400-700 / test 300-500 |
| 11 | Form-pattern matrix candidate | existing-plugin-spec-upgrade 先驅 |

## §4 Replay Cache 7-Contributor `aly:` Prefix (per D-§1-B 23/0 UNANIMOUS)

| # | Plugin | Prefix |
|:-:|--------|:------:|
| 1 | Plan52 pushInput | `psh:` |
| 2 | Plan54 AC-9 | `ac9:` |
| 3 | Plan56 D-30-4 | `mvq:` |
| 4 | Plan57 D-30-5 (plugin) | `vsn:` |
| 5 | Plan58 Mesh | `msh:` |
| 6 | Plan59 API Runtime | `apr:` |
| **7** | **Plan60 Blackboard-Alaya** | **`aly:`** |

`aly:` per ASANGA Sanskrit ālaya transliteration; 3-char-lowercase + colon-suffix; KNUTH algorithm rigor + GUARDIAN prefix-collision Hamming distance ≥ 2 PASS。

**R2-C 5-item AND-condition same-PR**: prefix structure verbatim + nonce length verbatim + replay cache table 7-row + audit script 7-contributor + F-13/14/15 v3 schema lint 7-contributor。任一缺漏 = Rule #75 §75.X 10th-enforced gate 阻 v0.57.0-alpha tag。

**Phase 6 完工 final replay cache topology N=7 stable post-this-cycle**;O(N choose 2) = 21 pairs 上界。

## §5 ε-surface 0-delta + 7-Sub-Check + Tri-Party MR-6 7-Sub-Attestation AND-Condition

**ε-surface 0-delta strict (MR-6 鐵律)**: 0 fields, 0 const vs Plan52 baseline; plugin-internal namespace (SeedStore / BlackboardKey / VectorClock) 不 leak; 7-sub-check verbatim cycle 03-17 D-§1-R2-B 繼承 (KERNEL R2 7/7 design-stage PASS)。

**Tri-party MR-6 AND-condition** (per R2-B 23/0 UNANIMOUS): TANENBAUM (plan-level boundary) + KERNEL (runtime ε-surface envelope; byte-level 比對) + GUARDIAN (security 5-vector defence-in-depth)。任一 sub-attestation FAIL = R3 阻;不可化約。

## §6 Storage Threat Model 5-Vector Defence-In-Depth

1. Alaya seed pollution (HMAC + nonce + replay attestation + key derivation audit + size limit; per R2-D 5-element)
2. Blackboard race condition (vector clock + serialise OR last-write-wins; per R2-E 22/1 super-majority; DSS-CY23-§1-C LEIBNIZ locking model alternative verbatim per MR-11)
3. 種子 retrieval consistency (append-only seed log + monotonic order + Merkle proof if scaling; per R2-F 6-element)
4. Key derivation reuse audit (Option A 既有 path 不變)
5. Replay cache prefix-collision (`aly:` Hamming distance ≥ 2 vs 6 existing prefixes PASS)

## §7 LOC Trajectory + Cross-OS CI + TW Sibling

**LOC trajectory** (Option A reuse): CP-1 R4 close indicative ~600 prod / ~400 test; CP-3 pre-tag v0.57.0-alpha hard ≤700 / ≤500。

**Cross-OS CI**: Linux + Windows + Tier δ extension to 8 emit-source × 2 OS = 16 cells (per R3-§2-B Class I substantive)。

**TW sibling**: `Plan60_Blackboard_Alaya_Binding.tw.md` deferred coordinator G5 sync stage post-Master ratification per Rule #78 §78.5 BINDING-tier reflexive same-PR。

## §8 Forward + Dissent (3 NEW DSS-CY23 Verbatim)

**Forward**:
- Phase 6 完工 達成本輪 → cycle 03-24 endpoint 10/0/0★ ratification + Tenet #10 升 COMPLIANT 條件齊備
- VasanaEngine + Mesh Plan58 integration: single-direction topology + Kahn cycle prevention (per D-§1-Clarif C3 23/0 UNANIMOUS)
- cycle 03-25 Phase 7 R-input formal opening: Plan52/54/56/58 整批 elevate

**3 NEW DSS-CY23 verbatim per MR-11**:
- DSS-CY23-§1-A (DARWIN): Plan60 Type 4 form-pattern matrix candidate not canonical until cycle 03-25 R-input formal opening review
- DSS-CY23-§1-B (LEIBNIZ): Option B 新建 alternative preferred for Phase 6 spec naming alignment; cycle 03-25+ Phase 7 elevation 期間 revisit
- DSS-CY23-§1-C (LEIBNIZ): R2-E locking model (read-write lock per blackboard key) alternative; cycle 03-25+ Phase 7 evaluation

## §9 Compliance

| Constraint | Status |
|------------|:------:|
| MR-5 hard / MR-6 鐵律 / MR-9 / MR-11 / MR-12 | ✅ PASS |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS |
| ENG-FAB v1.8 = 48 canonical (v1.9 retired) | ✅ PASS |
| Rule #72 (new SPC limits cycle 03-23+ binding) / #74 / #75 §75.X 10th-enforced / #76 §76.6+§76.7 / #77 / #78 §78.5 | ✅ PASS |
| F-13/14/15 v3 reflexive | ✅ PASS |
| **Phase 6 strict 7-list anchor** | ✅ **Plan60 = 7/7 完工** |
| Master directive 2026-05-01 4 防線 fifth enforce | ✅ |
| Master directive cycle 03-23 §6.4 SPC backfill Option α | ✅ |
| Master directive 2026-05-03 5-point isolation v2 third sustained | ✅ |
| F-16 / FORBIDDEN-phrasings / chair-rule | ✅ ALL RETIRED (cycle 03-21 binary final) |
| Plan55 sunset | ✅ |
| Tenet #10 | ⏸ NC PENDING (cycle 03-24 才升 COMPLIANT) |

---

*Plan60 Blackboard-Alaya BINDING — cycle 03-23 R3 D-§1 ratified — 2026-05-05*
*Master Ratification Batch 20 #2 dispatch ready / v0.57.0-alpha minor-bump*
*Phase 6 7/7 ✅ 完工 final functional landing / 阿賴耶識 (Vijnana 第八)*
