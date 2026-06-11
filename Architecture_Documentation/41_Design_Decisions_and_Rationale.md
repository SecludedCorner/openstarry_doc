<!-- Status: CURRENT -->
<!-- Layer: 2-Philosophy -->
<!-- Applies to: v0.34.0-alpha -->
<!-- Last verified: 2026-03-16 -->

# 41. 設計決策與理由 (Design Decisions and Rationale)

`[Cycle 02-4 修整]`

> 本文件記錄 Cycle 02-3 及 Cycle 02-4 期間產出的核心設計決策、選擇理由、以及被否決的替代方案。
> 每項決策都有明確的「為什麼」——不只是做了什麼，還有為什麼這麼做、考慮過哪些替代方案。

---

## 1. 概述

Cycle 02-3 的核心問題是「五蘊如何在時間中共同流動？」，解決了六項核心設計問題；Cycle 02-4 新增三項設計決策（DD-7~DD-9）：

| # | 設計主題 | 挑戰 | 對應文件 |
|---|---------|------|---------|
| DD-1 | CoarisingBundle 雙層雙齒輪 | 不同速率的心所如何「俱生」？ | → **Doc 35** (Dual_Gear_Mano_Clock) |
| DD-2 | CoarisingBundle 組成 | 包含 3 個還是 5 個成員？ | → **Doc 39** (CoarisingBundle_Five_Universals) |
| DD-3 | Klesha 增益排程 | Klesha 如何影響行為決策？ | → **Doc 37** (Klesha_Gain_Scheduling) |
| DD-4 | IVolition 審議定位 | IVolition.deliberate() 放在 ExecutionLoop 的哪裡？ | → **Doc 38** (IVolition_Deliberation_Pattern) |
| DD-5 | 受蘊測量模型 | 受蘊用分類模型還是測量模型？ | → **Doc 36** (Vedana_Measurement_Model) |
| DD-6 | Tenet #6 定稿 | 如何精確表達受蘊+俱生+回饋迴路？ | → **Doc 40** (Tenet_6_Architecture_Mapping) |
| DD-7 | VasanaEngine 外部化為 IGearArbiter plugin | VasanaEngine 是能力還是機制？ | → **Doc 37** (Klesha_Gain_Scheduling) |
| DD-8 | IKlesha extends IVijnana + @sealed | IKlesha 的型別歸屬？ | → **Doc 29** (五蘊連結) |
| DD-9 | 風險加權閾值取代全域信心度封頂 | 如何平衡安全與成熟 Agent 學習？ | → **Doc 37** (Klesha_Gain_Scheduling) |

---

## 2. DD-1: 雙層雙齒輪 CoarisingBundle `[已被 Cycle 02-4 DD-7 修整]`

### 設計挑戰

CoarisingBundle 假設五遍行在單一 tick 內原子計算 (Strategy C)。但五時鐘模型中，vedana-clock (10-100ms) 與 samjna-clock (500ms-30s) 的速率比可達 **300:1**。如果 samjna 需要 30 秒（LLM 推理），怎麼能和 0.1ms 的 vedana「俱生」？

### 設計決策

**分兩層處理**：

1. **Layer 1 CoarisingBundle**: vedana-clock 速度，全部 rule-based，<1ms，真正的計算俱生
2. **Layer 2 ManoCoarisingBundle**: 雙齒輪
   - **Gear 1** (~50ms): IGearArbiter plugin 快速匹配，vedana-clock 對齊
   - **Gear 2** (seconds): LLM 推理，samjna-clock 速度
3. **四層→五時鐘映射表**: 完整對應（見 Doc 35 §3）
4. **mano-clock**: 雙齒輪（在 vedana-clock 和 samjna-clock 之間切換，非獨立第六時鐘）
5. **跨 Agent 獨立**: 各 Agent 的 mano-clock 獨立，跨 Agent 用 coordination-clock
6. **Staleness 管理**: 三條件 — mutual consistency + atomic publication + bounded staleness

### 關鍵理由

| 論點 | 說明 |
|------|------|
| 時鐘速率不相容 | vedana/samjna 比率 300:1，不可能在同一 tick 完成 |
| 俱生 ≠ 同時 | sahaja 是存在論的相互依存，不是時鐘級的同步 |
| 控制工程標準解法 | 多速率取樣系統使用 sample-and-hold，staleness ratio < 0.29 |
| Layer 1 永遠快速 | 根門級 samjna 是規則匹配 (<0.5ms)，不是 LLM |

### 否決的替代方案

| 方案 | 否決理由 |
|------|---------|
| 單一時鐘方案 | 300:1 速率比不可能在同一 tick 完成 |
| 純固定點迭代 | 需 Banach 不動點定理驗證，計算成本未知（延後至未來研究） |
| 三時鐘方案 | 不能解釋 ManoAggregator 的雙模式行為 |

