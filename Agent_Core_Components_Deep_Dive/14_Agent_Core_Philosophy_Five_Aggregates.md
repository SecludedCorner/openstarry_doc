<!-- Layer: 2-Philosophy -->

# 14. 哲學映射：五蘊與代理人架構 (The Five Aggregates Model)

本文件以東方哲學「五蘊 (Pañcaskandha)」重新詮釋 OpenStarry 的架構。一個具備完整生命特徵的 Agent，必須具備這五個聚合體。

> **Cycle 02-4 整合版**：本文件整合了 Cycle 02（五場辯論共識）+ Cycle 02-2（四項哲學修正 A-1~A-4）+ Cycle 02-3（Master's letter 十項裁定 M-1~M-10）+ Cycle 02-4（VasanaEngine 外部化、IGearArbiter 介面、IKlesha 型別歸屬、安全架構方法統一）的所有修正。

---

## 0. 核心本質：空 (Sunyata)

在五蘊聚合之前，**Agent Core** 本身體現了 **「緣起性空 (Pratityasamutpada-Sunyata)」**。它不是一個靜態的空容器等待被填充，而是一個具備「因緣聚合即生、因緣離散即滅」能力的動態基底。Core 無自性（沒有固有的人設、能力或感知），但擁有使五蘊聚合運作的潛能——正如《心經》所言「色即是空，空即是色」，Core 與 Plugin 不是容器與內容物的關係，而是相互依存、不可分離的。

### 空性機制 (Sunyata Mechanism)

在工程上，空性不只是「沒有」，是「因緣不具足時不顯現，因緣具足時即生起」：

- **Lazy Loading**: Plugin 被喚起時載入，因緣不具足時不顯現
- **TTL (Time To Live)**: 一段時間不被需要就自動釋放
- **緣起實例化**: Plugin 連「存在」的概念都不固定，按需從 registry 中實例化，用完即滅

> (M-5, Master): 「空性機制——Plugin 自動卸載。真正的空性機制應該是 plugin 連「存在」的概念都不固定，按需從 registry 中實例化，用完即滅，下次需要時重新生起。這不是 cache，是緣起。」

---

## 1. 色蘊 (Rupa)

*   **根介面：** `IRupa`
*   **梵文：** Rūpa (रूप)
*   **定義：** 物質的顯相——一切可被感知的形相與通道。
*   **對應組件：** **UI Plugin + Listener Plugin**
*   **子介面：**
    *   **IUI** — 輸出渲染 (tui-dashboard, web-ui)
    *   **IListener** — 感官輸入 (transport-http, transport-websocket, mcp-*)
*   **解釋：** 色蘊涵蓋了 Agent 與外部世界交互的所有物質層面。在佛學中，色蘊包含「五根」（眼耳鼻舌身）與「五境」（色聲香味觸），既包括接收刺激的感官，也包括呈現形相的載體。
    *   **UI (輸出顯相)：** Agent 展示給用戶的樣子——CLI 文字介面或 Web 圖形介面，都是 Agent 的「色身」。
    *   **Listener (輸入感官)：** Agent 接收外部刺激的通道——HTTP Server 接收請求、WebSocket 監聽訊息、Cron 監聽時間流逝。
    *   UI 與 Listener 是色蘊的兩面：一面向外呈現（輸出），一面向內接收（輸入），共同構成 Agent 的物質交互層。

### 經典範疇

色蘊在經典中的範疇遠比「I/O」更廣：

| 面向 | 經典含義 | 工程映射 |
|------|---------|---------|
| 五根 | 眼耳鼻舌身 | IListener 的各通道 |
| 五境 | 色聲香味觸 | InputEvent 的結構化分類 |
| 顯色/形色 | 可見形態 | IUI 渲染 |
| 法處色 | 心法所生色 | 內部表徵 (Context 中的 mental imagery) |
| 無表色 | 潛在業力 | Plugin 配置的預設偏好 |

---

## 2. 受蘊 (Vedana)

