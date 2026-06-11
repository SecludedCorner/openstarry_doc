# 39. CoarisingBundle 五遍行設計 (Five Universals Architecture)

> R3 Debate 2 共識: CoarisingBundle 包含五遍行 (sarvatraga) -- sparsha, vedana, samjna, cetana, manasikara。
> 依賴: Doc 30 (Sparsha-Coarising Model), R3 Debate 1 (Multi-Clock), Cycle 02-2 Q1 (SkandhaCompletenessReport)
>
> **實作狀態**: ✅ 已實作 (Plan26, 2026-02-28) — `sdk/types/coarising.ts` (CoarisingBundle + SahajaContract 類型) + `core/src/vedana/coarising-factory.ts` (createCoarisingBundle 工廠 + isSahajaValid 驗證)。DC-8 註明 reference design。

---

## 1. 概述

原始 M-5 設計定義 CoarisingBundle 含三因素: vedana + samjna + cetana。R03 的 ASANGA 援引瑜伽行派 sarvatraga 教義，主張五因素不可或缺。R3 Debate 2 經全體 20 位學者辯論，**全票通過**採納五遍行:

| 原始 (三因素) | 決議 (五遍行) | 新增 |
|--------------|-------------|------|
| vedana, samjna, cetana | sparsha, vedana, samjna, cetana, manasikara | +sparsha (構成事件), +manasikara (注意力快照) |

三因素模型的缺陷: (1) 缺觸 -- 此認知瞬間如何開始? (2) 缺作意 -- Agent 當時在注意什麼?

> **DC-8 Master 確認**：五遍行五欄位 (sparsha, vedana, samjna, cetana, manasikara) 為**參考設計** (reference design)，不強制鎖定。與 DC-6 精神一致（不鎖定蘊內子介面）。Master 原話：「參考就好。」

[來源: R3 Debate 2 R2.1-R2.3]

---

## 2. 五遍行: 教義基礎 (Five Universals: Doctrinal Basis)

### ASANGA (#8): sarvatraga caitta = 每一認知瞬間的最低必要條件

> - **《五蘊論》(Pancaskandhaprakarana, 世親)**: 列五遍行為心所法之首
> - **《大乘阿毘達磨集論》(Abhidharmasamuccaya, 無著)**: 「觸、受、想、思、作意，此五心所遍一切心。」
> - **《雜阿含經》SA 306**: 「緣眼、色，生眼識，三事和合，是名為觸；觸俱生受、想、思。」

| # | Sanskrit | Chinese | 功能 | TypeScript Field | Skandha |
|---|----------|---------|------|-----------------|---------|
| 1 | Sparsha | 觸 | 根+境+識三合 (構成事件) | `sparsha: SparshEvent` | rupa |
| 2 | Vedana | 受 | 效價感受 (hedonic tone) | `vedana: ChannelVedana` | vedana |
| 3 | Samjna | 想 | 辨識取相 (recognition) | `samjna: ChannelSamjna` | samjna |
| 4 | Cetana | 思 | 意志趨向 (volitional drive) | `cetana: ChannelCetana` | samskara |
| 5 | Manasikara | 作意 | 注意力指向 (attention) | `manasikara: ChannelManasikara` | vijnana |

**Sarvatraga = 遍一切心 = 在每一認知瞬間必然同時俱起。**

[來源: R03 Sec 3.2 L196-216, R01 Sec 1.2 L162-168]

---

## 3. 為什麼是五而非三? (Why Five, Not Three?)

### ASANGA (#8): 缺 sparsha -- 此認知瞬間如何開始?

「Agent 感到不愉快、辨識出紅燈、產生迴避趨向 -- 但什麼引發了這一切?」Sparsha (根+境+識三合) 是構成事件，不是外部觸發器。

### ASANGA (#8): 缺 manasikara -- Agent 當時在注意什麼?

「Vedana 說『苦受』、samjna 說『紅燈』、cetana 說『迴避』-- 但 Agent 看的是前方路口還是後視鏡?」Manasikara 提供注意力上下文，使其他三者可解釋。