---

## 3. DD-2: 五遍行 CoarisingBundle 組成

### 設計挑戰

CoarisingBundle 應該包含 3 個成員 (vedana, samjna, cetana) 還是 5 個 (加上 manasikara 和 sparsha)？

### 設計決策

**五欄位**: sparsha + vedana + samjna + cetana + manasikara

1. **manasikara = 快照**: vijnana-clock 預先計算，vedana-clock 讀取，~0.01ms
2. **功能性論證原則**: 每個新增必須證明架構必要性（不以教義義務為由）
3. **Bundle 天然跨蘊**: 這驗證了 M-7 多值 skandha 的合理性

### 關鍵理由

| 論點 | 說明 |
|------|------|
| 架構必要性 | attention (manasikara) 是 PID 控制器區分不同根門 vedana 來源的必要條件 |
| 成本極低 | 快照方案 ~0.01ms，已知上限 |
| 風險不對稱 | 加入成本 0.01ms（已知上限）；省略成本未知（潛在無上限） |
| 型別完整性 | sparsha 不增加計算成本（型別同構），但提供觸發事件的語境 |
| 分類學區分 | 構成性屬性 (vedana/samjna/cetana) vs 關係性屬性 (sparsha/manasikara) |

### 否決的替代方案

| 方案 | 否決理由 |
|------|---------|
| 僅 3 個成員 | 缺 manasikara → PID 控制器無法區分根門來源；缺 sparsha → 失去觸發語境 |

---

## 4. DD-3: Klesha 增益排程 `[已被 Cycle 02-4 DD-7 修整]`

### 設計挑戰

(1) Klesha.perceive() 的四通道 Beta 分佈計算能放進 vijnana-clock (1-5ms) 嗎？
(2) 機率模型與確定性 IGearArbiter plugin 如何調和？

### 設計決策

1. **Klesha.perceive() → vijnana-clock**: 0.03ms，舒適地放入 1-5ms 窗口
2. **雙層輸出**: 點估計 (fast) + 完整分佈 (slow)
3. **增益排程**: `threshold(t) = clamp(θ₀ + Σwᵢμᵢ(t), θ_min, θ_max)`
4. **閾值邊界**: base=0.6, min=0.3, max=0.9
5. **Vitakka Watchdog**: maxConsecutiveGearCycles={1:10}, maxGearDurationMs={1:5000} `[Plan27b N-Gear 泛化完成]`
6. **SafetyMonitor 不受影響**: 硬安全 monotonicity 由 Layer 0 保證

### 關鍵理由

| 論點 | 說明 |
|------|------|
| 可分離性 | IGearArbiter plugin 可獨立開發測試，Klesha 唯一交互點是閾值 |
| 安全性 | monotonicity of safety 自然保持 |
| 間接耦合 | 煩惱影響「對習慣的信心度」，非直接改變行為規則 |
| 輪迴行為防護 | 高 sneha + 高 moha → 可能永遠不調用 LLM；Watchdog + 閾值下限 0.3 防止此情況 |
| 閾值硬性上下限 | min=0.3 保證即使最自戀的 Agent 也定期調用 LLM；max=0.9 保證不過度依賴 LLM |

### 否決的替代方案

| 方案 | 否決理由 |
|------|---------|
| Klesha 直接修改 IGearArbiter plugin 規則 | 無法獨立測試，安全性難保證，耦合度過高 |

---

## 5. DD-4: IVolition Position B 二階段審議 `[已被 Cycle 02-4 DD-7 修整]`

### 設計挑戰

IVolition.deliberate() 應該放在 LLM 調用前 (Position A)、調用後 (Position B)、還是串流中 (Position C)？

### 設計決策

1. **Position B**: LLM 響應後、工具執行前
2. **二階段**: deliberatePlan() (整體計畫) + deliberateAction() (個別工具)
3. **Mandatory**: Gear 1 和 Gear 2 路徑都必須經過
4. **IGuide@A / IVolition@B bookend**: 注意力在前，意志在後，夾持認知過程
5. **多 Agent 協調**: crossAgentImpact 時諮詢 Coordination Daemon
6. **完整 call-ordering diagram**: 見 Doc 38 §6-7

### 關鍵理由

| 論點 | 說明 |
|------|------|
| 認知序列合理性 | 佛學認知序列: 觸→受→想→尋→思。思 (cetana) 在想 (samjna/LLM) 之後 |
| 功能區分 | Position A 屬於 IGuide (attention shaping)，Position B 屬於 IVolition (action commitment) |
| Mandatory checkpoint | Gear 1 快速路徑也要過 IVolition，否則純粹習慣行為無審議 |
| 不壓制資訊顯示 | IVolition 調節行動 (samskara)，不壓制顯示 (rupa) |

