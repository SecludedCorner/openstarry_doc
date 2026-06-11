<!-- Layer: 2-Philosophy -->

# 29. 五蘊在 Agent Core 的連結架構 (Five Aggregates Core Connection)

> [Cycle 02-4 修整]

> Master 文件需求 #1: 五蘊在 Agent Core 的連結架構圖
> Cycle 02-3 整合版

---

## 1. 設計時架構 (Static / 五蘊分類)

> **[Master DC-6 修正]**
> - IUI 屬色蘊（輸出面），非行蘊。色蘊 = IListener（輸入）+ IUI（輸出），雙重繼承。
> - 行蘊保持開放，不鎖定為身行/語行/意行三類。「不同的樣態都可以存在，只要跑的動。」

```
                    ┌──────────────────────┐
                    │     IVijnana (識蘊)    │
                    │                      │
                    │  IIdentity ── 自我認同 │
                    │  IGuide    ── 行為引導 │
                    │  IVolition ── 行動承諾 │ ←── Klesha DI
                    │  IReflection ── 自省  │     (Moha/Drishti/
                    │                      │      Mana/Sneha)
                    └──────────┬───────────┘
                               │ constrains & guides
      ┌────────────────────────┼────────────────────────┐
      │                        │                        │
┌─────┴──────┐         ┌──────┴──────┐          ┌──────┴──────┐
│ IRupa (色蘊) │         │ ISamjna (想蘊)│          │ISamskara(行蘊)│
│ (雙重繼承)  │         │             │          │ (開放範疇)    │
│ IUI ─輸出面 │ ───────→│ IProvider   │────────→│ ITool        │
│ IListener─輸入│ events │IInference.  │ decides │ ISlashCmd    │
│            │         │             │          │ 其他造作…     │
└─────┬──────┘         └──────┬──────┘          └──────┬──────┘
      │                       │                        │
      └───────────────────────┼────────────────────────┘
                              │ all events carry vedana tag
                    ┌─────────┴─────────┐
                    │  IVedana (受蘊)     │
                    │                   │
                    │  VedanaPlugin      │
                    │  DukkhaSensor(苦)  │
                    │  SukhaSensor (樂)  │
                    │  UpekkhaSensor(捨) │
                    │  PID Controller    │
                    └─────────┬─────────┘
                              │
                    ┌─────────┴─────────┐
                    │   Agent Core (空)  │
                    │   EventBus         │
                    │   ServiceRegistry  │
                    │   PluginLoader     │
                    │   緣起性空          │
                    └───────────────────┘
```

## 2. 運行時架構 (Dynamic / 觸→俱生迴路)

> [Research Team Update — R3 Debate 1-6] 整合二層雙齒輪架構、五普遍心所、連續受蘊模型、二階段審議。