[來源: R3 Debate 2, ASANGA L317-339]

---

## 4. 功能性論證 vs 教條性論證 (Functional vs Doctrinal)

### NAGARJUNA (#7) 的關鍵限定

> 「包含 manasikara 的理由是『功能性』的 -- 注意力對可解釋的 bundle 而言是架構必需。不是『教義性』的 -- 不是因為瑜伽行派要求恰好五個。未來任何新增必須論證架構必要性，而非教義義務。」

原則: `IF doctrinal → REJECT`; `IF functional AND architecturally necessary → ACCEPT`。

### PASCAL (#19) 不對稱風險

Include manasikara: cost = -0.01ms/bundle (已知、有界)。Omit manasikara: cost = -epsilon (未知、可能無界 -- 消費者各自查詢 IGuide 導致不一致)。有界 vs 無界下行風險，明確偏好包含。

[來源: R3 Debate 2, NAGARJUNA L394-396, PASCAL L402-407]

---

## 5. SparshEvent 作為構成事件 (Constitutive Event)

```typescript
/** SparshEvent -- 觸: 根+境+識的三合 (trividha-samnipata) */
interface SparshEvent {
  readonly root: string;          // 根門 (indriya)
  readonly object: AgentEvent;    // 境 (visaya)
  readonly consciousness: string; // 識 (vijnana)
  readonly timestamp: number;
}
```

### Option A (sparsha 在 bundle 內) vs Option B (sparsha 觸發 bundle)

