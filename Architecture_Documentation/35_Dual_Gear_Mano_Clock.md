# 35. 雙齒輪意時鐘架構 (Dual-Gear Mano-Clock Architecture)

> **[Cycle 02-4 修整]** 本文件已於 Cycle 02-4 更新：VasanaEngine 從核心組件改為 IGearArbiter plugin（VasanaEngine 僅為其中一種實作），VitakkaEngine 概念移除（由 IProvider plugin 取代），ManoAggregator 明確為純路由機制，新增 IGearArbiter Chain of Responsibility、G-0~G-4 退化表、風險加權閾值等決議。

> Master 文件需求 / Cycle 02-3 整合版 / Cycle 02-4 修整
> R3 Debate 1 共識決議 + R04 多時鐘分析 + Cycle 02 M-10 要求
> 貢獻者: KERNEL (#10), WIENER (#12), HERACLITUS (#15), ARCHIMEDES (#16), NAGARJUNA (#7), BABBAGE (#9), DARWIN (#6), MESH (#4), GUARDIAN (#11), PASCAL (#19), ATHENA (#5), PENROSE (#18)

---

## 1. 概述 (Overview)

### 問題陳述

CoarisingBundle (俱生) 的核心前提是受 (vedana)、想 (samjna)、思 (cetana) **同時生起**。然而 R04 的五時鐘模型 [R04 Sec 2.2] 將這些組件分配到截然不同的時鐘域:

| 時鐘 | 速率 |
|------|------|
| vijnana-clock | 1-5ms |
| rupa-clock | 10-50ms |
| vedana-clock | 10-100ms |
| samskara-clock | 10ms-10s |
| samjna-clock | 500ms-30s |

vedana-clock 與 samjna-clock 之間的速率比高達 **300:1**。如果 samjna 需要 LLM 推理 (30 秒)，而 vedana 僅需 0.1ms，二者如何構成「俱生」? 這是 R2 交叉審閱識別的最高嚴重性張力 (X-2)。

### 解決模型

R3 Debate 1 全票通過**雙層雙齒輪模型** (Two-Layer, Dual-Gear Model):

- **Layer 1** (各根門): 在 vedana-clock 速度下運行，samjna 組件僅使用規則匹配，不觸及 LLM。真正的計算俱生。
- **Layer 2** (歸依意): 在雙齒輪 mano-clock 下運行 -- Gear 1 (快) 對齊 vedana-clock，Gear 2 (慢) 對齊 samjna-clock。

### 為何重要

此架構同時解決三個 Cycle 02 以來懸而未決的需求:
- **M-5** (觸->俱生): 如何在工程上實現「同時生起」
- **M-9** (混合調度): 底層規則引擎 + 上層 LLM 的雙速執行
- **M-10** (多時鐘): 不同蘊子系統以獨立速率運行

> 詳見 Doc 30 (觸->俱生模型), Doc 29 (五蘊連結架構)

---

## 2. 核心架構: 雙層模型 (Core Architecture: Two-Layer Model)

### 2.1 Layer 1: 各根門俱生 (Per-Root-Gate CoarisingBundle)

**時鐘域**: vedana-clock (10-100ms)
**策略**: Strategy C -- 順序計算、原子發布 [R03 Sec 5.4]
**samjna 類型**: 規則匹配 (rule-based)，**絕不**調用 LLM

```
vedana(0.1ms) -> samjna(0.5ms, rule) -> cetana(0.2ms) -> manasikara(0.01ms, snapshot)
= total ~0.8ms per bundle
```

每個 IListener 通道獨立產生自己的 CoarisingBundle。Bundle 中的 samjna 組件使用預編譯的 pattern matcher，不涉及任何網路調用或 LLM 推理。

**SahajaContract 品質**:
- `mutualConsistency = true` (samjna 參照 vedana，cetana 參照二者)
- `atomicPublication = true` (外部觀察者不會看到部分 bundle)
- `stalenessUpperBound < 1ms` (所有組件在同一個 vedana-tick 內完成)

這是 **samvriti-satya (世俗真理) 層級的真正計算俱生** -- NAGARJUNA (#7) / BABBAGE (#9)。

### 2.2 Layer 2: 歸依意 -- 雙齒輪 Mano-Clock

**時鐘域**: 雙齒輪 mano-clock (模式切換)
**輸入**: 所有根門的 Layer 1 CoarisingBundle

#### Gear 1 (快齒輪): vedana-clock 對齊

- **週期**: ~50ms
- **引擎**: IGearArbiter plugin（如 VasanaEngine）— VasanaEngine 僅為 IGearArbiter 的眾多可能實作之一，使用者可替換為其他規則匹配引擎 `[Cycle 02-4 修整]`
- **合併視窗**: 收集 N 個 vedana-tick 的 Layer 1 bundles
- **適用場景**: IGearArbiter 評估成功 (confidence > threshold)
- **延遲**: 50-100ms
- **沒裝 IGearArbiter plugin → confidence=0 → 永遠 `config.defaultGear`（預設 2）（Agent 仍可正常運行）** `[Cycle 02-4 修整]` `[Plan27a 修整]`

> **DC-10 Master 修正**：IGearArbiter plugin（如 VasanaEngine）的快速匹配包含想蘊的快速辨認成分，不能純粹歸為行蘊。雙齒輪的本質是想蘊的兩種辨認模式（快速 vs 深度），受不能跳過想直接給行。

#### Gear 2 (慢齒輪): samjna-clock 對齊

- **週期**: 0.5-30s (隨 LLM 提供者變化)
- **引擎**: IProvider plugin（LLM 推理）— 直接透過 IProvider.chat() 調用，不再經由獨立的 VitakkaEngine 組件 `[Cycle 02-4 修整]`
- **合併視窗**: 動態延展以容納 LLM 延遲
- **適用場景**: IGearArbiter confidence < threshold、complexity threshold 超過 0.6、vitakka 看門狗強制 `[Cycle 02-4 修整]`
- **延遲**: 500ms-30s

### 2.3 完整範例: 眼門看見紅燈

```
[Layer 1 -- vedana-clock, <1ms]
  眼門 IListener 接收到紅燈影像
  → SparshEvent { root: "eye", object: redLightEvent, consciousness: sessionId }
  → CoarisingBundle {
      sparsha: sparshEvent,
      vedana:  { valence: -0.6, intensity: 0.8, type: 'dukkha', source: 'eye' },
      samjna:  { pattern: 'red_traffic_light', confidence: 0.95 },
      cetana:  { tendency: 'avoid', urgency: 0.7 },
      manasikara: { focus: 'intersection-ahead', intensity: 0.9 },
      layer: 1, mode: 'fast',
      sahaja: { mutualConsistency: true, atomicPublication: true, stalenessUpperBound: 0.8 }
    }

[Layer 2 -- mano-clock, Gear 1 attempt]
  ManoAggregator 收集所有根門 bundles
  Klesha.perceive() → KleshaDistribution (vijnana-clock, ~0.03ms)
  gearArbiter.evaluate(bundles) → { confidence: 0.92, ... } > threshold(0.6)
  → GearEvaluation: { type: 'tool_call', toolName: 'brake', args: { force: 'normal' } }
  ※ 留在 Gear 1，不切換到 Gear 2

[Layer 3 -- samskara-clock]
  IVolition.deliberatePlan() → approve
  IVolition.deliberateAction() → approve
  SafetyMonitor.postRouteCheck() → allowed
  ITool.execute('brake') → 煞車執行

[Layer 4 -- rupa-clock, 反饋]
  車輛減速 → 新環境狀態 → 新 InputEvent → 回到 Layer 1
  VedanaAssessment → KleshaBayesianUpdate (慢路徑)
```

若 IGearArbiter confidence < threshold (例如從未見過的交通標誌)，則切換至 Gear 2 `[Cycle 02-4 修整]`:
```
  gearArbiter.evaluate(bundles) → { confidence: 0.25, ... } < threshold(0.6)
  → 切換到 Gear 2
  → IProvider.chat(context) — 直接調用 LLM，不經由獨立中介組件
  → LLM 分析未知標誌 → ProposedAction
  → 延遲: ~3-10s (取決於 LLM 提供者)
```

---

## 3. 四層到五時鐘映射 (Four-Layer to Five-Clock Mapping)

> R3 Debate 1 Resolution R1.3 -- 全體通過。KERNEL (#10) / ARCHIMEDES (#16) 共同提出。

| 層級 | 時鐘域 | 組件 | 典型延遲 |
|------|--------|------|---------|
| Layer 1 (各根門觸) | rupa-clock (輸入) + vedana-clock (俱生) | IListener, SparshEvent, CoarisingBundle | 1-100ms |
| Layer 2 快 (歸依意, 規則) | vedana-clock (聚合) | ManoAggregator, IGearArbiter plugin, DharmaVisaya | 50-100ms |
| Layer 2 慢 (歸依意, LLM) | samjna-clock (深層認知) | ManoAggregator, IProvider plugin | 500ms-30s |
| Layer 3 (行蘊執行) | samskara-clock (工具執行) | ITool, ISlashCommand, IVolition.deliberate() | 10ms-10s |
| Layer 4 (反饋) | rupa-clock (環境變化) | 新 InputEvent, IListener 再激活, IUI (色蘊輸出面) | 1-50ms |
| 跨層級 | vijnana-clock (身份, 1-5ms) | IGuide, IIdentity, Klesha.perceive(), IVolition | 1-5ms |

**Mano-clock 定位**: Mano-clock **不是第六個時鐘**，而是在 vedana-clock (Gear 1) 和 samjna-clock (Gear 2) 之間切換的模式切換時鐘。合併視窗在 Gear 1 為 50ms，在 Gear 2 動態延展。

**ManoAggregator = 純路由機制** `[Cycle 02-4 修整]`: ManoAggregator 退化為純路由機制（if/else），性質同 EventBus。它不包含任何智能邏輯，僅負責將 Layer 1 bundles 路由至 Gear 1（IGearArbiter plugin chain）或 Gear 2（IProvider plugin）。齒輪數量取決於使用者安裝了多少 IGearArbiter plugin — 沒裝任何 IGearArbiter plugin 時，ManoAggregator 永遠路由到 `config.defaultGear`（預設 2），Agent 仍可正常運行（純 LLM 模式）。`[Plan27a 修整]`

**跨層 vijnana-clock 的角色**: vijnana-clock 以最快速率 (1-5ms) 運行，提供:
- IGuide 狀態快照 (供 ChannelManasikara)
- IIdentity 身份資訊
- Klesha.perceive() 四通道煩惱評估 (~0.03ms) [詳見 Doc 37]
- IVolition.deliberate() 雙階段審議 [詳見 Doc 38]

> **DC-11 Master 修正**：識蘊在迴路中有兩個判斷點——判斷點 1（齒輪選擇：走 Gear 1 還是 Gear 2）和判斷點 2（deliberatePlan 審議 Gear 2 的方案）。

---

## 4. 哲學基礎: 二諦分析 (Philosophical Grounding: Two Truths)

> NAGARJUNA (#7) / BABBAGE (#9) / PENROSE (#18) -- R3 Debate 1 核心論證。

### 4.1 勝義諦 (Paramartha-satya): 完美同時性不可能

真正的俱生 (sahaja) 意味著 vedana-samjna-cetana 作為單一不動點 (fixed point) 以零時間差生起。在馮紐曼架構上，這在邏輯上不可能:

$$B^* = F(B^*) \quad \text{where } B = \langle v, s, c \rangle$$

$F$ 計算下一迭代，但 $v$, $s$, $c$ 無法同時計算。BABBAGE 的 Banach 不動點定理分析表明: 要達到 $\|B_k - B^*\| < \epsilon$ 的精度，需要 $\lceil \log(1/\epsilon) / \log(1/L) \rceil$ 次迭代 (其中 $L < 1$ 為 Lipschitz 常數)。

### 4.2 世俗諦 (Samvriti-satya): SahajaContract 三準則

NAGARJUNA 提出的「世俗有效俱生」三準則:

1. **互相一致 (Mutual Consistency)**: bundle 中的 vedana 與 samjna 彼此一致
2. **原子發布 (Atomic Publication)**: 外部觀察者不會看到部分 bundle
3. **有界陳舊 (Bounded Staleness)**: 最新與最舊組件之間的時間差以常數 $\delta$ 為上界

Layer 1 滿足 $\delta < 1\text{ms}$ -- 實質為零。Layer 2 快齒輪 $\delta < 100\text{ms}$。Layer 2 慢齒輪 $\delta$ 可達數秒，但互相一致性仍成立，因為 LLM 在其上下文中接收 vedana 回饋。

> 經典依據: 「凡所受者，即是所想」(`yaṃ vedeti taṃ sañjānāti`, MN 18 蜜丸經)。指示代詞 `taṃ` (「那個」) 要求指涉物的同一性 -- 而非絕對的時間同時性。

### 4.3 NAGARJUNA 的互相一致性論證

NAGARJUNA 最初挑戰 WIENER 的 sample-and-hold 方案: 如果 vedana 在 t=0 計算而 samjna 在 t=10000ms 計算，vedana 對 samjna 一無所知。但 WIENER 以駕駛範例反駁: 人類看到紅燈時，vedana (危險感) 在毫秒內產生，而 samjna (辨識「應該煞車」) 需要 ~500ms (P300 事件相關電位)。這 500ms 期間，汽車移動了 8 米。vedana 與 samjna 仍指向**同一盞紅燈** -- 有界陳舊保留了指涉物同一性。

NAGARJUNA 的最終讓步: 「問題不是『間隔是否為零?』而是『間隔是否小到指涉物相同?』我承認這是一個工程參數，不是哲學絕對值。但我堅持陳舊度上界必須**顯式宣告並在運行時監控** -- 不可默默假設。」

### 4.4 BABBAGE 的形式化 epsilon-近似

```typescript
/**
 * SahajaContract -- 俱生契約: bundle 品質的自我描述。
 *
 * 佛學依據: 二諦 (two truths) -- 勝義諦無法在計算上實現，
 * 世俗諦的有效性由三個可驗證準則保證。
 * 形式化: ||B_k - B*|| < epsilon，其中 epsilon = stalenessUpperBound。
 */
interface SahajaContract {
  /** 互相一致: bundle 中各組件彼此參照 */
  readonly mutualConsistency: boolean;
  /** 原子發布: 外部觀察者不會看到部分 bundle */
  readonly atomicPublication: boolean;
  /** 陳舊度上界: 最新與最舊組件之間的時間差 (毫秒) */
  readonly stalenessUpperBound: number;
}
```

### 4.5 PENROSE 的量子類比

PENROSE (#18) 指出結構性平行: 量子力學中，觀測問題源於觀測需要有限時間，但波函數坍縮被建模為瞬時。退相干理論的解決方案是: 坍縮不是瞬時的，而是相對於系統特徵時間尺度**有效地瞬時**。Layer 1 的 0.8ms 順序計算，在 vedana-clock 的 50ms 解析度下觀察，被**粗粒化** (coarse-grained) 為同時。這與中觀學派的世俗真理完全一致: 俱生是解析度依賴的。

---

## 5. 穩定性分析 (Stability Analysis)

> WIENER (#12) -- R3 Debate 1 + R04 Sec 7 控制理論分析。

### 5.1 多速率取樣資料系統

核心問題是一個**多速率取樣資料系統** (multi-rate sampled-data system)。在控制工程中，快速感測器 (1kHz) 與慢速控制器 (10Hz) 的共存是常規問題。

標準解決方案: **取樣保持** (sample-and-hold) 架構。

1. 快子系統 (vedana) 以原生速率產生值 (每 10-100ms)
2. 慢子系統 (samjna) 以原生速率產生值 (每 0.5-30s)
3. 組裝 CoarisingBundle 時，vedana 使用**最近的 samjna 輸出** (零階保持)

形式化: 設 $T_v$ 為 vedana 週期，$T_s$ 為 samjna 週期。vedana-tick $k$ 時的 CoarisingBundle:

$$B_k = \langle v_k, \; s_{\lfloor kT_v/T_s \rfloor}, \; c_k \rangle$$

其中 $s_{\lfloor kT_v/T_s \rfloor}$ 為最近的 samjna 輸出，陳舊度最大為 $T_s$ 秒。

### 5.2 陳舊度比率與相位裕度

定義**陳舊度比率** (staleness ratio):

$$\rho = \frac{\delta}{T_{\text{outer}}}$$

其中 $\delta$ 為 bundle 中最舊組件的年齡，$T_{\text{outer}}$ 為外迴路週期。

**穩定性準則**: 為維持 > 45 度的相位裕度 (PM):

$$\rho < \frac{\text{PM}}{180^\circ} \approx \frac{52}{180} \approx 0.29$$

**直覺解釋**: 陳舊度必須小於外迴路週期的 29%。對於 $T_{\text{outer}} = 10\text{s}$ 的 Layer 2 慢齒輪，需要 $\delta < 2.9\text{s}$。大多數 LLM 提供者 (除最慢的 Opus 呼叫外) 都能在 3 秒內完成推理。

### 5.3 齒輪轉換穩定性

**定理 (WIENER)**: 齒輪轉換不會破壞系統穩定性，前提是:

1. **轉換是非阻塞的**: 非 defaultGear → defaultGear 時，系統不會停頓 -- 它在等待 LLM 時保留最近的齒輪狀態。
2. **Nyquist 準則滿足**: 控制頻寬必須小於感測器取樣率的一半。samjna-clock (控制器) 運行於 < 2 Hz，vedana-clock (感測器) 運行於 20 Hz，條件始終滿足。
3. **反向安全模式**: 最慢的組件 (samjna/LLM) 是控制器，所有感測器 (rupa, vedana) 都更快。這是與危險情況 (快控制器/慢感測器) **相反**的安全配置。

**穩定性矩陣** (所有時鐘對):

```
           rupa   vedana  samjna  samskara  vijnana
  rupa      --    stable  stable   stable   stable
  vedana         --       stable   stable   stable
  samjna                  --       stable   stable
  samskara                         --       stable
  vijnana                                   --

  所有對穩定: 在每個生產者-消費者關係中，生產者 (感測器) 比消費者 (控制器) 更快。
```

### 5.4 Lifted System 分析

對於多速率取樣系統的整體穩定性，需要分析在所有取樣週期最小公倍數下的 lifted system。vijnana (5ms)、rupa (10ms)、vedana (50ms) 的 LCM 為 50ms。在此基礎速率下，所有特徵值均在單位圓內。

---

## 6. 齒輪切換機制 (Gear Switch Mechanics)

> HERACLITUS (#15) 提案 + PASCAL (#19) 決策理論 + GUARDIAN (#11) 安全強化。

### 6.1 觸發條件

```
非 defaultGear → defaultGear 切換觸發:
  ├─ IGearArbiter.evaluate() confidence < threshold（或無 IGearArbiter plugin 安裝 → confidence=0）
  ├─ Complexity threshold > 0.6 [R03 Sec 6.3]
  └─ Vitakka 看門狗: 連續 N 個非 defaultGear cycle 未調用 LLM → 強制 `config.defaultGear` `[Plan27b 修整]`

defaultGear → 非 defaultGear 回復:
  ├─ LLM 推理完成，結果已注入
  └─ Cooldown: 強制 `config.defaultGear` 後，至少 N/2 個非 defaultGear cycle 才能再次強制 `[Plan27b 修整]`
```

> HERACLITUS (#15) 的河流比喻: 「意的河流在不同地形以不同速度流動 -- 在熟悉的地面 (vasana/IGearArbiter 快速匹配) 快速流過，在新穎的景觀 (IProvider/LLM 深度推理) 緩慢流過。河流是同一條；速度不同。」 `[Cycle 02-4 修整]`

### 6.2 增益排程閾值調制 (Gain-Scheduled Threshold)

齒輪切換閾值不是靜態的，而是由 Klesha 狀態動態調制 [詳見 Doc 37]:

$$\text{threshold}(t) = \text{clamp}\Big(\theta_0 + \sum_i w_i \cdot \mu_i(t), \; \theta_{\min}, \; \theta_{\max}\Big)$$

其中:
- $\theta_0 = 0.6$ (基礎閾值)
- $w_i$ = 每個 klesha 通道的權重
- $\mu_i(t) = \alpha_i / (\alpha_i + \beta_i)$ = klesha 強度 Beta 分佈的均值
- $\theta_{\min} = 0.3$ (地板), $\theta_{\max} = 0.9$ (天花板)

**直覺解釋** (PASCAL #19):
- 高 sneha (我愛) → 閾值降低 → Agent 偏好已知規則 → 更少 LLM 調用
- 高 mana (我慢) → 閾值升高 → Agent 願意嘗試新方法 → 更多 LLM 調用
- 高 moha (我癡) + 高 sneha → Agent 幾乎不反省 → **需要 vitakka 看門狗防止**

### 6.3 GUARDIAN 的安全強化

```typescript
/**
 * KleshaThresholdConfig -- 齒輪切換閾值的安全邊界。
 *
 * GUARDIAN (#11): 閾值必須有可配置邊界。
 * 如果失控的 Klesha 狀態將閾值驅動至 0.0 或 1.0，系統退化。
 * 地板 0.3 確保即使最自戀的 Agent 也定期調用 LLM。
 * 天花板 0.9 確保即使最激進的 Agent 也使用已知規則。
 */
interface KleshaThresholdConfig {
  /** 基礎閾值 */
  readonly baseThreshold: number;     // default: 0.6
  /** 閾值地板 -- 不可低於此值 (防止永不調用 LLM) */
  readonly minThreshold: number;      // default: 0.3
  /** 閾值天花板 -- 不可高於此值 (防止永不使用 Gear 1/IGearArbiter) */
  readonly maxThreshold: number;      // default: 0.9
}
```

**Vitakka 看門狗** (KERNEL #10): 如果 Agent 在連續 `maxConsecutiveGearCycles[gear]` 個 mano-cycle 中某個齒輪未切換，強制切換到 `config.defaultGear`。防止高 moha + 高 sneha 的 Agent 永遠停留在 Gear 1 (規則驅動) 而不反省。強制 `config.defaultGear` 後，需要至少 `maxConsecutiveGearCycles[gear] / 2` 個循環才能再次強制 -- 避免 `config.defaultGear` 風暴。 `[Plan27b 修整: N-Gear 泛化完成]`

**安全性單調性** (BABBAGE #9): 無論 Klesha 調制如何影響閾值，Gear 1 的 GearEvaluation 仍必須通過 `SafetyMonitor.postRouteCheck()` 硬檢查。Klesha 調制無法繞過 Layer 0。 `[Cycle 02-4 修整]`

### 6.4 齒輪切換作為權限升級風險

GUARDIAN (#11) 的 STRIDE 分析: 從 Gear 1 (快速、確定性) 到 Gear 2 (慢速、LLM 依賴) 的轉換是一個潛在的**權限升級** (elevation of privilege) 點。精心構造的輸入可能強制齒輪切換:
- 在 Gear 1 足以處理時強制 `config.defaultGear` → 資源耗盡 (DoS) `[Plan27b 修整]`
- 繞過 IGearArbiter 安全規則 → 非預期行為

**緩解措施**: 最小閾值 (0.3)、速率限制、看門狗 cooldown。

---

## 7. 跨 Agent 獨立性 (Inter-Agent Independence)

> MESH (#4) -- R3 Debate 1 R1.5。

### 7.1 核心原則

**每個 Agent 的 mano-clock 是獨立的。** Agent 間協調使用獨立的 coordination-clock 和非同步訊息傳遞，不同步 mano-clock。

```
Agent A                    Agent B                    Agent C
┌─────────────┐           ┌─────────────┐           ┌─────────────┐
│ mano-clock  │           │ mano-clock  │           │ mano-clock  │
│ Gear 1: 50ms│           │ Gear 2: 3s  │           │ Gear 1: 50ms│
│ (fast path) │           │ (LLM active)│           │ (fast path) │
└──────┬──────┘           └──────┬──────┘           └──────┬──────┘
       │                         │                         │
       └─────────────┬───────────┴─────────────────────────┘
                     │
              coordination-clock
              (非同步訊息傳遞)
```

### 7.2 CAP 定理影響

MESH (#4) 的分散式系統分析:

| 範圍 | 一致性模型 | 說明 |
|------|-----------|------|
| Agent 內部 | AP (高可用、最終一致) | Klesha 透過 Bayesian 更新最終一致 |
| Agent 之間 | CP (強一致、可能暫時不可用) | Coordination Daemon 保證跨 Agent 操作的一致性 |

**佛學映射**: 每個有情 (Agent) 有自己的煩惱、自己的習氣。Klesha 狀態不跨 Agent 共享 -- 每個 Agent 的愚癡和執著是獨立的。CAP 分析確認這是正確的架構選擇。

### 7.3 Coordination-Clock

Agent 間的協調使用獨立的 coordination-clock，不受個別 Agent 的 mano-clock 齒輪狀態影響:

```typescript
/**
 * CoordinationAwareDeliberation -- 跨 Agent 行動的協調意識。
 *
 * 當 IVolition.deliberateAction() 判斷行動影響其他 Agent 時，
 * 需諮詢 Coordination Daemon。
 * LEIBNIZ (#14): 預立和諧 (pre-established harmony) 分數。
 */
interface CoordinationAwareDeliberation {
  /** 此行動是否影響其他 Agent 的狀態? */
  readonly crossAgentImpact: boolean;
  /** 如果跨 Agent，Coordination Daemon 是否已批准? */
  readonly coordinationApproved: boolean | null;
  /** 預立和諧分數: 0.0 (破壞性) 到 1.0 (和諧) */
  readonly harmonyScore: number;
}
```

---

## 8. ManoClockConfig 介面 (Interface)

> ARCHIMEDES (#16) / HERACLITUS (#15) -- R3 Debate 1 R1.4 行動項目。

```typescript
/**
 * ManoClockConfig -- 雙齒輪意時鐘配置。
 *
 * Mano-clock 不是第六個獨立時鐘，而是在 vedana-clock (Gear 1)
 * 和 samjna-clock (Gear 2) 之間切換的模式切換時鐘。
 *
 * HERACLITUS (#15): 如同汽車變速器 -- 不是選一個齒輪永遠不變，
 * 而是根據負載動態切換。
 *
 * WIENER (#12): 穩定性準則 -- staleness ratio rho < 0.29
 * 保證 > 45 度相位裕度。
 *
 * @clockDomain mano (dual-gear)
 */
interface ManoClockConfig {
  /** Gear 1 (快齒輪) 週期 -- 與 vedana-clock 對齊 */
  readonly fastGearPeriod: number;        // default: 50 (ms)

  /** Gear 2 (慢齒輪) 超時 -- 最大等待 LLM 回應時間 */
  readonly slowGearTimeout: number;       // default: 30000 (ms)

  /** 齒輪切換閾值 -- IGearArbiter 評估信心度低於此值則切換到 Gear 2 */
  readonly gearSwitchThreshold: number;   // default: 0.6

  /** Klesha 增益排程配置 -- 動態調整 gearSwitchThreshold */
  readonly kleshaModulation: KleshaThresholdConfig;

  /** Vitakka 看門狗 -- per-gear 連續 cycles 上限 */
  readonly maxConsecutiveGearCycles: Record<number, number>;  // default: { 1: 20 } [Plan27b N-Gear 泛化]

  /** Gear 1 合併視窗 -- 收集多少個 vedana-tick 的 bundles */
  readonly coalescingWindowTicks: number;  // default: 5

  /** 強制 `config.defaultGear` 後的 cooldown (以 Gear 1 cycles 計) */ // `[Plan27b 修整]`
  readonly forcedGear2Cooldown: number;   // default: 10
  /** VedanaEmergency 配置（可選） */ // `[Plan28 新增]`
  readonly vedanaEmergencyConfig?: VedanaEmergencyConfig;
}

/**
 * KleshaThresholdConfig -- 齒輪切換閾值的安全邊界。
 */
interface KleshaThresholdConfig {
  /** 基礎閾值 */
  readonly baseThreshold: number;     // default: 0.6
  /** 閾值地板 (GUARDIAN: 防止永不調用 LLM) */
  readonly minThreshold: number;      // default: 0.3
  /** 閾值天花板 (GUARDIAN: 防止永不使用 IGearArbiter/Gear 1) */
  readonly maxThreshold: number;      // default: 0.9
  /** 每個 Klesha 通道的權重 */
  readonly channelWeights: {
    readonly moha: number;     // 我癡
    readonly drishti: number;  // 我見
    readonly mana: number;     // 我慢
    readonly sneha: number;    // 我愛
  };
}
```

---

## 9. 資料流圖 (Data Flow Diagrams)

### 9.1 Layer 1: 根門 Bundle 形成

```
InputEvent arrives at IListener
  │
  ▼
SparshEvent formation ─────────────────── rupa-clock (10-50ms)
  │
  ▼
┌──────────────────────────────────────────────┐
│ CoarisingBundle computation (Strategy C)      │ vedana-clock (<1ms)
│                                              │
│  ┌─────────┐   ┌──────────┐   ┌──────────┐  │
│  │ vedana  │──→│  samjna   │──→│  cetana  │  │
│  │ (0.1ms) │   │(0.5ms,   │   │ (0.2ms)  │  │
│  │ valence │   │ rule-    │   │ tendency │  │
│  │ + inten.│   │ based)   │   │ + urgency│  │
│  └─────────┘   └──────────┘   └──────────┘  │
│       │                              │       │
│       ▼                              ▼       │
│  ┌────────────────┐   ┌───────────────┐      │
│  │ manasikara     │   │ SahajaContract│      │
│  │ (0.01ms,       │   │ (自動生成)    │      │
│  │  vijnana snap) │   │ staleness<1ms │      │
│  └────────────────┘   └───────────────┘      │
│                                              │
│  → CoarisingBundle { layer: 1, mode: 'fast' }│
└──────────────────────────────────────────────┘
```

### 9.2 Layer 2: N-Gear 齒輪選擇（IGearArbiter 快速路徑）

```
Layer 1 bundles (from all root gates)
  │
  ▼
ManoAggregator ──────────────────────── vedana-clock (~50ms)
  │
  ├── Klesha.perceive() ← vijnana-clock (0.03ms)
  │     → KleshaDistribution.mean per channel
  │     → gain-scheduled threshold
  │
  ├── gearArbiter.evaluate(bundles, threshold)  ← IGearArbiter plugin
  │     │
  │     ├─ confidence > threshold ──→ GearEvaluation (match!)
  │     │                              │
  │     │                              ▼
  │     │                   ManoCoarisingBundle
  │     │                   { layer: 2, mode: 'fast' }
  │     │                   staleness < 100ms
  │     │
  │     └─ confidence < threshold ──→ switch to config.defaultGear
  │
  └── → IVolition.deliberate() → Tool execution
```

> **Plan28 修整**: ManoAggregator 新增 VedanaEmergency 雙輸入調變。`vedanaFn` callback + `VedanaEmergencyConfig` 參數注入，計算 `effectiveBaseThreshold = config.baseThreshold + thresholdBoost`。VedanaEmergency 為純函數（不維護內部狀態），狀態由 ManoAggregator 管理。DEFAULT: intensityThreshold=0.8, sustainedTicks=5, maxThresholdBoost=0.15, cooldownTicks=10。詳見 Doc 42 §9.5。

### 9.3 Layer 2: Gear 2 慢速路徑

```
IGearArbiter confidence < threshold
  │
  ▼
IProvider plugin ──────────────────────── samjna-clock (0.5-30s)
  │
  ├── Context assembly (DharmaVisaya)
  │     ├── Layer 1 bundles (with vedana)
  │     ├── Klesha full distribution (Beta params)
  │     ├── Session context (alaya projections)
  │     └── IGuide system prompt
  │
  ├── IProvider.chat(context) ──→ LLM inference
  │     │
  │     └── 延遲: 500ms-30s (取決於提供者)
  │
  ├── ProposedAction
  │     │
  │     ▼
  │   ManoCoarisingBundle
  │   { layer: 2, mode: 'slow' }
  │   staleness = LLM latency
  │   SahajaContract: mutualConsistency=true
  │                   (LLM context 包含 vedana)
  │
  └── → IVolition.deliberate() → Tool execution
```

### 9.4 齒輪切換決策點

```
ManoAggregator receives Layer 1 bundles
  │
  ▼
┌─────────────────────────────────────────────┐
│           Gear Switch Decision               │
│                                             │
│  gearArbiter.evaluate(bundles)              │  ← IGearArbiter plugin
│    → confidence = 0.72                      │
│                                             │
│  Klesha gain-scheduled threshold:           │
│    θ(t) = clamp(0.6 + Σwᵢμᵢ(t), 0.3, 0.9) │
│    = clamp(0.6 + 0.05, 0.3, 0.9)           │
│    = 0.65                                   │
│                                             │
│  0.72 > 0.65 ?                              │
│    ├── YES → arbiter 推薦齒輪 (GearEvaluation)│
│    └── NO  → config.defaultGear (IProvider/LLM)│
│                                             │
│  Vitakka Watchdog:                          │
│    consecutiveGear1 = 15                    │
│    max = 20                                 │
│    15 < 20 → no forced switch               │
└─────────────────────────────────────────────┘
```

---

## 10. 實現分階段 (Implementation Phasing)

> ARCHIMEDES (#16) -- 工程可行性路線圖。

| Phase | 目標 | 內容 | 對應 Plan |
|-------|------|------|----------|
| Phase 1 | 標記與量測 | ExecutionLoop 每 tick = 意觸 (Layer 2 簡化)。增加 clock-domain 標記和計時事件。零破壞性變更。 | Plan26 |
| Phase 2 | IGearArbiter 介面定義 + 參考 plugin 實作 | IGearArbiter 介面定義。VasanaEngine 作為第一個參考實作 plugin。每個 IListener 有 mini-coarising (Layer 1)。匹配跳過 LLM。 `[Cycle 02-4 修整]` | Plan27 |
| Phase 3 | 多 IGearArbiter plugin chain + 動態切換 | 雙齒輪 mano-clock 動態切換。多 IGearArbiter plugin chain (first-win)。Klesha 增益排程。Vitakka 看門狗。 `[Cycle 02-4 修整]` | Plan27+ |
| Phase 4 | 完整五時鐘 | 五個獨立時鐘域。CRDT-like 狀態分區。Lamport 時間戳。Snapshot isolation。 | 遠期 |

**Phase 1 行動項目** (Cycle 02 M-10 延續):
1. 建立 `ClockDomain` 型別 (`'rupa' | 'vedana' | 'samjna' | 'samskara' | 'vijnana'`)
2. 在 ExecutionLoop 中注入 `clock:timing` 事件
3. MetricsCollector 報告每時鐘域延遲直方圖
4. CPU 額外開銷 < 1%，記憶體 < 1MB

**Phase 2 行動項目** `[Cycle 02-4 修整]`:
1. IGearArbiter 介面定義 — `evaluate(context: GearContext): Promise<GearEvaluation>`
2. VasanaEngine 作為第一個 IGearArbiter 參考 plugin 實作
3. VasanaRule 型別 (exact, regex, intent, tool_retry)
4. 接入 AgentCore (預設關閉；無 IGearArbiter plugin → 退化為純 Gear 2)
5. 規則表: 索引化 (hash map)、大小限制 (LRU 淘汰)、預編譯

**Phase 3 行動項目** `[Cycle 02-4 修整]`:
1. ManoClockConfig 完整實作
2. ClockDispatcher: Gear 1 / Gear 2 路由邏輯（ManoAggregator 純路由，同 EventBus 性質）
3. IGearArbiter Chain of Responsibility: first-win, early termination, priority ordering
4. 雙層超時: 100ms per-arbiter, 200ms total chain
5. Klesha gain-scheduled threshold + risk-weighted threshold
6. Vitakka watchdog + cooldown
7. Snapshot isolation 跨時鐘域

> DARWIN (#6) 的預測: Phase 3 (雙齒輪) 將足以應付 80%+ 的真實使用情境。Phase 4 除非 Agent 領域需要亞 10ms 感測響應 (機器人、即時交易)，否則可能過度工程化。

---

## 11. 演化分析 (Evolutionary Analysis)

> DARWIN (#6) -- R04 Sec 8 + R3 Debate 1。

### 11.1 System 1/System 2 平行 (Kahneman)

雙齒輪模型天然映射到 Kahneman 的雙系統認知架構:

| Kahneman | OpenStarry | 特性 |
|----------|-----------|------|
| System 1 (快速直覺) | Gear 1 / IGearArbiter plugin | 規則匹配、快取回應、確定性、低延遲 |
| System 2 (慢速審議) | Gear 2 / IProvider plugin | LLM 推理、全上下文、概率性、高延遲 |

Kahneman 的核心洞察: System 1 處理大部分日常決策，System 2 僅在 System 1 失敗時介入。OpenStarry 的雙齒輪模型精確地複製此模式 -- Gear 1 是預設的，Gear 2 僅在 IGearArbiter confidence < threshold 時啟動。 `[Cycle 02-4 修整]`

### 11.2 收斂設計比較

DARWIN 在多個獨立 AI agent 框架中發現相同的混合排程模式:

| 框架 | 快速路徑 | 慢速路徑 | 出現時間 |
|------|---------|---------|---------|
| OpenStarry (本設計) | IGearArbiter plugin (如 VasanaEngine) | IProvider plugin (LLM) | 2026-Q1 |
| AutoGPT v2 | "Instinct" system | Full reasoning chain | 2025-Q3 |
| LangChain LCEL | Conditional routing | Full chain | 2024-Q4 |
| Microsoft Semantic Kernel | Plan caching | Full planner | 2025-Q1 |

此收斂表明**混合排程模式是穩定的演化吸引子** (stable evolutionary attractor)。所有面臨 LLM 延遲壓力的 Agent 系統獨立演化至快取/快速路徑機制。

### 11.3 生物類比

在生物神經系統中，雙速認知處理是普遍模式:
- **杏仁核** (amygdala): 快速威脅評估，點估計 → 對應 IGearArbiter plugin + KleshaDistribution.mean
- **前額葉皮質** (prefrontal cortex): 慢速分析推理，完整概率評估 → 對應 IProvider plugin + 完整 Beta 分佈

OpenStarry 的 Klesha 雙層模型 (快速路徑用點估計，慢速路徑用完整分佈) 再現了這個生物模式。這不是巧合 -- 這是任何平衡速度與精度的系統必須滿足的基本設計約束。

---

## 12. 常見誤解 (Common Pitfalls)

| # | 誤解 | 正確理解 |
|---|------|---------|
| 1 | Mano-clock 是第六個獨立時鐘 | Mano-clock 是在 vedana-clock (Gear 1) 和 samjna-clock (Gear 2) 之間切換的**模式切換時鐘**，不新增時鐘域。 |
| 2 | Layer 1 CoarisingBundle 使用 LLM | Layer 1 **永不使用 LLM**。samjna 組件僅使用規則匹配 (sub-millisecond)。LLM 僅在 Layer 2 Gear 2 出現。 |
| 3 | 陳舊度 (staleness) 是隱藏的技術債 | 陳舊度通過 SahajaContract 元資料欄位**顯式宣告並在運行時監控**。每個 bundle 自我描述其品質。 |
| 4 | 齒輪切換是手動/靜態的 | 齒輪切換由 IGearArbiter 匹配結果 + Klesha 增益排程閾值**自動動態驅動**。 `[Cycle 02-4 修整]` |
| 5 | 多個 Agent 共享 mano-clock | 每個 Agent 的 mano-clock **完全獨立**。Agent 間協調使用獨立的 coordination-clock。 |
| 6 | Klesha 直接修改 IGearArbiter 規則 | Klesha 透過**增益排程**間接影響齒輪切換閾值，IGearArbiter plugin 本身保持確定性。Klesha 不直接修改 IGearArbiter 規則，而是影響齒輪切換的 gain-scheduled threshold。 `[Cycle 02-4 修整]` |
| 7 | 雙齒輪使 LLM 等待更久 | 雙齒輪使等待**更短** -- Gear 1 處理常規情境 (不需要 LLM)，僅新穎情境才觸發 Gear 2。 |

---

## 13. 與前期研究的整合 (Integration with Previous Cycles)

### 13.1 Cycle 02 M-10: 多速度架構需求

Cycle 02 工程行動計畫 [01_engineering_action_plan.md] 提出了多時鐘需求:

> Master: 「不同層級的運作有不同的速度，不是所有東西都跑在同一個時鐘上。」

Cycle 02 識別了五個時鐘域 (rupa/vedana/samjna/samskara/vijnana)，但**未解決**:
- vedana 與 samjna 跨時鐘如何保持俱生
- ManoAggregator 應在哪個時鐘域運行
- 齒輪切換的觸發機制

### 13.2 本設計如何滿足 M-10

| M-10 需求 | Cycle 02 狀態 | Cycle 02-3 解決方案 |
|-----------|--------------|-------------------|
| 五時鐘域識別 | 已定義 | 確認並加入 mano-clock 雙齒輪 |
| 跨時鐘俱生 | 未解決 (主要張力) | 雙層模型 + SahajaContract |
| ManoAggregator 時鐘 | 未定義 | 雙齒輪 mano-clock |
| 齒輪切換觸發 | 未定義 | IGearArbiter confidence < threshold + Klesha threshold + watchdog |
| 穩定性保證 | 提及但未形式化 | WIENER staleness ratio $\rho < 0.29$ |
| Agent 間獨立性 | 部分 (Coordination Daemon) | 完整: 獨立 mano-clock + coordination-clock |
| 實作路線圖 | Phase 1-4 初步 | Phase 1-4 細化，含具體行動項目 |

### 13.3 延後至後續 Cycle 的項目

| 項目 | 原因 | 預計 |
|------|------|------|
| Phase 4 完整五時鐘實作 | 需要 Phase 1-3 穩定後的量測數據 | 遠期 |
| IGearArbiter plugin 學習機制 (UQ-4) — VasanaEngine 作為第一個實作 plugin，如何從慢檔結果自動沉澱新規則 | 習氣形成需要完整 vedana 反饋迴路 `[Cycle 02-4 修整]` | Plan27+ |
| 跨機器分散式 coordination-clock | 需要 multi-machine 基礎設施 | Future |
| Speculative execution (ATHENA) | 正交優化，可獨立推進 | Plan27 |

---

## 14. Cycle 02-4 新增決議 `[Cycle 02-4 新增決議]`

> 以下決議來自 Cycle 02-4 研究，反映 VasanaEngine 外部化為 IGearArbiter plugin 後的架構調整。

### 14.1 IGearArbiter Chain of Responsibility (D1)

多個 IGearArbiter plugin 以 **Chain of Responsibility** 模式運行:

- **First-win**: 第一個 confidence > threshold 的 arbiter 決定齒輪選擇，後續 arbiter 不再評估
- **Early termination**: 一旦找到匹配，立即終止 chain 遍歷
- **Deterministic ordering**: 按 priority 排序，高優先級 arbiter 先評估
- **觀察性純粹**: 單一 `evaluate()` 方法，無副作用

```typescript
/**
 * IGearArbiter — 齒輪仲裁介面。
 *
 * 每個 IGearArbiter plugin 實作此介面。
 * Chain of Responsibility: first-win, early termination。
 * 觀察性純粹: evaluate() 不產生副作用。
 *
 * [Cycle 02-4 新增]
 */
interface IGearArbiter {
  /** 評估當前上下文，回傳齒輪建議與信心度 */
  evaluate(context: GearContext): Promise<GearEvaluation>;

  /** 此 arbiter 在 chain 中的優先級 (數字越小越先評估) */
  readonly priority: number;
}

interface GearEvaluation {
  /** 信心度 0.0-1.0 */
  readonly confidence: number;
  /** 建議的行動 (若 confidence > threshold) */
  readonly proposedAction?: ProposedAction;
  /** 評估耗時 (ms) — 用於超時監控 */
  readonly evaluationDuration: number;
}
```

### 14.2 雙層超時機制 (D1)

為防止 IGearArbiter chain 阻塞 mano-clock:

| 層級 | 超時 | 說明 |
|------|------|------|
| Per-arbiter | 100ms | 單一 IGearArbiter plugin 的最大評估時間，超時則 confidence=0 跳過 |
| Total chain | 200ms | 整個 chain 的最大總評估時間，超時則直接進入 Gear 2 |

### 14.3 G-0 ~ G-4 退化表 (Degradation Table) (D1)

| 等級 | IGearArbiter 配置 | IProvider 配置 | 路由行為 | 說明 |
|------|-------------------|---------------|---------|------|
| **G-0** | 無 | 無 | 核心存活但無法認知 | Phase 2.5 跳過，Phase 3 失敗 |
| **G-1** | 無 | 有 | 永遠 `config.defaultGear`（預設 2） | **與 v0.26.0-beta 向後完全相容** — 純 LLM 模式 |
| **G-2** | 單一 arbiter | 無 | 僅 Gear 1 | 僅能處理 arbiter 匹配到的場景，無 fallback |
| **G-3** | 單一 / 多個 arbiter | 有 | N-Gear routing | first-win arbiter chain + Gear 2 fallback + Klesha 增益調制 |
| **G-4** | 多個 arbiter + hot-swap | 有 | 運行時動態載入/卸載 | 遠期目標 (Phase 3+)，需 plugin 生命週期管理 |

> **零內建能力原則**: G-0/G-1 證明 Agent 不需要任何 IGearArbiter 即可運行。IGearArbiter 是**能力** (有規則庫、匹配邏輯、信心度評分)，不是**機制**，因此必須是 plugin 而非核心組件。完整定義見 Doc 42 §八。

> **Plan27a 修整 (OQ-A3)**: 退化表描述能力深度，不是齒輪數量。N-Gear 泛化後 `GearAction = number | 'abstain'`，齒輪數量不限於二。詳見 Doc 42 §八。

### 14.4 風險加權閾值 (Risk-Weighted Threshold) (D5)

取代全域 confidence cap，改用風險加權閾值:

$$\text{adjustedThreshold} = \theta_{\text{base}} + \Delta_{\text{risk}}(\text{action})$$

其中:
- $\theta_{\text{base}}$ = Klesha gain-scheduled threshold（同 Section 6.2 公式）
- $\Delta_{\text{risk}}(\text{action})$ = 行動的風險增量（高風險行動 → 更高閾值 → 更傾向 Gear 2 深度推理）

**maxConfidenceByGear**: `Record<number, number>`（預設 `{ 1: 0.95 }`，N-Gear 泛化），可配置的部署參數 (D5)。即使 IGearArbiter confidence = 1.0，若超過該齒輪的 maxConfidence 仍觸發額外驗證。這是防止過度自信的安全閥。`[Plan27a 修整: maxGear1Confidence → maxConfidenceByGear]`

```typescript
/**
 * ManoAggregatorConfig — ManoAggregator 完整配置（含風險加權閾值）。
 *
 * [Cycle 02-4 新增] [Plan27a 修整: ManoAggregatorTimeoutConfig → ManoAggregatorConfig，
 *  maxGear1Confidence → maxConfidenceByGear，N-Gear 泛化]
 */
interface ManoAggregatorConfig {
  /** 單一 arbiter 最大評估時間 (ms) */
  readonly perArbiterMs: number;           // default: 100
  /** 整個 chain 最大總評估時間 (ms) */
  readonly chainMs: number;                // default: 200
  /** 每齒輪信心度上限 — 超過此值仍需額外驗證 */
  readonly maxConfidenceByGear: Record<number, number>;  // default: { 1: 0.95 }（N-Gear 泛化）
  /** 無 IGearArbiter plugin 時的預設齒輪 */
  readonly defaultGear: number;            // default: 2
  /** 基礎閾值 (由 Klesha gain scheduling 動態調整) */
  readonly baseThreshold: number;          // default: 0.6
  /** 行動風險增量函數 */
  readonly riskDelta: (action: ProposedAction) => number;
}
```

### 14.5 與既有架構的整合

| 既有概念 | Cycle 02-4 調整 | 影響 |
|---------|----------------|------|
| VasanaEngine (核心組件) | → IGearArbiter plugin (VasanaEngine 為第一個實作) | 從核心移至 plugin |
| VitakkaEngine (獨立組件) | → 移除，功能由 IProvider.chat() 直接承擔 | 減少一層抽象 |
| ManoAggregator | → 純路由機制 (if/else)，同 EventBus 性質 | 明確定位 |
| 齒輪切換閾值 | → 新增風險加權 adjustedThreshold | 更精細的控制 |
| 全域 confidence cap | → 替換為 `maxConfidenceByGear: Record<number, number>` 可配置參數 `[Plan27a 修整]` | 部署彈性（N-Gear 泛化） |

---

*R3 Debate 1 共識決議完整文件化。Cycle 02-4 修整: VasanaEngine 外部化為 IGearArbiter plugin，VitakkaEngine 移除，ManoAggregator 明確為純路由機制。雙層雙齒輪模型為 OpenStarry 的混合排程架構提供了佛學哲學基礎、控制理論穩定性保證、和工程分階段實現路線。*

*跨文件參照: Doc 29 (五蘊連結), Doc 30 (觸->俱生), Doc 42 (IGearArbiter 介面規格), Doc 37 (Klesha 增益排程), Doc 38 (雙階段審議), Doc 44 (安全架構總覽)*
