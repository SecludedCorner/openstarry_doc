---
title: Redaction 安全性負債登記冊
title_en_sibling: Redaction Security Debt Register
author: SCRIBE + LINNAEUS (TW translation backfill cycle 03-20)
date: 2026-05-02
cycle: 03-20 (TW backfill per D-§5 ratified + Batch 17 #4 APPROVED)
status: BINDING (TW sibling parity per Rule #78 §78.5 BINDING-tier reflexive)
authority: research-team R4 deliver (cycle 03-20)
supersedes: none
cross_refs:
  - openstarry_eco/share/openstarry_doc/Calibration_Reports/redaction_security_debt.md (EN sibling)
  - research record/cycle03-20/openstarry_doc/Reference/TW_backfill_list_cycle03-20.md
  - research record/cycle03-20/deliver/Master_Ratification/Batch_17_Request.md (Item #4)
hard_rule_restated: "TW sibling per Rule #78 §78.5 BINDING-tier reflexive same-PR forward backfill; MR-12 forward-only."
---

# Redaction 安全性負債登記冊（Redaction Security Debt Register）

**Status**: BINDING canonical（cycle 03-19 R3 D-§4 ratified 23/0 UNANIMOUS；pending Master Ratification Batch 16 #10）
**Authority**: SCRIBE-internal authority（Reference/15 Security Retroactive Precedent first concrete application）
**Cycle of origin**: 03-19
**Reference**: `Reference/15_Security_Retroactive_Precedent.md`（cycle 03-18 Master directive 2026-04-30）

---

## §1 目的（Purpose）

Per cycle 03-18 Master directive 2026-04-30（γ）γ retrofit + Reference/15 Security Retroactive Precedent：~15-18 actionable plugin **MUST** retrofit 至統一 redaction format（per cycle 03-18 D-§1-R2-B `<redacted-volition-payload len:NN first4:abcd>` N=4 + category-aware）至 **cycle 03-19 R4 close hard sunset** 為止。

本 register **透明列出** retrofit 進度與 known security debt per Master directive 2026-04-30 §4.2。

---

## §2 登記冊 Schema（每 entry 13 欄位）

| Field | Description |
|-------|-------------|
| `plugin_id` | Plugin package identifier |
| `family` | Family group（例如：provider / context / guide / comm / auditor）|
| `sensitivity_class` | HIGH/MED/LOW per category-aware redaction |
| `pre_format` | Retrofit 前 format |
| `post_format` | Retrofit 後 format（uniform target）|
| `retrofit_cycle` | Retrofit 套用 cycle |
| `retrofit_pr` | PR 編號 / commit hash |
| `tri_party_audit` | TANENBAUM + KERNEL + GUARDIAN sign-off status |
| `f15_lint_status` | F-15 v3 linter pass/fail |
| `cross_os_ci` | Linux + Windows CI status |
| `w2r22_regression` | W2-R22 regression test status |
| `dissent_inheritance` | DSS preservation tracking（例如：DSS-CY18-02 KERNEL N=8）|
| `notes` | Free-form context |

**Append-only discipline**: entries 只增不減；既有 entries 透過後續 row 加 timestamp annotation 修訂。**禁止刪除 entry**（per Reference/15 保留 audit trail）。

---

## §3 Plugin Retrofit 狀態（~15-18 actionable）

### §3.1 IN-SCOPE plugins（10 families）

| # | plugin_id | family | sensitivity | retrofit_cycle | status |
|:-:|-----------|--------|:-----------:|:--------------:|:------:|
| 1 | provider-openai-chat | provider-* | HIGH | 03-19 | PENDING DEV |
| 2 | provider-anthropic-chat | provider-* | HIGH | 03-19 | PENDING DEV |
| 3 | provider-google-chat | provider-* | HIGH | 03-19 | PENDING DEV |
| 4 | provider-cohere-chat | provider-* | HIGH | 03-19 | PENDING DEV |
| 5 | provider-mistral-chat | provider-* | HIGH | 03-19 | PENDING DEV |
| 6 | provider-deepseek-chat | provider-* | HIGH | 03-19 | PENDING DEV |
| 7 | provider-local-llm | provider-* | HIGH | 03-19 | PENDING DEV |
| 8 | provider-test-mock | provider-* | LOW | 03-19 | PENDING DEV |
| 9 | guide-character-init | guide-* | MED | 03-19 | PENDING DEV |
| 10 | context-sliding-window | context-* | MED | 03-19 | PENDING DEV |
| 11 | context-summarization | context-* | MED | 03-19 | PENDING DEV |
| 12 | context-state-extraction | context-* | MED | 03-19 | PENDING DEV |
| 13 | context-vector-store | context-* | HIGH | 03-19 | PENDING DEV |
| 14 | comm-mcp-bridge | comm-* | MED | 03-19 | PENDING DEV |
| 15 | comm-mesh-protocol | comm-* | MED | 03-19 | PENDING DEV |
| 16 | auditor-trail | auditor-* | HIGH | 03-19 | PENDING DEV |
| 17 | spc-monitor-control / spc-monitor-westgard / spc-monitor-l2-l3 | spc-monitor (3) | LOW | 03-19 | PENDING DEV |
| 18 | confirmation-gate-standard | confirmation-gate | HIGH | 03-19 | PENDING DEV |

**Total IN-SCOPE**: 18 actionable plugin（透過共用 `redact_volition_payload` helper deduplicated 至 ~15-18，per BABBAGE R2 §2.4）。

### §3.2 共用 helper deduplication

Per cycle 03-19 R3 D-§4 + BABBAGE R2：共用 helper `redact_volition_payload` 位於 Plan52 `sourceContext` utility module。Provider-* family（8）+ context-* family（4）全部集中消費 helper → 實際 retrofit touched-plugin 數 = 15-18（非 24）。

### §3.3 OUT-of-scope plugins（~20 plugins；無敏感資料）

| family | plugins | rationale |
|--------|---------|-----------|
| `static-*` | static-fs-loader / static-config-loader | 讀取結構化 config；無 payload redaction 需求 |
| `fs-*` | fs-readwrite / fs-snapshot | 僅 path-metadata；非透過 Plan52 sourceContext |
| `mcp-*` | mcp-server / mcp-client | protocol-level；redaction 在 app layer |
| `devtools-*` | devtools-inspector / devtools-tracer | dev-only；無 production runtime |
| `policy-*` | policy-evaluator | rule evaluation；無 payload data |
| `lifecycle-*` | lifecycle-monitor | 僅 metadata |

---

## §4 6 道關卡 Non-bypassable Enforcement

Per cycle 03-19 R3 D-§4-R2-06 23/0 UNANIMOUS：
1. **Rule #75 §75.X gate** — `pnpm install --frozen-lockfile && pnpm build && pnpm test && pnpm test:purity` 全 0 exit
2. **Tri-party MR-6 audit** — TANENBAUM + KERNEL + GUARDIAN sign-off
3. **F-15 v3 linter** — false-positive 0% / false-negative 0% target
4. **Cross-OS CI matrix** — Linux + Windows persistent
5. **W2-R22 verification** — regression test
6. **R3 review** — cycle 03-19 R3 D-§4（this round）

**任何 waive request 上呈 Master prerogative**。

---

## §5 Sunset 機制（cycle 03-19 R4 close hard）

Per Master directive 2026-04-30 §4.3 BINDING：
- **Cycle 03-19 R4 close**（2026-05-01）= hard sunset deadline
- 任何 IN-SCOPE plugin 未 retrofit → **DEPRECATED label** + **WARN log** + **replacement schedule**
- DEPRECATED entries 須附 `capability_orphan_status` attestation（per RUSSELL R2 amendment）以防 silent capability degradation

---

## §6 Scope-strict Reject-on-violation

Per cycle 03-19 R3 D-§4-R2-08 23/0 UNANIMOUS：

**嚴格界定於 redaction format**。Plugin logic changes OUT-OF-SCOPE。任何 retrofit PR 引入 logic changes → **REJECT at R2/R3** + **return to Dev for scope split**。

---

## §7 Dissent 保留 per MR-11

DSS preservation tracking：
- **DSS-CY18-02**（KERNEL N=8 hex preference；cycle 03-18 carryover）— preserved verbatim across all 18 retrofit entries；non-blocking；N=4 alphanumeric ratified per cycle 03-18 D-§1-R2-B 22/1
- **DSS-CY19-§1-C**（KERNEL N=8 carryover；cycle 03-19 D-§1-R2-B reaffirm 22/1）— preserved

---

*Redaction Security Debt Register — BINDING canonical — cycle 03-19 R3 D-§4 ratified 23/0 — 2026-05-01*
*Reference/15 Security Retroactive Precedent 之首次具體適用*
*Master Ratification Batch 16 #10 dispatch ready*
