# Cycle 03-1 合規影響分析

**日期**: 2026-03-24
**分析者**: SYNTHESIST (#1), BABBAGE (#9)
**審批**: SUNYATA (#0)
**基準版本**: v0.36.1-alpha
**決議來源**: D1 (5), D2 (10), D3 (3), D4 (4) = 22 decisions total

---

## 當前狀態 (v0.36.1-alpha): 8/1/1

| 類別 | 數量 | 宣言 |
|------|------|------|
| COMPLIANT | 8 | #1, #2, #3, #4, #5, #7, #8, #9 |
| CONDITIONAL | 1 | #6 (Eight Consciousnesses) |
| NON-COMPLIANT | 1 | #10 (Fractal Social Structure) |

---

## Per-Tenet Impact

### Tenet #1 (Agent as OS Process): COMPLIANT — no change

**當前狀態**: COMPLIANT。每個 Agent 為獨立 OS process（child_process.spawn）。

**Cycle 03-1 影響**: 無變化。
- D2-R4（零 Core 修改）明確保護此宣言：Core 維持 per-Agent EventBus、per-Agent StateManager、per-Agent ExecutionLoop，不因多代理而改變。
- D2-R8（五層故障隔離）L1 = Process Isolation（既有），確認進程邊界為基礎安全層。
- D3-R1（八識分配）前七識 per-agent = per-process 認知獨立。

**相關決議**: D2-R4, D2-R8
**風險**: 無。

---

### Tenet #2 (Everything Plugin): COMPLIANT — strengthened (ICommChannel as plugin)

**當前狀態**: COMPLIANT。所有功能透過 plugin 介面提供。

**Cycle 03-1 影響**: 強化。
- D2-R2: ICommChannel 定義為 `PluginHooks.commChannels?: ICommChannel[]`（array slot），與 vedanaSensors[]、tools[]、listeners[] 平行。通訊 channel 為 plugin，非 Core 元件。
- D2-R3: Claude Code Channels 若未來整合，亦為 plugin（plugin-transport-channels）。
- D2-R7: EventBridge 為 Daemon 服務，Agent 端透過 comm plugin 參與。
- D3-R2: AC-6 typed IListener 為 SDK 型別擴充，plugin 宣告 senseType。
- PENROSE 撤回 GlobalManoAggregator 為 Core-level 提案，改為 future plugin spec。

**相關決議**: D2-R2, D2-R3, D2-R7, D3-R2
**風險**: 無。Plugin pattern 一致性進一步加強。

---

### Tenet #3 (Five Aggregates): COMPLIANT — no change

**當前狀態**: COMPLIANT。五蘊（色受想行識）映射為 Agent 五要素。

**Cycle 03-1 影響**: 無變化。
- D3-R1 明確保留 per-agent 五蘊完整性：每個 Agent 為完整認知單元，擁有全部五蘊。
- 多代理環境中五蘊不跨 Agent 共享（阿賴耶共享為第八識層，非五蘊層）。

**相關決議**: D3-R1
**風險**: 無。

---

### Tenet #4 (Directory as Protocol): COMPLIANT — no change

**當前狀態**: COMPLIANT。目錄結構定義協定。

**Cycle 03-1 影響**: 無直接影響。
- 多代理通訊不改變目錄協定。ICommChannel 為 SDK 型別，不影響目錄結構。
- agent.json 新增 `communication` 區段（D2-R5）遵循既有 JSON 設定模式，不改變目錄即協定的原則。

**相關決議**: D2-R5
**風險**: 無。

---

### Tenet #5 (Directory as Permission): COMPLIANT — strengthened (capability model extends to multi-agent)

**當前狀態**: COMPLIANT。allowedPaths 定義 Agent 權限邊界。

**Cycle 03-1 影響**: 強化。
- D2-R5: seL4 啟發式 capability model 將權限原則從目錄存取延伸至多代理通訊。`canSendTo`、`canReceiveFrom`、`exposedTools` 在 agent.json 中宣告，遵循 Tenet #5「明確宣告、預設拒絕」原則。
- D2-R5: 預設零 capability — 無 `communication` 區段的 Agent 無法收發多代理訊息。與 Tenet #5 預設無權限一致。
- D3-R3: ICompositeAgent 權限格 `child.allowedPaths ⊆ parent.allowedPaths`（mechanism, spawn 時強制）。子 Agent 永遠不能超越父 Agent 權限。
- D2-R5: `C_child.canSendTo ⊆ C_parent.canSendTo` AND `C_child.exposedTools ⊆ C_parent.exposedTools`。通訊 capability 同樣遵守向下收窄原則。

**相關決議**: D2-R5, D3-R3
**風險**: 無。權限模型自然延伸，無設計衝突。

---

### Tenet #6 (Eight Consciousnesses): CONDITIONAL -> advancing (AC-6 in Plan37, AC-7 in Plan38+)

**當前狀態**: CONDITIONAL。兩個活躍合規項：
- AC-6: 前五識缺乏型別區分（全為 generic IListener）
- AC-7: 阿賴耶識分散式架構（Phase 6+ 範疇）

**Cycle 03-1 影響**: 朝升級推進。
- D3-R1: 八識多代理分配混合模型確立。前七識 per-agent，第八識共享為世俗諦慣例指定。二諦宣言 REQUIRED。此為 AC-7 的設計基礎。
- D3-R2: AC-6 排入 Plan37 W3（~150-200 LOC）。定義 5 個 typed IListener 子介面（IVisualListener, IAuditoryListener, IOlfactoryListener, IGustatoryListener, ITactileListener），senseType 辨別欄位。Union type 向後相容。
- D3-R2: AC-7 排入 Plan38+（~800+ LOC）。Dependencies: Process Tree + ICommChannel。需 GUARDIAN 安全審查。

**Post-Plan37 預測**: AC-6 解決後，CONDITIONAL 狀態維持（AC-7 尚未完成）。但 CONDITIONAL 的理由從「兩個 deviation」縮減為「一個 deviation（AC-7 only）」。

**Post-Plan38 預測**: 若 AC-7 設計完成且初步實作，可能升級為 COMPLIANT 或接近 COMPLIANT。

**相關決議**: D3-R1, D3-R2, D4-R1 (W3)
**風險**: AC-7 實作複雜度高（分散式狀態、一致性模型、安全攻擊面）。需謹慎設計，不可急於升級。

---

### Tenet #7 (Microkernel Purity): COMPLIANT — strengthened (D2-R4: zero Core modification)

**當前狀態**: COMPLIANT。Core 不含政策常數，MR-6 強制。

**Cycle 03-1 影響**: 強化。
- D2-R4 為 Phase 6 最重要的 Tenet #7 保證：**Core 新增零 IPC 原語，EventBus 不變，所有多代理通訊透過 Daemon + Plugin**。
- TANENBAUM 撤回原始 R3 建議（Core EventBus 加 sendTo/onRemote），確認 Plugin-level wrappers over ICommChannel 可達成相同效果而不侵入 Core。
- D2-R2: ICommChannel 為 SDK 型別（非 Core code）。
- D2-R7: EventBridge + L2 ServiceRegistry 為 Daemon 服務，非 Core。
- D1-R1: BUG-3 修復中，worst-risk RISK_ORDER 映射位於 gear-arbiter-static（plugin），非 Core。per-tool event emission 使用既有 EventBus 基礎設施。
- D1-R1: 修復未引入新政策常數至 Core（遵守 MR-6）。

**相關決議**: D2-R4, D2-R2, D2-R7, D1-R1
**風險**: 無。D2-R4 的明確決議為後續 Phase 6 開發提供清晰邊界。

---

### Tenet #8 (Control Theory): COMPLIANT with remediation (BUG-3 fix in Plan37)

**當前狀態**: COMPLIANT（v0.36.1-alpha 評估）。

**Cycle 03-1 影響**: COMPLIANT with remediation。
- D1-R5: BUG-3 歸類為實作缺陷（plugin logic + architecture wiring），非設計層級控制缺口。控制迴路**設計**正確 — ThresholdAuditor 定義四類 delta 規則，destructive delta <= 0 安全約束存在於 ManoAggregator。
- BUG-3 是**接線缺陷**（first-match riskCategory instead of worst-risk），類比恆溫器邏輯正確但溫度感測器故障。
- 維持 COMPLIANT 的三個條件：
  1. 修復必須在 Plan37 交付（若延後則重新評估為 CONDITIONAL）
  2. W2-R1 必須確認 4/4 category coverage（SC-3b）
  3. 修復後 sigma > 0
- sigma=0.000245（W2 v2）證明迴路在功能類別上存活。WIENER 預測修復後 sigma ~0.012。
- D4-R4: W2-R1 協定加入 SC-5（category representation）和 SC-6（bidirectional delta），直接驗證修復效果。

**Post-Plan37 預測**: 若 W2-R1 全 SC pass，Tenet #8 恢復為不帶 "with remediation" 修飾的完整 COMPLIANT。

**相關決議**: D1-R1, D1-R2, D1-R5, D4-R4
**風險**: 中。修復本身低風險（~45 LOC），但 W2-R1 結果尚待確認。若 SC-3 或 SC-6 在修復後仍 FAIL，需進一步調查。

---

### Tenet #9 (Pluggable Memory): COMPLIANT — no change

**當前狀態**: COMPLIANT。記憶元件皆為 plugin。

**Cycle 03-1 影響**: 無直接變化。
- D3-R1 共享阿賴耶模型為記憶擴展提供方向（種子庫跨 Agent），但實作延後至 AC-7（Plan38+）。
- D2-R7 L2 Registry 與 Blackboard 分離，Blackboard 設計延後至 Plan39+。

**相關決議**: D3-R1, D2-R7
**風險**: 無。

---

### Tenet #10 (Fractal Social): NON-COMPLIANT -> advancing (ICompositeAgent + Process Tree in Plan37)

**當前狀態**: NON-COMPLIANT。v0.36.1-alpha 無子代理合成（zero sub-agent composition）。

**Cycle 03-1 影響**: 朝 COMPLIANT 推進。
- D3-R3: ICompositeAgent 遞迴介面正式採納。定義 `children`、`spawn()`、`terminate()`、`broadcast()`、`aggregate()` 方法。
- D3-R3: 權限格 `child.allowedPaths ⊆ parent.allowedPaths`（mechanism, spawn 時強制）。
- D3-R3: 資源預算 `reserve_ratio` = policy（SDK default 0.3）。Fractal depth limit = 3（policy）。
- D3-R3: Backward-compatible bisimulation — 單子代 CompositeAgent 弱 bisimilar 於獨立 Agent。
- D4-R1: Plan37 W2 包含 Process Tree（parentAgentId, spawnChildAgent, permission lattice validation）~300 LOC。
- D2-R1: Pipeline 通訊為 CompositeAgent 子代間的第一種協調模式。
- D2-R10: 研究團隊四組結構作為 fractal 社會結構的案例研究。

**Post-Plan37 預測**: 若 W2 Process Tree + ICompositeAgent 成功交付，Tenet #10 從 NON-COMPLIANT 升級為 CONDITIONAL（基礎設施就位但尚未完整實現所有 fractal patterns）。

**Post-Plan38 預測**: 加入 Hub-and-Spoke + Supervisor Strategy 後，可能升級為 COMPLIANT。

**相關決議**: D3-R3, D4-R1, D2-R1, D2-R5, D2-R8, D2-R10
**風險**: 中。Process Tree 是全新架構元件，需確保 BABBAGE continuity test 通過（零子代 bisimulation）。權限格實作需與既有 SecurityLayer 正確整合。

---

## Post-Plan37 Prediction: 8/1/1 (deepening, no status change)

| Tenet | Pre-Plan37 | Post-Plan37 | 變化 |
|-------|-----------|-------------|------|
| #1 | COMPLIANT | COMPLIANT | 無變化 |
| #2 | COMPLIANT | COMPLIANT | 強化（ICommChannel plugin） |
| #3 | COMPLIANT | COMPLIANT | 無變化 |
| #4 | COMPLIANT | COMPLIANT | 無變化 |
| #5 | COMPLIANT | COMPLIANT | 強化（capability model） |
| #6 | CONDITIONAL | CONDITIONAL | 推進（AC-6 完成，AC-7 未完成） |
| #7 | COMPLIANT | COMPLIANT | 強化（D2-R4 零 Core 保證） |
| #8 | COMPLIANT w/ remediation | COMPLIANT | 修復完成 + W2-R1 驗證 |
| #9 | COMPLIANT | COMPLIANT | 無變化 |
| #10 | NON-COMPLIANT | NON-COMPLIANT* | 推進（Process Tree 就位，但尚未完整 fractal） |

*Note: #10 可能在 Plan37 完成後即升級為 CONDITIONAL，取決於 Process Tree + ICompositeAgent 的完整度評估。

**計數**: 8 COMPLIANT / 1 CONDITIONAL / 1 NON-COMPLIANT（數字不變，但 #8 移除 remediation 修飾，#2/#5/#7 深化）。

---

## Post-Plan38 Prediction: 9/1/0 or 9/0/1 (potential #6 upgrade)

**情境 A (9/1/0)**: AC-7 設計完成 + 初步實作 -> #6 升級 COMPLIANT; #10 CONDITIONAL（Hub-and-Spoke 就位但 Mesh 未完成）。

**情境 B (9/0/1)**: #6 維持 CONDITIONAL（AC-7 複雜度高，可能需更多時間）; #10 升級 COMPLIANT（Hub-and-Spoke + Supervisor Strategy 完成 fractal patterns）。

兩種情境皆為 **9 non-violating / 1 remaining**。

---

## Cycle 03 Target: 10/0/0

**路徑**:
1. **Plan37**: BUG-3 fix + Pipeline + Process Tree + AC-6 -> 8/1/1 (deepened)
2. **Plan38**: Hub-and-Spoke + AC-7 design + Supervisor -> 9/1/0 or 9/0/1
3. **Plan39+**: API Runtime + remaining AC items + full fractal -> 10/0/0

**關鍵風險**:
- AC-7（阿賴耶分散式）為最大技術挑戰：分散式狀態一致性、安全攻擊面、效能影響
- Tenet #10 完整 fractal 需 API Runtime 作為先決條件（MESH R2 challenge）
- W2-R1 失敗可能延遲 Tenet #8 完整確認

---

## Baseline Rules Summary

### 規則總覽

| 範圍 | 數量 |
|------|------|
| #1-#28（既有，不變） | 28 |
| #29（修改：三類安全保證） | 1 |
| #30（既有，不變） | 1 |
| #31-#39（新增） | 9 |
| **Current Baseline 總計** | **39** |
| PROVISIONAL | 2（PROC-SPEC-3 + 累積負向 floor -0.05） |

### 新增/修改規則索引

| # | 規則摘要 | 來源 | 分類 |
|---|---------|------|------|
| #29 | 三類安全保證：fail-open / fail-closed / **must-invoke** | D1-R4 | Modified |
| #31 | ICommChannel 統一通訊介面 | D2-R2 | New |
| #32 | 多代理零 Core 修改 | D2-R4 | New |
| #33 | seL4 capability model（default zero, child subset of parent） | D2-R5 | New |
| #34 | 五層故障隔離 | D2-R8 | New |
| #35 | 優雅關機協定（force-kill after grace period） | D2-R8 | New |
| #36 | 八識多代理分配混合模型 | D3-R1 | New |
| #37 | AC-6 before AC-7 ordering | D3-R2 | New |
| #38 | ICompositeAgent 權限格 + bisimulation | D3-R3 | New |
| #39 | W2-R1 enhanced SC (SC-1~SC-6) | D4-R4 | New |

### 規範層級確認

1. **十大宣言**（最高）— 22 項決議全數通過宣言檢查，零違反
2. **永久規則** — MR-1~MR-6 無變動，全數遵守
3. **基線規則** — 39 條 Current Baseline + 2 條 PROVISIONAL

---

*Compliance Impact Analysis, Cycle 03-1*
*SYNTHESIST (#1) + BABBAGE (#9)*
*Reviewed by: SUNYATA (#0)*
*2026-03-24*
