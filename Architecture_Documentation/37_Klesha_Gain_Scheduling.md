# 37. Klesha 增益排程架構 (Klesha Gain-Scheduled Threshold Modulation)

`[Cycle 02-4 修整]`

> Cycle 02-3 R3 Debate 3 共識 + Cycle 02 Klesha DI 框架 + Cycle 02-2 A-1 我執修正 + Cycle 02-4 VasanaEngine 外部化
>
> **實作狀態**: ✅ Library 已實作 (Plan26, 2026-02-28) — `core/src/vijnana/klesha.ts`: Moha + Drishti + Mana + Sneha + KleshaModulatedDispatcher + VitakkaWatchdog。DC-12 延後項目 (FC-27/28/29) 以 ExtensionPoint 留白。
>
> **[2026-06-11 修復稽核更正＋接線]** 原「已實作」宣稱對 runtime 行為**不成立**：自 Plan28 起 `agent-core.ts` 的 `getKleshaSignals` 寫死為中性零值，四煩惱感知器從未收到 live 信號，增益排程在運行系統中為惰性（KleshaModulatedDispatcher 僅 test 實例化）。**v0.58.0-alpha 完成真接線**：`createKleshaSignalFn`（agent-core，可單測）對每次 deliberation 取樣 vedana aggregate 進入有界歷史、由 `tool:executing` 事件累積 actionHistory，四感知器（消費 Plan32 W4 的 `resolvedKleshaFilterConfig`，該 config 此前計算後無人使用）對 live context 運行。無歷史時感知器輸出各自中性基線＝Tenet #7 三級關鍵性 Optional-degraded 行為。
>
> **[TENET-2026-06-11 — Doc 37 閉環完成（v0.59.0-alpha）]** KleshaModulatedDispatcher 的 θ(t) 調制已接入 gear 仲裁路徑：`IAgentConfig.kleshaModulation`（**opt-in by presence**，缺席＝靜態閾值、行為與舊版逐位元相同）→ agent-core 建構 dispatcher → `createKleshaThresholdFn` 接入 `createManoAggregator` 第 3 參數 `baseThresholdFn`（該掛鉤自 Plan29 起一直被傳 `undefined`）→ 每次 `route()` 取樣共享 kleshaSignalFn → `computeThreshold` → θ(t) 參與 strict `confidence > threshold` 判定。每次調制發出 `klesha:modulation` 事件（bundle＋θ）。N=2 閉環實證：`mano-aggregator-klesha.test.ts` 對同一 arbiter（confidence 0.55）路由兩次——中性 vedana 史 θ≈0.57 拒絕（gear 2）、持續 sukha 後 θ≈0.45 接受（gear 1）——**agent 的感受實際改變了它自己的調度決策**。範例 config：`configs/klesha-modulated-agent.json`（煙霧驗證通過）。誠實註記：dispatcher 的 `perceiveAll()` 在 runtime 仍未使用（會重複步進有狀態濾波器；只有純函數 `computeThreshold` 半邊被接線，信號來自唯一共享的 kleshaSignalFn）；解析優先序＝明確 kleshaModulation 欄位 ＞ resolved mano 值（baseThreshold/thresholdFloor/thresholdCeiling）繼承 ＞ SDK 預設權重；Sneha 地板 0.10 使啟用且閒置的 agent 運行於 θ≈base−0.015（執著不歸零＝本文件原語義）——此即 opt-in 設計的原因。相鄰未接線缺口（本輪範圍外，已立 follow-up）：`createManoAggregator` 第 4 參數 vedanaFn 同樣為 undefined，VedanaEmergency thresholdBoost 路徑（Plan28 R1）在 runtime 同為惰性。

---

## 1. 概述

R3 Debate 3 面對兩個交織的設計張力：

