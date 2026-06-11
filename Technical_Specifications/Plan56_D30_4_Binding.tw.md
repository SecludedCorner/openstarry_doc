---
title: Plan56 — D-30-4 多重意志系統 綁定規範（繁中）
author: ASANGA + NAGARJUNA + LEIBNIZ + RUSSELL + TANENBAUM + ARCHIMEDES + Master directive 2026-04-29
date: 2026-04-30
cycle: 03-18 (ratified) → 03-19 (canonical doc backfilled by coordinator)
status: 程式庫已移除於 v0.58.0-alpha（2026-06-11 修復稽核）— v0.53.0-alpha 交付且有測試，但從未接入執行迴圈（0 個 production import）。多重意志設計保留作未來工作參考；現役意志層 = core loop 的單一 IVolition 路徑。
authority: Master ratified（Cycle 03-18 Batch 15 Item #1；APPROVED Option A 22/1 super-majority + Master doctrinal annotation 2026-04-29）
supersedes: Plan56_D30_4_Binding.md（EN sibling；本檔為 TW 翻譯版 per Rule #78 BINDING-tier reflexive）
cross_refs:
  - openstarry_doc/Technical_Specifications/Plan56_D30_4_Binding.md（EN sibling）
  - openstarry_doc/Technical_Specifications/Plan52_pushInput_Binding.md（同構基準）
  - openstarry_doc/Technical_Specifications/Plan54_AC9_Binding.md（同構）
  - openstarry_doc/Reference/11_Rule_78_TW_Translation.md（TW parity 規範）
---

# Plan56 — D-30-4 多重意志系統 綁定規範

**狀態**: BINDING
**ratified cycle**: 03-18（R3 Round A 22/1 super-majority Option A + Master directive 2026-04-29 義理註解）
**Phase 6 軌跡**: 第三棒 functional；Plan52 pushInput (1/7) + Plan54 AC-9 (2/7) → **Plan56 D-30-4 (3/7)** → D-30-5 / Mesh / API Runtime / Blackboard-Alaya 餘 4

---

## §1 Master 義理註解（per Master directive 2026-04-29 verbatim）

> OpenStarry 的多重意志系統，骨架取自行蘊的兩個層次：
>
> 產生層（並行）：仿六思身對應六根的結構，系統允許多個 volition 候選同時生起，各自緣其所對應的輸入域。
> 輸出層（序列）：仿心剎那遷流的結構，最終現行為動作時收斂為單一序列輸出。
>
> 此設計不引入外部仲裁器：收斂由內部動力學決定（候選之間的強度、優先級、上下文相關性），而非由裁判機制判決。這在概念上對應經文中「思即是業」的單一業現行，以及「心轉變迅速」的剎那遷流圖像。

**架構意涵**:
- **產生層並行** = 多個 volition 候選同時生起；各自緣對應輸入域；對應六思身/六根
- **輸出層序列** = 收斂為單一序列輸出；對應心剎那遷流；對應「思即是業」單一業現行
- **無外部仲裁器** = 由內部動力學（強度 / 優先級 / 上下文相關性）決定收斂；不引入裁判機制

---

## §2 架構：選項 A 單一序列多重意志佇列

per R3 §2 Round A 22/1 super-majority + Master 義理註解 §1：

### §2.1 Plugin 層配置

D-30-4 多重意志系統為 **plugin 層 composition primitive**，**非 Core**（per MR-6 鐵律）：
- 不在 `packages/core/*` 加任何 policy constants
- 透過既有 Plan52 sourceContext 路徑 emit InputEvents
- 視需要透過 Plan54 AC-9 pathway spawn sub-agent

### §2.2 兩層架構

```
產生層（並行）
  candidate_1（緣於輸入域 A）
  candidate_2（緣於輸入域 B）
  candidate_3（緣於輸入域 C）
        ↓
        ↓ 內部動力學收斂（強度 / 優先級 / 上下文相關性）
        ↓ 無外部仲裁器
        ↓
輸出層（序列）
  → emit_1 → emit_2 → emit_3 → ...（單一序列）
```

### §2.3 三條架構性禁止（per R1 §1 ASANGA + NAGARJUNA 唯識/中觀）

1. **無中央註冊**：無 global volition-state coordination object
2. **無仲裁基底**：無 priority arbiter mediating across volitions
3. **構成性閉合**：volitions 透過 temporal ordering 構成；不另有 substrate 累積狀態

