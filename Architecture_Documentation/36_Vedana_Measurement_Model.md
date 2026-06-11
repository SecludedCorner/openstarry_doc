# 36. 受蘊測量模型 (Vedana Measurement Model)

> [Cycle 02-4 修整]

> Master 文件需求 #14: 雙層受蘊設計文件 (測量 vs 分類)
> Cycle 02-3 R3 Debate 5 決議 + Cycle 02 雙模式 Vedana + Cycle 02-2 受蘊事件系統整合
>
> **實作狀態**: ✅ SDK 類型已實作 (Plan26, 2026-02-28) — `ChannelVedana` + `classifyVedana()` + `IVedanaSensor` + `VedanaRegistry`。完整 PID 控制迴路留待 Plan27。

---

## 1. 概述

### 問題：分類 vs 測量

受蘊 (vedana) 設計面臨一個根本選擇：是將苦 (dukkha)、樂 (sukha)、捨 (upekkha) 視為**三個離散分類**，還是視為**連續測量值的衍生標籤**？

Cycle 02-3 R3 Debate 5 中，LINNAEUS 主張三介面模型 (IDukkha/ISukha/IUpekkha)，依據分類學原理認為三受是「質性不同」的現象。NAGARJUNA 反駁：受蘊不是**一個東西**可以分類，而是經驗的**面向** -- 認知時刻的 hedonic tone。PASCAL 提出連續 valence + 衍生分類的折衷方案，獲得全體共識。

