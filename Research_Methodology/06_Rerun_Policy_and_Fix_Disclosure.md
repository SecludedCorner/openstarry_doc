# 06. Re-Run Policy and Fix Disclosure Framework

`[Cycle 03-9 新增]`

> **來源**: Cycle 03-9 R3 Debate D1 + D3
> **核心學者**: KERNEL (#10), BABBAGE (#9), NAGARJUNA (#7), ASANGA (#8)
> **相關規則**: Rules #64, #65, #66, #67
> **相關文件**: 70_Rules_64_to_68_Rerun_Policy_Fix_Disclosure.md

---

## 1. 概述

Cycle 03-9 Research 團隊建立了**正式的重跑政策與 fix 揭露框架**，回應 W2-R9 的 3 次重跑揭露的治理缺口。本文件記錄政策的研究方法論依據。

核心規則：
- **Rule #64**: 重跑條件（基礎設施或系統性失敗）
- **Rule #65**: Fix-between-reruns 規範
- **Rule #66**: 強制揭露
- **Rule #67**: Fix 揭露為正面信號

---

## 2. 問題背景：W2-R9 的 3 次重跑

### 時序

| Run | 結果 | 根因 | Fix 應用 |
|-----|------|------|---------|
| 1 | FAIL | API 429 + plugin not found | Fix 12b（+9 deps） |
| 2 | FAIL | shadow=0（StateTracker 不持久） | Fix 12c（serialize/fromSnapshot） |
| 3 | FAIL | shadow=0（config 未傳入） | Fix 12d（factory(ref.config)） |
| 4 | PASS | — | — |

3 次重跑的執行過程中，研究團隊實時觀察到：
- 無規則說明「何時可重跑」
- 無規則說明「重跑間 fix 是否合規」
- 無規則說明「如何記錄這些重跑」
- 無規則說明「如何從反造假視角評估 4 次嘗試」

### 識別的治理缺口

這不是單次事件，而是**系統性方法論缺口**。若不制度化，未來 cycle 可能：
- 濫用重跑至「試到對」
- 未揭露的失敗 runs 構成隱性造假
- Fix-between-reruns 演變為 scope creep
- 多次嘗試的 survivorship bias 污染統計

---

## 3. 研究方法論原則

### 原則 1: 觀測獨立性（Tenet #8）

控制理論要求觀測程序**先定義後執行**。若重跑政策「按需調整」，觀測結果喪失獨立性。

### 原則 2: 透明化即反造假（Rule #58 延伸）

Rule #58 的 H0 = fabrication 框架預設「有證據才 REJECT」。Omission（省略不報）等同於「無證據」。

結論：**未揭露的重跑 = 造假**（Rule #66）。

### 原則 3: 補回是德行（MR-10 + MR-12）

MR-10 要求錯誤必須補回。補回的**正當方式**：
- 根因分析
- 明確分類（Rule #62）
- 揭露過程

這一組合**反轉**了「post-delivery fix = 負面」的預設。

### 原則 4: 雙層 H0 考量（NAGARJUNA + ASANGA 辯論）

#### NAGARJUNA（中觀）視角

- 兩條獨立路徑得出相同結論 = 強證據（pramāṇa）
- Path A（研究）+ Path B（測試）對 NC4 皆 PASS
- 無「phantom agreement」（僅因共享方法論而看似一致）

#### ASANGA（唯識）視角

- Fix 揭露 = 「造假外觀轉化為非造假證明」（pratibhāsa）
- 誠實揭露是最強的 H0 REJECTION 信號
- Phase 3 shadow 成為真實（非幻影）解決所有先前 cycle 的 fabrication 疑慮

兩觀點**收斂**支持 Rule #67。

---

## 4. 規則框架細節

見 `Architecture_Documentation/70_Rules_64_to_68_Rerun_Policy_Fix_Disclosure.md` 的完整規則文字。本節僅記錄研究方法論依據。

### 為什麼 4 次嘗試是上限？

- 1 次初次 = 基礎執行
- 3 次重跑 = 容許 3 個獨立系統性問題（與 W2-R9 的 3 個 fix 對應）
- 若 4 次皆失敗：問題超出局部修復範圍，升級 Master

### 為什麼允許 fix-between-reruns？

若禁止，則任何 run 中發現的 bug 必須等到下個 cycle 修復。這與 MR-8（品質第一）矛盾。

允許但強制 **Rule #62 分類 + scope 受限** 是妥協：快速補回 + 防止 scope creep。

### 為什麼 disclosure 不是可選？

若可選，測試團隊面臨「報告漂亮 vs 報告完整」的誘因衝突。強制揭露消除此誘因。

---

## 5. Plan44 case study: 4 Fixes 的反造假意義

### 4 Fixes 全景

| Fix | 揭露 | 根因 | Tier | L1-L4 |
|-----|:----:|:----:|:----:|:-----:|
| 12a | delivery_report §12a | 文件未同步 | N/A（doc） | PASS |
| 12b | delivery_report §12b | pnpm strict mode | Tier 1 | PASS |
| 12c | delivery_report §12c | 每 cycle 重建 core | Tier 1 | PASS |
| 12d | delivery_report §12d | 24 版本空窗 | Tier 1 | PASS |

### 評估

- 全數揭露 ✅
- 全數有根因 ✅
- 全數符合 Rule #62 分類 ✅
- 全數 L1-L4 clear ✅

### Rule #67 應用

4 個 fix 的**存在**（而非其**數量**）是 H0 REJECTION 的強證據：
- 揭露顯示透明
- 根因顯示追溯能力
- L1-L4 clear 顯示修復品質

這是 Plan42/43/44 連續 3 個 0F Plan 的深層含義：**0 fabrication 不僅指「無幻影 code」，也指「錯誤被誠實對待」**。

---

## 6. 對 DEV-1b 的影響

DEV-1b 追蹤 fabrication pattern evolution（Plan36 起 OPEN）。

NC4 = dual verification，要求兩條獨立路徑確認無 fabrication。

### 研究團隊的裁決邏輯

1. Path A（TURING + GUARDIAN）：41P/2D/0F + 4 fixes L1-L4 clear + CLEAN retroactive
2. Path B（BABBAGE + ARCHIMEDES）：GO 50/50 + audit integrity + ENG-FAB 39/39
3. 兩條路徑**獨立得出** PASS
4. 4 fixes 的存在 per Rule #67 **加強** H0 REJECTION

### 推薦 CLOSE

Research 團隊 R3 D3-Q18 決議：**推薦 CLOSE**（待 Master 於 03-10 ratification）。

---

## 7. 對未來 Cycle 的影響

### 正式制度化

- W2-R10 起：Rules #64-66 正式生效
- 測試團隊的 todo_test_instructions.md 已更新
- ENG-FAB v1.4 新增 F-7 項

### Fabrication 追蹤的演化

歷史上 fabrication 追蹤關注「幻影 code」。Plan44 之後，追蹤範圍擴展至：
- Omission（未揭露）
- Configuration-layer（Gen4）
- Process（重跑未揭露）

每一類都有對應規則支持：
- Code fabrication → Rule #58 L1-L4
- Configuration fabrication → Rule #63
- Omission → Rule #66
- 雙路徑盲區 → Rule #68

---

## 8. 方法論反思

### 成熟度提升

03-9 的 R3 debates（42 unanimous 決策）體現了研究團隊的方法論成熟度：
- 從單純「找 bug」演化為「建立治理」
- 從「事後辯論」演化為「實時觀察 + 制度化」

### MR-11 的體現

MR-11（十大宣言是一種抉擇，每輪以此為基調）在 03-9 的體現：
- 透明度是抉擇（Rule #66, #67）
- 完整性是抉擇（Rule #68）
- 觀測獨立性是抉擇（Rule #64, #65）

每條新規則都是**選擇對齊**的產物。

---

## 9. Open Questions for Future Cycles

1. **Rule #66 的邊界**：若 re-run 本身合規但 fix 有問題，如何分類？
2. **Rule #67 的邊界**：若 fix 揭露但 Tier 分類錯誤（應為 Tier 2 卻報為 Tier 1），是正面還是負面信號？
3. **Rule #68 的擴展**：未來可能有 Path C（例如 service registry-based config）？是否需進一步擴展？

這些問題留待後續 cycle 基於實證決議。

---

*Research Methodology #06 — Re-Run Policy and Fix Disclosure*
*Cycle 03-9 R3 D1 + D3, 2026-04-15*