---

## §3 ε-surface（0-delta strict equality；MR-6 鐵律）

per Plan52 / Plan54 同構：
- **ε-surface 與 Plan52 baseline delta = 0 fields, 0 const**
- 7-sub-check ε-surface 自 cycle 03-17 D-§1-R2-B 繼承
- v0.53.0-alpha tag 時 `pnpm test:purity` PASS
- `packages/core/**` 0 new policy constants

## §4 LOC 預算（per R3 A2 23/0 UNANIMOUS）

- prod: **600-900 LOC ceiling**
- test: **400-600 LOC ceiling**
- R1 預估值: ~500 prod / ~320 test（80%+ buffer）

## §5 Tri-Party MR-6 Audit Sign-Off（per cycle 03-17 Batch 14 #1 carry-forward）

- **TANENBAUM**（Plan56 spec authority）+ **KERNEL**（Core surface authority）+ **GUARDIAN**（security authority）
- 4-column Markdown table + Git GPG-signed commit per persona
- 落於 `delivery_report.md § Tri-Party MR-6 Sign-Off`

## §6 Spec 釐清（per R3 A7+A8+A9）

### §6.1 A7 — Volition replay 去重複語義（23/0 UNANIMOUS）

HMAC nonce keyed deduplication；replay cache 共用 topology declaration

### §6.2 A8 — Operator override 環境變數（20/2/1 super-majority；DSS-CY18-03 保留）

- 環境變數: `OPENSTARRY_MAX_VOLITION_QUEUE`
- 預設 cap with operator override
- 允許 per-spawn override

### §6.3 A9 — 多重意志 emit parent quota 計入（23/0 UNANIMOUS）

- 每次 emission 計入 parent quota
- 同 AC-9 NEG-6 模式

## §7 Plan52 InputEvent 整合不變量（3 → 6 per R3 A6 23/0 UNANIMOUS）

per cycle 03-18 D-§1-R2-D codification：

1. （既有 inv-1）InputEvent.sourceContext shape 保留
2. （既有 inv-2）nonce 唯一性保證
3. （既有 inv-3）HMAC tokenSig 驗證
4. **（新增 inv-4）多重意志 emit ordering 不變量**（temporal ordering 跨 emission 保留）
5. **（新增 inv-5）parent quota counter 單調性**（per A9）
6. **（新增 inv-6）env var override audit trail**（per A8 + cycle 03-17 Item #6 MAX_SPAWN_DEPTH override audit）

## §8 Replay Cache Topology — 三方貢獻者宣告

per cycle 03-18 D-§1-R2-A 23/0 UNANIMOUS：

| 貢獻者 | Plan | Cycle |
|------------|------|:-----:|
| 1 | Plan52 pushInput | 03-14 |
| 2 | Plan54 AC-9 | 03-17 |
| 3 | **Plan56 D-30-4** | **03-18** |

- 共用同一 replay cache instance
- topology 明文宣告（誰寫 / 寫順序 / 保留時間 / eviction 策略）
- nonce collision 偵測 + lock 機制保證

## §9 NEG Adversarial 案例（NEG-1~6）

per spec + A7+A8+A9 ratification：

| NEG | 場景 | 預期行為 |
|------|------|-------------------|
| NEG-D1 | multi-emit limit | 超過 parent quota 拒絕 |
| NEG-D2 | invalid priority weight | 拒絕 |
| NEG-D3 | capability bypass attempt | 拒絕 |
| NEG-D4 | invalid HMAC | 拒絕 |
| NEG-D5 | timing window 超過 TTL | 拒絕 |
| NEG-D6 | env var out-of-range（per A8）| 拒絕 + audit log |

## §10 驗證 Gates（per Rule #75 §75.X）

- v0.53.0-alpha tag 時 `pnpm install --frozen-lockfile && pnpm build && pnpm test && pnpm test:purity` 全 0 exit
- Cross-OS CI matrix（Linux + Windows）
- F-13 / F-14 / F-15 v3 reflexive PASS（含 TW parity per Rule #78）
- Tri-party MR-6 sign-off 4-column GPG-signed table
- v0.53.0-alpha 邊界 lexical-token 重新背書