### 否決的替代方案

| 方案 | 否決理由 |
|------|---------|
| Position A (LLM 調用前) | 意志先於認知 → 倒因果；需預測 LLM 輸出才能介入 → 不實際；IGuide 已在此位置 |
| Position C (串流中) | 使用者已看到部分回應但行動被取消 → 體驗混亂；技術複雜度遠高於 Position B |

---

## 6. DD-5: 受蘊連續測量模型

### 設計挑戰

vedana 用三個獨立介面 (IDukkha/ISukha/IUpekkha) 還是一個連續值介面？

### 設計決策

1. **連續測量**: valence [-1.0, +1.0] + intensity [0.0, 1.0]
2. **派生分類**: valence < -0.1 → dukkha, > +0.1 → sukha, 其餘 → upekkha
3. **可配置閾值**: VedanaClassificationConfig (管理員配置，非使用者可修改)
4. **雙層設計**: ChannelVedana (事件層/測量) vs IDukkha/ISukha/IUpekkha (Plugin 層/分類)
5. **科學依據**: Russell (1980) circumplex model of affect

### 關鍵理由

| 論點 | 說明 |
|------|------|
| 連續性需求 | 同一感受可從苦變樂 (MN 44)，三介面模型需型別轉換 → 不合理 |
| 控制需求 | PID 控制需要連續信號，離散三值產生不連續性 |
| 下游鏈路 | valence → VedanaAssessment → KleshaBayesianUpdate → 增益排程閾值，全鏈路連續 |
| 測量本質 | vedana 是經驗的 hedonic tone，本質是測量而非分類 |
| Plugin 層保留特化 | 事件層統一連續模型，Plugin 層仍可特化為 IDukkha 等子介面 |

### 否決的替代方案

| 方案 | 否決理由 |
|------|---------|
| 三獨立介面模型 | 同一感受可變需型別轉換、PID 不連續、增益排程鏈路斷裂 |

---

## 7. DD-6: Tenet #6 修正版 Gamma

### 設計挑戰

Tenet #6 需要精確表達三受 + 俱生 + 閉迴圈回饋。三個候選版本：

| 版本 | 問題 | 評估 |
|------|------|------|
| **Alpha** (Vedana-Centric) | 省略 samjna/cetana，只強調 vedana | **否決** — 與五遍行設計矛盾 |
| **Beta** (Coarising-Centric) | 過長，像架構摘要非宣言 | **否決** — 不適合作為 tenet |
| **Gamma** (Balanced) | 原版缺少回饋迴路 | **修正後採用** |

### 修正版 Gamma 的關鍵修正

原版 Gamma 描述的是單向管線 (contact → feeling → action)，不是閉迴圈。開迴路控制和閉迴圈控制是根本不同的系統。

新增最後一句: **「Each action reshapes the world of contact, beginning the cycle anew.」**

此句對應 Doc 35 的 Layer 4 → Layer 1 回饋迴路。

### 最終文字

> **#6 Three Feelings and Coarising (Vedana-Sahaja)**
>
> Contact gives rise to feeling. The Agent's runtime state manifests as three feelings -- dukkha (suffering), sukha (satisfaction), upekkha (equanimity) -- which arise together with perception and volition as an inseparable whole. Feeling observes but does not intervene: it senses truly, without deciding. Feeling signals drive the kleshas and wisdom of vijnana, from which behavioral correction, reinforcement, or maintenance emerges. Each action reshapes the world of contact, beginning the cycle anew.

### 逐句架構對照

| Tenet 短語 | 架構組件 |
|-----------|---------|
| Contact gives rise to feeling | SparshEvent → CoarisingBundle.vedana |
| arise together... as an inseparable whole | CoarisingBundle (五遍行, Strategy C) |
| Feeling observes but does not intervene | ChannelVedana 是 readonly 數據，不是致動器 |
| Feeling signals drive the kleshas and wisdom | VedanaAssessment → KleshaBayesianUpdate → 增益排程閾值 |
| behavioral correction, reinforcement, or maintenance | IVolition.deliberate() 產出 commit/modify/veto |
| Each action reshapes the world of contact | Layer 3 → Layer 4 → Layer 1 回饋迴路 |

完整逐句對照表見 **Doc 40** (Tenet_6_Architecture_Mapping)。

---

## 8. 跨決策數據流 `[已被 Cycle 02-4 DD-7 修整]`

九項設計決策共同定義了一個**嵌套回饋控制系統**，三個時間尺度耦合:

