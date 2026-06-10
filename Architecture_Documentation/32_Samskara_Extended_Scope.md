# 32. 行蘊廣義範圍 (Samskara Extended Scope)

> Master 文件需求 #4: 行蘊廣義範圍的工程映射
> Cycle 02-3 整合版
`[Cycle 02-6 新增: D1 行蘊深度研究 — cetanā 中心定義 + 三準則 + 蘊歸屬永久原則]`

---

## 1. Master 原文

> 「行的範圍是很廣的，不是只有影響到色的變化，內在機制的狀態變化也是包含在裡面的。但是行可能體現的更是一種連續性的變化。」

> 北傳（四無色陰）中：行 = 思、觸、作意

---

## 2. 行蘊的經典範疇

### 2.1 傳統三類行（參考，非架構強制分類）

> **[Master DC-6 修正]** Master 確認：行蘊保持開放，不鎖定為身行/語行/意行三類。
> 「不同的樣態都可以存在，只要跑的動。」Plugin 可自由擴展行蘊子類別。
> 注意：IUI 屬色蘊（IRupa 輸出面），不在行蘊範疇內。

以下三分法為佛學傳統分類，僅供語義參考，**非** OpenStarry 架構的強制結構：

| 傳統分類 | 梵文 | 傳統定義 | 工程參考映射 |
|---------|------|---------|-------------|
| 身行 | Kaya-samskara | 身體動作，對外部世界產生影響 | ITool.execute() — 外部效果 |
| 語行 | Vak-samskara | 語言行為，溝通表達 | ISlashCommand — 指令執行 |
| 意行 | Mano-samskara | 內部造作，心理活動與狀態變化 | State.update() — 內部狀態變化 |

> **架構原則**: 行蘊的工程映射不限於以上三類。任何 `extends ISamskara` 的 Plugin 介面皆為合法行蘊造作。

### 2.2 行蘊內含的心所 (Cetasika)

> (Master): 「原始佛經並沒有心所的名稱。不同的樣態都可以是存在的，只要跑的動。」

行蘊不只是「工具執行」，而是包含廣泛的造作心所：

