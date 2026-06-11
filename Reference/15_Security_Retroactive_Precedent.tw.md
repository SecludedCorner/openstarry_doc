---
title: 安全性回溯先例 — Master Directive 2026-04-30
title_en_sibling: Security Retroactive Precedent — Master Directive 2026-04-30
author: SCRIBE + LINNAEUS (TW translation backfill cycle 03-20)
date: 2026-05-02
cycle: 03-20 (TW backfill per D-§5 ratified + Batch 17 #4 APPROVED)
status: BINDING (TW sibling parity per Rule #78 §78.5 BINDING-tier reflexive)
authority: research-team R4 deliver (cycle 03-20)
supersedes: none
cross_refs:
  - openstarry_eco/share/openstarry_doc/Reference/15_Security_Retroactive_Precedent.md (EN sibling)
  - research record/cycle03-20/openstarry_doc/Reference/TW_backfill_list_cycle03-20.md
  - research record/cycle03-20/deliver/Master_Ratification/Batch_17_Request.md (Item #4)
hard_rule_restated: "TW sibling per Rule #78 §78.5 BINDING-tier reflexive same-PR forward backfill; MR-12 forward-only."
---

# 安全性回溯先例 — Master Directive 2026-04-30

**Status**: BINDING
**Authority**: Master directive 2026-04-30
**Class**: Security gap closure retroactive precedent（NEW；獨立於 documentation precedent class）

---

## §1 Master Directive 原文（per Cycle 03-18 Master_Confirmation §4）

per cycle 03-18 Master_Confirmation §4.4：

> **新立 precedent 類別**：「security gap closure retroactive」
>
> - **不同於** cycle 03-13 documentation backfill（cycle 03-15 §6.3 ratified one-off for documentation）
> - Security 嚴重性 > documentation；**獨立 precedent class**
> - 未來 security gap 比照本次辦理（盤點 + 限時 retrofit + sunset）
> - **Cycle 03-15 §6.3 documentation one-off 限制仍有效**；本 directive 不重啟 documentation retroactive

---

## §2 先例類別區分（Precedent Class Distinction）

OpenStarry retroactive 補回（per MR-10 補回）分**兩類**：

### §2.1 Documentation 類別 — cycle 03-15 §6.3 one-off（cycle 03-13 backfill）

- **Scope**: documentation gaps（例如：missing TW translation、doc structural retrofit）
- **Precedent**: cycle 03-13 plugin-gear-arbiters.md TW backfill 一次性
- **Going forward**: forward-only per Rule #78 §78.5；不可再援引 documentation retroactive
- **Source**: Reference/12_Cycle03-13_Backfill_Classification.md

### §2.2 Security 類別 — cycle 03-18 NEW（Master directive 2026-04-30）

- **Scope**: security gap closures（例如：redaction format retrofit、authn/z gap closure、敏感資料外洩修補）
- **Precedent**: cycle 03-18（γ）full retrofit ~15-20 既有 plugin redaction format
- **Going forward**: 未來 security gap 比照辦理；獨立援引 security retroactive precedent
- **Source**: 本 doc

---

## §3 Security Retroactive Precedent 適用準則（Application Criteria）

當以下情況**全部成立**時，可援引本 precedent class：

1. **Security gap 性質**: gap 涉及敏感資料外洩、認證/授權繞過、replay attack、injection、或其他資訊安全紀律破壞
2. **Master directive**: Master 明文發 directive 援引 security retroactive
3. **盤點 + 限時 retrofit + sunset 三件全做**：
   - 盤點（audit）：盤查既有 plugin / system 哪些受影響
   - 限時 retrofit：排定明確 deadline（通常 1-2 cycles）
   - Sunset：超期未修 → 標 DEPRECATED + schedule replacement
4. **6 道關卡保護**（per Master directive 2026-04-30 §4.5；不可繞過）：
   - Rule #75 §75.X pre-delivery gate
   - Tri-party MR-6 audit
   - F-15 v3 linter
   - Cross-OS CI matrix
   - W2 verification regression test
   - R3 review（fail = 退回 Dev iterate）

---

## §4 Cycle 03-18 適用 — Volition-Payload Redaction Format（γ）Full Retrofit

per Master directive 2026-04-30 §4.1-§4.6：

| Element | Detail |
|---------|--------|
| Gap | volition-payload redaction format codification（cycle 03-18 Item #3）ad-hoc per-plugin behavior 留 security gap |
| Scope | ~15-20 既有 plugin（provider-* + guide-* + context-* + comm-* + auditor-* + spc-monitor + confirmation-gate + distributed-alaya + 部分 gear-arbiter / standard-core-commands）|
| Timing | cycle 03-19 dispatch（同 cycle 與 D-30-5 並行）|
| Sunset | cycle 03-19 R4 close 為止全 plugin 必修完；超期 plugin 標 DEPRECATED |
| 6 道關卡 | 全 enforce per Master directive §4.5 |
| Compliance | MR-5 hard / MR-6 / MR-9 / MR-11 / MR-12（security exception）/ MR-10 retroactive（security class）/ Tenet #2 誠實（known-security-debt 透明）/ ZT-1/2/3 全 PASS |

---

## §5 合規性聲明（Compliance Attestation）

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-5 hard（Tenet #10 status 不變）| ✅ PASS | Security retroactive precedent 不動 Tenet #10 |
| MR-6（Core 零）| ✅ PASS | Security retrofit 屬 plugin/log 層 |
| MR-9（no MUST WAIVE）| ✅ PASS | strengthening；既有 plugin **必修不豁免** |
| MR-10（retroactive precedent）| ✅ PASS | 援引 security 類別新立 precedent；獨立於 documentation one-off |
| MR-11（dissent preservation）| ✅ PASS | cycle 03-18 R-team R3 22/1 ratified Item #3 with 1 dissent verbatim 保留 |
| MR-12（既有不破壞）| ✅ PASS | 援引 security exception；非破壞既有 governance default 而是 security override per Master directive |
| Tenet #2 誠實 | ✅ PASS | known-security-debt 透明列出（per Master directive §4.2）|
| Tenet #4 嚴謹 | ✅ PASS | 強化 security discipline；6 道關卡 enforce |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | 無 tenet rewrite；endpoint 10/0/0★ 不變；control-range 加嚴 |

---

## §6 邊界 — 本 Precedent **不**授權的事項

本 precedent **不授權**：

- ❌ Documentation gap retroactive 補回（per cycle 03-15 §6.3 documentation one-off 限制仍有效）
- ❌ Functional / feature retrofit（純 functional 改動仍 forward-only per MR-12）
- ❌ Performance optimization retroactive
- ❌ 任何不滿足 §3 application criteria 的 retroactive 動作
- ❌ 繞過 6 道關卡（Master directive §4.5 不可協商）

---

## §7 Activation 紀錄

| Cycle | Application | Master directive |
|:-----:|-------------|------------------|
| **03-18** | Volition-payload redaction format（γ）full retrofit ~15-20 plugin | 2026-04-30 |
| 03-19+ | future security gap | TBD |

---

## §8 交叉引用（Cross-References）

- `openstarry_doc/Reference/12_Cycle03-13_Backfill_Classification.md`（documentation precedent class；distinct）
- `claude research/research record/cycle03-18/deliver/Master_Ratification/Master_Confirmation.md` §4（Master directive 2026-04-30 完整原文）
- `claude research/research record/cycle03-15/deliver/Master_Ratification/Master_Confirmation.md` §6.3（documentation one-off 限制 reaffirm）
- `openstarry_doc/Research_Methodology/15_ENG_FAB_v1.9_F_16_StructuredError.md`（F-16 schema；strength change 維持 forward-only per MR-9）

---

*Reference/15 Security Retroactive Precedent — BINDING — Master directive 2026-04-30（Cycle 03-18 Batch 15 Item #3（γ）full retrofit extension）*
*獨立於 cycle 03-15 §6.3 documentation one-off；security 類別新立 precedent；6 道關卡保護不可繞過*
*9/0/1★ ACTIVE preserved / Tenet #10 NC PENDING per MR-5 / endpoint 10/0/0★ unchanged*
