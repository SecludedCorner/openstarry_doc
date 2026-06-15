---
title: Reference/19 — Audit 方法論 Codification（中文版）
author: research-team (TURING + KNUTH + ARCHIMEDES + GUARDIAN + SUSSMAN + DARWIN + LINNAEUS)
date: 2026-05-07
cycle: 03-25
status: BINDING
authority: Master Ratification Batch 22 #1 + cycle 03-25 R3 D-§2 23/0 UNANIMOUS
supersedes: (無 — audit 方法論首次 concrete codification)
cross_refs:
  - claude research/research record/cycle03-25/deliver/O1_M1_audit_methodology_final.md（完整 deliver）
  - claude research/research record/cycle03-25/R3/R3_decision_log.md §1.1 D-§2
  - openstarry_eco/share/openstarry_doc/Reference/16_R4_Folder_Convention_Discipline.md
  - openstarry_eco/share/openstarry_doc/Reference/20_Audit_Verdict_Format_Codification.md（配對 A9；cycle 03-25 R3 D-§13 22/1）
  - F-15 v3 governance docs schema（ENG-FAB v1.8 #15）
  - Rule #74 L1' 5 sub-check / Rule #78 §78.5 TW parity
  - Tenet #2 + #8 / MR-7 + MR-11 + MR-12 / ZT-1/2/3
---

# Reference/19 — Audit 方法論 Codification（中文版）

**Status**：**BINDING**（Master Ratification Batch 22 #1；cycle 03-25 R3 D-§2 23/0 UNANIMOUS；cycle 03-26+ 生效 per MR-12）
**Date**：2026-05-07
**權威**：research-team R3 + R4；coordinator G5 sync 後落 canonical

---

## §1 背景 + Cycle 03-26 矩陣 Sweep 前置條件

per Master letter cycle 03-25 §二 §2.1：cycle 03-24 R1 §1 §7 Item A1「Tenet 真實 COMPLIANT 驗證」明文「不是 declared，不是 paper-only」，但 audit **方法**未定。沒方法 → 矩陣 sweep 跑起來會空轉。**M1 = 方法論先定，cell verdict 才能 rigorous**。

Cycle 03-26 矩陣 sweep 規模：
- 38 plugin × 10 Tenets = 380 plugin cells
- 15-30 functions × 10 Tenets = 150-300 function cells
- **總計 ≈ 530-680 cells**（per Master directive 2026-05-07 雙矩陣 sweep）

本 Reference/19 + 配對 Reference/20（A9 verdict format canonical）配對 mandatory：
- **Reference/19（本）說「用什麼方法」** → cell.method = L1 | L2 | L3 | L4 | conjunct
- **配對 Reference/20（A9）說「verdict 怎麼寫」** → cell.verdict = Full PASS | Conditional PASS | FAIL | FAIL fix=&lt;scope&gt; | N/A

Audit verifiability 三原則 inheritance（cycle 03-24 R1 §1 §7 Item A1）：
1. 不是 declared
2. 不是 paper-only
3. 第三方 reproducible

---

## §2 4 方法 Definition Table