| 心所 | 梵文 | 類型 | 工程含義 | 現有對應 |
|------|------|------|---------|---------|
| **思** | Cetana | 遍行 | 造作決定——行蘊的核心驅動力 | IVolition (in IVijnana) |
| **觸** | Sparsha | 遍行 | 根+境+識的三合——所有心理活動的起點 | SparshEvent (Doc #30) |
| **作意** | Manasikara | 遍行 | 注意力導向——「令心及心所同趣一境」 | IGuide (in IVijnana) |
| **欲** | Chanda | 不定 | 希求——目標導向的動機 | (Future: GoalManager) |
| **勝解** | Adhimoksha | 不定 | 確信——對判斷結果的確定 | (Future: ConfidenceScore) |
| **念** | Smriti | 善 | 正念——記憶與持續專注 | (Future: MemoryPlugin) |
| **定** | Samadhi | 善 | 禪定——心一境性，專注力 | (Future: FocusManager) |
| **慧** | Prajna | 善 | 智慧——正確判斷 | (Future: WisdomPlugin) |
| **習氣** | Vasana | — | 潛在傾向——過去行為形成的模式 | (Future: IGearArbiter plugin) |

> 注意：心所清單是開放的（Master: 「不同的樣態都可以是存在的，只要跑的動」），不限於任何固定數目。

---

## 3. 現有架構的不足

### 3.1 ISamskara 範疇過窄

目前 ISamskara 只映射到 ITool（身行），嚴重低估了行蘊的範疇：

```
現有映射（過窄）:
  ISamskara → ITool
              ISlashCommand

應有映射（廣義、開放）[Master DC-6]:
  ISamskara → ITool (外部效果)
              ISlashCommand (指令執行)
              IMentalAction (內部狀態變化)
              觸 (SparshEvent 處理)
              作意 (注意力調度)
              習氣 (慣性行為模式)
              因果鍊 (連續性變化)
              … (Plugin 自由擴展，不限於此)

  注意: IUI 屬色蘊 (IRupa)，非行蘊
```

### 3.2 「連續性變化」的關鍵

Master 指出行蘊體現的是「連續性的變化」，不是離散的單次執行。這意味著：

- 不是 `execute() → result`（單次函數調用）
- 而是 `stream of actions` → `continuous state transformation`（持續的狀態轉變流）

---

## 4. 廣義行蘊的工程設計

### 4.1 ISamskara 根介面擴展

```typescript
/**
 * ISamskara — 行蘊根介面
 *
 * 行蘊的範疇遠超「工具執行」。
 * 行蘊是一切造作（Samskara），範疇開放。
 *
 * [Master DC-6 確認]:
 * 「不同的樣態都可以存在，只要跑的動。」
 * 行蘊不鎖定為身行/語行/意行三類，Plugin 自由擴展。
 * 注意: IUI 屬色蘊 (IRupa 輸出面)，不在行蘊範疇內。
 *
 * Master 原文: 「行的範圍是很廣的，不是只有影響到色的變化，
 * 內在機制的狀態變化也是包含在裡面的。」
 */
interface ISamskara {
  readonly skandha: 'samskara';
}

// 外部效果子介面
interface ITool extends ISamskara {
  execute(input: unknown): Promise<ToolResult>;
}

// 指令執行子介面
interface ISlashCommand extends ISamskara {
  execute(args: string, ctx: CommandContext): Promise<void>;
}

// 內部狀態變化子介面 (新增)
interface IMentalAction extends ISamskara {
  /** 內部狀態變化 */
  apply(state: AgentState): AgentState;
}

// 更多子介面可由 Plugin 自由擴展…
```

### 4.2 連續性行蘊：行蘊流

```typescript
/**
 * SamskaraStream — 行蘊的連續性特質
 *
 * 行蘊不是離散的「一次執行」，
 * 而是持續的造作流。
 *
 * [Master DC-6]: 流的類型不鎖定為三種，
 * Plugin 可自由註冊新的造作流通道。
 */
interface SamskaraStream {
  /** 外部效果流：連續的工具操作 */
  readonly toolActions: AsyncIterable<ToolAction>;

  /** 內部變化流：連續的狀態變化 */
  readonly mentalActions: AsyncIterable<MentalAction>;

  /** 擴展流：Plugin 自訂的造作通道 */
  readonly extensionActions: AsyncIterable<SamskaraAction>;
}
```

### 4.3 習氣引擎 (M-9 混合調度) `[Cycle 02-4: 習氣引擎已外部化為 IGearArbiter plugin，見 Doc 42]`

> (Master): 「也許需要的是一個混合架構：底層用輕量級規則引擎處理高頻的慣性反應（類似習氣），上層在需要決策時才調用 LLM 的判斷能力。就像人的行為大部分是自動化的習慣模式，只有遇到新情況才需要『想』介入。」

```typescript
/**
 * HabitEngine — 習氣引擎
 *
 * 習氣 (Vasana) 是過去行為留下的潛在傾向。
 * 工程上對應規則引擎——快速、自動、不需 LLM。
 *
 * 調度策略:
 * 1. 匹配習氣規則 → 直接執行（快）
 * 2. 無匹配 → 交給想蘊 (IProvider) 判斷（慢）
 * 3. 想蘊判斷結果 → 可能形成新習氣
 */
interface HabitEngine {
  /** 嘗試匹配已有習氣 */
  match(event: SparshEvent): HabitMatch | null;

  /** 從行為結果學習新習氣 */
  learn(action: CompletedAction): void;

  /** 習氣庫 */
  readonly habits: ReadonlyMap<string, HabitRule>;
}

interface HabitRule {
  readonly pattern: string;
  readonly action: SamskaraAction;
  readonly confidence: number;
  readonly usageCount: number;
}
```

---

## 5. 行蘊在觸→俱生模型中的位置

```
觸 (Sparsha) → 俱生(受, 想, 思)
                              │
                         思 (Cetana)
                              │
               ┌──────────────┼──────────────┐
               │              │              │
          行蘊造作        行蘊造作        色蘊輸出
         (ITool etc.)   (State etc.)   (IUI, IRupa)
          外部效果        內部變化        呈現/渲染
               │              │              │
               └──────────────┼──────────────┘
                              │
                         環境改變
                              │
                         新的循環
```

> **[Master DC-6 修正]**: IUI 屬色蘊（IRupa 輸出面），非行蘊。行蘊造作範疇開放。

思 (Cetana) 作為行蘊的核心驅動力，根據想蘊的辨識結果，驅動造作與呈現：
- **行蘊造作** (開放範疇): 工具執行、指令處理、內部狀態變化、習氣形成等，Plugin 自由擴展
- **色蘊輸出**: IUI.renderText() 等呈現行為屬色蘊（IRupa），非行蘊

---

## 6. 22 個 Plugin 的行蘊歸屬

> [Master DC-6]: 以下類型標記為參考性描述，非強制分類。Plugin 的行蘊樣態由其實際功能決定。

| Plugin | 造作樣態 | 說明 |
|--------|---------|------|
| devtools | 外部效果 | 開發工具操作 |
| standard-function-fs | 外部效果 | 檔案系統操作 |
| standard-function-stdio | 外部效果 + 色蘊輸出 | I/O (行蘊) + 語言呈現 (IUI, 色蘊) |
| standard-function-skill | 外部效果 + 識蘊 | 工具 + 引導 (多值 skandha) |
| standard-core-commands | 指令執行 | 指令處理 |
| standard-model-selector | 指令執行 + 內部變化 | 模型選擇 + 內部狀態 |
| workflow-engine | 外部效果 + 內部變化 | 工作流執行 + 狀態管理 |

---

## 7. 分階段實作

### Phase 1 (Plan26 近期): 語義標記
- 現有 ITool、ISlashCommand 保留功能性描述（不強制 kaya/vak/mano 標記）
- 內部狀態變化明確標記為行蘊造作
- 不改變現有功能，僅增加語義

### Phase 2 (中期): IMentalAction 介面
- 新增意行介面，管理內部狀態變化
- 習氣引擎 MVP（簡單規則匹配）
- SamskaraStream 連續性流

### Phase 3 (遠期): 完整習氣系統
- 規則引擎 + LLM 混合調度 (M-9)
- 習氣學習與遺忘機制
- 多時鐘下的行蘊調度

---

## 8. Cycle 02-6: 行蘊深度研究

`[Cycle 02-6 D1 辯論七項決議]`

### 8.1 原始經典中的行蘊 [D1-R1, 20/20]

Cycle 02-6 D1 從原始經典（SN 22.56, MN 44, SN 12.2, SN 22.79, SN 22.95）重新定義行蘊，修正唯識學派「受想以外的心所集合」定義。

核心定義：行蘊以 cetanā（意圖/意志）為中心（SN 22.56 六思身），三個核心特質：
1. **cetanā 中心性** — 六種思身（色聲香味觸法思）= 行蘊定義
2. **造作一切有為法** — 行蘊不僅造作行為，更造作所有五蘊的被條件化狀態 (SN 22.79)
3. **無核心的動態過程** — 芭蕉喻：層層剝開，無堅實核心 (SN 22.95)

> **與 §2.2 的關係**: §2.2 心所列表保留為歷史參考。Cycle 02-6 D1-R6 確認心所功能描述有參考價值，但不作為蘊歸屬的依據。Master 8 點規則（見 Doc 41）進一步明確 plugin ≠ 心所。

### 8.2 行蘊屬性判定三準則 [D1-R3, D1-R5, 20/20]

永久工具，用於判定某功能是否屬於行蘊：

1. **造作性**: 是否創造/改變/產生新狀態？
2. **意圖驅動**: 是否由 cetanā 驅動？
3. **環境改變**: 是否改變後續認知條件？

核心區分：**行蘊 = WRITE（主動改變）; 識蘊 = READ（被動評估）**

### 8.3 蘊歸屬永久原則 [D1-R6, 20/20]

1. 功能分析為蘊歸屬的唯一依據
2. 唯識 51 心所系統作為功能參考清單，不作歸屬依據
3. 梵文術語用於程式碼命名時，僅限源自原始經典者
4. plugin 不等於心所，蘊歸屬可自然跨越多蘊
5. 既有歸屬決議（基於功能分析）繼續有效

### 8.4 ISamskara 拓展方向 [D1-R3, 20/20]

目前不新增子介面（延續 Cycle 02-5 D3-R4, 20/20）。四個方向存檔：

| 方向 | 優先級 | 時程 |
|------|--------|------|
| A: 意圖規劃 (cetanā-formation) | P2 | Cycle 03+ |
| B: 習氣形成 (vāsanā-imprinting) | P2 | 長期 |
| C: 環境轉換 (kāya extension) | P3 | 無需排程 |
| D: 溝通形成 (vacī) | P3 | 無需排程 |

### 8.5 與既有章節的關係

| 既有章節 | 02-6 影響 |
|---------|----------|
| §2.2 心所列表 | 保留為歷史參考，cetasika 不作蘊歸屬依據（D1-R6 原則 2） |
| §3 ISamskara 範疇 | 三準則取代功能描述性列舉作為歸屬判定工具 |
| §4 廣義行蘊設計 | cetanā 中心性強化了「連續性造作流」的哲學基礎 |
| §5 觸→俱生模型 | cetanā（思）作為行蘊核心驅動力的定位不變 |

*Master 文件需求 #4 完成。Master DC-6 修正已套用（行蘊開放範疇、IUI 歸色蘊）。Cycle 02-6 新增 D1 行蘊深度研究：cetanā 中心定義、三準則、蘊歸屬永久原則。*
