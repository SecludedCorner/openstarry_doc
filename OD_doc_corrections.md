# Cycle 02-10 文件修正清單（OD Corrections）

**來源**：R1 SCRIBE (#02) 文件漂移盤點 + D4 辯論 Track A 決議
**狀態**：（**HISTORICAL — cycle 02-10**）供工程團隊於 Plan34 DOC SOP 執行；已於 cycle 03-29 H-2 完成 README 重編號（見下方 §Historical Note）。

> **Historical Note (cycle 03-29 H-4 fix, 2026-05-13)**: 本文件記錄的 Doc 52/53 編號方案為 cycle 02-10 當時的 README 編號狀態。canonical Architecture_Documentation 後續經歷多輪 R3 ratify renumber（cycle 02-11 ~ cycle 03-2 加入 Doc 54-58；後續更名/重編號）。現行 (cycle 03-29 後) canonical 真實檔名：`50_Microkernel_Purification_Rationale.md`、`51_Codex_Review_Response.md`、`52_Alaya_Partial_Mapping.md`、`53_Multi_Agent_Communication_Interface_Spec.md`。本文件保留為**歷史證跡**，不再作為 README 編號權威；現行權威見 `README.md` §文檔導航地圖 §1 Architecture Documentation。

---

## P0 — 必須立即修正

### OD-3：openstarry_doc/README.md 導航補齊（**HISTORICAL — superseded by cycle 03-29 H-2**）

**問題**：Architecture Documentation 區段僅列 Doc 00-51，**缺少 Doc 52-53**（Cycle 02-9 產出）。

**修正**（cycle 02-10 prescription — OLD numbering scheme）：

在 Architecture Documentation 導航區段末尾新增：

```markdown
# NOTE (cycle 03-29 H-4): 以下兩條為 cycle 02-10 OLD numbering；
# 現行 canonical 真實檔名為 50_Microkernel_Purification_Rationale.md
# 和 53_Multi_Agent_Communication_Interface_Spec.md（不再是 52_/53_ 此處引用）。
- [52 - Microkernel Purification Rationale](./Architecture_Documentation/52_Microkernel_Purification_Rationale.md)
- [53 - Security Analysis](./Architecture_Documentation/53_Security_Analysis.md)
```

在 Implementation Plans 區段新增 Plan33 條目：

```markdown
- [Plan33 - DX Wrap-up](./Implementation_Plans/Plan33_DX_Wrapup.md) — v0.33.0-alpha
```

**估算**：~10 行

---

## P1a — 必須修正（Plan34 DOC SOP）

### OD-4：Implementation Plans 缺 Plan33

**問題**：Implementation Plans 區段列至 Plan32，缺 Plan33。

**修正**：新增 Plan33 條目（見 OD-3 合併處理）。

**估算**：~2 行

### OD-1：Tenet #7 註記時態修正

**問題**：openstarry_doc/README.md 中 Tenet #7 註記使用未來式：「Plan32...將達成本宣言完全合規」

**修正**：改為過去式：「Plan32-33 已達成本宣言完全合規（REM-7.1 於 v0.33.0-alpha 清除）」

**估算**：~1 行

### OD-2：Tenet #9 註記時態修正

**問題**：openstarry_doc/README.md 中 Tenet #9 註記使用未來式：「Plan32 Wave 6 將 context manager 從 Core 提取」

**修正**：改為過去式：「Plan32 已於 Wave 6 將 context manager 從 Core 提取為 @openstarry-plugin/context-sliding-window」

**估算**：~1 行

---

## P1b — 語言精確性修正（Plan34 DOC SOP）

### IL-2：Iteration_Log 缺 Hotfix 記錄

**問題**：`Iteration_Log_Cycle02.md` Plan33 區段**缺少 Hotfix SOP 記錄**。三項 Hotfix（SEC-033-001, SEC-033-002, skandha）記載於 engineering `lessons_learned.md` 但未回寫 Iteration_Log。

**修正**：在 Plan33 Standard SOP 區段下新增：

```markdown
### Hotfix SOP (Post-Standard)

- **SEC-033-001**: config-migrate.ts redaction regex — pattern 覆蓋 secret|token|password|key|auth|credential (Medium)
- **SEC-033-002**: plugin-loader.ts dependencies array cap = 50 (Low)
- **Skandha validation**: standard-core-commands + standard-model-selector 移除錯誤 `skandha: 'samskara'` 宣告
```

**估算**：~6 行
**PROC-DOC-5 驗證**：此修正正是 PROC-DOC-5（Hotfix 回寫規則）的首次執行案例。

### R-2：文件連結有效性驗證

**問題**：`openstarry/README.md` 中 5 個指向 `docs/EN/` 的連結**未確認可存取**。

**行動**：工程團隊於 Plan34 DOC SOP 執行時驗證所有 README 連結。

**估算**：~5 分鐘驗證任務

---

## P2 — 可延遲至 Plan35

### R-1：Architecture 表更新

**問題**：README 的 Five Aggregates mapping 表未含 Plan33 新功能（ITool.metadata, skandha-check）。

### OD-5：Plugin 命名驗證

**問題**：openstarry_doc/README.md config 範例中引用 `guide-pain-mechanism` plugin 名稱，需驗證是否仍為現行名稱。

### OD-6：Config 範例更新

**問題**：config 範例引用 `policy.safety` 與 `max_consecutive_errors: 3`，需更新至 Plan32/33 外化後的正確欄位名稱。

---

## 修正追蹤表

| ID | 優先級 | 文件 | 類型 | 狀態 |
|----|--------|------|------|------|
| OD-3 | P0 | openstarry_doc/README.md | 導航補齊 | 待工程執行 |
| OD-4 | P1a | openstarry_doc/README.md | Plan33 條目 | 待工程執行 |
| OD-1 | P1a | openstarry_doc/README.md | 時態修正 | 待工程執行 |
| OD-2 | P1a | openstarry_doc/README.md | 時態修正 | 待工程執行 |
| IL-2 | P1b | Iteration_Log_Cycle02.md | Hotfix 回寫 | 待工程執行 |
| R-2 | P1b | openstarry/README.md | 連結驗證 | 待工程執行 |
| R-1 | P2 | openstarry/README.md | 表格更新 | 可延遲 |
| OD-5 | P2 | openstarry_doc/README.md | 命名驗證 | 可延遲 |
| OD-6 | P2 | openstarry_doc/README.md | 範例更新 | 可延遲 |

---

*Cycle 02-10 文件修正清單*
*SCRIBE (#02) 盤點；KNUTH (#21) 流程規則映射*
*2026-03-13*