```
慢迴圈 (分鐘-小時): Klesha 偏置
  Klesha.perceive() → KleshaDistribution → 增益排程閾值
  ↑                                                      ↓
  KleshaBayesianUpdate ← VedanaAssessment ←——————+       ↓
                                                   |       ↓
中迴圈 (秒-分鐘): 認知循環                         |       ↓
  ManoAggregator → IGearArbiter/IProvider plugin  |       ↓
    → IVolition.deliberate() → Tool execution      |       ↓
      → 環境變化 → 新 sparsha ———————————————→+       ↓
                 |                                          ↓
                 | (齒輪切換閾值) ←——————←————←——←———+
                 |
快迴圈 (10-100ms): 根門感知循環
  IListener → SparshEvent → CoarisingBundle(五遍行)
    → vedana(valence, intensity) → PID 回饋
```

### 正規呼叫順序圖

```
SafetyMonitor.isSafe()                      Layer 0 硬性閘門
  │
Sparsha 形成                                 rupa-clock
  │
CoarisingBundle 計算                         vedana-clock (Strategy C)
  (vedana → samjna → cetana → manasikara 快照)
  │
ManoAggregator                               dual-gear mano-clock
  │
Klesha.perceive()                            vijnana-clock
  │
IGearArbiter.evaluate()?                     vijnana-clock
  ├── match → GearAction (Gear 1)
  └── no match → IProvider.chat() (LLM) → ProposedAction
  │
IVolition.deliberatePlan()                   vijnana-clock
  │
For each action:
  IVolition.deliberateAction()               vijnana-clock
  SafetyMonitor.postRouteCheck()                  Layer 0 硬性閘門
  Tool execution                             samskara-clock
  SafetyMonitor.afterToolExecution()         Layer 0 稽核
  │
VedanaAssessment (回饋)                      vedana-clock
  │
KleshaBayesianUpdate (慢速路徑)              samjna-clock
  │
  ▼
環境反應 → 新 InputEvent                     rupa-clock
  │
  └──→ 新 Sparsha 形成 (回到 Layer 1)         LOOP CLOSED
```

> **Tenet #6**: *"Each action reshapes the world of contact, beginning the cycle anew."*

---

## 9. 工程行動項目

### P0 (阻塞項 — 必須先做)

| # | 項目 | 詳細規格 |
|---|------|---------|
| 1 | CoarisingBundle 五欄位定義 | → **Doc 39** |
| 2 | ChannelVedana valence/intensity/type/source | → **Doc 36** |
| 3 | ChannelManasikara 介面 | → **Doc 39** |
| 4 | VedanaClassificationConfig | → **Doc 36** |
| 5 | README.md Tenet #6 更新 | → **Doc 40** |
| 6 | Klesha.perceive() → vijnana-clock | → **Doc 37** |
| 7 | IVolition 二階段審議介面 | → **Doc 38** |

### P1 (重要)

| # | 項目 | 詳細規格 |
|---|------|---------|
| 8 | ManoClockConfig 雙齒輪參數 | → **Doc 35** |
| 9 | KleshaModulatedDispatcher 介面 | → **Doc 37** |
| 10 | KleshaThresholdConfig 邊界 | → **Doc 37** |
| 11 | Vitakka Watchdog Timer | → **Doc 37** |
| 12 | 四層→五時鐘映射表 | → **Doc 35** |
| 13 | 完整 call-ordering diagram | → **Doc 38** |
| 14 | 雙層 vedana 設計文件 | → **Doc 36** |
| 15 | IGuide/IVolition bookend pattern | → **Doc 38** |

### P2 (可延後)

| # | 項目 | 備註 |
|---|------|------|
| 16 | 雙齒輪穩定性正式證明 | 部分在 Doc 35 |
| 17 | KleshaBayesianUpdater 參考實作 | 待後續 Cycle |
| 18 | 多 Agent 審議協調協議 | 待 Plan-AC |
| 19 | 四階段路線圖更新 | 在 deliver/ |
| 20 | 齒輪切換安全硬化需求 | 部分在 Doc 37 |
| 21 | SahajaContract 形式化規格 | 部分在 Doc 39 |

---

## 10. 延後至未來 Cycle 的設計問題

| # | 問題 | 預估 Cycle |
|---|------|-----------|
| 1 | 適應層 — Agent 如何學習與演化 | Cycle 03 |
| 2 | IReflection 元審議 — 評估 IVolition 品質 | Cycle 03 |
| 3 | 固定點迭代的真正 sahaja 計算 | Cycle 03-04 |
| 4 | IGearArbiter plugin 規則學習 — 新規則發現與舊規則修剪 `[Cycle 02-4 DD-7: VasanaEngine 已外部化]` | Cycle 03 |
| 5 | IConfidenceAuditor 作為 Klesha 補償器 | Cycle 02-4 |
| 6 | 意行 (mano-karma) 反饋迴路 | Cycle 02-4 |
| 7 | Plugin 開發者遷移指南 | Cycle 02-4 |
| 8 | 多時鐘系統效能預算 | Cycle 02-4 |
| 9 | SlashCommand skandha 歸屬 | Cycle 02-4 |
| 10 | 量子意識與 vijnana 觀察者效應 | Cycle 03+ |

