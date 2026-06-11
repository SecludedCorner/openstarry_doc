<!-- Status: CURRENT -->
<!-- Layer: 3-Research -->
<!-- Applies to: v0.34.0-alpha -->
<!-- Last verified: 2026-03-16 -->

# 30. 觸→俱生模型的工程映射 (Sparsha-Coarising Engineering Model)

> [Cycle 02-4 修整]

> Master 文件需求 #2: 觸→俱生模型的工程映射文件
> Cycle 02-3 整合版

---

## 1. 佛學模型

### 經典依據

- **《雜阿含經》**: 「緣眼、色，生眼識，三事和合，是名為觸；觸俱生受、想、思。」
- **《蜜丸經》(MN 18)**: `phassapaccayā vedanā, yaṃ vedeti taṃ sañjānāti, yaṃ sañjānāti taṃ vitakketi`
- **《大拘絺羅經》(MN 43)**: 「凡所受者，即是所想；凡所想者，即是所思；凡所思者，即是所識。」

### 公式

```
根 (Indriya) + 境 (Visaya) + 識 (Vijnana) → 觸 (Sparsha)
  → 俱生 (Sahaja): 受 (Vedana) + 想 (Samjna) + 思 (Cetana)
```

### 特性

1. **俱生性**: 受想思同時浮現，不可分割
2. **順序性**: 受→想→思 有內在順序（開迴路）
3. **根門別**: 每個根門獨立觸發，結果經歸依意路由至齒輪選擇
4. **循環性**: 行為結果→色蘊變化→新根境→新一輪觸

---

## 2. 工程型別映射

### 2.1 觸 (Sparsha)

```typescript
/**
 * SparshEvent — 觸：根+境+識的三合。
 * 觸是事件處理的原子單位。
 */
interface SparshEvent {
  /** 根門 (哪個 IListener 通道) */
  readonly root: string;
  /** 境 (輸入事件) */
  readonly object: AgentEvent;
  /** 識 (處理上下文: session/context) */
  readonly consciousness: string;
  /** 時間戳 */
  readonly timestamp: number;
}
```

### 2.2 俱生 (CoarisingBundle) — 五普遍心所

> [Research Team Update — R3 Debate 1+2] Bundle 從三因素 (受想思) 擴展為五普遍心所 (sarvatraga)：觸 + 受 + 想 + 思 + 作意。新增 layer/mode/sahaja 元資料以支持二層雙齒輪架構。ChannelVedana 從離散分類改為連續測量模型 (R3 Debate 5)。

```typescript
/**
 * CoarisingBundle — 俱生：五普遍心所同時生起。
 *
 * Master: 「你不可能有沒有想的受或沒有受的想」
 * 每個欄位都是 required — 型別系統強制完備性。
 *
 * R3 Debate 2 R2.1: 五普遍心所 (sarvatraga) 全部包含。
 * R3 Debate 2 R2.3: 功能性論證 — 每個欄位因架構必要性而存在。
 */
interface CoarisingBundle {
  readonly sparsha: SparshEvent;              // Universal 1: 觸 (constitutive event)
  readonly vedana: ChannelVedana;             // Universal 2: 受 (hedonic tone)
  readonly samjna: ChannelSamjna;             // Universal 3: 想 (recognition)
  readonly cetana: ChannelCetana;             // Universal 4: 思 (volitional drive)
  readonly manasikara: ChannelManasikara;     // Universal 5: 作意 (attention snapshot)
  readonly layer: 1 | 2;                      // R3 Debate 1: 第一層(根門) / 第二層(歸依意)
  readonly mode: 'fast' | 'slow';             // R3 Debate 1: 雙齒輪 (layer 2 only)
  readonly sahaja: SahajaContract;            // R3 Debate 1: 俱生品質自我描述
  readonly timestamp: number;
}

/**
 * ChannelVedana — 受：連續測量模型。
 *
 * R3 Debate 5 R5.1: 連續 valence + intensity，離散 type 為派生值。
 * 科學依據: Russell (1980) circumplex model of affect。
 */
interface ChannelVedana {
  /** 效價：-1.0 (最大苦受) 到 +1.0 (最大樂受) */
  readonly valence: number;
  /** 強度：0.0 (幾乎不覺) 到 1.0 (壓倒性) */
  readonly intensity: number;
  /** 分類 (由 valence 派生): dukkha(<-0.1), upekkha([-0.1,+0.1]), sukha(>+0.1) */
  readonly type: 'dukkha' | 'sukha' | 'upekkha';
  /** 來源根門 */
  readonly source: string;
}

interface ChannelSamjna {
  readonly pattern: string;      // 取相結果
  readonly confidence: number;
}

interface ChannelCetana {
  readonly tendency: 'approach' | 'avoid' | 'neutral';
  readonly urgency: number;
}

/**
 * ChannelManasikara — 作意：vijnana-clock 狀態快照。
 *
 * R3 Debate 2 R2.2: 不是計算，而是快照。成本 ~0.01ms。
 */
interface ChannelManasikara {
  /** 當前注意焦點 (from IGuide) */
  readonly focus: string;
  /** 注意強度：0.0 (邊緣) 到 1.0 (焦點) */
  readonly intensity: number;
}

/**
 * SahajaContract — 俱生契約：bundle 的品質自我描述。
 *
 * R3 Debate 1 R1.7: 三個準則驗證世俗諦層級的俱生有效性。
 */
interface SahajaContract {
  /** 互相一致：bundle 中的各組件彼此參照 */
  readonly mutualConsistency: boolean;
  /** 原子發布：外部觀察者永遠不會看到部分 bundle */
  readonly atomicPublication: boolean;
  /** 過時上限：最新與最舊組件之間的時間差 (毫秒) */
  readonly stalenessUpperBound: number;
}
```