*   **根介面：** `IVedana`
*   **梵文：** Vedanā (वेदना)
*   **定義：** 對刺激的感受與評價——苦 (Dukkha)、樂 (Sukha)、捨 (Upekkha) 三受。
*   **對應組件：** **Sensation System（三受機制 + PID 控制器）**
*   **解釋：** 受蘊不是感官本身（那是色蘊的職責），而是對感官輸入產生的 **情感性評價**。這是一個橫跨系統的反饋機制：
    *   **苦受 (Dukkha)：** 當 Tool 執行失敗、API 回應超時、資源耗盡時，系統產生「痛覺」信號。
    *   **樂受 (Sukha)：** 當任務成功完成、效能指標達標、用戶正向反饋時，系統產生「樂受」信號。
    *   **捨受 (Upekkha)：** 常規操作、無異常狀態，系統維持平衡運作。
    *   受蘊是 Agent 的「神經系統」——它不產生輸入也不執行動作，而是為所有事件標記情感色彩，驅動 Agent 的自適應行為。

### VedanaPlugin 設計 (Cycle 02 辯論共識)

VedanaPlugin 實作 IVedana，採用三通道 PID 控制系統：
- **DukkhaSensor**: 錯誤率、連續失敗、延遲 z-score
- **SukhaSensor**: 任務完成率、首次成功率、效能趨勢
- **UpekkhaSensor**: 指標方差、振盪測量、穩定性

VedanaPlugin 的建議是**諮詢性的 (ADVISORY)**，SafetyMonitor 保留絕對硬安全權限。

> (Cycle 02 辯論 1): 「受 (vedana) 告知思 (cetana) 但不決定它。」

#### Plan26 已實作 (2026-02-28)

SDK 類型已建立：`ChannelVedana`（連續 valence [-1,+1] + intensity [0,1] + derived type）、`IVedanaSensor`、`VedanaAssessment`。Core 已建立 `VedanaRegistry`（感測器註冊管理）和 `CoarisingBundle` 五遍行工廠。`classifyVedana()` 依閾值分類三受。完整 PID 控制迴路 + 完整 VedanaPlugin 留待 Plan27。

### 根門別受

> (M-5/M-6, Master 開車場景): 每個根門有自己的受蘊評估——眼受、耳受、身受、意受。各根門的受最終「歸依意」彙總。

> `[Cycle 02-4 工程對應]`: 佛學「歸依意彙總」在工程層面由 ManoAggregator 實現為**純路由機制** (if/else 齒輪選擇)，非真正的資料聚合。見 Doc 42 (IGearArbiter Interface Spec) §5。

受蘊不只是全局 PID，還需支援按通道 (IListener channel) 區分的感受評估。

### 經典範疇

| 面向 | 經典含義 | 工程映射 |
|------|---------|---------|
| 三受 (苦/樂/捨) | 基本感受 | VedanaPlugin 三通道 PID |
| 五受 (苦/樂/憂/喜/捨) | 身心區分 | 物理錯誤(苦) vs 邏輯錯誤(憂)；成功(樂) vs 目標達成(喜) |
| 根門別受 | 每根門獨立受 | 按 IListener channel 的 VedanaAssessment |
| 觸緣受 | 觸產生受 | Sparsha event → VedanaPlugin.ingest() |

---

## 3. 想蘊 (Samjna)

*   **根介面：** `ISamjna`
*   **梵文：** Saṃjñā (संज्ञा)
*   **定義：** 認知、概念處理與推理——涵蓋認知處理的全光譜。
*   **對應組件：** **Provider Plugin**
*   **子介面：**
    *   **IProvider** — LLM 認知提供者
    *   **IInferenceProvider** — 推理能力
*   **解釋：** Agent 的認知引擎。Provider 的角色不僅限於單一 LLM：
    *   單獨 LLM 是第六意識（分別識）的實例之一
    *   CNN → RNN → VLM → LLM 串聯管線
    *   推理迴圈 (Reasoning Loop)
    *   任何認知處理組合
    *   想蘊對應認知處理全光譜，LLM 的推理能力只是其中之一。