```
InputEvent (外部刺激)
    │
    ▼
┌──────────────────────────────────────────────────┐
│ 第一層：各根門觸事件 (Layer 1, vedana-clock, <1ms)  │
│                                                  │
│  IListener_A + event + ctx → 觸A (SparshEvent)    │
│    → CoarisingBundle {                           │
│        受A: valence[-1,+1] + intensity[0,1],     │
│        想A: rule-based pattern match,            │
│        思A: tendency + urgency,                  │
│        作意A: vijnana-clock snapshot              │
│      } (五普遍心所, Strategy C, ~0.8ms)           │
│  IListener_B + event + ctx → 觸B                 │
│    → CoarisingBundle { ... }                     │
│  ...                                             │
└───────────────────┬──────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────────┐
│ 第二層：歸依意 (Dual-Gear ManoAggregator — 純路由)   │
│                                                  │
│  Klesha.perceive() ← vijnana-clock (0.03ms)      │
│  all 俱生 bundles → 意觸                          │
│                                                  │
│  [Cycle 02-4 修整: VasanaEngine → IGearArbiter plugin]
│  (沒裝 IGearArbiter → confidence=0 → 永遠 Gear 2)  │
│                                                  │
│  ┌─ Gear 1 (快, IGearArbiter plugin, ~50ms):     │
│  │   IGearArbiter.evaluate() → GearEvaluation    │
│  │   threshold = gain-scheduled by Klesha        │
│  │                                               │
│  └─ Gear 2 (慢, IProvider plugin/LLM, seconds):  │
│      IProvider.chat() → ProposedAction            │
│      (triggered by low confidence or watchdog)    │
└───────────────────┬──────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────────┐
│ 第三層：行蘊執行 (含二階段審議)                      │
│  [Master DC-6: 行蘊開放，不鎖定三分類]              │
│                                                  │
│  IVolition.deliberatePlan() → 整體計畫審議         │
│  For each action:                                │
│    IVolition.deliberateAction() → 個別審議         │
│    SafetyMonitor.postCheck() → 硬安全 (ABSOLUTE)   │
│    行蘊造作: ITool.execute() → 外部效果            │
│              ISlashCommand → 指令執行             │
│              State update → 內部變化              │
│              (其他 Plugin 自由擴展…)               │
│    色蘊輸出: IUI.renderText() → 語言呈現 (IRupa)   │
│  SafetyMonitor.afterToolExecution() → 稽核       │
└───────────────────┬──────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────────┐
│ 第四層：反饋 → 新循環                               │
│                                                  │
│  行為結果 → 環境改變 → 新的 InputEvent              │
│  → 回到第一層                                     │
│  VedanaAssessment → KleshaBayesianUpdate (慢路徑) │
│                                                  │
│  "Each action reshapes the world of contact,     │
│   beginning the cycle anew." — Tenet #6          │
└──────────────────────────────────────────────────┘
```

## 3. 五蘊在 AgentCore 中的連結方式

### 3.1 事件驅動連結

五蘊之間的連結不是直接函數調用，而是**事件驅動**：

```
IRupa (IListener) ──pushInput()──→ EventBus        ← 色蘊·輸入面
EventBus ──onAny()──→ IVedana (VedanaPlugin tags events)
EventBus ──processEvent()──→ ExecutionLoop
  ExecutionLoop:
    ISamjna (IProvider.chat()) ── 想蘊認知
    ISamskara (ITool.execute() etc.) ── 行蘊造作 (開放範疇)
    IVijnana (EgoFramework.checkAction()) ── 識蘊約束
IRupa (IUI) ←──emit()──── EventBus                 ← 色蘊·輸出面
```

### 3.2 Pub/Sub 感應模式

> (Master): 「不是 A 呼叫 B，而是 A 的變化自然引發 B 的響應。」

五蘊之間的互動更接近**事件流相互感應**：
- VedanaPlugin 訂閱所有事件 → 自然感應系統狀態
- EgoFramework 接收 VedanaAssessment → 自然調整行為約束
- SafetyMonitor 監控異常 → 自然觸發安全機制

### 3.3 多時鐘架構

> [Research Team Update — R3 Debate 1 R1.3] 五時鐘模型 + 雙齒輪 mano-clock。

| 時鐘 | 頻率 | 組件 |
|------|------|------|
| vijnana-clock | 1-5ms | IGuide, IIdentity, Klesha.perceive(), IVolition.deliberate() |
| rupa-clock | 10-50ms | IListener (輸入), IUI (輸出), Layer 4 反饋 |
| vedana-clock | 10-100ms | CoarisingBundle (Layer 1), VedanaPlugin.assess(), PID Controller |
| samskara-clock | 10ms-10s | ITool.execute(), ISlashCommand |
| samjna-clock | 500ms-30s | IProvider.chat() (LLM), KleshaBayesianUpdate (慢路徑) |
| mano-clock (雙齒輪) | 50ms / seconds | ManoAggregator: Gear 1 (IGearArbiter plugin) / Gear 2 (IProvider plugin/LLM) |