### 2.3 歸依意 (ManoAggregation)

```typescript
/**
 * ManoAggregation — 歸依意：各根門結果路由至齒輪選擇（純路由機制）。
 *
 * 《大拘絺羅經》: 「五根有歸依意，意為彼盡領受境界。」
 */
interface ManoAggregation {
  /** 各根門的俱生 bundles */
  readonly channelBundles: readonly CoarisingBundle[];
  /** 意觸 */
  readonly manoSparsha: SparshEvent;
  /** 意受 (綜合感受) */
  readonly manoVedana: VedanaAssessment;
  /** 意想 (綜合辨識) */
  readonly manoSamjna: { decision: string; reasoning: string };
  /** 意思 (最終決定) */
  readonly manoCetana: { action: string; target: string };
}
```

---

## 3. 四層迴路工程映射

> [Research Team Update — R3 Debate 1] 整合二層雙齒輪架構和四層到五時鐘映射。

### 四層到五時鐘映射 (R3 Debate 1 R1.3)

| 層 | 時鐘域 | 組件 | 典型延遲 |
|----|--------|------|---------|
| 第一層 (根門觸) | rupa-clock (輸入) + vedana-clock (俱生) | IListener, SparshEvent, CoarisingBundle | 1-100ms |
| 第二層快 (歸依意, 規則) | vedana-clock (路由) | ManoAggregator, IGearArbiter plugin | 50-100ms |
| 第二層慢 (歸依意, LLM) | samjna-clock (深度認知) | ManoAggregator, IProvider plugin | 500ms-30s |
| 第三層 (行蘊) | samskara-clock (工具執行) | ITool, ISlashCommand, IVolition.deliberate() | 10ms-10s |
| 第四層 (反饋) | rupa-clock (環境變化) | 新 InputEvent, IListener 再激活 | 1-50ms |
| 跨層 | vijnana-clock (身份, 1-5ms) | IGuide, IIdentity, Klesha.perceive() | 1-5ms |

### 第一層：各根門觸事件 (Layer 1, vedana-clock, <1ms)

```
InputEvent arrives at IListener channel
  │
  ├─ channel: "http"
  │    SparshEvent { root: "http", object: event, consciousness: sessionId }
  │    → CoarisingBundle {
  │        sparsha: sparshEvent,
  │        vedana: { valence: 0.0, intensity: 0.7, type: 'upekkha', source: 'http' },
  │        samjna: { pattern: 'api_request', confidence: 0.95 },
  │        cetana: { tendency: 'approach', urgency: 0.3 },
  │        manasikara: { focus: 'http-input', intensity: 0.8 },  // vijnana-clock snapshot
  │        layer: 1, mode: 'fast',
  │        sahaja: { mutualConsistency: true, atomicPublication: true, stalenessUpperBound: 0.8 }
  │      }
  │
  ├─ channel: "websocket"
  │    → CoarisingBundle { layer: 1, ... }
  │
  └─ (other channels...)

  ※ Layer 1 永遠使用規則匹配 (rule-based samjna), 不觸及 LLM。
  ※ 計算順序 (Strategy C): vedana(0.1ms) → samjna(0.5ms) → cetana(0.2ms) → manasikara(0.01ms) = ~0.8ms
```

### 第二層：歸依意 — 雙齒輪 (Dual-Gear Mano-Clock)

