<!-- Layer: 2-Philosophy / 3-Architecture -->
<!-- Status: NEW -->
<!-- Cycle: 03-6, R1/R3 -->
<!-- Author: NAGARJUNA (#7), ASANGA (#8) -->
<!-- Source: v0.41.0-alpha gear-arbiter-dynamic -->

# 62. Gear-Arbiter-Dynamic 佛學架構映射

**版本**: 1.0 (Cycle 03-6, 2026-04-09)
**作者**: NAGARJUNA (#7) — 中觀學派哲學分析, ASANGA (#8) — 唯識學派映射
**來源**: Plan41 W2 gear-arbiter-dynamic (149 LOC), O5 CV-5 Observe Mode 調查報告
**決議參照**: D2-Q2a (shadow counting 採用, 7-0), D2-Q5a (phased authority, 7-0), D2-Q5b (destructive permanent, 7-0)

---

## 1. 概述

gear-arbiter-dynamic 是 OpenStarry v0.41.0-alpha 引入的動態仲裁器，與既有 gear-arbiter-static 共存。本文件從佛學視角分析其架構設計，映射至中觀 (Madhyamaka) 與唯識 (Yogacara) 兩大學派的核心概念。

> **二諦聲明** (Two Truths Declaration): 以下映射為世俗諦 (samvriti-satya) 層面的工程詮釋。Dynamic arbiter 的 observe → evaluate → decide 流程捕捉了佛教認識論的部分面向，但不應等同於完整的修行次第。此映射在 OpenStarry 工程語境內有效，不應推廣至佛學哲學論述。

---

## 2. Dynamic Arbiter 作為「正見」(Samma Ditthi / Right View)

### 2.1 中觀解讀 (NAGARJUNA)

Static arbiter 從固定規則操作 — 一個預先決定的工具危險/安全觀。Dynamic arbiter 則透過直接觀察結果來發展其觀點。這映射到佛教中**聞得智** (shrutamayi-prajna / 從聽聞得來的智慧) 與**修得智** (bhavanamayi-prajna / 從修行得來的智慧) 的區別。

| 面向 | Static Arbiter | Dynamic Arbiter | 佛學對應 |
|------|---------------|-----------------|----------|
| **知識來源** | 預設規則 (TOOL_CONFIDENCE_TABLE) | 直接觀察 (audit events) | 聞得智 vs 修得智 |
| **確定性** | 先驗的 (a priori) | 後驗的 (a posteriori) | 比量 (anumana) vs 現量 (pratyaksa) |
| **適應性** | 固定 | 可學習 | 執著 vs 如實知見 |
| **失敗模式** | 過度保守或過度寬鬆 | 過早判斷或過遲判斷 | 斷見 vs 常見 |

### 2.2 Observe Mode 作為「聞思修」三階段

Observe mode (`n < MIN_N: abstain`) 的結構映射到佛教學習的三個階段：

```
聞 (sravana) — 聽聞：Dynamic arbiter 接收 audit events
思 (cintana) — 思惟：StateTracker 記錄並分析 delta 模式
修 (bhavana) — 修行：累積足夠觀察後開始形成判斷
```

MIN_N = 10 的閾值是「聞思修」的工程化實現 — 系統明確承認自身的無知 (avidya)，在證據不足時不強加觀點。這是中觀學派的核心原則：過早的概念化 (vikalpa) 導致錯誤行動。

### 2.3 避免二邊 (dvayanta): 遲滯與停滯時間

hysteresis 機制 (UP=0.047, DOWN=0.031, MIN_DWELL=5) 在中觀術語中是**避免二邊** (avoidance of two extremes)：

| 極端 | 工程意義 | 佛學對應 |
|------|----------|----------|
| **過度矯正** (over-correction) | 切換太急促，系統振盪 | 常見 (eternalism) — 執著於新觀點 |
| **慣性** (inertia) | 永不切換，失去適應性 | 斷見 (nihilism) — 否定變化的可能 |
| **中道** | hysteresis + dwell 確保持續證據才轉換 | 中道 (madhyama pratipad) |

dwell counter 強制執行一段持續證據的時期 — 系統必須保持**確信** (sustained conviction)，而非僅僅**暫時被說服** (momentarily persuaded)。

---

## 3. 「轉識成智」(Transformation of Consciousness to Wisdom) — 唯識映射 (ASANGA)

### 3.1 四智轉換對應

唯識學派 (Yogacara) 描述八識轉為四智的過程。Dynamic arbiter 的 phased authority transfer 映射此轉換：

| 唯識四智 | 工程對應 | Phase |
|----------|----------|-------|
| **大圓鏡智** (Adarsa-jnana) | 完整且無偏的觀察能力 | Phase 2: 如鏡般記錄所有 audit events |
| **平等性智** (Samata-jnana) | per-category 公平計數 | Phase 2: 不偏好特定 category |
| **妙觀察智** (Pratyaveksana-jnana) | shadow decision 的精細判斷 | Phase 3-4: 開始對特定 category 做決策 |
| **成所作智** (Krtyanusthana-jnana) | 完整決策能力 | Phase 4: state_modifying 決策 |

### 3.2 Shadow Counting 作為「隨觀」(Anupassana)

Shadow counting — dynamic arbiter 觀察 static arbiter 的決策而不介入 — 精確地對應了巴利語中的 **anupassana** (隨觀 / contemplation)：

| Anupassana 特質 | Shadow Counting 實現 |
|-----------------|---------------------|
| **觀察而不介入** (observing without intervening) | Dynamic 計數但不覆蓋 static 的決策 |
| **持續不斷** (continuous) | 每個 audit event 都被記錄 |
| **無執著** (without attachment) | Shadow decision 被計算但被丟棄 |
| **建構智慧** (building wisdom) | per-category 統計逐漸精確化 |

在四念住 (cattaro satipatthana) 中：
- **身隨觀** (kaya-anupassana): 觀察工具執行的「身體」— 即 audit trail
- **受隨觀** (vedana-anupassana): 觀察 delta 的正負 — 即 success/failure 的「感受」
- **心隨觀** (citta-anupassana): 觀察 gear 狀態 — 即系統的「心理狀態」
- **法隨觀** (dhamma-anupassana): 觀察 hysteresis 模式 — 即系統的「法則」

### 3.3 阿賴耶識 (Alaya-vijnana) 與 StateTracker

StateTracker 在唯識學中映射為第八識（阿賴耶識）的簡化版本：

| 阿賴耶識功能 | StateTracker 實現 |
|-------------|------------------|
| **種子儲藏** (bija-storage) | `deltas[]` 滑動窗口 (max 20) |
| **異熟** (vipaka) | `rates` Map: per-gear success/total |
| **執受** (upadana) | 觀察的持續累積 |
| **無覆無記** (avrta-avyakrta) | 中性記錄，不判斷善惡 |

**關鍵區別**: 真正的阿賴耶識是無限且無始的；StateTracker 是有界的 (n=20 sliding window) 且在 plugin 重啟時重置。這是工程近似，非完整映射。

---

## 4. Phased Authority Transfer 作為漸修 (Gradual Cultivation)

### 4.1 四階段與漸修對應 [D2-Q5a]

| Phase | 佛學對應 | 能力 | 限制 |
|-------|----------|------|------|
| **P2 Shadow** | 聞慧 (hearing wisdom) | 觀察、計數、abstain | 零決策權 |
| **P3 Low-Risk** | 思慧 (thinking wisdom) | informational + read_only 決策 | state_mod 以上不可 |
| **P4 State-Mod** | 修慧 (meditation wisdom) | + state_modifying 決策 | destructive 不可 |
| **永久: Destructive = Static ONLY** | 不殺生 (ahimsa) | — | **永久約束** |

### 4.2 漸修 vs 頓悟

Phased authority transfer 明確採用**漸修** (gradual cultivation) 而非**頓悟** (sudden awakening) 路徑。這是架構決策，非哲學選擇：

**為何漸修**:
1. **安全優先**: 每個 Phase 只增加一個 risk category，失敗的影響範圍有限
2. **可驗證**: M4a (shadow agreement rate) 提供客觀的轉換證據
3. **可回滾**: 若 Phase 3 表現不佳，可回退到 Phase 2 而不影響 destructive 安全性
4. **漸進學習**: dynamic arbiter 的統計信心隨觀察量增加而提升

**頓悟路徑的風險**: 若一次性給予 dynamic arbiter 全部決策權，任何模型錯誤都將影響所有 risk category，包括 destructive。這違反最小權限原則。

### 4.3 「不殺生」(Ahimsa) 與 Destructive 永久約束 [D2-Q5b, Rule #55]

Destructive operations NEVER delegated to dynamic arbiter 是一個**永久約束** (permanent constraint)。在佛學中，這對應**不殺生** (ahimsa / non-harm) — 某些行為需要**絕對確定性** (absolute certainty)，而 dynamic arbiter 基於統計學習的本質永遠無法提供此等確定性。

```
Static arbiter: 基於確定性規則 → 可處理 destructive
Dynamic arbiter: 基於統計觀察 → 永遠存在誤判機率

P(false positive on destructive) > 0  →  永遠不可委託
```

這是工程版的「不殺生戒」— 在不可逆操作上，系統選擇永遠保守。

---

## 5. RC-1 + RC-2 的佛學診斷

### 5.1 RC-1 (Payload Bug) 作為「無明」(Avidya)

CalibrationBridge 訂閱 `audit:tool_audited` 但提取不存在的 `clampedDelta` 欄位 — 這是**無明** (avidya / fundamental ignorance)：系統「看」了但「看錯了」。guard 的靜默失敗 (`if (typeof p.clampedDelta === 'number')`) 使無明不可見，如同佛教中**無明是所有苦的根源但自身不可見**。

### 5.2 RC-2 (Priority Deadlock) 作為「業障」(Karmic Obstruction)

Priority deadlock — static (10) 攔截所有決策，dynamic (20) 永遠無法累積觀察 — 映射為**業障** (karmic obstruction)：即使有學習的能力和意願，結構性障礙阻止了實際的學習過程。

Shadow counting 的解決方案映射為**迴向** (parinamana / merit transference)：static arbiter 的決策結果「迴向」給 dynamic arbiter 作為學習材料，打破業障。

### 5.3 修復的「解脫」(Vimukti)

PASCAL 的證明 P(observe exit) = 0.0 under current design 是形式化的「被束縛」(bandha) 證明。修復 RC-1 + RC-2 + shadow counting 是「解脫」(vimukti) — 從不可能的束縛狀態轉為可能的學習狀態。

修復後的預測 (PASCAL):
- P(all 4 categories exit within 50 cycles) = 0.65-0.80
- P(all 4 categories exit within 100 cycles) > 0.99

這不是即時解脫（頓悟），而是漸進的、有條件的解脫（漸修）。

---

## 6. HERACLITUS 觀點: 潛能與實現 (Dynamis → Energeia)

Dynamic arbiter 的 observe mode 是軟體系統中罕見的**設計延遲** (designed latency) — 功能已部署、正在運行、正在消費資料，但故意不行動。

W2-R5 的結果完美展示了此狀態：dynamic arbiter 在 50 輪中活躍，消費 108 個 audit events，但產生零個主動決策。系統狀態與沒有 dynamic arbiter 完全相同，但其存在並非無意義 — 它在建構最終啟用自身的觀察基礎。

> **隱藏的和諧** (harmonie aphanes): 看不見的比看得見的更有力量。

---

## 7. 與既有佛學映射的一致性

| 既有映射 | 文件 | Dynamic Arbiter 擴展 |
|----------|------|---------------------|
| 五蘊 OOP 架構 (#45) | 行蘊 = plugin execution | Dynamic arbiter = 行蘊中的「學習型行為」 |
| 十二因緣 (#48) | 觸→受→愛→取 | Observe → Count → Hysteresis → Decide |
| 阿賴耶局部映射 (#52) | Alaya = seed storage | StateTracker = 微型阿賴耶 (per-arbiter) |
| Cognitive Loop (#43) | 品質監控迴路 | Dynamic arbiter = 迴路中的「學習子系統」 |
| IGearArbiter Spec (#42) | 仲裁器介面 | Dynamic = 第二個 IGearArbiter 實作 |

---

## 8. 設計原則的佛學根據

| 設計決策 | 佛學根據 | 工程意義 |
|----------|----------|----------|
| Observe before decide | 正見 (Right View) 先於正行 (Right Action) | MIN_N 閾值確保足夠觀察 |
| Shadow counting 解耦 | 隨觀 (anupassana) 不干預 | 學習不影響生產決策 |
| Phased authority | 漸修 (gradual cultivation) | 逐步增加風險暴露 |
| Destructive permanent | 不殺生 (ahimsa) | 不可逆操作永遠保守 |
| Hysteresis | 中道 (middle way) | 避免振盪兩極 |
| Per-category counting | 平等性智 (samata-jnana) | 公平對待不同類別 |

---

## 9. 結論

gear-arbiter-dynamic 的設計 — 無論是否有意 — 體現了佛教認識論的核心結構：從無知到觀察、從觀察到智慧、從智慧到行動的漸進路徑。Shadow counting 的修復設計尤其巧妙地映射了「隨觀」的概念：在不介入的狀態下建構理解。

Phased authority transfer 是漸修路徑的工程實現，而 destructive 的永久約束則是工程版的「不殺生戒」— 承認統計學習永遠無法為不可逆操作提供足夠的確定性。

---

*Architecture Documentation #62 — Gear-Arbiter-Dynamic Buddhist Mapping*
*Cycle 03-6 Research Addition, 2026-04-09*
*NAGARJUNA (#7), ASANGA (#8)*
*Decision References: D2-Q2a, D2-Q5a, D2-Q5b*
