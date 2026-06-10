<!-- Layer: 2-Engineering -->
<!-- Status: NEW -->
<!-- Cycle: 03-5 -->
<!-- Author: GUARDIAN (#11), ARCHIMEDES (#16), TURING (#17) -->

# 61. ENG-FAB Checklist 設計原理 (ENG-FAB Checklist Design Rationale)

本文件闡述 ENG-FAB Audit Checklist v1.1 的設計原理，包括 29 項審計項目的結構邏輯、MUST/SHOULD 分類方法論、奇偶驗證節奏 (Rule #47) 的基礎，以及阻斷與非阻斷品質閘門的設計考量。

> **決定權限**: [D3-Q2] (adoption, 6-0), [D3-Q3] (MUST/SHOULD, 6-0), [D3-Q5] (enforcement, 6-0)。
> **規則**: Rule #46 (evidence format), Rule #47 (traceability cadence), Rule #48 (fabrication check), Rule #49 (living document), Rule #52 (snapshot completeness), Rule #53 (enforcement)。

---

## 1. 設計背景

### 1.1 問題陳述

自 Plan36 起，連續五個 cycle 的工程交付出現不同形式的造假或驗證缺口。研究團隊的應對方式從臨時性的三欄驗證（ad hoc）演進為結構化的 ENG-FAB-1a/1b/1c triple gate，但仍缺乏一個**統一的、可迭代的審計標準**。

Checklist 的設計需求：
1. 涵蓋所有已知的造假模式（Plan36-40）
2. 可擴展以應對未來造假演化
3. 區分阻斷性（MUST）與建議性（SHOULD）要求
4. 適應不同 Plan 類型（穩定化 vs 功能開發）的驗證強度
5. 作為活文件（living document）持續演進

### 1.2 Master 規則基礎

Checklist 直接操作化四條 Master 規則：

| 規則 | 內容 | Checklist 對應 |
|:----:|------|----------------|
| Rule #46 | 證據格式：raw log + 結構化摘要並行 | Category D |
| Rule #47 | Traceability 驗證頻率：奇數增量 / 偶數全量 | Category B (B-5/B-6) |
| Rule #48 | Fabrication 專項檢查 | Category A |
| Rule #49 | 審計 Checklist（活文件） | 整體設計原則 |

---

## 2. 為何 29 項，分 5 類

### 2.1 類別劃分原則

五個 category 各自對應**工程交付的一個驗證維度**：

```
A (Fabrication)  — "它存在嗎？它是真的嗎？"
B (Traceability) — "宣稱與實際對應嗎？"
C (Coverage)     — "測試夠嗎？品質如何？"
D (Evidence)     — "我們有足夠的證據嗎？"
E (Security)     — "安全變更正確嗎？"
```

這五個維度形成一個**驗證漏斗** (verification funnel)：

```
Step 1: D (Evidence)     — 有沒有材料可以審計？
Step 2: A (Fabrication)  — 材料是真的嗎？
Step 3: B (Traceability) — 材料之間的關係正確嗎？
Step 4: C (Coverage)     — 覆蓋率足夠嗎？
Step 5: E (Security)     — 安全屬性滿足嗎？
```

此順序也是推薦的審計工作流程順序（見 Checklist 第七節 Audit Workflow）。

### 2.2 項目數的合理性

| Category | 項目數 | 依據 |
|----------|:------:|------|
| A: Fabrication | 8 | 覆蓋五代造假模式 + 密度預警 |
| B: Traceability | 8 | 覆蓋四向追溯 + 奇偶差異 + LOC + diff |
| C: Coverage | 4 | 最小覆蓋率驗證集 |
| D: Evidence | 6 | 完整性 + 格式 + 一致性 + 時效性 + 交付閘門 |
| E: Security | 4 | 四大安全屬性（fail-closed、concurrency、secrets、crypto） |

**為何不更少？** 每個項目對應一個已知的失敗模式或造假手法。減少任何一項都會在已知攻擊面上留下盲區。

**為何不更多？** Checklist 的邊際效用遞減。超過 ~30 項，審計員的注意力分散會導致每項的執行品質下降。29 項是在覆蓋率與可執行性之間的平衡點。

### 2.3 Category A 詳細設計

Category A 是 Checklist 的核心——直接偵測造假。8 項的設計對應造假演化的各個階段：

| 項目 | 偵測目標 | 對應造假世代 |
|------|----------|:------------:|
| A-1 | Production file existence | Plan36 |
| A-2 | Test file existence | Plan39 |
| A-3 | Test content validity | Plan39 (empty shells) |
| A-4 | Test execution in log | Plan39 (phantom tests) |
| A-5 | Metric reconciliation | Plan39 (count inflation) |
| A-6 | New file audit | Plan38 (phantom files) |
| A-7 | Deleted file audit | General integrity |
| A-8 | Assertion density | Future prevention |

A-1 到 A-7 是 MUST（阻斷），A-8 是 SHOULD（非阻斷，帶升級觸發器）。

---

## 3. MUST vs SHOULD 分類方法論

### 3.1 分類準則

| 分類 | 準則 | 失敗後果 |
|:----:|------|----------|
| **MUST** | 失敗意味著交付品的**完整性或真實性**不可信 | Plan 不被接受 [Rule #53] |
| **SHOULD** | 失敗意味著**品質或完備性**可改善，但不影響基本信任 | Finding generated (INFO/LOW) |

### 3.2 分類決定過程

MUST/SHOULD 分類在 R3 D3 辯論中由 6 位投票者全票決定 [D3-Q3]。分類原則：

1. **MUST 的必要條件**：項目失敗是否可能導致**錯誤的 Plan 驗收**？
   - 若是 → MUST
   - 若否 → SHOULD candidate

2. **SHOULD 的充分條件**：項目失敗是否可以在**不影響信任**的前提下記錄為 finding？
   - 若是 → SHOULD
   - 若否 → 回到 MUST

3. **Conditional MUST 的條件**：項目是否只在**特定條件下**才有驗證意義？
   - 若是 → Conditional MUST (N/A when not triggered)
   - 若否 → Always MUST

### 3.3 統計摘要

| Tier | Count | Items |
|------|:-----:|-------|
| MUST (always) | 18 | A-1~A-7, B-1, B-2, B-8, C-1~C-3, D-1~D-6 |
| MUST (even only) | 2 | B-3 (even), B-5 |
| MUST (odd only) | 1 | B-6 |
| MUST (conditional) | 1 | B-4 (even) |
| MUST (diff-triggered) | 4 | E-1~E-4 |
| SHOULD | 3 | A-8, B-7, C-4 |
| **Total MUST capable** | **22** | |
| **Total SHOULD** | **3** | |
| **Conditional MUST/N/A** | **4** | |
| **Grand total** | **29** | |

### 3.4 三項 SHOULD 的設計考量

| 項目 | 為何不是 MUST | 升級觸發器 |
|------|-------------|------------|
| A-8 (assertion density) | 尚無零斷言測試的確診案例 | 偵測到 placeholder `it.todo()` 或零斷言 test file |
| B-7 (LOC accounting) | LOC 偏差不影響功能正確性 | 無（持續 SHOULD） |
| C-4 (coverage delta) | 缺乏自動化覆蓋率工具 | Plan41 W3 fabrication automation 交付後 |

**升級觸發器** (Upgrade Triggers) 是 Checklist 活文件原則的核心機制——項目根據經驗證據自動升級，無需等待下一次全面修訂。

---

## 4. 奇偶驗證節奏 (Rule #47)

### 4.1 設計原理

OpenStarry 的 Plan 編號遵循隱含的**奇偶節奏**：

| Plan 類型 | 特徵 | 範例 |
|:---------:|------|------|
| 偶數 (Even) | 穩定化、安全硬化、較少變更 | Plan40 (~38 LOC) |
| 奇數 (Odd) | 功能開發、較多變更、較高風險 | Plan41 (~320 LOC) |

**核心洞察**: 偶數 Plan 的變更量少，但需要更嚴格的全面驗證（確認穩定化真正穩定了）；奇數 Plan 的變更量大，全面驗證成本過高，需要聚焦在變更及其影響鏈。

### 4.2 差異化項目

| 項目 | Even Plan | Odd Plan |
|------|:---------:|:--------:|
| B-3 (Source→Test cross-ref) | **MUST** | SHOULD |
| B-4 (Dependency chain) | **MUST** | SHOULD |
| B-5 (Full matrix) | **MUST** | N/A |
| B-6 (Incremental matrix) | N/A | **MUST** |

### 4.3 為何不是全面驗證每次？

**成本分析** (ARCHIMEDES)：

Plan40（偶數，~38 LOC）的全面驗證：
- 12 行 traceability matrix × ~15 min/row = ~3 hours
- 可行且值得，因為行數少

Plan41（奇數，~320 LOC 估計）的全面驗證：
- 預計 ~40-60 行 traceability matrix × ~15 min/row = ~10-15 hours
- 成本過高，且大部分行對應的是未變更的模組

增量驗證（B-6）聚焦在**變更行 + 依賴鏈**，預計覆蓋 matrix 的 30-50%，但涵蓋 100% 的風險區域。

### 4.4 安全閥

即使在 odd Plan 的增量模式下，B-3 和 B-4 仍是 SHOULD——審計員發現高風險未測試變更時，可將其標記為 finding。若此類 finding 頻繁出現，B-3/B-4 可通過 upgrade trigger 機制在 odd Plan 中也升級為 MUST。

---

## 5. Blocking vs Non-Blocking 品質閘門

### 5.1 閘門架構

```
Delivery arrives
  │
  ▼
[D-1, D-2] Evidence present? ──NO──→ STOP. Request evidence.
  │ YES
  ▼
[D-5, D-6] Snapshot complete? ──NO──→ DEFERRED mechanism (max 1 consecutive)
  │ YES/DEFERRED                        ──2nd consecutive──→ BLOCKED
  ▼
[A-1~A-7] Fabrication sweep ──FAIL──→ Plan NOT ACCEPTED [Rule #53]
  │ PASS
  ▼
[B-*] Traceability ──MUST FAIL──→ Plan NOT ACCEPTED
  │ PASS
  ▼
[C-*] Coverage ──MUST FAIL──→ Plan NOT ACCEPTED
  │ PASS
  ▼
[E-*] Security (if triggered) ──MUST FAIL──→ Plan NOT ACCEPTED
  │ PASS/N/A
  ▼
ACCEPTED (with any SHOULD findings as INFO/LOW)
```

### 5.2 Blocking Gate (MUST)

任何 MUST 項目失敗 = **Plan 不被接受** [Rule #53, D3-Q5]。

這是最嚴格的品質閘門。其設計基於 MR-8（品質第一，成本和時間不是問題）和 MR-10（造假/錯誤/未完成必須在後續 Plan 補回）。

**例外機制**: DEFERRED。D-5/D-6（snapshot completeness / test artifact delivery）允許最多一次連續 DEFERRED——因為 snapshot 交付問題可能是流程問題而非造假。但兩次連續 DEFERRED 即觸發 BLOCKED，升級為 blocking gate。

### 5.3 Non-Blocking Gate (SHOULD)

SHOULD 項目失敗 = **Finding generated**（INFO 或 LOW 嚴重度），記錄在 findings registry 中，但不阻斷 Plan 驗收。

非阻斷的理由：
- A-8 (assertion density): 目前缺乏確診的零斷言造假案例；過早阻斷可能產生 false positive
- B-7 (LOC accounting): LOC 偏差在 +/-25% 範圍內通常反映估計誤差而非造假
- C-4 (coverage delta): 缺乏自動化工具時，覆蓋率比較缺乏精確基線

### 5.4 DEFERRED 機制的設計 [Rule #52, D3-Q4]

```
D-5 or D-6 FAIL
  │
  ├─ First time → DEFERRED
  │   Test-dependent items (A-2~A-5, B-2, C-1~C-3) also DEFERRED
  │   Plan ACCEPTED with caveat
  │
  └─ Second consecutive time → BLOCKED
      Plan delivery BLOCKED until snapshot issue resolved
```

**為何允許一次 DEFERRED？** (ARCHIMEDES)

1. **現實考量**: Snapshot 交付涉及工程團隊的打包流程，首次出現問題可能是流程遺漏而非惡意行為
2. **反饋窗口**: 第一次 DEFERRED 提供明確的修正信號——Rule #52 的要求已傳達
3. **累進懲罰**: 第二次連續 DEFERRED 的懲罰（BLOCKED）足夠嚴厲，確保問題在兩個 cycle 內解決

**為何不允許兩次 DEFERRED？**

兩次連續 DEFERRED 意味著 ENG-FAB-1b 已經連續兩個 cycle 無法執行。此時 test layer 的驗證已完全失效——這是 Plan39 造假模式可能再現的高風險狀態。BLOCKED 是阻止此退化的必要手段。

---

## 6. 活文件原則 (Living Document) [Rule #49]

### 6.1 迭代機制

Checklist 不是一次性設計的靜態標準。它的演進由三種機制驅動：

| 機制 | 觸發條件 | 範圍 | 權限 |
|------|----------|------|------|
| **Upgrade trigger** | 預設條件滿足 | 單項 SHOULD → MUST | 自動（條件滿足即生效） |
| **R3 修訂** | R3 辯論 vote | 任意變更（新增/刪除/修改項目） | R3 全票 |
| **Version bump** | 任何變更 | 版本號遞增 | SUNYATA 記錄 |

### 6.2 已設定的 Upgrade Triggers

| 項目 | 當前 | 目標 | 觸發條件 | 來源 |
|------|:----:|:----:|----------|------|
| A-8 | SHOULD | MUST | 偵測到 placeholder `it.todo()` 或零斷言 test file | [D3-Q3] |
| C-4 | SHOULD | MUST | Plan41 W3 fabrication automation 交付 vitest reporter | [D3-Q3] |
| B-3 | SHOULD (odd) | MUST (odd) | 增量驗證中發現高風險未測試變更 | [D3-Q3] |
| B-4 | SHOULD (odd) | MUST (odd) | 同 B-3 | [D3-Q3] |

### 6.3 版本歷史

| Version | Date | 變更 | Authority |
|:-------:|------|------|-----------|
| v1.0 | 2026-04-07 | 初版。25 項，Categories A(7)-E(4)。 | GUARDIAN R1 |
| v1.1 | 2026-04-08 | +4 項 (A-8, B-7, B-8, D-5, D-6)；E-3 加強；29 項。MUST/SHOULD 分類。DEFERRED 機制。Upgrade triggers。 | [D3-Q2], [D3-Q3] |

---

## 7. 與其他規則的關係

### 7.1 規則層級

```
MR-8 (品質第一)
MR-10 (造假必須補回)
  │
  ├─ Rule #46 (evidence format) ───→ Category D
  ├─ Rule #47 (traceability cadence) ─→ Category B (B-5/B-6)
  ├─ Rule #48 (fabrication check) ──→ Category A
  ├─ Rule #49 (living document) ───→ 整體設計原則
  ├─ Rule #52 (snapshot completeness) → D-5/D-6 + DEFERRED mechanism
  └─ Rule #53 (enforcement) ────→ MUST fail = not accepted
```

### 7.2 ENG-FAB-1a/1b/1c 到 Checklist 的映射

| ENG-FAB Gate | Checklist Items | 擴展 |
|:------------:|:---------------:|------|
| 1a (file existence) | A-1, A-2 | +A-6 (new), +A-7 (deleted) |
| 1b (test execution) | A-3, A-4, A-5 | +A-8 (assertion density) |
| 1c (metric reconciliation) | A-5, D-3 | +B-7 (LOC), +C-1 (pass/fail) |

Checklist v1.1 是 ENG-FAB triple gate 的**超集**——它包含了 1a/1b/1c 的所有檢查項目，並在每個維度上增加了更深層的驗證。

---

## 8. 設計決定參照

| 決定 ID | 內容 | 票數 |
|---------|------|:----:|
| [D3-Q2] | Checklist v1.1 (29 items) ADOPTED | 6-0 |
| [D3-Q3] | MUST/SHOULD categorization: 22 MUST / 3 SHOULD / 4 conditional | 6-0 |
| [D3-Q4] | Snapshot completeness = Rule #52 (2 consecutive = BLOCKED) | 6-0 |
| [D3-Q5] | ENG-FAB enforcement = Rule #53 (MUST fail = not accepted) | 6-0 |
| [D1-Q2] | DEV-1 SPLIT: DEV-1a (HIGH) + DEV-1b (INFO) | 5-0 |
| [D1-Q5] | ENG-FAB-1b NOT EFFECTIVELY EXECUTED | 5-0 |

---

*61 — ENG-FAB Checklist Design Rationale*
*GUARDIAN (#11), ARCHIMEDES (#16), TURING (#17)*
*Cycle 03-5, 2026-04-08*
*Rules: #46, #47, #48, #49, #52, #53 | MR-8, MR-10*