> [R3 Debate 1 R1.2] ManoAggregator 使用雙齒輪模型。

```
[Cycle 02-4 修整: ManoAggregator = 純路由機制]
(沒裝 IGearArbiter → confidence=0 → 永遠 Gear 2)

ManoAggregator.aggregate(allBundles)
  │
  ├─ Gear 1 (快速, IGearArbiter plugin chain, ~50ms):
  │   gearArbiter.evaluate(aggregatedBundles)
  │   ├─ confidence > threshold → 快速結果
  │   └─ confidence <= threshold → switch to Gear 2
  │
  └─ Gear 2 (慢速, IProvider plugin/LLM, seconds):
      IProvider.chat(context)
      ├─ 意受: aggregated valence (continuous)
      ├─ 意想: IProvider.chat() with full Klesha distribution context
      └─ 意思: ProposedAction

  ※ Gear switch threshold = gain-scheduled by Klesha state (R3 Debate 3 R3.3)
  ※ threshold(t) = clamp(θ₀ + Σwᵢμᵢ(t), θ_min=0.3, θ_max=0.9)
  ※ Vitakka Watchdog: 連續 N 次 Gear 1 後強制 Gear 2 (R3 Debate 3 R3.6)
```

### 第三層：行蘊執行 (包含二階段審議)

> [R3 Debate 4 R4.1-R4.3] IVolition 在 Position B (LLM 後、執行前) 進行二階段審議。

```
ProposedActions (from Gear 1 or Gear 2)
  │
  ├─ IVolition.deliberatePlan(proposedActions, kleshaSignals, vedanaAssessment)
  │   → modifiedPlan or accept as-is (Phase 1: 整體計畫審議)
  │
  ├─ For each action:
  │   IVolition.deliberateAction(action, kleshaSignals, vedanaAssessment)
  │   │  → veto? → skip action, log reasoning
  │   │  → approve? → proceed
  │   │
  │   SafetyMonitor.postCheck() → 硬安全 (ABSOLUTE, 不受 Klesha 影響)
  │   │
  │   ├─ 身行: ITool.execute() → 外部效果
  │   ├─ 語行: IUI.renderText() → 語言輸出
  │   └─ 意行: State.update() → 內部變化
  │
  └─ SafetyMonitor.afterToolExecution() → 稽核
```

### 第四層：反饋 → 新循環

```
行為結果 (file created)
  → 環境改變 (filesystem state changed)
  → 如果有 watcher → 新的 InputEvent → 回到第一層
  → VedanaAssessment (feedback) → KleshaBayesianUpdate (slow path)

  ※ Tenet #6: "Each action reshapes the world of contact, beginning the cycle anew."
```

---

## 4. 分階段實作策略

> [Research Team Update — R3 Debate 1] Phase 邊界由二層架構自然確定。

### Phase 1 (Plan26 近期): Layer 2 Only — 意觸

- ExecutionLoop 每個 tick = 一次「意觸」(Layer 2 ManoCoarisingBundle)
- VedanaPlugin.assess() = 「意受」(ChannelVedana with continuous valence)
- IProvider.chat() = 「意想」(or IGearArbiter plugin in Gear 1)
- ITool.execute() = 「意思→身行」(with IVolition.deliberate() at Position B)
- 不需要根門級別觸 (Layer 1)，是當前架構的自然延伸
- **新增**: IVolition 二階段審議 (R3 Debate 4)
- **新增**: Klesha gain-scheduled modulation (R3 Debate 3)

### Phase 2 (中期): Layer 1 + Layer 2 — 根門觸 + 歸依意

- 每個 IListener 通道有自己的 CoarisingBundle (Layer 1, 五普遍心所)
- 新增 ManoAggregator with dual-gear mano-clock (Layer 2)
- VedanaPlugin 支援 per-channel ChannelVedana (valence/intensity/type/source)
- ChannelManasikara 快照機制 (vijnana-clock → vedana-clock 讀取)
- SahajaContract 元資料

### Phase 3 (遠期): 完整四層 + 多時鐘

- 完整四層反饋迴路 with five-clock model
- 多時鐘架構 (M-10): vijnana(1-5ms) / rupa(10-50ms) / vedana(10-100ms) / samskara(10ms-10s) / samjna(500ms-30s)
- Dual-gear mano-clock 完整實作 (Gear 1 ↔ Gear 2 動態切換)
- Vitakka Watchdog 防止 samsaric stall
- Stability guarantee: staleness ratio ρ < 0.29 (WIENER criterion)

---

*Master 文件需求 #2 完成。R3 Debate 1-6 整合完畢。*