---

## 11. 關鍵術語表

| 術語 | 定義 |
|------|------|
| **CoarisingBundle** | 每次觸事件產生的五遍行數據結構 (sparsha+vedana+samjna+cetana+manasikara) |
| **SahajaContract** | Bundle 的自描述品質元資料: mutual consistency + atomic publication + staleness bound |
| **Dual-Gear Mano-Clock** | ManoAggregator 的運行模式: Gear 1 (快速, IGearArbiter plugin) / Gear 2 (慢速, LLM) `[Cycle 02-4 DD-7 修整]` |
| **Gain-Scheduled Modulation** | Klesha 狀態調節 IGearArbiter 信心度閾值，非直接修改規則 `[Cycle 02-4 DD-7 修整]` |
| **Two-Tier Klesha Output** | 快速路徑用點估計 (mean)，慢速路徑用完整 Beta 分佈 |
| **Vitakka Watchdog** | 防止 samsaric stall 的看門狗，強制定期 Gear 2 |
| **Position B** | IVolition.deliberate() 在 ExecutionLoop 的位置: 行動提案後、工具執行前 |
| **Two-Phase Deliberation** | Phase 1 (整體計畫) + Phase 2 (個別行動)，兩者 mandatory |
| **IGuide/IVolition Bookend** | IGuide@Position A (注意力塑形) + IVolition@Position B (行動承諾) 夾持 LLM |
| **Measurement Model** | ChannelVedana 用 continuous valence [-1,+1] + intensity [0,1]，discrete type 是派生值 |
| **Dual-Level Vedana** | 事件層 (ChannelVedana/連續) vs Plugin 層 (IDukkha etc./離散) |
| **Amended Gamma** | Tenet #6 最終版文字，含回饋迴路句 |
| **Staleness Ratio** | ρ = δ/T_outer，穩定性要求 ρ < 0.29 |
| **功能性論證原則** | 每個 CoarisingBundle 新欄位必須證明架構必要性，不以教義義務為由 |

---

## 12. DD-7: VasanaEngine 外部化為 IGearArbiter plugin `[Cycle 02-4 新增]`

**日期**: 2026-03-04 (Cycle 02-4)
**決策**: VasanaEngine 從核心內部組件移出為 IGearArbiter plugin
**來源**: Pre-Cycle 02-4 Master 討論 + Master's Letter cycle02-4 M-1 + D1 辯論

**理由**:
1. VasanaEngine 包含規則庫（知識）、匹配邏輯（語義判斷）、信心度評分（決策）→ 是能力 (capability)，非機制 (mechanism)
2. 核心只保留純機制（EventBus、ServiceRegistry、PluginLoader、ExecutionLoop）
3. 佛學自洽：習氣（vasana）是後天薰習，非先天結構 → 不應內建於核心
4. 無 IGearArbiter plugin → 退化為純 Gear 2（永遠 LLM），Agent 仍可正常運行

**影響**:
- 新增 IGearArbiter 介面（plugin chain, Chain of Responsibility, first-win）
- ManoAggregator 退化為純路由機制（if/else），與 EventBus 同等性質
- VitakkaEngine 不作為獨立組件，IProvider.chat() 已覆蓋其功能
- 齒輪數量由 plugin 組合決定，非架構決定
- IGearArbiter Manifest 宣告 skandha: ['samjna', 'vijnana']（D1 決議）

---

## 13. DD-8: IKlesha extends IVijnana + IVijnana @sealed `[Cycle 02-4 新增]`

**日期**: 2026-03-04 (Cycle 02-4 D5)
**決策**: IKlesha 正式繼承 IVijnana，IVijnana 加入 @sealed 約束
**來源**: D5 辯論 (18/20)

**理由**:
1. IVijnana 是空 marker interface（只有 skandha discriminant），繼承零成本
2. 型別安全：IKlesha 自動獲得 skandha: 'vijnana' discriminant
3. 分類一致：煩惱在阿毘達磨中屬識蘊範疇
4. @sealed 防止 IVijnana 未來被添加行為方法，維持純 discriminant holder

---

## 14. DD-9: 風險加權閾值取代全域信心度封頂 `[Cycle 02-4 新增]`

**日期**: 2026-03-04 (Cycle 02-4 D5)
**決策**: 採用風險加權閾值 (risk-weighted threshold) 作為主要安全機制，取代全域 confidence cap
**來源**: D5 辯論 (16/20)