| Method | 定義 | 主要 tooling | 輸出格式 | 可驗證 evidence 要求 |
|---|---|---|---|---|
| **L1 程式碼閱讀** | 直接讀 plugin source + manifest + factory pattern + dependency graph，依 Tenet 條文判 cell | `Read` / `Grep` / `Glob` 工具；IDE 跳轉；TURING manual review checklist | File path + line number 引用；quoted code excerpt（≤10 行）；verdict reasoning | Cell verdict 必引 ≥1 file path + line number；reviewer reproduces by reading same code |
| **L2 測試套件執行** | 跑 `pnpm test` / `pnpm test:purity` / 等價自動驗證；對行為 Tenet 收 PASS/FAIL evidence | pnpm runner；vitest；CI matrix（Linux + macOS + Windows × Node 20/22 = 16 cells）| Test command + flags；test result summary；failed test detail；test ID reference | Cell verdict 必引 ≥1 test ID + test command；reviewer reproduces by same command；flaky → 自動 FAIL |
| **L3 Static Analysis** | F-15 v3 governance schema lint + Rule #74 L1' 5 sub-check + ENG-FAB v1.8 checklist 自動化驗證 | `tools/f15_check.py`；F-15 v3 lint suite；ENG-FAB checklist runner；AST traversal；cross-ref scanner | Lint report（per-rule pass/fail）；violation file + line + rule ID；aggregated compliance % | Cell verdict 必引 ≥1 lint output entry + tool command；lint MUST deterministic（non-flaky）|
| **L4 Runtime Trace** | provider-claude-cli / 等價 LLM 運行時觀察 + replay cache + ε-surface delta + F1/F2/F3 counter | provider-claude-cli；Gemini-oauth（M2 primary）；Codex 25-cycle subset（M2 secondary）；audit_calc.py state-(a) | Runtime trace log（timestamped）；LLM API request/response（sanitized）；replay hit/miss；ε-surface delta；F1/F2/F3 counter | Cell verdict 必引 ≥1 runtime trace timestamp + log location；trace 不可 fabricate（Tenet#2）；LLM stochastic → 多次 sample 取趨勢 |

---

## §3 10 Tenets × 4 方法 Matrix

| Tenet # | 名稱 | 性質 | **主要適用方法** | Secondary 方法 |
|:-------:|------|------|----------------|---------------|
| #1 | 微核心優先 (microkernel discipline) | 結構性 | **L1** | L3 |
| #2 | 誠實透明 (honesty) | Composite | **L1 + L3 conjunct** | L4 |
| #3 | 接續斷裂 (continuity break) | 行為 | **L2 + L4 conjunct** | L1 |
| #4 | 隱藏錯誤 (hidden error) | 行為 | **L2 + L4 conjunct** | L3 |
| #5 | 漸進改良 (incremental refinement) | 規範 | **L3** | L1 |
| #6 | Plugin 治理 (plugin governance) | 結構性+規範 | **L1 + L3 conjunct** | L4 |
| #7 | Hub 中立 (hub neutrality) | 結構性 | **L1** | L3 |
| #8 | 證據>意見 (evidence over opinion) | 規範 | **L3** | L2 + L4 |
| #9 | 可重現 (reproducibility) | 行為 | **L2 + L4 conjunct** | L3 |
| #10 | 絕對純淨 (absolute purity) | Composite | **L1 + L3 conjunct** | L2 + L4 |

**性質 → 方法分工**：
- 結構性（Tenet #1, #7）→ **L1 primary**（manifest + factory 是 source of truth）
- 行為（Tenet #3, #4, #9）→ **L2 + L4 conjunct primary**（test + runtime 缺一不可）
- 規範（Tenet #5, #8）→ **L3 primary**（F-15 + ENG-FAB checklist 直接 codify）
- Composite（Tenet #2, #10）→ **L1 + L3 conjunct primary**（跨層 conjunct）
- 結構性+規範（Tenet #6）→ **L1 + L3 conjunct primary**

**Method coverage** verified non-vestigial（LINNAEUS taxonomy）：4 方法每個 primary ≥1 Tenet + secondary cross-coverage；matrix 完整。

---

## §4 Conjunct 規則 + Escalation + N/A 紀律

### §4.1 Conjunct 全有全無（3×3 State-Space Exhaustive）

對 行為 + composite Tenet conjunct primary（Tenets #2/#3/#4/#6/#9/#10）：