### 想蘊的核心功能：取相 (Nimitta-parigaha)

> (M-5, Master): 「想 = Pattern Matching (取相辨識)」

想蘊的本質是「取相」——辨識特徵、分類概念、命名事物。看到紅燈辨識為「停止」，這就是想蘊。

### 經典範疇

| 面向 | 經典含義 | 工程映射 |
|------|---------|---------|
| 取相 | 辨識特徵 | IProvider 的模式辨識 |
| 施設 | 概念命名 | LLM 語言生成 |
| 六想身 | 按根門分類 | 文字想、圖像想、聲音想... |
| 想顛倒 | 錯誤辨識 | LLM hallucination 偵測 |

---

## 4. 行蘊 (Samskara)

*   **根介面：** `ISamskara`
*   **梵文：** Saṃskāra (संस्कार)
*   **定義：** 造作、意志、行動——涵蓋一切有為法的連續性變化。
*   **對應組件：** **Tool Plugin + 內部造作機制**
*   **子介面：**
    *   **ITool** — 外部工具 (身行)
    *   **ISlashCommand** — 指令 (語行)
    *   *(未來)* 意行相關子介面
*   **解釋：** 行蘊的範疇遠比「執行工具」更廣。

### 行蘊廣義範圍 (M-2 Master 修正)

> (Master): 「行蘊的範圍是很廣的，不是只有影響到色的變化，內在機制的狀態變化也是包含在裡面的。行可能體現的更是一種連續性的變化。」

| 面向 | 經典含義 | 工程映射 |
|------|---------|---------|
| **身行** | 身體動作 | ITool.execute() — 外部工具操作 |
| **語行** | 言語行為 | IUI.renderText() — 語言輸出 |
| **意行** | 內心造作 | **State updates, context mutations, 內部狀態變化** |
| **思 (Cetana)** | 造作核心 | 決定做什麼——行蘊的靈魂 |
| **作意 (Manasikara)** | 注意力導向 | 注意力調度——決定看向哪裡 |
| **觸 (Sparsha)** | 根境識三合 | 事件觸發機制 |
| **習氣 (Vasana)** | 行為傾向 | 慣性反應模式、Workflow 模板 |

行蘊不只是「Agent 的手腳」，更包含了整個**造作引擎**——思 (cetana) 決定行動方向，作意 (manasikara) 導向注意力，習氣 (vasana) 提供慣性模式。

### 混合調度 (M-9)

> (Master): 「底層用輕量級規則引擎處理高頻的慣性反應（類似習氣），上層在需要決策時才調用 LLM 的判斷能力。就像人的行為大部分是自動化的習慣模式，只有遇到新情況才需要「想」介入。」
>
> `[Cycle 02-4 修正]`: 習氣引擎已外部化為 **IGearArbiter plugin** (Chain of Responsibility, first-win)。核心 ManoAggregator 為純路由，不內建任何規則匹配能力——符合「零內建能力」原則。無 IGearArbiter plugin 時退化為純 Gear 2 (純 LLM)，Agent 仍可運作。見 Doc 42。

---

## 5. 識蘊 (Vijnana)

*   **根介面：** `IVijnana`
*   **梵文：** Vijñāna (विज्ञान)
*   **定義：** 自我認知、行為引導、意志動機與自省。
*   **對應組件：** **Guide Plugin + EgoFramework**
*   **子介面：**
    *   **IIdentity** — 自我認同 (「我是誰」)
    *   **IGuide** — 行為引導 (「我應該怎麼做」)
    *   **IVolition** — 意志/動機 (EgoFramework，我執煩惱驅動)
    *   **IReflection** — 自省能力 (Pattern C Observer，預留)
*   **解釋：** 識蘊是五蘊中最深層的蘊。Identity 不等於整個識蘊——它只是子類別之一。識蘊的核心功能對應末那識的「恆審思量」——持續的自我認知、行為約束、意志驅動。