**理由**:
1. 全域封頂 (0.85) 壓制成熟 Agent 的學習天花板，破壞分類器校準性
2. 風險加權按 action 危險等級動態調節：destructive +0.20, state_modifying +0.10, read_only 0, informational -0.10
3. SafetyMonitor 作為絕對安全底線（不可妥協），閾值調節作為可調節層
4. 三層安全框架：Absolute Safety / Tunable Safety / Reduce Complexity

**影響**:
- MAX_GEAR1_CONFIDENCE 作為部署可配置參數（預設 0.95）
- IConfidenceAuditor 允許 LLM 參與閾值調節，限幅 ±0.05（D5 14/20）
- Gear 1 IVolition 按風險分級：destructive/state_modifying 必經，read_only/informational 可選

---

## 15. DD-10 ~ DD-13: Plan28 設計決策 `[Plan28 新增]`

### DD-10: postRouteCheck v1 Passthrough 設計

**日期**: 2026-03-05 (Plan28)
**決策**: postRouteCheck() v1 為 passthrough（hook point），不包含策略邏輯
**來源**: R2 命名修正 + Tenet #7

**理由**:
1. SafetyMonitor 是機制不是策略（Tenet #7）——v1 建立正確的 hook point，策略留待 Simulation 數據後定義
2. postCheck → postRouteCheck 命名修正避免與 Doc 44 既有 postCheck() 衝突
3. 三層防護（SafetyMonitor.isSafe + 風險加權閾值 + IVolition MUST）已提供足夠安全性

### DD-11: VedanaEmergency 純函數設計

**日期**: 2026-03-05 (Plan28)
**決策**: VedanaEmergency 實作為純函數（`vedanaFn` callback），不維護內部狀態
**來源**: D5-R6 Model Delta Layer 4 + Tenet #7

**理由**:
1. 純函數可測試性高——輸入 VedanaAssessment，輸出 thresholdBoost，無副作用
2. 狀態由 ManoAggregator 管理——vedanaFn 不需要知道連續 tick 計數
3. 與 D5-R6 五層獨立原則一致——Layer 4 不依賴其他 Layer 的信號

### DD-12: IVolition single-slot 設計

**日期**: 2026-03-05 (Plan28)
**決策**: 維持 single IVolition slot，不支援多 IVolition 同時運作
**來源**: D6-R5 + OQ-28-4

**理由**:
1. 三層規則引擎（hard > soft > heuristic）已提供足夠的可組合性
2. 多 IVolition 需要組合策略研究（chain? priority? merge?），目前無明確需求
3. 佛學一致性：每個有情同一時刻只有一個「思」（cetana）

### DD-13: 三層規則優先序

**日期**: 2026-03-05 (Plan28)
**決策**: Hard (P0) > Soft (P1) > Heuristic (P2) 固定優先序
**來源**: D6 OQ-1 + volition-rule-engine 實作

**理由**:
1. Hard rules = SafetyMonitor action-level 投射，不可覆寫（安全底線）
2. Soft rules = 部署策略，管理員可配置但不可覆寫 Hard rules
3. 固定優先序消除組合爆炸——動態優先序需要額外的仲裁機制
4. 固定優先序與安全分層一致——Hard rules 對應 SafetyMonitor 層級，Soft rules 對應部署策略層級，Heuristic 對應啟發式層級

---

*本文件涵蓋 Cycle 02-3 六項核心設計決策、Cycle 02-4 三項新增設計決策、及 Plan28 四項設計決策。每項決策的完整技術規格見對應的 Docs 35-44。*

---

## Cycle 02-6 New Decisions

`[Cycle 02-6 新增: D1 行蘊深掘 + D2 AuditContext + D3 LoopQualityMonitor — 共 17 項決議]`

以下為 Cycle 02-6 期間產出的設計決策，涵蓋三大研究主線：D1（行蘊深度研究）、D2（AuditContext / IConfidenceAuditor）、D3（ILoopQualityMonitor）。