**三層時間迴路** (R3 Debate 6 SYNTHESIST meta-synthesis):
```
SLOW LOOP (分鐘-小時): Klesha 偏差
  Klesha.perceive() → KleshaDistribution → gain-scheduled threshold

MEDIUM LOOP (秒-分鐘): Mano 認知循環
  ManoAggregator → IGearArbiter chain / IProvider → IVolition → Tool execution

FAST LOOP (10-100ms): 根門感知循環
  IListener → SparshEvent → CoarisingBundle(5 universals) → vedana PID feedback
```

---

## 4. 介面層級圖

> [Research Team Update — R3 Debate 2+4+5] 新增 CoarisingBundle 型別、ChannelVedana 連續模型、IVolition 二階段審議。

```typescript
// 五蘊根介面 (梵文命名, M-1 裁定)
IRupa     { skandha: 'rupa' }      // 色蘊
IVedana   { skandha: 'vedana' }    // 受蘊
ISamjna   { skandha: 'samjna' }    // 想蘊
ISamskara { skandha: 'samskara' }  // 行蘊
IVijnana  { skandha: 'vijnana' }   // 識蘊

// 色蘊子介面 (雙重繼承: 輸入 + 輸出) [Master DC-6 確認]
IListener extends IRupa       // 輸入面 — 感官接收
IUI       extends IRupa       // 輸出面 — 呈現/渲染 (非行蘊)

// 受蘊 (VedanaPlugin 實作)
// ChannelVedana: continuous valence [-1,+1] + intensity [0,1] + derived type
// 三受分類 = 派生值 (R3 Debate 5 R5.1)
// Plugin 層: IDukkha, ISukha, IUpekkha 保留為特化介面 (R5.3)

// 想蘊子介面
IProvider          extends ISamjna
IInferenceProvider extends IProvider

// 行蘊子介面 (開放範疇, 不鎖定三分類) [Master DC-6 確認]
ITool         extends ISamskara     // 外部效果
ISlashCommand extends ISamskara     // 指令執行
// Plugin 可自由擴展其他 ISamskara 子類別
// ICetana, IManasikara, ISparsha — 待確認是否正式化 (PD-4)

// 識蘊子介面
IIdentity   extends IVijnana     // 被動數據實體
IGuide      extends IVijnana     // Position A: 注意力形塑 (before LLM)
IVolition   extends IVijnana     // Position B: 行動承諾 (after LLM)
  // deliberatePlan(input): Promise<PlanDeliberationResult>
  // deliberateAction(input): Promise<ActionDeliberationResult>
IReflection extends IVijnana     // 延後到 Cycle 03 (UQ-2 meta-deliberation)

// Klesha DI (識蘊·煩惱系統)
// [Cycle 02-4 DD-8: IKlesha extends IVijnana + @sealed]
IKlesha extends IVijnana   // 型別歸屬: 識蘊子介面 (D5 18/20)
  @sealed                  // 凍結契約，不可被 plugin 繼承
Klesha (基類 implements IKlesha)
  Moha    extends Klesha   // 我癡 — low-pass filter
  Drishti extends Klesha   // 我見 — band-pass, 含 IIdentity 被動數據
  Mana    extends Klesha   // 我慢 — PD controller
  Sneha   extends Klesha   // 我愛 — integral controller

// CoarisingBundle (俱生，跨蘊事件型別)
CoarisingBundle {
  sparsha: SparshEvent           // 觸
  vedana: ChannelVedana          // 受 (連續 valence)
  samjna: ChannelSamjna          // 想
  cetana: ChannelCetana          // 思
  manasikara: ChannelManasikara  // 作意 (快照)
  layer: 1 | 2
  mode: 'fast' | 'slow'
  sahaja: SahajaContract
}

// 觀察者 (Composition, R3 Debate 2 R2.4)
// 不再需要獨立 IObserver — CoarisingBundle 天然跨蘊
```

---

*Master 文件需求 #1 完成。R3 Debate 1-6 整合完畢。Master DC-6 修正已套用（IUI 歸色蘊、行蘊開放範疇）。*