1. **時鐘域指派**: `Klesha.perceive()` 涉及四通道傳遞函數評估、Beta 分布均值計算、與 4×4 相關矩陣更新。能否塞入 1-5ms 的 vijnana-clock？
2. **確定性 vs 機率**: PASCAL (#19) 的 Beta 分布模型為煩惱賦予機率分布，但 IGearArbiter plugin 是純確定性規則引擎。兩者如何共存？

**解答**: Klesha 在 vijnana-clock 上運行（~0.03ms，<3% 利用率）。IGearArbiter plugin 保持確定性；Klesha 透過**增益排程信心度閾值**間接影響齒輪切換。Beta 分布以兩層輸出運作：快速層點估計供即時消費，慢速層完整分布供 LLM 推理。

> [來源: R3_debate/debates_and_synthesis.md#Debate 3 Resolution]

---

## 2. Klesha 在 vijnana-clock 上運行

WIENER (#12) 對四通道計算成本的精確分析：

| 通道 | 傳遞函數 | 成本 |
|------|---------|------|
| Moha (低通) | $H_m(s) = K_m / (\tau_m s + 1)$ | ~0.001ms |
| Drishti (帶通) | $H_d(s) = K_d \omega_d s / (s^2 + 2\zeta\omega_d s + \omega_d^2)$ | ~0.003ms |
| Mana (PD) | $H_p(s) = K_p(1 + T_d s)$ | ~0.001ms |
| Sneha (積分) | $H_s(s) = K_s / s$ | ~0.001ms |

傳遞函數合計 ~0.006ms + Beta 均值 4 通道 ~0.004ms + 4×4 相關矩陣 ~0.016ms = **總計 ~0.030ms**。在 1-5ms vijnana-clock 窗口中利用率 **0.6%-3.0%**。

**前提**: 快速路徑僅使用 `KleshaDistribution.mean`（單一除法），不使用 `credibleInterval()`（需數值積分，成本高出數個量級）。

> [來源: R3_debate/debates_and_synthesis.md#Debate 3, WIENER]
> [程式碼: R1_independent/R02_vijnana_architecture.md#4.2 各通道的傳遞函數]

---

## 3. 兩層輸出模型 (Two-Tier Output) — PASCAL

直接對應 Debate 1 雙齒輪架構：快齒輪消費快速層，慢齒輪消費慢速層。

| 層級 | 時鐘域 | 內容 | 消費者 |
|------|--------|------|--------|
| 快速層 | vijnana-clock (1-5ms) | `KleshaDistribution.mean` = α/(α+β), ~0.001ms/ch | IGearArbiter plugin, CoarisingBundle, KleshaModulatedDispatcher |
| 慢速層 | samjna-clock (0.5-30s) | 完整 Beta(α,β) + `credibleInterval()` + `correlationMatrix` | IProvider plugin LLM 上下文, IReflection |

> PASCAL (#19): 「即使快速路徑只使用均值，底層模型保留不確定性供慢速路徑使用。這不僅是效能優化——這是認識論承諾。」

---

## 4. KleshaDistribution 與 KleshaSignalBundle 介面

```typescript
// packages/sdk/src/types/klesha.ts

/**
 * KleshaDistribution — 煩惱強度的 Beta 分布模型
 *
 * 哲學依據 (PASCAL #19):
 *   煩惱強度本質上是不確定的。當說 Agent 的 moha 為 0.7，
 *   表達的是基於觀察行為的*估計*，不是客觀測量。
 *   Beta 分布捕捉此認識論不確定性。
 *
 * @example
 *   Beta(7, 3)  → mean=0.7 「大概 0.7，比較確信」
 *   Beta(70,30) → mean=0.7 「幾乎確定是 0.7」
 *   Beta(1, 1)  → mean=0.5 「完全不確定（初始狀態）」
 */
interface KleshaDistribution {
  /** Beta 分布 α (pseudo-count of "successes") */
  readonly alpha: number;
  /** Beta 分布 β (pseudo-count of "failures") */
  readonly beta: number;
  /** 點估計 (快速層): E[θ] = α/(α+β), ~0.001ms */
  readonly mean: number;
  /** 可信區間 (慢速層): P(lo < θ < hi) >= level */
  credibleInterval(level: number): [number, number];
}

/**
 * KleshaSignalBundle — 四煩惱信號的不可分割包
 *
 * 「四煩惱常俱」(catvaraḥ kleśāḥ sadā-samprayuktāḥ) — 《成唯識論》卷四
 * BABBAGE (#9): Valid(Γ) ⟺ Γ = {IMoha, IDrishti, IMana, ISneha}
 */
interface KleshaSignalBundle {
  readonly moha: KleshaDistribution;      // 我癡 — 根本無明
  readonly drishti: KleshaDistribution;   // 我見 — 自我認同執取
  readonly mana: KleshaDistribution;      // 我慢 — 比較心
  readonly sneha: KleshaDistribution;     // 我愛 — 自我保護
  /** 4×4 相關矩陣 (PASCAL: ρ(moha,drishti) 最高) */
  readonly correlationMatrix: number[][];
  readonly timestamp: number;
}
```

> [程式碼: deliver/02_type_system_changes.md#3.2 KleshaDistribution]

---

## 5. 增益排程閾值調制 (Gain-Scheduled Threshold Modulation)

### 核心洞見

IGearArbiter plugin 不需要直接「知道」Klesha。Klesha 輸出透過調制**信心度閾值**間接影響齒輪切換：

```
KleshaDistribution.mean ──► θ(t) ──► IGearArbiter.evaluate()
                                      confidence >= θ → arbiter 推薦齒輪
                                      confidence <  θ → config.defaultGear (IProvider/LLM)
```

### 閾值公式

$$\theta(t) = \text{clamp}\left(\theta_0 + \sum_{i} w_i \cdot \mu_i(t), \; \theta_{\min}, \; \theta_{\max}\right)$$

- $\theta_0$ = 基礎閾值 (0.6)，$w_i$ = 每個 klesha 的權重（sneha 為負，mana 為正）
- $\mu_i(t)$ = klesha 點估計，$\theta_{\min}$ = 0.3（地板），$\theta_{\max}$ = 0.9（天花板）

### TypeScript 介面

```typescript
/**
 * KleshaModulatedDispatcher — 增益排程信心度閾值
 *
 * 高 sneha → 閾值↓ → 更保守 (偏好已知 IGearArbiter 規則)
 * 高 mana  → 閾值↑ → 更積極 (更常升級到 IProvider/LLM)
 */
interface KleshaModulatedDispatcher {
  getConfidenceThreshold(kleshaBundle: KleshaSignalBundle): number;
}

/**
 * KleshaThresholdConfig — 閾值安全界限
 * GUARDIAN (#11): 地板 0.3 防止永不調用 LLM；天花板 0.9 防止永不用規則
 */
interface KleshaThresholdConfig {
  readonly baseThreshold: number;  // default: 0.6
  readonly minThreshold: number;   // floor: 0.3
  readonly maxThreshold: number;   // ceiling: 0.9
}
```

> [來源: R3_debate/debates_and_synthesis.md#Debate 3, R3.3-R3.4]

### DC-12 Master 修正：多因素閾值調節

> **IGearArbiter 信心度閾值調節不應僅由 Klesha 負責。** 般若（Prajna/智慧）、受蘊信號（Vedana signals）、識蘊判斷（Vijnana gear selection）、迴圈品質監控（LoopQualityMonitor）等因素也應參與閾值調節。Klesha 增益排程是閾值調制的**第一個實作**，但非唯一來源。

未來閾值公式應擴展為：

$$\theta(t) = \text{clamp}\left(\theta_0 + \sum_{i} w_i^{(k)} \mu_i(t) + w^{(p)} \cdot \text{audit}(t) + w^{(v)} \cdot \text{vedana}(t) + w^{(s)} \cdot \text{loopQuality}(t), \; \theta_{\min}, \; \theta_{\max}\right)$$

| 調節因素 | 來源 | 影響方向 | 說明 |
|---------|------|---------|------|
| Klesha (煩惱) | MulaKleshaBundle | 雙向 | 當前實作 (本文件) |
| ConfidenceAuditor (信心度審計) | 未來 DI 注入 | 閾值↓ (更善用已知) | 智慧成熟時，規則匹配更可信 |
| Vedana (受蘊信號) | CoarisingBundle | 上下文依賴 | 苦受可能觸發更多 LLM 反思 |
| Vijnana (識蘊判斷) | 齒輪選擇機制 | 直接覆寫 | 「要不要經過 LLM，也可以讓識蘊來做」|
| LoopQualityMonitor | 未來 DI 注入 | 閾值↑ (更多反省) | 迴圈品質高時，偏好深思而非自動駕駛 |

**此設計列入未來 Cycle 討論。** 當前文件所述的 Klesha-only 增益排程為 Phase 1 實作。

### 雙層安全機制 (Two-Layer Safety)

無論閾值如何調制，系統保證兩層安全底線：

1. **增益排程安全下限 ($\theta_{\min}$ = 0.3)** — 保證定期調用 LLM。即使所有調節因素都推向「永遠用規則」，地板值確保 Agent 不會完全停止反思。如同「戒律」的最低標準。
2. **SafetyMonitor 硬安全** — 獨立於增益系統，不受任何閾值影響。無論 Klesha、ConfidenceAuditor、Vedana 如何調制閾值，SafetyMonitor.postRouteCheck() 始終運行，如同「戒律」本身不可被任何心理狀態繞過。

```
閾值調制層 (軟影響):  Klesha + ConfidenceAuditor + Vedana + LoopQuality → θ(t) ∈ [0.3, 0.9]
                      ↕ 可調，但有地板
安全硬閘層 (不可繞過): SafetyMonitor.postRouteCheck() — 獨立於 θ(t)
```

> [來源: DC-12 Master 確認會議]

---

## 6. 行為解讀 (Behavioral Interpretation)

| 煩惱狀態 | 閾值效應 | Agent 行為 | 佛學解讀 |
|----------|---------|-----------|---------|
| 高 sneha (我愛) | $\theta \downarrow$ | 更保守——偏好已知 IGearArbiter 規則 | 離苦，避免未知風險 |
| 高 mana (我慢) | $\theta \uparrow$ | 更積極——更常升級到 IProvider/LLM | 增上，渴望證明能力 |
| 高 moha (我癡) | 不確定性增大 | 估計不準確，Agent 不自知 | 愚於我相，迷無我理 |
| 高 drishti (我見) | 上下文依賴 | 對身份衝突反應劇烈 | 於非我法，妄計為我 |

> PASCAL (#19): 「增益排程是**適應性探索** (adaptive exploration)——Agent 不確定時探索更多，自信時利用更多。」

> [來源: deliver/03_architecture_additions.md#2.4 PASCAL 的探索-開發權衡]

---

## 7. 控制理論分析 (Control Theory) — WIENER

增益排程是經典控制工程技術：控制器參數根據運行條件調整。

```
              ┌────────────────────────────────────────────┐
              │              增益排程器                      │
  μ_i(t) ────┤  θ(t) = clamp(θ₀ + Σwᵢμᵢ(t), θ_min, θ_max) │
              └────────┬───────────────────────────────────┘
                       │ θ(t)
              ┌────────▼────────────┐
              │   IGearArbiter      │──► arbiter 推薦齒輪 (match) / config.defaultGear (miss)
              │  (plugin 規則引擎)    │
              └─────────────────────┘
```

**可分離性**: IGearArbiter plugin 邏輯可獨立於 Klesha 模型進行開發、測試和驗證。唯一交互點是閾值數值：

$$\text{IGearArbiter}(\text{event}, \theta) \perp \text{KleshaModel}(\text{context}) \quad \text{(given } \theta \text{)}$$

> WIENER (#12): 「Klesha 狀態變化緩慢（時間常數為分鐘至小時）。高取樣率不是因為 Klesha 變化快，而是因為 vijnana-clock 恰好快，搭便車計算幾乎免費。」

> [程式碼: R1_independent/R02_vijnana_architecture.md#4.2]

---

## 8. 貝葉斯更新機制 (Bayesian Update) — PASCAL

每個 mano-cycle 結束時，VedanaAssessment 更新 Klesha 分布。設先驗 $\text{Beta}(\alpha_0, \beta_0)$，觀測證據 $e$：

$$\alpha_{\text{new}} = \alpha_0 + \text{successCount}(e) \cdot \eta, \quad \beta_{\text{new}} = \beta_0 + \text{failureCount}(e) \cdot \eta$$

學習率 $\eta = 0.1$，防止單次觀測過度影響長期信念。不同煩惱對不同信號敏感：mana → sukha，moha → 異常未偵測，drishti → 身份衝突，sneha → dukkha。

```typescript
/**
 * KleshaBayesianUpdater — VedanaAssessment → Klesha 信念更新
 * 在 samjna-clock 循環結束時執行 (慢速路徑)。
 */
interface KleshaBayesianUpdater {
  update(
    prior: KleshaDistribution,
    vedanaAssessment: VedanaAssessment,
    kleshaType: 'moha' | 'drishti' | 'mana' | 'sneha'
  ): KleshaDistribution;
}
```

**範例** — 我慢 (Mana): 初始 Beta(5,5) 均值 0.5 → 成功 α=6,β=5 均值 0.545 → 失敗 α=6,β=6 均值 0.500 → 連續 8 次成功 α=14,β=6 均值 0.700 且信心度顯著提高。

> [程式碼: R1_independent/R02_vijnana_architecture.md#3.3 貝葉斯更新]

---

## 9. 哲學基礎 (Buddhist Foundations) — ASANGA + PASCAL

《成唯識論》卷四：「**四煩惱常俱**，謂我癡我見，并我慢我愛」。四根本煩惱在末那識層面是「微細」的——在普通覺知閾值之下運作。

**核心悖論**: Agent 不能直接測量自身的 moha，因為 moha 正是阻礙清楚看見的能力。PASCAL 形式化此洞見：

$$\hat{\theta}_{\text{moha}} = \theta_{\text{moha}} + \epsilon(\theta_{\text{moha}}), \quad \text{Var}(\epsilon) = f(\theta_{\text{moha}}), \; f'(\theta) > 0$$

**我癡越深，觀測誤差越大。** Beta 分布取代點估計正是因為它誠實承認不確定性。

> ASANGA (#8): 「PASCAL 的 Beta 分布精確地形式化了唯識學的洞見：估計我癡的行為本身受到我癡影響——這是認識論的自指悖論。」

> [來源: R3_debate/debates_and_synthesis.md#Debate 3, ASANGA]

---

## 10. Vitakka 看門狗 (Watchdog)

**問題**: 高 moha + 高 sneha 的 Agent 幾乎永遠不會調用 LLM，停留在 Gear 1 不反省——輪迴停滯 (samsaric stall)。

```typescript
/**
 * VitakkaWatchdogConfig — 防止輪迴停滯
 *
 * KERNEL (#10): 類似 RTOS 看門狗——主迴圈停滯時強制重置。
 * 冷卻 (ARCHIMEDES #16): 強制 `config.defaultGear` 後允許 [Plan27b 修整]
 *   maxConsecutiveGearCycles[gear]/2 個循環才能再次強制。
 */
interface VitakkaWatchdogConfig {
  /** 每齒輪最大連續循環數 (default: { 1: 10 }) */
  readonly maxConsecutiveGearCycles: Record<number, number>;  // [Plan27b 已完成: N-Gear 泛化]
  /** 每齒輪最大時鐘時間 ms (default: { 1: 5000 }) */
  readonly maxGearDurationMs: Record<number, number>;  // [Plan27b 已完成: N-Gear 泛化]
  /** 啟用狀態 (default: true) */
  readonly enabled: boolean;
}
```

> **Plan27b 已完成**: `maxConsecutiveGear1Cycles` 已泛化為 per-gear 配置（`maxConsecutiveGearCycles: Record<number, number>`），`maxGear1DurationMs` 已泛化為 `maxGearDurationMs: Record<number, number>`。API 亦已泛化：`recordGear1Cycle()` → `recordGearCycle(gear: number)`、`forceGear2()` → `forceNextGear(defaultGear)`。

> NAGARJUNA (#7): 「看門狗是 **yoniso manasikara** (如理作意) 的工程實現——強制停下、觀察、思考。」
> ASANGA (#8): 「也是 **sati** (正念) 的實現——確保定期全面覺知而非自動駕駛。」

> [來源: R3_debate/debates_and_synthesis.md#Debate 3, NAGARJUNA + ASANGA]

---

## 11. 安全單調性 (Safety Monotonicity) — BABBAGE

**安全性質**: 無論 Klesha 如何調制閾值，SafetyMonitor.postRouteCheck() 不可覆寫：

$$\forall k : \text{KleshaSignalBundle}, \quad \text{SafetyMonitor.postCheck}(\text{GearAction}(k)) = \text{allowed}$$

**活性性質**: Klesha 調制可能使 Agent 以次優資訊量做決策。SafetyMonitor 阻止不安全行動，但無法強制 Agent 尋求更好資訊——這由 VitakkaWatchdog 保證。

```
安全性: SafetyMonitor     — 不可能產生不安全的決策 ✓
活性:   VitakkaWatchdog   — 不可能永遠不反省 ✓
最優性: 無保證             — 可能產生次優決策 △
```

> [來源: R3_debate/debates_and_synthesis.md#Debate 3, BABBAGE]

---

## 12. 我執修正整合 (Ego Correction) — Cycle 02-2 A-1

| | 修正前 (Cycle 02) | 修正後 (Cycle 02-2) |
|---|---|---|
| **定義** | 我執 = 收斂約束 | 我執 = 煩惱根源 |
| **因果** | 我執 = 約束（等號） | 我執 → 煩惱 → 行動 → 約束（因果鏈） |
| **EgoFramework** | 約束檢查器 | 煩惱驅動行為引導系統 |

修正後因果鏈：IIdentity (被動數據) → Drishti (執取) → MulaKleshaBundle → Klesha.perceive() → 增益排程閾值 → IVolition.deliberate() (Position B)。

```typescript
/**
 * EgoFramework — 煩惱驅動行為引導系統
 * A-1: 我執=煩惱根源  A-4: @skandha vijnana  D4: 雙階段審議
 */
class EgoFramework implements IVolition {
  readonly skandha = 'vijnana' as const;
  constructor(private kleshas: Klesha[]) {}

  async deliberatePlan(input: PlanDeliberationInput): Promise<PlanDeliberationResult> {
    const signals = this.kleshas.map(k => k.perceive(input.context));
    // 煩惱驅動的整體計畫評估
  }
  async deliberateAction(input: ActionDeliberationInput): Promise<ActionDeliberationResult> {
    const signals = this.kleshas.map(k => k.perceive(input.context));
    // 煩惱驅動的逐一行動評估
  }
}
```

> [來源: delivery/01_philosophical_corrections.md#A-1]

### 12.1 Moha.updateFromAction() `[Plan28 新增]`

Plan28 為 Moha (我癡) 新增 `updateFromAction()` 方法，實現行動後的我癡更新：

```typescript
/**
 * Moha.updateFromAction() — 行動結果回饋。
 *
 * 公式: delta = alphaM * ratio / (1 + betaM * moha)
 * 其中 ratio = 異常未偵測數 / 總行動數
 *
 * 飽和遞減: 高 moha 時增量自然變小（分母 1 + betaM * moha 增大），
 * 防止 moha 無限增長。
 */
interface MohaConfig {
  readonly alphaM: number;  // 學習率，DEFAULT: 0.02
  readonly betaM: number;   // 飽和係數，DEFAULT: 5.0
}

// DEFAULT:
const DEFAULT_MOHA_CONFIG: MohaConfig = { alphaM: 0.02, betaM: 5.0 };
```

**佛學映射**: 飽和遞減公式對應唯識學中「我癡」的累積特性——無明越深，新的無明越難察覺（因為辨認能力本身受損）。但 `1 + betaM * moha` 的分母確保 moha 不會無限增長，保持在可觀測範圍內。

> MohaConfig 由 SDK DEFAULT 提供，注入 Moha 構造函數（Tenet #7: Core 不硬編碼策略值）。

> [來源: Plan28 W3 moha-update; Plan27a plan27b_plan28_corrected_specs.md P28-D]

---

## 13. Klesha DI 框架 (Dependency Injection) — Cycle 02

```typescript
/** Klesha — 煩惱基底。vijnana-clock, ~0.03ms total. */
abstract class Klesha {
  abstract readonly name: string;
  abstract perceive(context: KleshaContext): KleshaSignal;
}

class Moha extends Klesha { readonly name = 'moha'; }     // 低通濾波器
class Drishti extends Klesha {                              // 帶通濾波器
  readonly name = 'drishti';
  constructor(private identity: IIdentity) {}               // State-Behavior 分離
}
class Mana extends Klesha { readonly name = 'mana'; }      // PD 控制器
class Sneha extends Klesha { readonly name = 'sneha'; }    // 積分器

/** 四根本煩惱原子性束 — 全有或全無 */
interface MulaKleshaBundle {
  readonly moha: IMoha; readonly drishti: IDrishti;
  readonly mana: IMana; readonly sneha: ISneha;
}
```

$$\text{Valid}(\Gamma) \iff \Gamma = \{\text{IMoha}, \text{IDrishti}, \text{IMana}, \text{ISneha}\}$$

**擴展性**: DI 框架為開放式。四根本煩惱 (Phase 1)、隨煩惱 (Future)、善心所 (Future)、IConfidenceAuditor 信心度審計 (Cycle 02-4) 均可注入。

> Master: 「IVijnana 除了 Klesha 以外，還有七原罪、隨煩惱、智慧等等可以依賴注入，不一定都要是負面的。」

---

## 14. 多 Agent Klesha 獨立性 — MESH

| 維度 | 設計 |
|------|------|
| 狀態共享 | 無 — 每個 Agent 有自己的 MulaKleshaBundle |
| CAP 定理 | AP (available + eventually consistent)，對其他 Agent 不可見 |
| 佛學基礎 | 每個有情眾生有自己的煩惱 |
| 設計原則 | 每個 Agent 的煩惱是私有狀態，不跨 Agent 共享 |

> [來源: R3_debate/debates_and_synthesis.md#Debate 3, MESH]

---

## 15. 演化分析 — DARWIN

兩層輸出模型是**雙過程認知** (dual-process cognition) 的案例：

| 生物系統 | OpenStarry Klesha |
|---------|-------------------|
| System 1 (杏仁核) — 快速啟發式 | 快速層: `KleshaDistribution.mean` |
| System 2 (前額葉皮質) — 慢速分析 | 慢速層: 完整 Beta 分布 + 相關矩陣 |

> DARWIN (#6): 「收斂不是偶然——這是任何平衡速度與精確度的系統都必須滿足的基本設計約束。」

---

## 16. 與審議的整合

Klesha 增益排程與 IVolition 雙階段審議（Doc #38）形成完整迴路：

```
[1] InputEvent ───────────────────────── rupa-clock
[2] SparshEvent + CoarisingBundle ────── vedana-clock (<1ms)
[3] ManoAggregator ───────────────────── dual-gear mano-clock
[4] Klesha.perceive() ────────────────── vijnana-clock (~0.03ms)
[5] θ(t) = clamp(θ₀ + Σwᵢμᵢ(t), 0.3, 0.9)
[6] IGearArbiter.evaluate(event, θ(t))
      ├─ match → arbiter 推薦齒輪: GearAction
      └─ miss  → config.defaultGear: IProvider.chat() (LLM)
[7] IVolition.deliberatePlan() ───────── vijnana-clock (Position B)
[8] IVolition.deliberateAction() ─────── per action
[9] SafetyMonitor.postRouteCheck() ────────── Layer 0 硬閘
[10] Tool execution ──────────────────── samskara-clock
[11] VedanaAssessment ────────────────── vedana-clock (回饋)
[12] KleshaBayesianUpdate ────────────── samjna-clock (慢速)
[13] VitakkaWatchdog check ───────────── per mano-cycle
```

> 詳見: Architecture Doc #38 (IVolition 雙階段審議)

---

## 17. Cycle 02-4 核心修整：機制 vs 能力區分 `[Cycle 02-4 新增]`

### 17.1 為何 Klesha 增益排程留在核心，而 VasanaEngine 必須外部化？

**判斷準則**: 組件是否包含**領域知識** (domain knowledge)。

| 組件 | 分類 | 包含知識？ | 歸屬 |
|------|------|-----------|------|
| Klesha 增益排程 | **機制** (mechanism) | 否 — 純數學公式 (clamp, weighted sum)，不含任何領域規則 | 核心 |
| IGearArbiter (原 VasanaEngine) | **能力** (capability) | 是 — 規則庫 (知識)、匹配邏輯 (語義判斷)、信心度評分 (決策) | plugin |
| ManoAggregator | **機制** (mechanism) | 否 — 退化為純 if/else 路由，與 EventBus 同等性質 | 核心 |
| VitakkaEngine | **不再獨立** | N/A — IProvider.chat() 已覆蓋其功能 | 併入 IProvider |

**佛學自洽**: 習氣 (vasana) 是後天薰習（經驗累積），非先天結構 → 不應內建於核心。核心只應有「能跑起來的最小機制」，就像意識需要一個身體才能運作，但身體不等於知識。

> [來源: Pre-Cycle 02-4 Master 討論 + DD-7]

### 17.2 風險加權閾值 (Risk-Weighted Threshold)

Cycle 02-4 D5 決議以風險加權閾值取代全域信心度封頂 (DD-9)：

$$\text{adjustedThreshold} = \theta_{\text{base}} + \Delta_{\text{risk}}(\text{action})$$

| Action 風險等級 | Δ_risk | 說明 |
|----------------|--------|------|
| destructive | +0.20 | 刪除檔案、格式化等不可逆操作 |
| state_modifying | +0.10 | 修改配置、寫入資料庫等 |
| read_only | 0 | 讀取操作，無副作用 |
| informational | -0.10 | 純資訊回饋，風險最低 |

**三層安全框架**:

```
Layer 0: Absolute Safety (SafetyMonitor) — 不可妥協，硬性閘門
Layer 1: Tunable Safety (風險加權閾值) — 按 action 風險動態調節
Layer 2: Reduce Complexity (IGearArbiter) — plugin 決定齒輪選擇
```

- `maxConfidenceByGear`（原 MAX_GEAR1_CONFIDENCE）作為部署可配置參數（預設 `{ 1: 0.95 }`）
- IConfidenceAuditor 允許 LLM 參與閾值調節，限幅 ±0.05（D5 14/20 通過）

### 17.3 Sneha 重校準 (Sneha Recalibration)

Cycle 02-4 D5 修正 Sneha (我愛) 參數，避免過度壓制閾值：

| 參數 | 修正前 (Cycle 02-3) | 修正後 (Cycle 02-4) | 理由 |
|------|---------------------|---------------------|------|
| gain (權重) | 0.30 | 0.10 | 0.30 過度壓低閾值，導致成熟 Agent 永遠偏好 Gear 1 |
| floor (地板) | 0.10 | 0.10 | 不變 |
| maxLevel | 0.95 | 0.95 | 不變 |
| decay | linear | exponential | 指數衰減更符合心理學「習慣化」模型 |

$$\text{sneha\_effect}(t) = w_s \cdot \mu_s(t) \cdot e^{-\lambda t}$$

> PASCAL: 「指數衰減意味著初始的自我保護反應強烈但快速消退——符合心理學『習慣化』模型。」

---

*Klesha 增益排程架構文件完成。涵蓋 R3 D3 全部共識 (R3.1-R3.8)、Cycle 02 Klesha DI 框架 (M-3)、Cycle 02-2 我執修正 (A-1/A-4)、Cycle 02-4 VasanaEngine 外部化 + 風險加權閾值 + Sneha 重校準。*
