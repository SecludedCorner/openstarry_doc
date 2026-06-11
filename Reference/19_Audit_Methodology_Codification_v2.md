---
title: Reference/19 v2 — Audit Methodology Codification (Canonical Tenet Axis Re-anchor)
date: 2026-05-11
cycle: 03-26
authors: SCRIBE (chapter lead) + KNUTH + ARCHIMEDES + LINNAEUS
status: BINDING (RATIFIED cycle 03-26 R3 D-§A.1 23/0 UNANIMOUS 2026-05-11; Master Ratification Batch 23 #1 dispatch ready)
authority: cycle 03-26 R1 §A; substance verbatim from Reference/19 v1 §2-§5 (cycle 03-25 R3 D-§2 23/0 UNANIMOUS BINDING); §3 matrix axis labels + 性質 classification re-derived per Master directive 2026-05-11 §4.2 Option B + §4.3
supersedes: openstarry_doc/Reference/19_Audit_Methodology_Codification.md (v1; DRIFT-FLAGGED archived per Master directive 2026-05-11 §3)
cross_refs:
  - claude research/research record/cycle03-25/deliver/O1_M1_audit_methodology_final.md (v1 full deliver)
  - openstarry_doc/Reference/20_Audit_Verdict_Format_Codification.md (paired A9; v2 in §B chapter)
  - openstarry_doc/README.md § 十大核心宣言 (canonical Tenet axis authority)
  - openstarry_doc/Reference/16_R4_Folder_Convention_Discipline_v2.md
  - openstarry_doc/Reference/21_Counter_Registry.md (counter ID discipline)
mr_zt_refs: [MR-2, MR-7, MR-11, MR-12, ZT-1, ZT-2, ZT-3, Tenet #2 canonical (一切皆插件), Tenet #8 canonical (控制理論閉環模型), F-15 v3, Rule #74 L1', Rule #78 §78.5]
---

# Reference/19 v2 — Audit Methodology Codification

**Status**: **BINDING** (RATIFIED cycle 03-26 R3 D-§A.1 23/0 UNANIMOUS 2026-05-11).
**Authority**: substance verbatim from Reference/19 v1 §2-§5 (cycle 03-25 R3 BINDING); §3 mapping axis labels + 性質 classification re-derived per Master directive 2026-05-11 §4.2 Option B (R-team from-scratch derivation; coordinator does NOT propose mapping).
**Effective**: cycle 03-26 R4 close (post-Master Ratification Batch 23).

---

## §1 Background + Cycle 03-26 Matrix Sweep Prerequisite

(verbatim from v1 §1; cycle reference updated)

Per Master letter cycle 03-25 §二: cycle 03-24 R1 §1 Item A1「Tenet 真實 COMPLIANT 驗證」明文「不是 declared，不是 paper-only」, 但 audit **方法**未定。沒方法 → 矩陣 sweep 跑起來會空轉。**M1 = 方法論先定，cell verdict 才能 rigorous。**

Cycle 03-26 矩陣 sweep 規模:
- 43 plugin × 10 canonical Tenets = 430 plugin cells
- 18 functions × 10 canonical Tenets = 180 function cells
- **Total = 610 cells** (per cycle 03-26 R0 enumeration §6 + §7)

本 Reference/19 v2 + paired Reference/20 v2 (Audit Verdict Format) 配對 mandatory:
- **Reference/19 v2 says "用什麼方法"** → cell.method = L1 | L2 | L3 | L4 | conjunct
- **Reference/20 v2 says "verdict 怎麼寫"** → cell.verdict ∈ {full_pass | conditional_pass | fail | fail_fix=<scope> | not_applicable}

Audit verifiability 三原則 inheritance (cycle 03-24 R1 §1 Item A1):
1. 不是 declared
2. 不是 paper-only
3. 3rd-party reproducible

**v2 vs v1 delta**: substance §2 + §4 + §5 + §6 verbatim from v1. §3 matrix axis labels re-anchor canonical 10 Tenets (per Master directive 2026-05-11 §2.1); 性質 classification + primary/secondary method mapping **re-derived from scratch** (per Master directive 2026-05-11 §4.2 Option B; coordinator does NOT propose mapping; R-team 24-agent collective derivation).

---

## §2 4 方法 Definition Table (verbatim from v1 §2)

| Method | Definition | Primary tooling | Output format | Verifiable evidence requirement |
|---|---|---|---|---|
| **L1 程式碼閱讀** | 直接讀 plugin source + manifest + factory pattern + dependency graph，依 Tenet 條文判 cell | `Read` / `Grep` / `Glob` tools; IDE 跳轉; TURING manual review checklist | File path + line number references; quoted code excerpt (≤10 lines); verdict reasoning | Cell verdict 必引 ≥1 file path + line number; reviewer reproduces by reading same code |
| **L2 測試套件執行** | 跑 `pnpm test` / `pnpm test:purity` / 等價自動驗證；對行為 Tenet 收 PASS/FAIL evidence | pnpm runner; vitest; CI matrix (Linux + macOS + Windows × Node 20/22 = 16 cells) | Test command + flags; test result summary; failed test detail; test ID reference | Cell verdict 必引 ≥1 test ID + test command; reviewer reproduces by same command; flaky → 自動 FAIL |
| **L3 Static Analysis** | F-15 v3 governance schema lint + Rule #74 L1' 5 sub-check + ENG-FAB v1.8 checklist 自動化驗證 | `tools/f15_check.py`; F-15 v3 lint suite; ENG-FAB checklist runner; AST traversal; cross-ref scanner | Lint report (per-rule pass/fail); violation file + line + rule ID; aggregated compliance % | Cell verdict 必引 ≥1 lint output entry + tool command; lint MUST deterministic (non-flaky) |
| **L4 Runtime Trace** | provider-claude-cli / 等價 LLM 運行時觀察 + replay cache + ε-surface delta + F1/F2/F3 counter | provider-claude-cli (sixth sustained per Master directive 2026-05-08-b); Gemini-oauth (M2 primary; defer cycle 04+); Codex 25-cycle subset (M2 secondary; defer cycle 04+); audit_calc.py state-(a) | Runtime trace log (timestamped); LLM API request/response (sanitized); replay hit/miss; ε-surface delta; F1/F2/F3 counter | Cell verdict 必引 ≥1 runtime trace timestamp + log location; trace 不可 fabricate (per Tenet#2 canonical 一切皆插件 + Tenet#7 canonical 微內核與絕對純淨); LLM stochastic → 多次 sample 取趨勢 |

---

## §3 Canonical 10 Tenets × 4 方法 Matrix (R-team R0/R1 re-derived per Master directive 2026-05-11 §4.2 Option B)

**Axis source**: `openstarry_doc/README.md` § 十大核心宣言 (canonical Tenet authority per Master directive 2026-05-11 §2.1).

| Tenet # | Canonical 名稱 | 性質 | **主要適用方法** | Secondary 方法 |
|:-------:|---------------|------|----------------|---------------|
| #1 | 代理人即操作系統進程 (Agent as OS Process) | 結構性 | **L1** | L3 |
| #2 | 一切皆插件 (Everything is a Plugin) | 結構性 | **L1 + L3 conjunct** | L4 |
| #3 | 五蘊聚合架構 (Five Aggregates Architecture) | 結構性+規範 | **L1 + L3 conjunct** | L4 |
| #4 | 目錄結構即協議 (Directory as Protocol) | 結構性 | **L1 + L3 conjunct** | L2 |
| #5 | 目錄結構即權限 (Directory as Permission) | 結構性+行為 | **L1 + L3 conjunct** | L4 |
| #6 | 八識俱轉 (Concurrent Consciousness) | 行為 | **L2 + L4 conjunct** | L1 |
| #7 | 微內核與絕對純淨 (Microkernel & Absolute Purity) | 結構性 | **L1 + L3 conjunct** | L2 |
| #8 | 控制理論閉環模型 (Control-Theoretic Loop Model) | 行為 | **L2 + L4 conjunct** | L1 |
| #9 | 可插拔的記憶策略 (Pluggable Context Strategy) | 結構性+行為 | **L1 + L3 conjunct** | L4 |
| #10 | 分形社會結構 (Fractal Social Structure) | 結構性+規範 | **L1 + L3 conjunct** | L4 |

**Per-Tenet rationale digest** (full derivation in §A.3 chapter — `research record/cycle03-26/R1/§A.3_性質_方法_mapping.md`):

- **Tenet #1 結構性 L1 primary**: Agent process model = code structure (lifecycle scaffolding, PID anchor, daemon supervisor) — directly readable from source. Secondary L3 for governance-schema attestation of process-spec doc class.
- **Tenet #2 結構性 L1+L3 conjunct**: Plugin discipline = (a) every replaceable concern declared as plugin in manifest (L1 manifest + factory read) AND (b) F-15 v3 governance schema + ENG-FAB checklist enforces no built-in policy (L3 lint). Conjunct because partial credit dangerous (e.g. plugin-form-without-manifest-discipline still violates).
- **Tenet #3 結構性+規範 L1+L3 conjunct**: Five Aggregates (色受想行識) plugin classification — (a) each plugin assignable to aggregate class (L1 manifest tag + L3 LINNAEUS taxonomy lint) AND (b) Core empty-of-policy (L3 0 policy constants check) verifies 緣起性空 substrate. Secondary L4 runtime: aggregate stream interaction observable in trace.
- **Tenet #4 結構性 L1+L3 conjunct**: Directory-as-protocol = `plugins/` + `configs/` standard recognized = (a) loader reads from canonical directory tree (L1) AND (b) directory schema linted F-15 v3 (L3). Secondary L2 boot test: integration test verifies auto-load from standard directory tree.
- **Tenet #5 結構性+行為 L1+L3 conjunct**: Directory-as-permission = (a) system layer vs project layer isolation (L1 path resolution + permission checks) AND (b) lint enforces no cross-layer plugin reference (L3). Secondary L4: runtime trace verifies permission enforcement under real LLM provider.
- **Tenet #6 行為 L2+L4 conjunct**: 8 streams concurrent — (a) test exercises concurrent invocation (L2 unit + integration concurrency tests) AND (b) runtime trace shows actual co-arising under LLM provider (L4 trace). Conjunct because static check insufficient for concurrency semantics. Secondary L1: deep code review of stream-arbitration architecture.
- **Tenet #7 結構性 L1+L3 conjunct**: Microkernel & purity = (a) Core binary 0 plugin code references (L1 dependency graph + `pnpm test:purity` 0-policy invariant) AND (b) F-15 v3 ENG-FAB 0-policy constant lint (L3). Secondary L2 purity test.
- **Tenet #8 行為 L2+L4 conjunct**: Control loop = (a) feedback-driven test (L2 control-loop tests; observable error minimization) AND (b) runtime trace captures actual error-correction behavior (L4). Conjunct because feedback semantics must be both unit-testable AND emergent in real runtime. Secondary L1: control plane architecture review.
- **Tenet #9 結構性+行為 L1+L3 conjunct**: Pluggable memory strategy — (a) memory strategy plugin interface defined + replaceable (L1 IContext interface + standard plugins) AND (b) governance lint enforces strategy not hard-coded (L3). Secondary L4: runtime trace verifies actual strategy swap behavior.
- **Tenet #10 結構性+規範 L1+L3 conjunct**: Fractal social structure — (a) Agent-of-agents pattern + unified MCP interface (L1 code + mesh/multi-agent plugin) AND (b) fractal recursion schema lint (L3 mesh routing + MCP envelope discipline). Secondary L4: real multi-agent runtime trace. **High N/A expected** for non-multi-agent plugins per Reference/19 v2 §4.3 LINNAEUS taxonomy review.

**性質 → 方法分工 (R-team derived)**:
- 結構性 single (Tenet #1) → **L1 primary** (manifest + factory + lifecycle source = source of truth)
- 結構性 + governance (Tenets #2/#4/#7) → **L1 + L3 conjunct primary** (code structure + lint discipline together)
- 結構性 + 規範 (Tenets #3/#10) → **L1 + L3 conjunct primary** (architecture + governance schema)
- 結構性 + 行為 (Tenets #5/#9) → **L1 + L3 conjunct primary** (declaration + lint; L4 secondary for behavioral verification)
- 行為 (Tenets #6/#8) → **L2 + L4 conjunct primary** (test + runtime; concurrent + control behavioral semantics)

**Method coverage** verified non-vestigial: all 4 methods primary ≥1 Tenet OR conjunct ≥1 Tenet + secondary cross-coverage; matrix complete. L2 secondary-only for Tenets #4/#7; remains relevant for boot + purity tests.

**Compared to v1 §3 mapping**: substantial restructuring vs v1 (v1 used drift-list Tenet labels: 微核心 / 誠實 / 接續斷裂 / etc.). Canonical-anchored mapping shifts emphasis:
- 結構性 Tenets dominate (5 of 10 in canonical vs 2 in v1 drift-list); L1+L3 conjunct becomes dominant primary
- Behavioral Tenets reduced (2 of 10 in canonical vs 3 in v1); L2+L4 conjunct narrower scope
- Composite + Tenet#10 fractal new (canonical) — high N/A expected for non-multi-agent plugins

---

## §4 Conjunct Rule + Escalation + N/A Discipline (verbatim from v1 §4 with canonical Tenet refs)

### §4.1 Conjunct 全有全無 (3×3 State-Space Exhaustive)

For Tenets requiring conjunct primary (canonical #2/#3/#4/#5/#7/#9/#10 with L1+L3 conjunct; #6/#8 with L2+L4 conjunct):

| method_a | method_b | Cell verdict |
|---|---|---|
| PASS | PASS | **PASS** |
| PASS / FAIL | FAIL / PASS | **FAIL** (any FAIL dominates) |
| FAIL | FAIL | **FAIL** |
| PASS / BORDERLINE | BORDERLINE / PASS | escalate borderline per §4.2 |
| FAIL | BORDERLINE | **FAIL** (any FAIL dominates regardless of borderline) |
| BORDERLINE | FAIL | **FAIL** (symmetric) |
| BORDERLINE | BORDERLINE | escalate both per §4.2; if still borderline → R3 debate |

**為何 conjunct 全有全無**: structural + governance / behavioral Tenets 單一方法不足。不允許半 PASS / partial credit (避免 verdict dilution)。

### §4.2 Borderline Escalation 4×4 (verbatim from v1 §4.2)

| Primary | Borderline 表徵 | Escalation 目標 |
|---------|-----------------|----------------|
| L1 borderline | Code review 意見分歧; manifest ambiguous | → **L3** (lint codify reading) |
| L2 borderline | Test flaky; intermittent fail | → **L4** (runtime trace 確認 root cause) |
| L3 borderline | Lint warning (not error); borderline schema | → **L1** (manual code read) |
| L4 borderline | Runtime stochastic (LLM); 多次 run 結果不一 | → **L2 + L3 conjunct** (deterministic test + lint) |

**Self-loop rule**: same-method re-run NOT valid escalation; must escalate to higher rigor method per row.

**Borderline → R3 final unambiguity**: cell verdict MUST terminate in 5-token enum per Reference/20 v2 §3; "ambiguous" / "TBD" / "skipped" non-emittable.

### §4.3 N/A Discipline (verbatim from v1 §4.3 with canonical Tenet refs)

**Allowed conditions**:
- 該 canonical Tenet 對該 plugin 性質本質不適用 (e.g. single-tool provider plugin 不適用 Tenet#10 分形社會結構 — non-multi-agent context)
- 須附 N/A rationale (≥1 sentence 解釋為何不適用; cite canonical Tenet text from README.md)
- 不允許「我不知道」「reviewer 沒時間」(per Tenet#2 canonical 一切皆插件 + 誠實 spirit inherited from cycle 03-24 endpoint chain)

**40% threshold (LINNAEUS taxonomic profile review trigger; NOT auto-fail)**:
- > 40% N/A per plugin → 觸發 LINNAEUS taxonomic profile review
- Plugin classes with structurally-justified-many-N/A (lightweight provider plugin / shim plugin / single-aggregate plugin) may exceed 40% **iff LINNAEUS profile substantiates non-applicability per Tenet-by-Tenet rationale**
- Lexical scrubbing for forbidden phrases ("不知道", "我不確定", "沒時間") mechanically auditable post-hoc
- **Canonical Tenet #10 特殊**: many plugins legitimately N/A (non-multi-agent context). LINNAEUS taxonomy profile expected to admit 50-70% N/A for Tenet #10 column without auto-fail trigger.

---

## §5 Cell Verdict 流程 (verbatim from v1 §5)

```
For each cell (plugin/function, canonical Tenet):
  1. Lookup §3 matrix → 取主要適用方法 (single OR conjunct)
  2. Run primary method → collect evidence:
     - L1: read source + manifest → file:line cite
     - L2: run test → test ID + result
     - L3: run lint → tool output entry
     - L4: run runtime trace → log timestamp + location
  3. If conjunct (per §3 mapping):
     必兩方法 PASS 才 cell PASS (per §4.1 3×3 state-space)
  4. If primary borderline: escalate per §4.2 (or self-loop higher-rigor rule)
  5. Verdict = full_pass | conditional_pass | fail | fail_fix=<scope> | not_applicable
     (per paired Reference/20 v2 format)
  6. Evidence reference required field 填齊 per Reference/20 v2 §3 YAML schema
  7. If cell N/A: 必填 N/A rationale (per §4.3) + LINNAEUS taxonomy reference
  8. Emit cell record per Reference/20 v2 YAML-primary schema
```

**Cell PASS threshold per method** (verbatim from v1):
- L1 PASS: Code 結構符合 canonical Tenet 條文 (README.md authority); reviewer manual review 0 violation
- L2 PASS: Test 100% PASS (所有相關 test); non-flaky
- L3 PASS: Lint 0 violation OR all violations have documented exemption (per Phase 6 strict 7-list anchor / F-15 v3 documented escape)
- L4 PASS: Runtime trace 符合預期 schema; F1/F2/F3 counter 0 (per 6-cycle Popperian target cycle 03-26); ε-surface delta within tolerance Δ=0 PRESERVED

**Disagreement DISPUTED → R3 vote 終裁**: two reviewers 對同 cell verdict 意見分歧 → cell verdict = "DISPUTED" → R3 主 session 24-agent debate → R3 vote 終裁 (per MR-7) → minority preserved per MR-11.

**Text-only L4 mode caveat (cycle 03-26 specific)**: per Master directive 2026-05-08-b, W2-R36 W-round 用 provider-claude-cli sixth sustained (text-only LLM artefact mode). L4 evidence for behavioral Tenets (#6/#8) accepts text-only sample with explicit `text-only-mode` caveat in Reference/20 v2 caveat field; real-LLM L4 deferred cycle 04+ Hybrid B+A per cycle 03-25 R3 D-§3 BINDING.

---

## §6 Forward + Compliance (verbatim from v1 §6 with canonical Tenet refs)

### §6.1 Forward — Cycle 03-26 R1 必引本 Matrix

cycle 03-26 R1 chapter §B.1~§B.10 必引本 §3 matrix 作為 per-Tenet primary method assignment **source of truth**. **不允許 R1 chapter 自定 method assignment** (per MR-12 forward-only; once R3 D-§A.1 + D-§A.3 BINDING, locked).

**Method tooling readiness (cycle 03-26 R0 close)**:
| Method | Status |
|---|---|
| L1 | Ready (cycle 03-X 慣例) |
| L2 | Ready (Cross-OS CI matrix 16 cells PASS sustained) |
| L3 | Ready (F-15 v3 self-lint extended to verify Reference/19 v2 + Reference/20 v2 drafts) |
| L4 | Ready (provider-claude-cli sixth sustained per Master directive 2026-05-08-b; text-only mode; real-LLM defer cycle 04+) |

### §6.2 Compliance References

| Constraint | 對齊 |
|---|---|
| **MR-2 (Tenet wording unchanged)** | §3 canonical axis re-anchor; no Tenet text edits; only audit-doc-side label/性質/method alignment |
| **MR-5 (Tenet #10 status; Phase 6 hard cap)** | Tenet #10 COMPLIANT effective cycle 03-24; this v2 carry-forward only |
| **MR-7** R3 ratification authority | §1 cycle 03-26 R3 D-§A.1 + D-§A.3 BINDING ratify target |
| **MR-11** DSS preservation verbatim | §5 DISPUTED cell minority preserved; v2 derivation may surface new DSS-CY26 entries |
| **MR-12** Forward-only | §6.1 cycle 03-26 R1+ inheritance forward-binding; v1 DRIFT-FLAGGED archived; cycle 03-25 audit verdict locked |
| **ZT-1** 違反十大宣言一律不接受 | §3 matrix re-anchored to canonical Tenet#1-10 (README.md authority) |
| **ZT-2** endpoint 10/0/0★ FINAL preserved | 無動 endpoint; methodology re-anchor in audit scope only |
| **ZT-3** control-range 加嚴 only | §4.2 escalation 嚴格 (no relaxation); canonical axis re-anchor = tightening not loosening |
| **Tenet #2 canonical 一切皆插件** | §3 L1+L3 conjunct primary anchor |
| **Tenet #8 canonical 控制理論閉環模型** | §3 L2+L4 conjunct primary anchor; control-loop semantics evidence-base |
| **F-15 v3** | 本 Reference/19 v2 自身 schema-compliant front-matter; self-lint PASS expected pre-R3 |
| **Rule #74 L1'** 5 sub-check | §2 L3 直接 invoke (carry-forward from v1) |
| **Rule #78 §78.5** TW parity | TW sibling deferred per same-PR coordinator G5 sync stage post-R3 ratify |
| **ENG-FAB v1.8 = 48 items** | §2 L3 ENG-FAB checklist 主 tooling (carry-forward) |
| **Reference/21 Counter Registry** | Counter ID discipline applied: C-S4 F1/F2/F3 Popperian counter; C-S5 9-datapoint pool; C-C5 SPC re-cal application |

### §6.3 Reference/19 v1 → v2 Transition

- v1 status: **DRIFT-FLAGGED archived** per Master directive 2026-05-11 §3
- v1 audit reports cycle 03-13~25 NOT retroactively re-verdicted (per MR-12 forward-only)
- v2 effective cycle 03-26 R4 close post-R3 D-§A.1 BINDING ratify
- TW sibling `Reference/19.tw.md` v2 produced at coordinator G5 sync stage per Rule #78 §78.5

---

## §7 Findings (R1 chapter §A)

| ID | Finding | Severity |
|----|---------|----------|
| F-CY26-§A-R1-01 | 性質 classification 結構性 dominates canonical axis (5 of 10) vs v1 drift-list (2 of 10) | INFO (architectural shift) |
| F-CY26-§A-R1-02 | L1+L3 conjunct emerges as dominant primary (7 of 10 Tenets) — reflects canonical's structural emphasis | INFO |
| F-CY26-§A-R1-03 | Behavioral Tenet count reduced (2 of 10 vs 3 of 10 v1); L2+L4 conjunct narrower scope | INFO |
| F-CY26-§A-R1-04 | Tenet #10 fractal social structure expected high N/A (50-70%) per LINNAEUS taxonomy; cell-level audit must NOT auto-fail | MED (N/A discipline gating) |
| F-CY26-§A-R1-05 | Behavioral Tenet #6 #8 L4 evidence text-only-mode caveat MUST annotate (provider-claude-cli sixth sustained per Master directive 2026-05-08-b) | MED |
| F-CY26-§A-R1-06 | Tenet #3 五蘊聚合架構 plugin aggregate classification needs LINNAEUS taxonomy table (色受想行識 mapping) — defer chapter §B.3 R1 detail | INFO |
| F-CY26-§A-R1-07 | Tenet #4 + Tenet #5 結構性 differ on protocol vs permission boundary — separate cell verdict required | INFO |
| F-CY26-§A-R1-08 | F-15 v3 self-lint MUST validate Reference/19 v2 + Reference/20 v2 drafts pre-R3 dispatch | LOW |

**Total: 8 findings** (target 8-15; within band).

---

*Reference/19 v2 — Audit Methodology Codification (DRAFT R1; cycle 03-26)*
*Substance verbatim from v1 §2/§4/§5/§6; §3 mapping R-team re-derived per Master directive 2026-05-11 §4.2 Option B*
*Canonical Tenet axis re-anchor; v1 DRIFT-FLAGGED archived per Master directive 2026-05-11 §3*
*4 方法 (L1/L2/L3/L4) × 10 canonical Tenets primary 適用 matrix + conjunct 全有全無 3×3 state-space + borderline escalation 4×4 + N/A discipline (Tenet #10 LINNAEUS profile 50-70% N/A expected)*
*Cycle 03-26 R3 D-§A.1 BINDING ratify target; D-§A.3 10 sub-vote 性質+方法 mapping target*