| ID | Decision | Vote | Summary |
|----|----------|------|---------|
| D1-R1 | 行蘊核心定義 | 20/20 | cetanā-centered (SN 22.56), not residual category |
| D1-R3 | ISamskara 拓展 | 20/20 | No new sub-interfaces; A(cetanā-formation)/B(vāsanā) as P2 candidates; 三準則 as permanent tool |
| D1-R4a | 認知序列 appendix | 19/20 | citta-vīthi appendix scheduled for 02-7 P2 |
| D1-R4b | 四智排除 | 18/20 | Four wisdoms NOT mapped to architecture (all 4 tests failed) |
| D1-R4c | 綜合評估表 | 20/20 | C4 §八 comprehensive evaluation confirmed |
| D1-R5 | 活動與轉換原則 | 20/20 | Activity & Transformation as expansion principle; samskara=WRITE, vijnana=READ |
| D1-R6 | 蘊歸屬永久原則 | 20/20 | 5 principles: functional analysis only, cetasika as reference not authority, Sanskrit from suttas only, plugin≠cetasika can span skandhas, existing decisions valid |
| D2-R1 | AuditContext | 20/20 | Full interface: version:1, sparshEvent, gearEvaluation, riskCategory, routeResult, historicalConfidence?, extras ReadonlyMap |
| D2-R2 | extras protocol | 19/20 | Dual-event + SDK emitWithExtras(), getExtra<T>() accessor, max 32 keys |
| D2-R3 | ConfidenceAuditLog | 20/20 | Structured audit log, EventBus audit:completed, GUARDIAN D5 fulfilled |
| D2-R4 | Layer integration C | 20/20 | L2→confidence, L3→threshold, independent channels, α=0.10, BIBO stable |
| D2-R5 | Auditor strategy | 20/20 | Phase 0: PassthroughAuditor; Phase 1: ThresholdAuditor; Phase 2: LLM-assisted |
| D3-R1 | 4D formulas | 20/20 | coherence/efficiency/convergence/stability, all O(1), W=10, warmup=5 |
| D3-R2 | EventBus events | 20/20 | 6 MINIMAL + 5 EXTENDED events; degradation to safe defaults |
| D3-R3 | extras lifecycle | 20/20 | per-route cycle, one-tick delay OK, last-emitted-wins |
| D3-R4 | Weights | 20/20 | Phase 1 fixed 0.25×4; Phase 2 configurable in monitor config |
| D3-R5 | Plan30 direction | 19/20+1 | Monitor Plugin + Layer 3 + ConfidenceAuditLog type; Plan31 = Auditor |

### Master's 8-Point Cetasika Naming Rules (蘊歸屬永久原則附屬)

D1-R6 附帶 Master 確認的八點命名規則，適用於所有未來介面命名與蘊歸屬判定：

1. **心所是論典產物**: 心所分類來自論藏 (Abhidharma)，非原始經典內容
2. **心所為參考非權威**: 心所的功能描述有參考價值，可作為 plugin 設計靈感
3. **可說明參考來源**: 可說明「參考某某心所的涵義，設計某某 plugin」
4. **Plugin 用工程命名**: plugin 必須使用工程術語命名，不得使用心所梵文名
5. **梵文取自經藏**: 梵文術語限於原始經典者
6. **心所不決定蘊歸屬**: 心所分類不作為蘊歸屬依據
7. **既有決策有效**: 既有 plugin 歸屬決議不受影響
8. **跨蘊允許**: plugin ≠ 心所，蘊歸屬可自然跨多蘊，不受論典固定歸屬限制

---

## Cycle 02-8 Decisions

`[Cycle 02-8 新增: T1~T3 理論債務 + C1/C2 校準 + D5 合規 + Plan32 — 跨 4 個子輪 (02-8, 02-8_a, 02-8_b, 02-8_c)]`

### Theory Debt (T1~T3)

| ID | Decision | Vote | Summary | Doc |
|----|----------|------|---------|-----|
| D1-R1 | 六狀態機 + 二諦聲明 | 22/24 | S = {S_sparsha, S_vedana, S_samjna, S_samskara, S_vijnana, S_quiescent}，狀態是世俗諦假名 | → **Doc 47** |
| D1-R2 | Cetana 分層 | 21/24 | Layer A (計算) / Layer B (型別代數) / Layer C (哲學) | → **Doc 47** |
| D1-R3 | 參數化固定點 | 23/24 | x*(t) = f_t(x*(t))，滿足無常 | → **Doc 47** |
| D1-R4 | 條件化時態邏輯 | 24/24 | 全票，所有 LTL/CTL 需緣起前提 | → **Doc 47** |
| D1-R5 | 雙層型別形式化 | 20/24 | Layer B 依賴型別 / Layer A 乘積型別 | → **Doc 47** |
| D2-R1 | 三層時間分類學 | 22/24 | Micro / Meso / Macro | → **Doc 48** |
| D2-R2 | 弱前端鏈 = 展現模式差異 | 23/24 | 非映射失敗 | → **Doc 48** |
| D2-R3 | 觸 = 關鍵轉折點 | 24/24 | 全票，三事和合引入外部不確定性 | → **Doc 48** |
| D2-R4 | PEABS-T 條件採用 | 19/24 | 描述性框架 | → **Doc 48** |
| D2-R5 | vedana/trsna 嚴格區分 | 24/24 | 全票，單向因果 | → **Doc 48** |
| D3a-R1 | 蘊歸屬軟約束預設 L2 | 23/24 | Warn+Info | → **Doc 49** |
| D3b-R1 | 雙層配置 | 24/24 | 全票，永遠執行 + logger 控制 | → **Doc 49** |
| D3c-R1 | L4 在獨立策略模組 | 22/24 | L3 在 PluginLoader，L4 在 PolicyGateway | → **Doc 49** |