## §11 合規證明

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 / MR-4（Tenet 措辭不變）| ✅ PASS | 0 D-item 提議 Tenet 措辭改 |
| MR-5 hard（Tenet #10 status 不變）| ✅ PASS | Plan56 plugin 層；0 status change 蘊含 |
| MR-6（Core 零）| ✅ PASS | ε-surface 0-delta；pnpm test:purity PASS |
| MR-9（無 MUST WAIVE）| ✅ PASS | F-16 SHOULD initial 維持 per cycle 03-17 OPT-C |
| MR-10（back-fill）| ✅ PASS | Plan56 spec 可透過未來 R-input 擴展 |
| MR-11（dissent 保留）| ✅ PASS | DSS-CY18-01（LEIBNIZ Option B）+ DSS-CY18-02（KERNEL N=8 hex）+ DSS-CY18-03（LEIBNIZ+RUSSELL env var）逐字保留 |
| MR-12（既有不破壞）| ✅ PASS | 既有 Plan52 / Plan54 部署未 retrofit |
| Rule #74 L1' | ✅ PASS | v0.53.0-alpha tag 時 code+doc sync |
| Rule #75 §75.X | ✅ PASS | 第 5 次強化 pre-delivery gate |
| Rule #76 §76.7 | ✅ PASS | composition_index regime caveat MUST emit |
| Rule #77 σ_regime | ✅ PASS | composition_index conjunct active |
| Rule #78 TW parity | ✅ PASS | 本檔即 TW sibling |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | 無 tenet rewrite；endpoint 10/0/0★ 不變；control-range 不放寬 |

## §12 Authority Notes

- **Master Ratification**: Cycle 03-18 Batch 15 Item #1 APPROVED 2026-04-30
- **Master 義理註解**: 2026-04-29 verbatim per §1
- **R3 Provenance**: §2 R3 Round A 22/1 + 23/0（LOC）+ 4 codifications + 3 clarifications

## §13 Cross-References

- `openstarry_doc/Technical_Specifications/Plan52_pushInput_Binding.md`（同構 baseline）
- `openstarry_doc/Technical_Specifications/Plan54_AC9_Binding.md`（同構）
- `openstarry_doc/Technical_Specifications/Plan56_D30_4_Binding.md`（EN sibling）
- `research record/cycle03-18/deliver/O1_D30_4_Plan56_implementation_final.md`（R-team R4 source）
- `research record/cycle03-18/deliver/Master_Ratification/Master_Confirmation.md`（Master directive 2026-04-29 source §3）

## §14 Dissent Slots — DSS-CY18-01..03 逐字（per MR-11）

### §14.1 DSS-CY18-01（LEIBNIZ，1 票；Option B）

> Arbiter sub-layer multi-volition arbitration would make multi-agent volition coordination semantically richer (per cooperation theory); single-stream queue collapses orthogonal volitions into temporal ordering. Defer to Option A for cycle 03-18 first-shipping; revisit at cycle 03-19+ VasanaEngine implementation if multi-arbiter pattern emerges.

**Partial resolution per Master directive 2026-04-29 §1 義理註解**：並行 in 產生層 already addressed；LEIBNIZ 主張的並行精神在產生層達成（dissent verbatim 仍保留）。

### §14.2 DSS-CY18-02（KERNEL，1 票；redaction format N=8 hex 偏好）

> N=4 alphanumeric ceiling is good security default but N=8 hex would preserve more forensic value during incident triage without significantly compromising redaction integrity. Acceptable trade-off; not blocking.

### §14.3 DSS-CY18-03（LEIBNIZ + RUSSELL，2 票；env var override 推遲）

> Adding env var override at first-shipping introduces operational surface area; prefer constant cap initially with env override added cycle 03-19+ if production data requires.

---

*Plan56 D-30-4 多重意志系統綁定規範 — BINDING — Master ratified Cycle 03-18 Batch 15 Item #1（2026-04-30）+ Master directive 2026-04-29 義理註解*
*Phase 6 第三棒；plugin 層 Plan52/Plan54 同構；ε-surface 0-delta；HMAC + tri-party MR-6 audit；MAX_SPAWN_DEPTH=4 + override；LOC 600-900/400-600；v0.53.0-alpha shipped 2026-04-30 01:33Z*
*9/0/1★ ACTIVE preserved / Tenet #10 NC PENDING per MR-5 / endpoint 10/0/0★ unchanged*
