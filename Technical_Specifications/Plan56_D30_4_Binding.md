---
title: Plan56 — D-30-4 Multi-IVolition Binding Specification
author: ASANGA + NAGARJUNA + LEIBNIZ + RUSSELL + TANENBAUM + ARCHIMEDES + Master directive 2026-04-29
date: 2026-04-30
cycle: 03-18 (ratified) → 03-19 (canonical doc backfilled by coordinator)
status: BINDING
authority: Master ratified (Cycle 03-18 Batch 15 Item #1; APPROVED Option A 22/1 super-majority + Master doctrinal annotation 2026-04-29)
supersedes: research record/cycle03-18/deliver/O1_D30_4_Plan56_implementation_final.md (R-team R4 source; this canonical doc is the binding extract)
cross_refs:
  - openstarry_doc/Technical_Specifications/Plan52_pushInput_Binding.md (isomorph)
  - openstarry_doc/Technical_Specifications/Plan54_AC9_Binding.md (isomorph)
  - claude research/research record/cycle03-18/deliver/Master_Ratification/Master_Confirmation.md (Master directive 2026-04-29 §3 doctrinal annotation source)
  - claude research/research input/master_letters/cycle03-15/Master_letter.md §十三 (strict 7-list anchor)
---

# Plan56 — D-30-4 Multi-IVolition Binding Specification

**Status**: BINDING
**Cycle ratified**: 03-18 (R3 Round A 22/1 super-majority Option A + Master directive 2026-04-29 doctrinal annotation)
**Phase 6 trajectory**: 第三棒 functional; Plan52 pushInput (1/7) + Plan54 AC-9 (2/7) → **Plan56 D-30-4 (3/7)** → D-30-5 / Mesh / API Runtime / Blackboard-Alaya 餘 4

---

## §1 Master Doctrinal Annotation (verbatim per Master directive 2026-04-29)

> OpenStarry 的多重意志系統，骨架取自行蘊的兩個層次：
>
> 產生層（並行）：仿六思身對應六根的結構，系統允許多個 volition 候選同時生起，各自緣其所對應的輸入域。
> 輸出層（序列）：仿心剎那遷流的結構，最終現行為動作時收斂為單一序列輸出。
>
> 此設計不引入外部仲裁器：收斂由內部動力學決定（候選之間的強度、優先級、上下文相關性），而非由裁判機制判決。這在概念上對應經文中「思即是業」的單一業現行，以及「心轉變迅速」的剎那遷流圖像。

**架構意涵**:
- **產生層並行** = multi-volition candidates 同時生起；各自緣對應輸入域；對應六思身/六根
- **輸出層序列** = 收斂單一序列輸出；對應心剎那遷流；對應「思即是業」單一業現行
- **無外部仲裁器** = internal dynamics（強度 / 優先級 / 上下文相關性）決定收斂；不引入裁判機制

---

## §2 Architecture: Option A Single-Stream Multi-Volition Queue

per R3 §2 Round A 22/1 super-majority + Master doctrinal annotation §1：

### §2.1 Plugin layer placement

D-30-4 Multi-IVolition 為 **plugin-layer composition primitive**，**non-Core**（per MR-6 鐵律）：
- 不在 `packages/core/*` 添加任何 policy constants
- 透過既有 Plan52 sourceContext 路徑 emit InputEvents
- 視需要透過 Plan54 AC-9 pathway spawn sub-agent

### §2.2 兩層架構

```
產生層（並行）
  candidate_1 (緣於輸入域 A)
  candidate_2 (緣於輸入域 B)  
  candidate_3 (緣於輸入域 C)
        ↓
        ↓ internal dynamics 收斂（強度 / 優先級 / 上下文相關性）
        ↓ NO external arbitrator
        ↓
輸出層（序列）
  → emit_1 → emit_2 → emit_3 → ...（單一序列）
```

### §2.3 三條架構性禁止（per R1 §1 ASANGA + NAGARJUNA Yogācāra/Madhyamaka）

1. **No central registry**：無 global volition-state coordination object
2. **No arbitration substrate**：無 priority arbiter mediating across volitions
3. **Compositional closure**：volitions compose via temporal ordering only；no orthogonal substrate accumulates state

---

## §3 ε-surface（0-delta strict equality；MR-6 鐵律）

per Plan52 / Plan54 isomorph：
- **ε-surface delta vs Plan52 baseline = 0 fields, 0 const**
- 7-sub-check ε-surface inheritance from cycle 03-17 D-§1-R2-B
- `pnpm test:purity` PASS at v0.53.0-alpha tag
- `packages/core/**` 0 new policy constants

## §4 LOC Budget (per R3 A2 23/0 UNANIMOUS)

- prod: **600-900 LOC ceiling**
- test: **400-600 LOC ceiling**
- R1 indicative: ~500 prod / ~320 test (80%+ buffer)

## §5 Tri-Party MR-6 Audit Sign-Off (per cycle 03-17 Batch 14 #1 carry-forward)

- **TANENBAUM**（Plan56 spec authority）+ **KERNEL**（Core surface authority）+ **GUARDIAN**（security authority）
- 4-column Markdown table + Git GPG-signed commit per persona
- 落 `delivery_report.md § Tri-Party MR-6 Sign-Off`

## §6 Spec Clarifications (per R3 A7+A8+A9)

### §6.1 A7 — Volition replay de-dup semantics (23/0 UNANIMOUS)

HMAC nonce keyed deduplication；replay cache 共用 topology declaration

### §6.2 A8 — Operator override env var (20/2/1 super-majority；DSS-CY18-03 preserved)

- env var: `OPENSTARRY_MAX_VOLITION_QUEUE`
- default cap with operator override
- per-spawn override allowed

### §6.3 A9 — Multi-volition emit parent quota consumption (23/0 UNANIMOUS)

- 每次 emission 計入 parent quota
- 同 AC-9 NEG-6 pattern

## §7 Plan52 InputEvent Integration Invariants (3 → 6 per R3 A6 23/0 UNANIMOUS)

per cycle 03-18 D-§1-R2-D codification：

1. （既有 inv-1）InputEvent.sourceContext shape preserved
2. （既有 inv-2）nonce uniqueness guaranteed
3. （既有 inv-3）HMAC tokenSig verification  
4. **（新增 inv-4）Multi-volition emit ordering invariant**（temporal ordering preserved across emissions）
5. **（新增 inv-5）parent quota counter monotonicity**（per A9）
6. **（新增 inv-6）env var override audit trail**（per A8 + cycle 03-17 Item #6 MAX_SPAWN_DEPTH override audit）

## §8 Replay Cache Topology — Three-Contributor Declaration

per cycle 03-18 D-§1-R2-A 23/0 UNANIMOUS：

| Contributor | Plan | Cycle |
|------------|------|:-----:|
| 1 | Plan52 pushInput | 03-14 |
| 2 | Plan54 AC-9 | 03-17 |
| 3 | **Plan56 D-30-4** | **03-18** |

- 共用同一 replay cache instance
- topology 明文宣告（誰寫 / 寫順序 / 保留時間 / eviction policy）
- nonce collision 偵測 + lock 機制保證

## §9 NEG Adversarial Cases (NEG-1~6)

per spec + A7+A8+A9 ratification：

| NEG | 場景 | Expected behavior |
|------|------|-------------------|
| NEG-D1 | multi-emit limit | reject if exceed parent quota |
| NEG-D2 | invalid priority weight | reject |
| NEG-D3 | capability bypass attempt | reject |
| NEG-D4 | invalid HMAC | reject |
| NEG-D5 | timing window outside TTL | reject |
| NEG-D6 | env var out-of-range（per A8）| reject + audit log |

## §10 Verification Gates (per Rule #75 §75.X)

- `pnpm install --frozen-lockfile && pnpm build && pnpm test && pnpm test:purity` 全 0 exit at v0.53.0-alpha tag
- Cross-OS CI matrix（Linux + Windows）
- F-13 / F-14 / F-15 v3 reflexive PASS（含 TW parity per Rule #78）
- Tri-party MR-6 sign-off 4-column GPG-signed table
- Lexical-token re-attestation at v0.53.0-alpha boundary

## §11 Compliance Attestation

| Constraint | Status | Evidence |
|------------|:------:|----------|
| MR-2 / MR-4 (Tenet wording 不變) | ✅ PASS | 0 D-item proposes Tenet wording change |
| MR-5 hard (Tenet #10 status 不變) | ✅ PASS | Plan56 plugin layer; 0 status change implication |
| MR-6 (Core 零) | ✅ PASS | ε-surface 0-delta; pnpm test:purity PASS |
| MR-9 (no MUST WAIVE) | ✅ PASS | F-16 SHOULD initial 維持 per cycle 03-17 OPT-C |
| MR-10 (back-fill) | ✅ PASS | Plan56 spec extensible via future R-input |
| MR-11 (dissent preservation) | ✅ PASS | DSS-CY18-01 (LEIBNIZ Option B) + DSS-CY18-02 (KERNEL N=8 hex) + DSS-CY18-03 (LEIBNIZ+RUSSELL env var) verbatim |
| MR-12 (既有不破壞) | ✅ PASS | 既有 Plan52 / Plan54 deployments not retrofitted |
| Rule #74 L1' | ✅ PASS | Code+doc sync at v0.53.0-alpha tag |
| Rule #75 §75.X | ✅ PASS | 5th-enforced pre-delivery gate |
| Rule #76 §76.7 | ✅ PASS | composition_index regime caveat MUST emit |
| Rule #77 σ_regime | ✅ PASS | composition_index conjunct active |
| Rule #78 TW parity | ✅ PASS | TW sibling Plan56_D30_4_Binding.tw.md |
| ZT-1 / ZT-2 / ZT-3 | ✅ PASS | No tenet rewrite; endpoint 10/0/0★ unchanged; control-range untouched |

## §12 Authority Notes

- **Master Ratification**: Cycle 03-18 Batch 15 Item #1 APPROVED 2026-04-30
- **Master Doctrinal Annotation**: 2026-04-29 verbatim per §1
- **R3 Provenance**: §2 R3 Round A 22/1 + 23/0 (LOC) + 4 codifications + 3 clarifications

## §13 Cross-References

- `openstarry_doc/Technical_Specifications/Plan52_pushInput_Binding.md`（isomorph baseline）
- `openstarry_doc/Technical_Specifications/Plan54_AC9_Binding.md`（isomorph）
- `research record/cycle03-18/deliver/O1_D30_4_Plan56_implementation_final.md`（R-team R4 source）
- `research record/cycle03-18/deliver/Master_Ratification/Master_Confirmation.md`（Master directive 2026-04-29 source §3）

## §14 Dissent Slots — DSS-CY18-01..03 Verbatim (per MR-11)

### §14.1 DSS-CY18-01 (LEIBNIZ, 1 vote; Option B)

> Arbiter sub-layer multi-volition arbitration would make multi-agent volition coordination semantically richer (per cooperation theory); single-stream queue collapses orthogonal volitions into temporal ordering. Defer to Option A for cycle 03-18 first-shipping; revisit at cycle 03-19+ VasanaEngine implementation if multi-arbiter pattern emerges.

**Partial resolution per Master directive 2026-04-29 §1 doctrinal annotation**: 並行 in 產生層 already addressed; LEIBNIZ 主張的並行精神在產生層達成（dissent verbatim 仍保留）。

### §14.2 DSS-CY18-02 (KERNEL, 1 vote; redaction format N=8 hex preferred)

> N=4 alphanumeric ceiling is good security default but N=8 hex would preserve more forensic value during incident triage without significantly compromising redaction integrity. Acceptable trade-off; not blocking.

### §14.3 DSS-CY18-03 (LEIBNIZ + RUSSELL, 2 votes; env var override deferred)

> Adding env var override at first-shipping introduces operational surface area; prefer constant cap initially with env override added cycle 03-19+ if production data requires.

---

*Plan56 D-30-4 Multi-IVolition Binding Specification — BINDING — Master ratified Cycle 03-18 Batch 15 Item #1 (2026-04-30) + Master directive 2026-04-29 doctrinal annotation*
*Phase 6 第三棒；plugin layer Plan52/Plan54 isomorph；ε-surface 0-delta；HMAC + tri-party MR-6 audit；MAX_SPAWN_DEPTH=4 + override；LOC 600-900/400-600；v0.53.0-alpha shipped 2026-04-30 01:33Z*
*9/0/1★ ACTIVE preserved / Tenet #10 NC PENDING per MR-5 / endpoint 10/0/0★ unchanged*