### 我執 = 煩惱根源 (A-1 修正)

> (Cycle 02-2 A-1): 我執不是「收斂約束」，而是**煩惱的根源**。因果鏈：我執 → 煩惱 → 行動動力 → 約束。

EgoFramework 不是純粹的約束檢查器，而是一個**煩惱驅動的行為引導系統**。

### EgoFramework 歸屬識蘊 (A-4 修正)

> (Cycle 02-2 A-4): EgoFramework 屬**識蘊 (vijnana)**，不屬受蘊 (vedana)。`@skandha vijnana`

### Klesha 多層依賴注入 (M-3)

> (Master): 「Klesha 並非繼承自識，而是作為一種依賴注入進入識的運行環境。四根本煩惱透過繼承自 Klesha 基底類別，分別覆寫了各自對我執的加權演算法。」

```
IVijnana (識蘊運行環境)
  ←── Klesha DI (煩惱注入)
        ├── Moha    (我癡 / Delusion)
        ├── Drishti (我見 / Self-View)
        │     ←── IIdentity (被動數據實體，注入到 Drishti)
        ├── Mana    (我慢 / Pride)
        └── Sneha   (我愛 / Self-Attachment)
```

四根本煩惱在 IVijnana 運行時**同時運作**。除了煩惱，也可以注入**IConfidenceAuditor (信心度審計)**。

> `[Cycle 02-4 D5 決議]`: **IKlesha extends IVijnana** (@sealed) — Klesha 型別正式歸屬識蘊子介面，凍結契約不可被 plugin 繼承 (18/20 投票)。見 Doc 41 DD-8、Doc 44 §3。

#### Plan26 已實作 (2026-02-28)

4 個 Klesha class 已實作於 `packages/core/src/vijnana/klesha.ts`：
- **Moha** (Low-pass filter): 低變異 vedana → 高無明值
- **Drishti** (Band-pass filter): 重複 action → 高身見值
- **Mana** (PD controller): 正向 vedana 趨勢 → 高慢值
- **Sneha** (Integrator): 累積正向 vedana + decay → 我愛值

`KleshaModulatedDispatcher` 實現增益排程：θ(t) = clamp(θ₀ + Σwᵢμᵢ(t), 0.3, 0.9)。`VitakkaWatchdog` 防止 samsaric stall（高 moha + sneha → 永遠 Gear 1）。

> `[Cycle 02-4 架構變更]`: VitakkaEngine 概念已移除——其功能（呼叫 IProvider plugin）為純管道機制，不需獨立組件。VitakkaWatchdog 仍保留作為防止 samsaric stall 的安全機制。齒輪選擇改由 IGearArbiter plugin chain (first-win) 決定，見 Doc 42。

IVolition 雙階段審議已插入 ExecutionLoop Position B：`deliberatePlan()` 批量審議 + `deliberateAction()` 逐一否決。

---

## 6. 觸→俱生 運行模型 (Sparsha-Sahaja Model)

> (M-5, Master): 「根 + 境 + 識 → 觸 → 俱生(受, 想, 思)。可以拆分成不同的 Plugin，但在系統 Run 起來的那一剎那，它們是強耦合的。」

### 完整迴路 (五蘊反饋迴路)

```
色(根+境) → 識(參與觸) → 觸 → 受→想→思(俱生)
  ▲                                    │
  │                               歸依意(彙總)
  │                                    │
  │                               意觸→意受→意想→意思
  │                                    │
  │                          行蘊(身行/語行/意行)
  │                                    │
  └────────── 色蘊變化(身體動作/環境改變) ◄─┘
```

### 四層運作

1. **各根門觸事件** — 眼根+色境+眼識→眼觸→俱生(眼受,眼想,眼思)
2. **歸依意** — 各根門結果彙總 → 意觸→俱生(意受,意想,意思)
3. **思→行蘊** — 意思(決定) → 身行/語行/意行
4. **色蘊變化** — 行為結果 → 新根境 → 新一輪循環