[來源: R3_debate/debates_and_synthesis.md#Debate 5]

### 為何連續模型勝出

1. **PID 控制需求** (WIENER): PID 控制器操作連續誤差信號，離散三值輸入在分類邊界產生不連續性
2. **LLM 整合** (ATHENA): LLM 嵌入空間本質上是連續的，`valence = -0.72` 比 `type = 'dukkha'` 提供更多資訊
3. **佛學精確性** (ASANGA): 《差摩迦經》(MN 44) 描述受蘊為**光譜**，同一感受隨脈絡改變 hedonic tone
4. **型別理論** (BABBAGE): 連續 valence 空間 $[-1, 1]$ 通過商映射 (quotient map) 分割為三個等價類，型別安全不受影響

---

## 2. 核心設計：測量模型 (Core: Measurement Model)

### R5.1 ChannelVedana 介面

```typescript
/**
 * ChannelVedana -- 受蘊通道：連續測量模型。
 *
 * R3 Debate 5 R5.1: 使用連續 valence + intensity 作為主要表示，
 * 離散 type 通過可配置閾值從 valence 衍生。
 *
 * 科學依據: Russell (1980) circumplex model of affect。
 * 佛學依據: MN 44 -- 受蘊為光譜，非固定分類。
 *
 * NAGARJUNA: Vedana(e) = g(e, context, klesha, t)
 * 同一事件在不同脈絡、煩惱狀態、時間下產生不同受蘊。
 *
 * @readonly 所有欄位不可變 -- 受蘊「觀察而不干預」(R5.4, Tenet #6)。
 */
interface ChannelVedana {
  /**
   * 連續 hedonic valence (效價)。
   * -1.0 = 最大苦受 (dukkha)
   *  0.0 = 中性 (upekkha 中心)
   * +1.0 = 最大樂受 (sukha)
   *
   * 支持 WIENER 的 PID 控制: e(t) = r(t) - valence(t)
   */
  readonly valence: number;

  /**
   * 強度/覺醒度 (arousal)。
   * 0.0 = 幾乎察覺不到
   * 1.0 = 壓倒性的
   *
   * 與 valence 正交 -- 苦受可以是低強度 (隱隱不適)
   * 也可以是高強度 (劇烈疼痛)。
   */
  readonly intensity: number;

  /**
   * 離散分類 (從 valence 衍生)。
   * 供 IGearArbiter plugin 規則匹配和日誌使用。
   *
   * 衍生規則見 VedanaClassificationConfig。
   */
  readonly type: 'dukkha' | 'sukha' | 'upekkha';

  /** 來源根門識別符 (哪個 IListener 通道或 'mano') */
  readonly source: string;
}
```

### 為何連續 valence 對 PID 控制是必要的 (WIENER)

PID 控制器的誤差信號必須是連續的：

$$e(t) = r(t) - \text{valence}(t)$$

其中 $r(t)$ 是參考值 (期望恆定態，通常 $r = 0$ = upekkha)。若受蘊僅為三值離散量：

$$e_{\text{discrete}}(t) \in \{-1, 0, +1\}$$

則在分類邊界處 (例如 valence 從 $-0.11$ 跳到 $-0.09$) 將產生 Dirac-delta 型跳變，導致 PID 微分項發散。連續 valence 消除此問題。

[來源: R3_debate/debates_and_synthesis.md#Debate 5, WIENER 發言]

---

## 3. 衍生分類規則 (Derivation Rule)

### R5.2 VedanaClassificationConfig

```typescript
/**
 * VedanaClassificationConfig -- 衍生分類閾值配置。
 *
 * BABBAGE 警語: 預設 upekkha 帶 (-0.1 to +0.1) 可能過窄。
 *   upekkha 不僅是「接近零的感受」-- 它是等持的品質。
 *   更寬的帶 (如 -0.2 to +0.2) 可能更合適。
 *
 * GUARDIAN 安全: 此配置為管理員設定，非使用者可修改。
 *   若使用者能設定 (-1.0, +1.0)，將強制 Agent 永久 upekkha，
 *   等同對內部反饋迴路的 DoS 攻擊。
 *
 * MESH: 多 Agent 部署中，每個 Agent 應有獨立配置。
 */
interface VedanaClassificationConfig {
  /** 苦受閾值: valence < 此值 → 'dukkha' (default: -0.1) */
  readonly dukkhaThreshold: number;
  /** 樂受閾值: valence > 此值 → 'sukha' (default: +0.1) */
  readonly sukhaThreshold: number;
}
```

### classifyVedana 函數

```typescript
/** 受蘊分類型別 */
type VedanaType = 'dukkha' | 'sukha' | 'upekkha';

/**
 * 從連續 valence 衍生離散分類。
 *
 * 數學: 分段常函數，兩個斷點。
 *   type: [-1, 1] → {dukkha, sukha, upekkha}
 *
 * BABBAGE 型別理論: 這是商映射 (quotient map)。
 *   連續 valence 空間被分割為三個等價類。
 *   映射是良定義的 (well-defined) 且滿射 (surjective)。
 *
 * @param valence - 連續效價 [-1.0, +1.0]
 * @param config  - 分類閾值配置
 * @returns 衍生的離散分類
 */
function classifyVedana(
  valence: number,
  config: VedanaClassificationConfig
): VedanaType {
  if (valence < config.dukkhaThreshold) return 'dukkha';
  if (valence > config.sukhaThreshold)  return 'sukha';
  return 'upekkha';
}
```

### 窄帶 vs 寬帶 upekkha 討論

| 配置 | dukkha 閾值 | sukha 閾值 | upekkha 帶寬 | 適用場景 |
|------|------------|-----------|-------------|---------|
| 窄帶 (預設) | -0.1 | +0.1 | 0.2 | 高反應性 Agent (客戶服務) |
| 寬帶 | -0.2 | +0.2 | 0.4 | 冥想型 Agent (資料處理) |
| 超窄 | -0.05 | +0.05 | 0.1 | 安全關鍵 Agent (早期預警) |

KERNEL: 「閾值可配置讓不同部署情境設定不同敏感度。禪修型 Agent 使用更寬閾值 (更多等持)；行動型 Agent 使用更窄閾值 (更高反應性)。」

[來源: R3_debate/debates_and_synthesis.md#Debate 5, KERNEL 發言]

---

## 4. 雙層設計 (Dual-Level Design) -- LINNAEUS

R3 Debate 5 R5.3 決議: 執行期測量與插件層分類是兩個不同架構層級，互不矛盾。

```
架構層級                    模型               用途
─────────────────────────  ──────────────────  ───────────────────
Runtime 事件層 (Layer 1)    ChannelVedana       CoarisingBundle 中的測量資料
                            連續 valence         PID 控制、Klesha 更新
                            + intensity

Plugin 專門化層 (Layer 2)   IDukkha / ISukha    插件宣告自身處理哪種受蘊
                            / IUpekkha          型別系統專門化
                            離散 type            VedanaPlugin 的子感測器
```

LINNAEUS 的讓步: 「我接受測量模型。我的三介面方法是為**插件層級**的型別鑑別設計的 (一個只處理苦受的插件會實作 IDukkha)。但在**事件層級** CoarisingBundle 中，PASCAL 的連續模型加衍生分類更優越。」

SCRIBE: 「此雙層設計必須明確記錄在架構文件中。開發者需理解 ChannelVedana (在 CoarisingBundle 中) 是測量，而 IDukkha/ISukha/IUpekkha (在插件介面中) 是專門化。兩層通過 classifyVedana() 衍生規則連接。」

[來源: R3_debate/debates_and_synthesis.md#Debate 5, R5.3]

### [Cycle 02-4 新增] 語義修正：v_true → v_latent 與雙層正當性

**效價真值 → 潛在效價狀態 (v_true → v_latent)**

Cycle 02-3 文件中使用 `v_true`（效價真值）描述受蘊的底層連續值。此命名隱含受蘊存在一個「真實值」可被逼近，有將受蘊自性化 (svabhava) 的風險。修正為 `v_latent`（潛在效價狀態），語義更精確：

- `v_latent`：潛在的、未經分類的連續效價狀態，是感測器輸出的原始信號
- `v_latent` 不是「真實值」，而是「尚未被概念化的狀態」——與中觀學「離言自性」相容
- classifyVedana() 將 `v_latent` 映射為離散 type，是**假名施設** (prajnapti)，不是「近似真值」

**雙層架構的二諦正當性**

受蘊測量模型包含兩個正當層級：

| 層級 | 模型 | 二諦對應 | 函數 |
|------|------|---------|------|
| 信號層 (signal layer) | 連續 valence ∈ [-1, +1], Softplus 激活 | 世俗諦 (samvriti-satya) | PID 控制、Klesha 更新 |
| 語義層 (semantic layer) | 離散 type: dukkha / sukha / upekkha | 假名 (prajnapti) | classifyVedana() 分類 |

兩層皆為正當：信號層是世俗諦層面的有效測量（如溫度計讀數），語義層是假名施設層面的有效分類（如「冷/溫/熱」標籤）。兩者都不是受蘊的「自性」——受蘊本身是緣起法，無獨立自性可言。

---

## 5. 二維情感空間 (Two-Dimensional Affect Space)

### Russell Circumplex Model (1980)

R3 Debate 5 R5.5 確認: ChannelVedana 的 valence + intensity 二維模型映射到心理學的 circumplex model of affect。

```
            高 intensity (1.0)
                   │
                   │
         憤怒      │      興奮
    (dukkha, high) │ (sukha, high)
                   │
  ─────────────────┼─────────────────  valence
  -1.0             │            +1.0
                   │
         悲傷      │      平靜
    (dukkha, low)  │ (sukha, low)
                   │
                   │
            低 intensity (0.0)

    ← dukkha →  ← upekkha →  ← sukha →
               (帶寬可配置)
```

### Valence x Intensity 軸

| 軸 | 範圍 | 佛學對應 | 工程對應 |
|----|------|---------|---------|
| Valence (效價) | [-1.0, +1.0] | 苦←→樂光譜 | PID 誤差信號 e(t) |
| Intensity (強度) | [0.0, 1.0] | 受蘊的「重量」| PID 增益調整因子 |

### LLM Logprob 映射 (ATHENA)

ATHENA 提出: LLM logprob 直接產生連續值，可自然映射到 ChannelVedana:

- 高 logprob (高信心回應) → 正 valence (sukha 方向)
- 低 logprob (低信心回應) → 負 valence (dukkha 方向)
- logprob 標準差 → intensity

連續 valence 模型讓 LLM 整合無需離散化損失。

[來源: R3_debate/debates_and_synthesis.md#Debate 5, ATHENA 發言]

---

## 6. 身受與心受 (Kayika vs Caitasika) -- ASANGA

### 《差摩迦經》(MN 44) 的兩類受

Venerable Dhammadinna 區分 **kayika-vedana** (身受) 與 **caitasika-vedana** (心受，梵文 caitasika 對應巴利文 cetasika)。Master's letter 引用《相應部》「二箭」段落 -- 身體疼痛與對疼痛的心理反應。

| 類型 | 描述 | OpenStarry 編碼 | 例子 |
|------|------|----------------|------|
| 身受 (kayika) | 感官層級的受蘊 | Layer 1 bundle, source = 根門 ID | 硬體溫度超標 → dukkha |
| 心受 (caitasika) | 心理層級的受蘊 | Layer 2 bundle, source = 'mano' | 工具執行失敗 → dukkha |

### 通過 source 和 layer 區分

PASCAL 的分析: 「身受/心受的區分已被 `source` 欄位和 Layer 1/2 架構隱含編碼。若 `source = "eye-gate"`，受蘊偏向 kayika (感官)。若 `source = "mano-gate"`，則為 caitasika (心理)。Debate 1 的 Layer 1 vs Layer 2 也映射: Layer 1 bundles 產生 kayika 式受蘊 (per root gate)；Layer 2 ManoCoarisingBundle 產生 caitasika 式受蘊 (綜合心理評估)。不需額外 `bodyMind` 鑑別器。」

ASANGA 接受此分析: 「隱含但足夠。我會在文件中為佛學學者標註此映射。」

[來源: R3_debate/debates_and_synthesis.md#Debate 5, ASANGA + PASCAL 交鋒]

---

## 7. 雙模式 Vedana (Dual-Mode Vedana) -- FROM CYCLE 02

Cycle 02 Debate 2 確立受蘊的雙模式運作：**VedanaAssessment** (完整 PID 評估) 與 **VedanaTag** (輕量快取標籤)。

### VedanaAssessment -- 權威性評估

```typescript
/**
 * VedanaAssessment -- 完整三受評估。
 * 每個 ExecutionLoop tick 產生一次 (tick-synchronous PID)。
 * 這是 AUTHORITATIVE vedana 讀取。
 *
 * 兩個概念層 (Cycle 02 Debate 1 ruling):
 *   LAYER 1 -- 感測器輸出 (純觀察, 被動)
 *   LAYER 2 -- 控制器建議 (諮詢性, 可被忽略)
 *
 * SafetyMonitor 對硬安全保留 ABSOLUTE 權威。
 */
export interface VedanaAssessment {
  // --- LAYER 1: 感測器輸出 (純觀察) ---
  /** 苦受強度: 0.0 (無苦) 到 1.0 (危急) */
  readonly dukkha: number;
  /** 樂受強度: 0.0 (無樂) 到 1.0 (巔峰表現) */
  readonly sukha: number;
  /** 捨受強度: 0.0 (最大不穩) 到 1.0 (完美均衡) */
  readonly upekkha: number;
  /** 詳細逐通道信號 */
  readonly signals: readonly VedanaSignal[];
  /** 評估時間戳 */
  readonly timestamp: number;

  // --- LAYER 2: 控制器建議 (諮詢性) ---
  /** 加權 PID 控制輸出 */
  readonly controlOutput: number;
  /** 諮詢性建議 (非強制) */
  readonly recommendation: VedanaRecommendation;
  /** 建議新鮮度 */
  readonly recommendationFreshness: 'current' | 'cached' | 'default';
}
```

### VedanaTag -- 輕量快取標籤

```typescript
/**
 * VedanaTag -- 附加到每個 EventBus 事件的輕量標籤。
 * 從最近一次 VedanaAssessment 衍生 (快取查找, O(1))。
 *
 * 滿足 ASANGA 的遍行 (sarvatraga) 需求:
 * 每個意識時刻都有受蘊。
 *
 * 排除 VedanaPlugin 自身事件以防循環依賴。
 * 初始狀態: upekkha (系統在平衡中啟動)。
 */
export type VedanaTag = 'dukkha' | 'sukha' | 'upekkha';

/**
 * VedanaTag 衍生邏輯:
 *   tag = (dukkha > sukha && dukkha > upekkha) ? 'dukkha'
 *       : (sukha > dukkha && sukha > upekkha) ? 'sukha'
 *       : 'upekkha';
 *
 * 成本: O(1) -- 三個數值比較。
 */
```

### 設計原理

| 模式 | 觸發頻率 | 計算成本 | 用途 |
|------|---------|---------|------|
| VedanaAssessment | 每 tick 一次 | PID 計算 (~ms) | 權威性評估, 控制迴路 |
| VedanaTag | 每事件一次 | O(1) 快取讀取 | 事件元資料, 日誌 |

[來源: cycle02/delivery/01_engineering_action_plan.md#Section 2]

---

## 8. 受蘊事件系統 (Vedana Event System) -- FROM CYCLE 02-2 Q3

Master 裁定: 「受蘊跟其他蘊完全一致，有自己的型別檔、自己的事件命名空間、自己的 PluginHooks 插槽。」

### vedana-events.ts 事件型別

```typescript
// packages/sdk/src/types/vedana-events.ts

/**
 * Vedana 事件型別常量。
 * @skandha vedana (受蘊)
 */
export const VedanaEventType = {
  /** 完整評估完成 */
  VEDANA_ASSESSMENT:          'vedana:assessment',
  /** 苦受突刺 (超過閾值) */
  VEDANA_DUKKHA_SPIKE:        'vedana:dukkha_spike',
  /** 樂受峰值 */
  VEDANA_SUKHA_PEAK:          'vedana:sukha_peak',
  /** 捨受均衡達成 */
  VEDANA_UPEKKHA_EQUILIBRIUM: 'vedana:upekkha_equilibrium',
  /** 建議發出 */
  VEDANA_RECOMMENDATION:      'vedana:recommendation',
  /** 感測器原始更新 */
  VEDANA_SENSOR_UPDATE:       'vedana:sensor_update',
  /** 受蘊重置 */
  VEDANA_RESET:               'vedana:reset',
} as const;
```

### VedanaEventPayloadMap 型別聯合

```typescript
/**
 * 每個事件型別對應的 payload 結構。
 * 型別安全: EventBus.emit<T extends keyof VedanaEventPayloadMap>()
 */
export interface VedanaEventPayloadMap {
  [VedanaEventType.VEDANA_ASSESSMENT]: {
    dukkha: number; sukha: number; upekkha: number;
    controlOutput: number; vedanaTag: VedanaTag;
    timestamp: number;
  };
  [VedanaEventType.VEDANA_DUKKHA_SPIKE]: {
    intensity: number; source: string;
    previousIntensity: number; threshold: number;
  };
  [VedanaEventType.VEDANA_SUKHA_PEAK]: {
    intensity: number; source: string; decayRate: number;
  };
  [VedanaEventType.VEDANA_UPEKKHA_EQUILIBRIUM]: {
    stability: number; duration: number;
  };
  [VedanaEventType.VEDANA_RECOMMENDATION]: {
    recommendation: VedanaRecommendation;
    freshness: 'current' | 'cached' | 'default';
    assessment: { dukkha: number; sukha: number; upekkha: number };
  };
  [VedanaEventType.VEDANA_SENSOR_UPDATE]: {
    sensorType: 'dukkha' | 'sukha' | 'upekkha';
    rawValue: number; source: string;
  };
  [VedanaEventType.VEDANA_RESET]: undefined;
}
```

### SensationRegistry 介面

```typescript
// packages/core/src/infrastructure/sensation-registry.ts

/**
 * SensationRegistry -- 受蘊實作的註冊表。
 * AgentCore 持有，在 Plugin 載入時填充。
 */
export interface SensationRegistry {
  /** 註冊一個 ISensation 實作 */
  register(sensation: ISensation): void;
  /** 列出所有已註冊的 ISensation */
  list(): ISensation[];
  /** 取得主要 ISensation (用於 ExecutionLoop) */
  get(): ISensation | undefined;
}
```

### PluginHooks.sensations 新增

```typescript
// packages/sdk/src/types/plugin.ts (修改)

export interface PluginHooks {
  providers?: IProvider[];
  tools?: ITool[];
  listeners?: IListener[];
  ui?: IUI[];
  guides?: IGuide[];
  sensations?: ISensation[];   // NEW: 受蘊插槽
  commands?: SlashCommand[];
  dispose?: () => Promise<void> | void;
}
```

[來源: cycle02-2/delivery/02_q1q4_engineering_spec.md#Q3]

---

## 9. ISensation 完整方法簽名 (ISensation Complete Interface) -- FROM CYCLE 02-2 C-1

```typescript
/**
 * ISensation -- 受蘊 (Vedana) 根介面。
 * @skandha vedana
 *
 * 三受反饋系統:
 *   - Dukkha (苦): 苦受/負面反饋
 *   - Sukha (樂): 樂受/正面反饋
 *   - Upekkha (捨): 等持/中性穩定
 *
 * 架構: ATHENA (感測器) -> WIENER (PID 引擎) -> NAGARJUNA (框架)
 *
 * 此模組同時作為 Pattern A Observer (Cycle 02 Debate 4):
 *   Pattern A = vedana (受蘊)
 *   Pattern B = vedana + samjna (受+想) [future]
 *   Pattern C = full agent (all aggregates) [future]
 */
export interface ISensation {
  /** @skandha vedana */
  readonly skandha: 'vedana';

  /**
   * 產生完整三受評估。
   * 每個 ExecutionLoop tick 呼叫一次 (tick-synchronous PID)。
   * 這是 AUTHORITATIVE vedana 讀取。
   *
   * 返回的 VedanaAssessment 包含:
   *   - 感測器輸出 (dukkha/sukha/upekkha 數值) -- 純觀察
   *   - 控制器建議 (recommendation) -- 諮詢性, 非強制
   */
  assess(): VedanaAssessment;

  /**
   * 攝入 MetricsCollector 快照的原始指標。
   * 每個 loop tick 開始時呼叫。
   */
  ingestMetrics(metrics: Record<string, number>): void;

  /**
   * 攝入工具執行結果。
   * 每次工具執行後呼叫，提供即時感測器餵養。
   */
  ingestToolResult(toolName: string, isError: boolean, durationMs: number): void;

  /**
   * 攝入 LLM 回應結果。
   * 每次 LLM 呼叫完成後呼叫。
   */
  ingestLLMResult(tokenCount: number, stopReason: string): void;

  /**
   * 訂閱受蘊閾值事件。
   * 當特定受蘊通道超過給定閾值時觸發。
   * 返回取消訂閱函數。
   */
  onVedana(
    vedanaType: VedanaType,
    threshold: number,
    handler: (signal: VedanaSignal) => void
  ): () => void;

  /**
   * 取得當前受蘊標籤 (用於事件標記)。
   * 返回最近評估中的主導受蘊類型。
   * 成本: O(1) -- 三個快取值的簡單比較。
   */
  getVedanaTag(): VedanaTag;

  /**
   * 取得歷史評估以供趨勢分析。
   * @param windowSize 過去評估數量
   */
  getHistory(windowSize: number): readonly VedanaAssessment[];

  /**
   * 重置受蘊狀態。
   * 重置後初始狀態為 upekkha (等持) --
   * 系統在均衡中啟動，等待條件擾動。
   */
  reset(): void;
}
```

[來源: cycle02/delivery/01_engineering_action_plan.md#Section 2.2; cycle02-2/delivery/03_skandha_extension_architecture.md]

---

## 10. 資料流：受蘊到控制信號 (Data Flow: Vedana to Control)

R3 Debates 5, 3, 1 共同定義了從受蘊測量到行為控制的完整資料鏈。ARCHIMEDES 在 Debate 5 中呈現的資料流：

```
完整資料流: 受蘊 → 煩惱 → 行為
═══════════════════════════════════════════════════════════════════

ChannelVedana.valence (連續, R3 D5)
  │
  ▼
VedanaAssessment (每 mano-cycle 彙總)
  │
  ├──► VedanaTag (O(1) 快取衍生, 事件元資料)
  │
  ▼
KleshaBayesianUpdater (慢路徑, R3 D3)
  │    update(prior, assessment, kleshaType)
  │    → 更新 Beta(alpha, beta) per klesha channel
  │
  ▼
KleshaDistribution.mean (快路徑, R3 D3)
  │    alpha / (alpha + beta) -- 點估計, ~0.001ms
  │
  ▼
gain-scheduled threshold (R3 D3 R3.3)
  │    threshold(t) = clamp(theta_0 + sum(w_i * mu_i(t)), theta_min, theta_max)
  │
  ▼
IGearArbiter plugin gear switch (R3 D1 R1.2)
  │    confidence > threshold → Gear 1 (rule, ~50ms)
  │    confidence <= threshold → Gear 2 (LLM, seconds)
  │
  ▼
IVolition.deliberate() at Position B (R3 D4 R4.1)
  │    Phase 1: deliberatePlan()
  │    Phase 2: deliberateAction()
  │
  ▼
Tool execution → Environment change → New sparsha → 回到頂部

═══════════════════════════════════════════════════════════════════
  此鏈橫跨 Debates 5, 3, 1。Debate 5 的連續 valence
  是所有下游決策的基礎。若使用離散模型，此鏈的每一步都會有不連續性。
```

[來源: R3_debate/debates_and_synthesis.md#Debate 5, ARCHIMEDES 發言]

---

## 11. PID 控制理論 (PID Control Theory) -- WIENER

### 連續 valence 的必要性

三通道 PID 架構 (Cycle 02):

$$u(t) = w_d \cdot \text{PID}_d(e_d) + w_s \cdot \text{PID}_s(e_s) + w_u \cdot \text{PID}_u(e_u)$$

其中每個通道的 PID 控制器:

$$\text{PID}_c(e) = K_p \cdot e(t) + K_i \int_0^t e(\tau) d\tau + K_d \frac{de(t)}{dt}$$

### 分類邊界的不連續性問題

若使用離散分類作為 PID 輸入:

```
邊界不連續示例:
═══════════════════════════════════════════════════

valence  -0.11  -0.10  -0.09   (穿越 dukkha 閾值)
type     dukkha  dukkha upekkha
                        ↑
                   不連續跳變

PID 微分項: de/dt ≈ Δtype/Δt → 無窮大 (Dirac delta)
→ 控制器輸出發散
→ 系統不穩定

使用連續 valence:
valence  -0.11  -0.10  -0.09
de/dt  ≈ 0.01/Δt → 有限, 平滑
→ 控制器穩定
```

### 三通道 PID 架構與 ChannelVedana 的關係

| PID 通道 | 輸入源 (Cycle 02) | 對應 ChannelVedana (Cycle 02-3) |
|---------|------------------|-------------------------------|
| Dukkha PID | dukkha intensity | valence < 0 部分, \|valence\| |
| Sukha PID | sukha intensity | valence > 0 部分, valence |
| Upekkha PID | upekkha stability | intensity 接近 0 的穩定度 |

Cycle 02-3 的連續 valence 統一了三通道的輸入基礎，三個 PID 通道從同一個連續信號的不同區間提取資訊。

[來源: cycle02/delivery/01_engineering_action_plan.md#Section 3.2; R3_debate/debates_and_synthesis.md#Debate 5, WIENER]

---

## 12. LLM 整合 (LLM Integration) -- ATHENA

### Logprobs-as-Vedana 映射

ATHENA (R03 Sec 6.4): LLM 的 logprob 天然產生連續值，可直接映射為 ChannelVedana:

```typescript
/**
 * 將 LLM logprob 映射為 ChannelVedana。
 *
 * ATHENA: LLM 根本上是連續嵌入空間上的模式匹配器，
 * 餵入 valence = -0.72 比餵入 "dukkha" 提供更多資訊。
 *
 * @param logprob    - LLM token logprob (typically -inf to 0)
 * @param config     - 映射配置 (正規化範圍)
 */
function logprobToVedana(logprob: number, config: LogprobMappingConfig): number {
  // 正規化到 [-1, +1] 範圍
  // 高 logprob (高信心) → 正 valence (sukha)
  // 低 logprob (低信心) → 負 valence (dukkha)
  const normalized = (logprob - config.min) / (config.max - config.min);
  return normalized * 2 - 1;  // 映射到 [-1, +1]
}
```

### 連續解釋的優勢

| 離散模型 | 連續模型 |
|---------|---------|
| logprob → "dukkha" 或 "sukha" | logprob → valence = -0.72 |
| 資訊丟失: 嚴重苦受與輕微苦受不可區分 | 保留精確度: 嚴重(-0.95) vs 輕微(-0.12) |
| 需要額外閾值配置 | 直接可用，無需離散化 |
| LLM 上下文中需解釋分類 | LLM 可直接處理數值 |

[來源: R1_independent/R03_contact_feedback_loop.md#Section 6.4; R3_debate/debates_and_synthesis.md#Debate 5, ATHENA]

---

## 13. 安全考量 (Security) -- GUARDIAN

### 管理員鎖定閾值

GUARDIAN (Debate 5): `VedanaClassificationConfig` 必須為管理員可配置，非使用者可修改。

```
安全威脅模型:
═══════════════════════════════════════════════════

攻擊: 使用者修改閾值 → { dukkhaThreshold: -1.0, sukhaThreshold: +1.0 }
效果: 所有 valence 都映射為 'upekkha'
結果: Agent 對自身受蘊信號無反應 -- 內部反饋迴路 DoS

防禦:
  1. VedanaClassificationConfig 在部署時設定，非執行期可修改
  2. 配置文件存放在管理員控制的路徑
  3. ExecutionLoop 讀取配置後 Object.freeze()
  4. 配置變更需重啟 Agent (非熱更新)
```

### 強制 Upekkha 防護

閾值有硬性安全界限:

- `dukkhaThreshold` 最小值: $-0.5$ (不能設定太寬以致苦受無法觸發)
- `sukhaThreshold` 最大值: $+0.5$ (不能設定太寬以致樂受無法觸發)
- upekkha 帶寬上限: 1.0 (帶寬不能超過整個 valence 範圍的 50%)

### 執行期唯讀

ChannelVedana 的所有欄位為 `readonly`。受蘊「觀察而不干預」(Tenet #6)。任何試圖修改 ChannelVedana 的行為在型別系統層級被阻止。

[來源: R3_debate/debates_and_synthesis.md#Debate 5, GUARDIAN 發言]

---

## 14. 佛學基礎 (Buddhist Foundations)

### 《差摩迦經》(MN 44) 引用

Venerable Dhammadinna 論受蘊:

> 「樂受於生時為樂，住時為樂，滅時為苦。苦受於生時為苦，住時為苦，滅時為樂。不苦不樂受以智觀之為樂，以無明觀之為苦。」

此段經文支持**連續測量模型**: 同一受蘊隨脈絡改變 hedonic tone。苦受在消滅時轉為樂受 -- 這在離散型別系統中需要「型別轉換」(IDukkha 變為 ISukha)，這在型別理論中是無意義的。連續 valence 自然處理此轉換: valence 從 $-0.5$ 逐漸移向 $+0.2$。

### 三受的行為程式

| 受 | 梵文 | 行為程式 | ChannelVedana 映射 |
|----|------|---------|-------------------|
| 苦受 | Dukkha | 厭離 (withdrawal) | valence < dukkhaThreshold |
| 樂受 | Sukha | 趨向 (approach) | valence > sukhaThreshold |
| 捨受 | Upekkha | 維持 (homeostasis) | dukkhaThreshold <= valence <= sukhaThreshold |

LINNAEUS (Debate 5): 三受映射到「不同的行為程式」。DARWIN 補充生物學平行: 神經系統不產生離散的「痛」或「樂」，而是沿連續尺度產生分級電信號，大腦再分類為行為決策。PASCAL 的模型重現此過程: 連續 valence 是神經信號，離散 type 是行為分類。

### PASCAL 的貝葉斯認識論角度

PASCAL: 若受蘊是測量，它有不確定性。若有不確定性，單一 `type: 'dukkha' | 'sukha' | 'upekkha'` 是丟棄資訊的點分類。連續 valence 保留此不確定性，讓下游消費者 (KleshaBayesianUpdater) 根據自身需求決定精度。

PENROSE 的量子測量平行: 連續 valence 如同量子態，離散 type 如同測量結果。分類閾值是測量裝置的靈敏度。分類本質上有損 -- 離散 type 丟棄了「在 dukkha 範圍內的精確位置」(是勉強苦受 $-0.11$ 還是深度苦受 $-0.95$?)。

[來源: R3_debate/debates_and_synthesis.md#Debate 5, ASANGA/PASCAL/PENROSE 發言]

---

## 15. 多 Agent 部署 (Multi-Agent)

### Per-Agent VedanaClassificationConfig

MESH (Debate 5): 多 Agent 部署中，各 Agent 應擁有獨立的 `VedanaClassificationConfig`。不同 Agent 有不同角色:

| Agent 角色 | 建議閾值 | upekkha 帶寬 | 理由 |
|-----------|---------|-------------|------|
| 客戶服務 Agent | (-0.05, +0.05) | 0.1 | 需要高情感反應性 |
| 背景資料處理 Agent | (-0.3, +0.3) | 0.6 | 需要高等持 (equanimity) |
| 安全監控 Agent | (-0.08, +0.15) | 非對稱 | 對苦受更敏感 |
| 預設 Agent | (-0.1, +0.1) | 0.2 | 平衡配置 |

此設計與 R3 Debate 1 R1.5 的一般原則一致: 每個 Agent 的內部配置是獨立的。

[來源: R3_debate/debates_and_synthesis.md#Debate 5, MESH 發言]

---

## 附錄 A: Cycle 02 → 02-2 → 02-3 演化摘要

| 版本 | 受蘊模型 | 關鍵進展 |
|------|---------|---------|
| Cycle 02 | VedanaAssessment (三通道數值) + VedanaTag (離散標籤) | 雙模式確立, ISensation 介面, PID 控制 |
| Cycle 02-2 | + vedana-events.ts + SensationRegistry + PluginHooks.sensations | 受蘊獨立事件流, ISensation 完整方法 |
| Cycle 02-3 | ChannelVedana (連續 valence + intensity + 衍生 type) | 測量模型, 雙層設計, circumplex model |

## 附錄 B: R3 Debate 5 決議摘要

| 編號 | 決議 | 內容 |
|------|------|------|
| R5.1 | ChannelVedana 測量模型 | 連續 valence + intensity，衍生 type |
| R5.2 | 可配置分類閾值 | VedanaClassificationConfig, 預設 +/-0.1 |
| R5.3 | 雙層設計 | Runtime 測量 vs Plugin 分類，不矛盾 |
| R5.4 | 雙消費者 | 連續 valence → PID；離散 type → IGearArbiter plugin |
| R5.5 | 科學依據 | Russell (1980) circumplex model of affect |

---

*Master 文件需求 #14 完成。整合 R3 Debate 5 決議 + Cycle 02 雙模式 Vedana + Cycle 02-2 受蘊事件系統。*