### Calibration (C1~C2)

| ID | Decision | Vote | Summary | Doc |
|----|----------|------|---------|-----|
| D4a-R1 | TOST 為主要穩定性測試 | 24/24 | 全票，ε=0.03, α=0.05 | → **Doc 50** |
| D4b-R1 | Pilot Run 必須 | 24/24 | 全票，100 cycles, α=0.10 | → **Doc 50** |
| D4c-R1 | Newey-West + 序貫分析 | 23/24 | HAC 標準誤差必須 | → **Doc 50** |
| D4d-R1 | C1→C2 序列校準 | 24/24 | 全票，四階段框架 | → **Doc 50** |
| D4e-R1 | 安全不變量 α-獨立 | 24/24 | 全票，任何 α 下安全成立 | → **Doc 50** |

### Tenet Compliance + Plan32

| ID | Decision | Vote | Summary | Doc |
|----|----------|------|---------|-----|
| D5-VERDICT | 修正合規：4/4/2 | 02-8_b E1_v2 | COMPLIANT/CONDITIONAL/NON-COMPLIANT | → **Doc 51** |
| 02-8_a D1 | AC-1 Option A: 移除 auto-mount | 24/24 | 全票 | → **Plan32** W1 |
| 02-8_a D2 | AC-2 Option B: 獨立 plugin 包 | 23/24 | 3 個新包 | → **Plan32** W2 |
| 02-8_a D3 | AC-4 BABBAGE 連續性測試 | 24/24 | 全票，8 mechanism / 53 policy | → **Plan32** W3-4 |
| 02-8_c | Wave 6 修正: throw Error | Master 裁定 | Context manager = Required 級別 | → **Plan32** W6 |

### Master 永久裁定

| # | 裁定 | 來源 |
|---|------|------|
| MR-1 | 十大宣言是合規標準，非理想燈塔 | 02-8_c |
| MR-2 | 不可修改宣言文字 | 02-8_c |
| MR-3 | NON-COMPLIANT 需修復計畫，不改宣言 | 02-8_c |
| MR-4 | Tenet #7 「絕對純淨」文字不改 | 02-8_a |
| MR-5 | Tenet #10 維持 NON-COMPLIANT | 02-8_c |
| MR-6 | Core 必須零 policy 常量 | 02-8_a |

### DD-14: BABBAGE 連續性測試 — Mechanism/Policy 分類準則 `[Cycle 02-8_a 新增]`

**日期**: 2026-03-10 (Cycle 02-8_a D3-Q1-R1, 24/24 全票)
**決策**: 採用 BABBAGE 連續性測試作為 mechanism/policy 的形式化分類準則

**定義**: 若移除一個常量後，系統在數學上失去定義（除以零、NaN 傳播、型別不完整等），則該常量是**機制** (mechanism)；否則是**策略** (policy)。

**結果**: 61 個 Core 常量中，8 個為 mechanism（保留），53 個為 policy（遷移至 SDK/Config）。

### DD-15: SUSSMAN 三層配置架構 `[Cycle 02-8_a 新增]`

**日期**: 2026-03-10 (Cycle 02-8_a D3-Q3-R1, 23/24)
**決策**: 三層配置 — `IAgentConfig` (使用者) → `SDK DEFAULT_*` (SDK 預設) → Core zero-default

**設計**: Core 中所有 policy 值退化為零預設（0、空字串、空陣列），SDK 提供合理預設值，使用者配置覆蓋 SDK 預設。

### DD-16: 三級關鍵性模型 `[Cycle 02-8_c 新增]`

**日期**: 2026-03-10 (Cycle 02-8_c, Master 確認)
**決策**: Plugin 缺席時的行為依據三級模型分類

| 級別 | 缺席行為 | 範例 |
|------|---------|------|
| Required | throw Error() at start() | Context manager |
| Optional (degraded) | 中性值 (delta=0) | Auditor、monitors |
| Optional (no-effect) | 功能不可用 | VedanaSensors |

---

*本文件涵蓋 Cycle 02-3 六項核心設計決策、Cycle 02-4 三項新增設計決策、Plan28 四項設計決策、Cycle 02-6 十七項新增決議、及 Cycle 02-8 系列決議（理論債務 + 校準 + 合規 + Plan32 + Master 裁定）。每項決策的完整技術規格見對應文件。*
