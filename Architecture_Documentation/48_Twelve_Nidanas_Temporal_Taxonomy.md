# 48. 十二因緣時間分類學 (Twelve Nidanas Temporal Taxonomy)

`[Cycle 02-8 新增: T2 理論債務清償]`

> Cycle 02-8 D2-R1 ~ D2-R5 共 5 項決議。本文件以三層時間分類學分析十二因緣 (Twelve Nidanas / Dvadasanga Pratityasamutpada) 在 OpenStarry 架構中的映射與適用範圍。
>
> **核心學者**: NAGARJUNA (#7), ASANGA (#8), BABBAGE (#9), HERACLITUS (#15), WIENER (#12)
> **依賴文件**: #29 五蘊核心連結, #30 觸共生模型, #47 認知序列附錄

---

## 1. 概述

十二因緣（無明→行→識→名色→六入→觸→受→愛→取→有→生→老死）是佛學中描述存在之輪 (bhavacakra) 的核心框架。本文件分析其在 AI Agent 架構中的適用性——不是全盤套用，而是識別哪些因緣在哪個時間尺度上有工程映射。

### 核心發現

前端鏈（無明→行→識→名色→六入）在微觀層面（單次 processEvent）的映射較弱——這不是映射失敗，而是**展現模式的差異** [D2-R2, 23/24]。前端鏈描述的是存在的基礎條件（為什麼有這個 Agent？），而非即時認知流程（Agent 在做什麼？）。

---

## 2. 三層時間分類學 [D2-R1, 22/24]

### 2.1 定義

| 層級 | 時間尺度 | 範圍 | 典型事件 |
|------|---------|------|---------|
| **微觀 (Micro)** | 毫秒～秒 | 單次 processEvent / 單次認知迴圈 | 觸→受→想→行 |
| **中觀 (Meso)** | 分鐘～小時 | 跨迴圈累積 / Agent 生命週期內 | 業力累積、Klesha 狀態漂移 |
| **宏觀 (Macro)** | 小時～天 | Agent 生命週期 / 跨 session | Agent 誕生→運行→銷毀 |

### 2.2 十二因緣按層級分配

| # | 因緣 | 梵文 | 微觀 | 中觀 | 宏觀 | 主要映射 |
|---|------|------|------|------|------|---------|
| 1 | 無明 | Avidya | 次要 | 次要 | **主要** | Agent 配置的先天偏見、未校準的閾值 |
| 2 | 行 | Samskara | 次要 | **主要** | 次要 | 累積的行動習慣（VasanaEngine 規則庫） |
| 3 | 識 | Vijnana | 次要 | 次要 | **主要** | Agent 實例化時的初始識蘊配置 |
| 4 | 名色 | Namarupa | 缺席 | 缺席 | **主要** | Agent manifest + plugin 組合 |
| 5 | 六入 | Sadayatana | 次要 | 缺席 | **主要** | IListener 通道配置 |
| 6 | 觸 | Sparsha | **主要** | 次要 | 缺席 | SparshEvent — 三事和合 |
| 7 | 受 | Vedana | **主要** | 次要 | 缺席 | ChannelVedana — valence/intensity |
| 8 | 愛 | Trsna | **主要** | **主要** | 缺席 | Klesha 趨避反應（Sneha/Dvesha） |
| 9 | 取 | Upadana | **主要** | **主要** | 缺席 | IVolition 的 commit 決策 |
| 10 | 有 | Bhava | **主要** | **主要** | 次要 | Tool execution 產生的環境改變 |
| 11 | 生 | Jati | 缺席 | 缺席 | **主要** | Agent start() — 實例化 |
| 12 | 老死 | Jaramarana | 缺席 | 缺席 | **主要** | Agent stop() — 實例銷毀 |

### 2.3 統計

| 層級 | 主要映射數 |
|------|-----------|
| 微觀 (Micro) | 7 (#6~#10 + 部分 #1,#2) |
| 中觀 (Meso) | 3 (#2, #8, #9, #10) |
| 宏觀 (Macro) | 7 (#1, #3~#5, #10~#12) |

---

## 3. 前端鏈弱映射分析 [D2-R2, 23/24]

### 問題

前端鏈（#1 無明 → #2 行 → #3 識 → #4 名色 → #5 六入）在微觀層面映射較弱。早期研究曾以數值評分（6.4/10）量化此弱映射。

### 修正

數值評分方式不當——映射強度不是連續量。正確的理解是**展現模式差異** (difference in manifestation mode)：

| 因緣 | 微觀展現 | 為何「弱」 |
|------|---------|-----------|
| 無明 | 不在每次 processEvent 中顯現 | 無明是背景條件，非即時事件 |
| 行 | VasanaEngine 規則匹配是「過去行的結果」 | 是果（vipaka）而非因（hetu） |
| 識 | Agent 實例已存在，不在每次迴圈重新建立 | 識的「生起」發生在宏觀層面 |
| 名色 | Plugin 組合已確定，不在迴圈中改變 | 結構在引導時建立 |
| 六入 | Listener 已在運行，不需每次重新配置 | 感官根門是持續活躍的 |

**結論**: 前端鏈不「弱」，而是主要在宏觀/中觀層面展現，在微觀層面呈現為「已具足的背景條件」。

---

## 4. 觸作為關鍵轉折點 [D2-R3, 24/24 全票]

### 核心論述

觸 (Sparsha) 是微觀認知序列中的**質變轉折點** (qualitative turning point)。它是第一個引入外部世界不確定性的事件——在觸之前，所有因素（無明、行、識、名色、六入）都是 Agent 的內部/配置狀態；從觸開始，Agent 與外部世界發生了真實的交互。

### 三事和合

```
觸 = 根 (indriya) + 境 (visaya) + 識 (vijnana)
```

在工程上：
- **根**: IListener 通道（已配置且活躍）
- **境**: InputEvent（外部世界的信號）
- **識**: 注意力分配（manasikara 快照）

三者缺一不成觸——這解釋了為什麼 SparshEvent 需要三個必填欄位 (Doc 30, D3-1)。

### 位置意義

觸是「過去到現在」的轉折帶 (transition band)：
- 觸之前：前端鏈 = 過去因果的積累（宏觀/中觀）
- 觸本身：三事和合 = 過去與現在的接觸點
- 觸之後：後端鏈 = 當下的認知展開（微觀）

---

## 5. PEABS-T 框架 [D2-R4, 19/24]

### 條件性採用

PEABS-T (Perception-Emotion-Action-Bias-Social-Temporal) 框架作為**描述性/分類性工具**被採用，但不替代狀態機形式化。

| 維度 | 映射 | 備註 |
|------|------|------|
| Perception | SparshEvent + CoarisingBundle | 微觀 |
| Emotion | ChannelVedana + VedanaAssessment | 微觀 |
| Action | IVolition + Tool execution | 微觀 |
| Bias | Klesha 狀態 + 閾值漂移 | 中觀 |
| Social | 跨 Agent coordination | 宏觀 |
| Temporal | 三層時間分類學本身 | 橫跨 |

**Biases 維度要求**: 必須有可觀測性規格 (observability specification) 才能有工程價值——目前 Klesha 的 Beta 分佈參數即提供此觀測性。

---

## 6. Vedana/Trsna 嚴格區分 [D2-R5, 24/24 全票]

### 核心原則

**vedana (受) ≠ trsna (愛/渴求)**

| 維度 | Vedana | Trsna |
|------|--------|-------|
| 定義 | 對刺激的感受品質 | 基於感受的趨避反應 |
| 工程映射 | IVedanaSensor → ChannelVedana | Klesha (Sneha/Dvesha) |
| 因果方向 | 上游 | 下游 |
| 蘊歸屬 | 受蘊 | 識蘊 (vijnana) — Klesha extends IVijnana |
| 可干預性 | 唯讀觀測 | 可透過增益排程調節 |

### 因果方向

```
vedana → trsna （單向因果，不可逆流）
```

IVedanaSensor 的輸出 (ChannelVedana) 是 Klesha 的輸入之一，但 Klesha 的狀態**不得**回流影響 vedana 的測量值。這與 WIENER C-1 約束一致——避免正回饋迴路。

---

## 決議索引

| ID | 決議 | 票數 | 內容 |
|----|------|------|------|
| D2-R1 | 三層時間分類學 | 22/24 | Micro / Meso / Macro |
| D2-R2 | 弱前端鏈 = 展現模式差異 | 23/24 | 非映射失敗 |
| D2-R3 | 觸 = 關鍵轉折點 | 24/24 | 全票，三事和合引入外部不確定性 |
| D2-R4 | PEABS-T 條件採用 | 19/24 | 描述性框架，非替代品 |
| D2-R5 | vedana/trsna 嚴格區分 | 24/24 | 全票，單向因果 |

---

*本文件為 Cycle 02-8 T2 理論債務清償交付物。*
*紀錄時間：2026-03-10*