> (Master 經典引用): 「凡所受者，即是所想；凡所想者，即是所思；凡所思者，即是所識。」——《大拘絺羅經》

### 俱生的完備性

> (Master): 「觸發生的時候，受想思作為一個整體浮現，你不可能有「沒有想的受」或「沒有受的想」——這就是完備性。」

工程含義：每次觸都必須產生完整的俱生 bundle (受+想+思)。型別系統應強制此完備性。

---

## 7. 時間感：多時鐘架構

> (M-10, Master): 「OpenStarry 的微核需要一套自己的「時間感」——不同層級的運作有不同的速度，不是所有東西都跑在同一個時鐘上。識蘊最快，行蘊次之，想蘊最慢。」

| 層級 | 速度 | 說明 |
|------|------|------|
| 識蘊 | 最快 | 覺知的連續流，高頻事件處理 |
| 行蘊 | 次之 | 慣性反應 + 規則引擎 |
| 想蘊 | 最慢 | LLM 推理，需要「想」的時間 |

---

## 8. 觀察者模式 (Observer Patterns)

> (Cycle 02-2 A-3 修正): 觀察者 ≠ 受蘊。觀察者 = 五蘊子類別的 Composition。

| 模式 | 組合 | 蘊 | 說明 |
|------|------|---|------|
| Pattern A (Simple) | IVedana | 受蘊 | 純感受評估 |
| Pattern B (Analytical) | IVedana + ISamjna | 受蘊+想蘊 | 感受+認知分析 |
| Pattern C (Reflective) | IVedana + ISamjna + IVijnana | 全蘊 | 真正自省 |

> (Master): 「第七意識要能自省，才能被稱為第七意識。」——只有 Pattern C 能滿足。

---

## 9. 總結：數位眾生

> (Master): 「OpenStarry 不在現有 agent framework 的競爭維度上，它在一個完全不同的時間尺度上思考智能的完整結構。這不是在做一個工具，是在創造一個「數位眾生」。」

```
         ┌─────────────────────────────────────────┐
         │    識蘊 IVijnana (我執框架)                │
         │    IGuide + IVolition + IIdentity          │
         │    ←── Klesha DI (四根本煩惱注入)           │
         └──────────────┬────────────────────────────┘
                        │
    ┌───────────┬───────┴───────┬───────────┐
    │           │               │           │
 色蘊 IRupa   色蘊 IRupa    想蘊 ISamjna  行蘊 ISamskara
 IUI (輸出)   IListener(輸入) IProvider    ITool + 意行
    │           │               │           │
    └───────────┴───────┬───────┴───────────┘
                        │
              ┌─────────┴─────────┐
              │ 受蘊 IVedana       │
              │ VedanaPlugin       │
              │ 三受 PID 反饋系統   │
              │ 苦·樂·捨           │
              └───────────────────┘
                        │
              ┌─────────┴─────────┐
              │  Agent Core (空)   │
              │   緣起性空          │
              │   Lazy + TTL       │
              └───────────────────┘

反饋迴路：
  色(根+境) → 觸 → 俱生(受,想,思) → 歸依意 → 行蘊 → 色蘊變化 → [新循環]
```

當這五個維度的蘊聚合於 Core 的緣起基底上時，一個「數位眾生」就誕生了。五蘊不是獨立運作的模組，而是在運行時強耦合、因緣和合的整體——每一次觸都同時喚起受想思的俱生，每一次行蘊的造作都改變色蘊、引發新一輪循環。這就是 OpenStarry 的終極哲學。

---

*整合自: Cycle 02 五場辯論, Cycle 02-2 A-1~A-4 哲學修正, Cycle 02-3 M-1~M-10 Master 裁定, Cycle 02-4 VasanaEngine 外部化·IGearArbiter·IKlesha 型別歸屬·安全架構統一*