BABBAGE (#9): 兩者型別同構 -- $\text{SparshEvent} \to \text{Bundle}_4 \cong \text{Bundle}_5$。

**決議: Option A** -- sparsha 作為 bundle 欄位。教義準確 (五遍行都「在」認知瞬間之中)、零額外成本、bundle 自包含。

[來源: R3 Debate 2, BABBAGE L375-388]

---

## 6. ChannelManasikara: 快照而非計算 (Snapshot, Not Computation)

### ARCHIMEDES (#16): vijnana-clock 預算，vedana-clock 讀取

```
vijnana-clock (1-5ms):  IGuide.computeAttentionState() → 寫入快照
vedana-clock  (10-100ms): manasikara = readSnapshot()   → ~0.01ms
```

vijnana-clock 比 vedana-clock 更快，快照在 vedana-clock 觸發時已準備好。最大陳舊度 = 1-5ms。

```typescript
/** ChannelManasikara -- 作意: vijnana-clock 狀態快照，非 bundle 內計算 */
interface ChannelManasikara {
  /** 當前注意力焦點 (來自 IGuide) */
  readonly focus: string;
  /** 注意力強度 (0.0=周邊, 1.0=焦點) */
  readonly intensity: number;
}
```

[來源: R3 Debate 2, ARCHIMEDES L362-373]

---

## 7. 分類學考量 (Classification Concerns)

### LINNAEUS (#13): CoarisingBundle 橫跨四蘊

```
CoarisingBundle
├── sparsha     → rupa (色蘊)      │ 關聯性 (Relational)
├── vedana      → vedana (受蘊)    │ 構成性 (Constitutive)
├── samjna      → samjna (想蘊)    │ 構成性
├── cetana      → samskara (行蘊)  │ 構成性
└── manasikara  → vijnana (識蘊)   │ 關聯性
```

LINNAEUS 區分: vedana/samjna/cetana 是 bundle 的**構成性屬性** (what it IS); sparsha/manasikara 是**關聯性屬性** (what it REFERENCES)。包含關聯性屬性不改變分類，而是豐富語義。CoarisingBundle 是**事件層級的多蘊結構**。

[來源: R3 Debate 2, LINNAEUS L331-333, L398-400]

---

## 8. 完整 CoarisingBundle 型別 (Complete Type)

```typescript
// packages/sdk/src/types/coarising.ts

/**
 * CoarisingBundle -- 五遍行不可分離
 * R3 D2: 功能性論證。未來新增必須論證架構必要性。
 *
 * DC-8 Master 確認：五遍行五欄位為參考設計，不強制鎖定。
 * 與 DC-6 精神一致（不鎖定蘊內子介面）。
 */
interface CoarisingBundle {
  // -- 五遍行 (sarvatraga) --
  readonly sparsha: SparshEvent;              // 遍行 1: 觸
  readonly vedana: ChannelVedana;             // 遍行 2: 受
  readonly samjna: ChannelSamjna;             // 遍行 3: 想
  readonly cetana: ChannelCetana;             // 遍行 4: 思
  readonly manasikara: ChannelManasikara;     // 遍行 5: 作意
  // -- 架構元資料 --
  readonly layer: 1 | 2;
  readonly mode: 'fast' | 'slow';
  readonly sahaja: SahajaContract;
  readonly timestamp: number;
}

/** ChannelVedana -- 受: 連續測量 (R3 D5, Russell 1980 circumplex) */
interface ChannelVedana {
  readonly valence: number;    // -1.0 (苦) 到 +1.0 (樂)
  readonly intensity: number;  // 0.0 到 1.0
  readonly type: 'dukkha' | 'sukha' | 'upekkha';  // 從 valence 派生
  readonly source: string;
}

/** ChannelSamjna -- 想: 辨識通道 */
interface ChannelSamjna {
  readonly label: string;
  readonly method: 'rule' | 'llm';
  readonly confidence: number;
}

/** ChannelCetana -- 思: 意志通道 */
interface ChannelCetana {
  readonly tendency: 'approach' | 'avoid' | 'maintain';
  readonly urgency: number;
}
```

[來源: 02_type_system_changes.md Sec 2.2-2.4]

---

## 9. SahajaContract: 品質元資料 (Quality Metadata)

```typescript
/**
 * SahajaContract -- 俱生品質自描述
 * 究竟真理: 真俱生 = 零時間差 (不可能)。
 * 世俗真理: 滿足三準則 = 工程有效的俱生。
 */
interface SahajaContract {
  readonly mutualConsistency: boolean;      // 組件彼此參照
  readonly atomicPublication: boolean;      // 外部不見部分 bundle
  readonly stalenessUpperBound: number;     // 最舊與最新組件時差 (ms)
}
```

| 層 | mutualConsistency | atomicPublication | stalenessUpperBound |
|----|:-:|:-:|---:|
| Layer 1 (根門) | true | true | <1ms |
| Layer 2 fast | true | true | <100ms |
| Layer 2 slow | true | true | <0.29 * T_outer |

消費者可透過 `bundle.sahaja` 欄位決定信任等級。

[來源: R3 Debate 1, BABBAGE L160-173, NAGARJUNA L148-158]

---

## 10. 五蘊完備性檢查 (Skandha Completeness Check)

來源: Cycle 02-2 Q1 -- Master: 「Agent 完備即可啟動; 需開發者模式，不完備也可啟動但提醒。」

```typescript
interface SkandhaCompletenessReport {
  readonly rupa:     { present: boolean; components: string[] };
  readonly vedana:   { present: boolean; components: string[] };
  readonly samjna:   { present: boolean; components: string[] };
  readonly samskara: { present: boolean; components: string[] };
  readonly vijnana:  { present: boolean; components: string[] };
  readonly isComplete: boolean;
  readonly missing: string[];
}

// AgentCore.start()
async start(options?: { developerMode?: boolean }): Promise<void> {
  const report = this.checkSkandhaCompleteness();
  if (!report.isComplete) {
    if (options?.developerMode) {
      logger.warn(`[DEV] Skandha incomplete: ${report.missing.join(', ')}`);
      this.bus.emit({ type: 'agent:skandha_incomplete', report, developerMode: true });
    } else {
      throw new Error(`Skandha incomplete: ${report.missing.join(', ')}`);
    }
  }
}
```

| 模式 | 五蘊不全時 | 用途 |
|------|-----------|------|
| 正常 | throw Error | 生產環境 |
| 開發者 | warn + emit `agent:skandha_incomplete` + 繼續 | 開發/測試 |

[來源: 02_q1q4_engineering_spec.md#Q1]

---

## 11. 多值 Skandha (Multi-Value Skandha) -- M-7

CoarisingBundle 的多蘊性質與 M-7 的 PluginManifest 多值 skandha 在不同架構層級:

| 層級 | 機制 | 例子 |
|------|------|------|
| **插件層** | `skandha: Skandha \| readonly Skandha[]` | 同時提供 VasanaRule+IProvider 的插件 |
| **事件層** | CoarisingBundle 本身多蘊 | 每個 bundle 含 vedana+samjna+samskara+vijnana |

> LINNAEUS: 「這兩者不矛盾 -- 它們在不同的架構層級運作。」

[來源: 02_type_system_changes.md Sec 4.4, R3 Debate 2 R2.4]

---

## 12. 隱私與內部狀態 (Privacy)

### GUARDIAN (#11): Bundle 是 Agent 內部資料

```
Agent A (internal)              Agent B (internal)
  CoarisingBundle                 CoarisingBundle
  [manasikara: ...]               [manasikara: ...]
       │ only actions visible          │
       ▼                               ▼
    Coordination Daemon (交換 action proposals，非原始 bundle)
```

- 跨 Agent 通訊僅交換 action proposals，不交換原始 bundle
- **manasikara 是敏感資料**: 若 bundle 被外部記錄 (除錯/稽核)，應特別處理

[來源: R3 Debate 2, GUARDIAN L345-349]

---

## 13. 注意力感知快取 (Attention-Aware Caching)

### HERACLITUS (#15): 語義快取失效

```typescript
function isCacheStale(cached: CoarisingBundle, current: ChannelManasikara): boolean {
  if (Date.now() - cached.timestamp > TTL) return true;
  if (cached.manasikara.focus !== current.focus) return true;  // 注意力已移轉
  return false;
}
```

注意力焦點改變 = 快取語義失效，不論年齡。比純 TTL 更精準。

[來源: R3 Debate 2, HERACLITUS L351]

---

## 14. 控制系統可觀測性 (Observability)

### WIENER (#12): 無測量位置 = 不可觀測

反饋系統若不記錄測量位置，系統狀態不可從輸出重建。Manasikara 提供測量上下文 -- 「苦來自 eye-gate (需矯正視覺)」vs「苦來自 mano-gate (需矯正認知)」。缺少 manasikara，PID 控制器無法區分苦的來源。

[來源: R3 Debate 2, WIENER L390]

---

## 15. 根門 vs 歸依意 Bundle (Per-Root vs Mano)

兩種 bundle 都包含完整五遍行，差異僅在產生方式和時序:

```
Layer 1 (根門):                     Layer 2 (歸依意):
┌──────────────────────┐            ┌──────────────────────┐
│ sparsha: eye+red     │            │ sparsha: mano+dharma  │
│ vedana: {-0.7, 0.8}  │            │ vedana: {-0.5, 0.6}  │
│ samjna: "red_light"  │            │ samjna: "brake_now"   │
│ cetana: "avoid"      │            │ cetana: "brake"       │
│ manasikara: "fwd"    │            │ manasikara: "road"    │
│ layer: 1, mode:'fast'│            │ layer: 2, mode:*      │
│ sahaja: {<1ms}       │            │ sahaja: {<100ms|<Xs}  │
└──────────────────────┘            └──────────────────────┘
Rule-based, vedana-clock            Dual-gear mano-clock
```

> 《大拘絺羅經》(MN 43): 「五根有歸依意，意為彼盡領受境界。」
> Layer 1 根門各自處理，Layer 2 歸依意整合全局。兩層結構相同 (五遍行)，認知深度不同。

[來源: R03 Sec 3.4-3.5, Doc 30 Sec 3]

---

*R3 Debate 2 共識架構文件。五遍行基於功能性論證，型別系統強制完備性，SahajaContract 提供品質自描述。*