| method_a | method_b | Cell verdict |
|---|---|---|
| PASS | PASS | **PASS** |
| PASS / FAIL | FAIL / PASS | **FAIL**（任一 FAIL 主導）|
| FAIL | FAIL | **FAIL** |
| PASS / BORDERLINE | BORDERLINE / PASS | escalate borderline per §4.2 |
| FAIL | BORDERLINE | **FAIL**（任一 FAIL 主導 regardless of borderline）|
| BORDERLINE | FAIL | **FAIL**（symmetric）|
| BORDERLINE | BORDERLINE | escalate 兩個 per §4.2；若仍 borderline → R3 debate |

**為何 conjunct 全有全無**：composite/行為 Tenet 單一方法不足。不允許半 PASS / partial credit（避免 verdict dilution）。

### §4.2 Borderline Escalation 4×4

| Primary | Borderline 表徵 | Escalation 目標 |
|---------|-----------------|----------------|
| L1 borderline | Code review 意見分歧；manifest ambiguous | → **L3**（lint codify reading）|
| L2 borderline | Test flaky；intermittent fail | → **L4**（runtime trace 確認 root cause）|
| L3 borderline | Lint warning（not error）；borderline schema | → **L1**（manual code read）|
| L4 borderline | Runtime stochastic（LLM）；多次 run 結果不一 | → **L2 + L3 conjunct**（deterministic test + lint）|

**Self-loop rule**：same-method re-run 不 valid escalation；must escalate to higher rigor method per row。

**Borderline → R3 final unambiguity**：cell verdict MUST terminate in {Full PASS / Conditional PASS / FAIL / FAIL fix=&lt;scope&gt; / N/A} per A9 schema；"ambiguous" / "TBD" / "skipped" non-emittable。

**Tenet#5 special escalation**：L3 borderline + L1 borderline → R3 debate（rather than re-iterate L3）；cycle-by-cycle diff content should not be reviewed by *same* lint twice。

### §4.3 N/A 紀律

**Allowed conditions**：
- 該 Tenet 對該 plugin 性質本質不適用（e.g. provider plugin 不適用 Tenet#7 Hub 中立）
- 須附 N/A rationale（≥1 sentence 解釋為何不適用）
- 不允許「我不知道」「reviewer 沒時間」（per Tenet#2 誠實）

**40% threshold（LINNAEUS taxonomic profile review trigger；NOT auto-fail）**：
- > 40% N/A per plugin → 觸發 LINNAEUS taxonomic profile review
- Plugin classes with structurally-justified-many-N/A（lightweight provider / shim plugin / etc.）may exceed 40% **iff LINNAEUS profile substantiates non-applicability per Tenet-by-Tenet rationale**
- Lexical scrubbing for forbidden phrases（"不知道", "我不確定", "沒時間"）mechanically auditable post-hoc

---

## §5 Cell Verdict 流程

```
For each cell (plugin/function, Tenet):
  1. Lookup §3 matrix → 取主要適用方法 (single OR conjunct)
  2. Run primary method → collect evidence:
     - L1: read source + manifest → file:line cite
     - L2: run test → test ID + result
     - L3: run lint → tool output entry
     - L4: run runtime trace → log timestamp + location
  3. If conjunct (Tenets #2/#3/#4/#6/#9/#10):
     必兩方法 PASS 才 cell PASS (per §4.1 3×3 state-space)
  4. If primary borderline: escalate per §4.2 (or self-loop higher-rigor rule)
  5. Verdict = Full PASS | Conditional PASS | FAIL | FAIL fix=<scope> | N/A
     (per paired A9 format)
  6. Evidence reference required field 填齊
  7. If cell N/A: 必填 N/A rationale (per §4.3) + LINNAEUS taxonomy reference
  8. Emit cell record per A9 YAML-primary schema
```

**Cell PASS threshold per method**：
- L1 PASS：Code 結構符合 Tenet 條文；reviewer manual review 0 violation
- L2 PASS：Test 100% PASS（所有相關 test）；non-flaky
- L3 PASS：Lint 0 violation OR all violations have documented exemption（per Phase 6 strict 7-list anchor / F-15 v3 documented escape）
- L4 PASS：Runtime trace 符合預期 schema；F1/F2/F3 counter 0（per 5-cycle Popperian）；ε-surface delta within tolerance

**Disagreement DISPUTED → R3 vote 終裁**：兩 reviewer 對同 cell verdict 意見分歧 → cell verdict = "DISPUTED" → R3 主 session 24-agent debate → R3 vote 終裁（per MR-7）→ minority preserved per MR-11。

---

## §6 Forward + Compliance

### §6.1 Forward — Cycle 03-26 R0 必引本 Matrix

cycle 03-26 R0 dispatch plan 必引本 §3 matrix 作為 per-Tenet R1 chapter primary method assignment **source of truth**。**不允許 cycle 03-26 R0 自定 method assignment**（per MR-12 forward-only；cycle 03-25 R3 BINDING 鎖定）。

**Method tooling readiness（cycle 03-25 R4 close）**：
| Method | Status |
|---|---|
| L1 | Ready（cycle 03-X 慣例）|
| L2 | Ready（Cross-OS CI matrix 16 cells PASS）|
| L3 | Ready（cycle 03-26 R0 verify F-15 v3 self-lint passes）|
| L4 | Ready post-cycle 03-25 R3 D-§3 RATIFY（Hybrid B-primary Gemini-oauth + A-secondary Codex subset）|

**M1 + A9 + M2 triple-BINDING same R3 session ratified**：
- D-§2 M1：23/0 UNANIMOUS BINDING（本 Reference/19）
- D-§13 A9：22/1 super-majority BINDING（配對 Reference/20）
- D-§3 M2：21/2 super-majority BINDING（Hybrid B+A real provider strategy）

### §6.2 Compliance References

| Constraint | 對齊 |
|---|---|
| **MR-7** R3 ratification authority | §1 cycle 03-25 R3 D-§2 23/0 UNANIMOUS BINDING |
| **MR-11** DSS preservation verbatim | §5 DISPUTED cell minority preserved；D-§2 0 NEW DSS surfaced |
| **MR-12** Forward-only | §6.1 cycle 03-26 R0 inheritance forward-binding；過去 cycles audit verdict 鎖定 |
| **ZT-1** 違反十大宣言一律不接受 | §3 matrix 對齊 Tenet#1-10 全 |
| **ZT-2** endpoint 10/0/0★ FINAL preserved | 無動 endpoint；methodology 在 audit scope |
| **ZT-3** control-range 加嚴 only | §4.2 escalation 嚴格（no relaxation）|
| **Tenet #2 誠實** | §2 4 方法每方法 require verifiable evidence；§4.3 N/A 禁「我不知道」|
| **Tenet #8 證據>意見** | §2 evidence requirement 強制；§5 cell verdict 必有 evidence reference |
| **F-15 v3** | 本 Reference/19 自身 schema-compliant front-matter |
| **Rule #74 L1'** 5 sub-check | §2 L3 直接 invoke |
| **Rule #78 §78.5** TW parity | TW sibling 同 PR coordinator G5 sync 階段 post-Master Ratification 完成（本檔即 TW sibling）|
| **ENG-FAB v1.8 = 48 items** | §2 L3 ENG-FAB checklist 主 tooling |

---

*Reference/19 — Audit 方法論 Codification（BINDING）（中文版）*
*Master Ratification Batch 22 #1 + cycle 03-25 R3 D-§2 23/0 UNANIMOUS BINDING（cycle 03-26+ effective per MR-12）*
*4 方法（L1/L2/L3/L4）× 10 Tenets primary 適用 matrix + conjunct 全有全無 3×3 state-space + borderline escalation 4×4 + N/A discipline 40% LINNAEUS taxonomic profile review*
*Cycle 03-26 530-680-cell 矩陣 sweep cell verdict 唯一方法論依據*
*M1 + A9 + M2 triple-BINDING same R3 session*
