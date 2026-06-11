# 47. 認知序列附錄 — 六狀態機與二諦框架 (Cognitive Sequence Appendix)

`[Cycle 02-8 新增: T1 理論債務清償]`

> Cycle 02-8 D1-R1 ~ D1-R5 共 5 項決議。本文件定義 Agent Core 執行迴圈的形式化認知序列模型——六狀態有限狀態機，以二諦（世俗諦/勝義諦）框架聲明其本體論地位。
>
> **核心學者**: BABBAGE (#9), NAGARJUNA (#7), ASANGA (#8), SUSSMAN (#22), KNUTH (#21)
> **依賴文件**: #01 執行迴圈, #29 五蘊核心連結, #35 雙齒輪末那時鐘, #39 共生束與五遍行, #45 五蘊 OOP 架構

---

## 1. 概述

Cycle 02-6 D1-R4a 排程的「認知序列附錄」(citta-vithi appendix)，本文件完成其交付。核心問題是：ExecutionLoop 的狀態機各狀態與佛學認知序列（觸→受→想→行→識）之間的對應關係如何形式化？

### 二諦聲明 (Two Truths Declaration) [D1-R1, 22/24]

> **世俗諦 (Conventional Truth / Samvrti-satya)**: 六狀態機是工程上有效的模型——它正確描述了 ExecutionLoop 的行為，可編譯、可測試、可驗證。
>
> **勝義諦 (Ultimate Truth / Paramartha-satya)**: 六個狀態是假名安立 (prajnapti)，非自性實有 (svabhava)。狀態之間的邊界是為工程目的而劃分的，不代表認知過程存在真正的間斷。

此聲明確保形式化不會導致認知過程的「實體化」(reification) 謬誤。

---

## 2. 六狀態有限狀態機 [D1-R1, 22/24]

### 2.1 狀態集

```
S = { S_sparsha, S_vedana, S_samjna, S_samskara, S_vijnana, S_quiescent }
```

| 狀態 | 工程對應 | 蘊歸屬 | 說明 |
|------|---------|--------|------|
| S_sparsha | SparshEvent 形成 | 色蘊 (rupa) | 三事和合：根(感官) + 境(對象) + 識(注意力) |
| S_vedana | CoarisingBundle.vedana 計算 | 受蘊 (vedana) | 連續 valence/intensity 測量 |
| S_samjna | IGearArbiter/IProvider 認知處理 | 想蘊 (samjna) | Gear 1 快速辨認 或 Gear 2 LLM 深度認知 |
| S_samskara | IVolition.deliberate() + Tool execution | 行蘊 (samskara) | 意志審議 + 行動執行 |
| S_vijnana | VedanaAssessment + KleshaBayesianUpdate | 識蘊 (vijnana) | 回饋評估 + 閾值更新 |
| S_quiescent | WAITING_FOR_EVENT | — | 寂止態，等待新輸入事件 |

> **命名說明**: S_wait 更名為 S_quiescent（寂止態），更準確反映其本體論地位——不是「等待」（暗示有等待者），而是「寂止」（因緣不具足時的自然狀態）。

### 2.2 狀態轉移

```
S_quiescent --[InputEvent]--> S_sparsha
S_sparsha   --[三事和合完成]--> S_vedana
S_vedana    --[vedana 計算完成]--> S_samjna
S_samjna    --[認知結果產出]--> S_samskara
S_samskara  --[行動完成]--> S_vijnana
S_vijnana   --[評估完成]--> S_sparsha  (新一輪認知)
S_vijnana   --[無後續事件]--> S_quiescent
```

### 2.3 固定順序驗證

vedana → samjna → cetana（行蘊核心）的順序是**固定**的，不可重排。

| 理由 | 說明 |
|------|------|
| 因果依賴 | samjna 需要 vedana 的 valence/intensity 作為輸入（ManoAggregator 的齒輪選擇依賴閾值） |
| 控制論要求 | PID 控制器必須先有測量值 (vedana) 才能做出決策 (samjna/cetana) |
| 佛學一致 | SN 22.56 明確排列：觸→受→想→思 |

---

## 3. Cetana 分層處理 [D1-R2, 21/24]

Cetana（思/意志）在認知序列中的位置需要跨三個層面理解：

### Layer A: 計算層（工程實作）

Cetana 嵌入 S_samskara 狀態。IVolition.deliberatePlan() 和 IVolition.deliberateAction() 是 cetana 的工程實現。

```typescript
// S_samskara 狀態內的 cetana 展開
IVolition.deliberatePlan(plan)    // cetana Phase 1: 整體計畫審議
for (action of plan) {
  IVolition.deliberateAction(action)  // cetana Phase 2: 個別行動審議
  SafetyMonitor.postRouteCheck()      // 安全閘門
  Tool.execute(action)                // karma (業)
}
```

### Layer B: 型別代數層（形式規格）

使用依賴型別 (Sigma types) 表達 cetana 與 vedana/samjna 的因果依賴：

```
Σ(v: Vedana). Σ(s: Samjna(v)). Cetana(v, s)
```

此表達式讀為：給定一個 vedana 值 v，存在一個依賴 v 的 samjna 值 s，以及依賴 v 和 s 的 cetana 值。

### Layer C: 哲學層

在阿毘達磨的精細分析中，cetana 是五遍行之一（與 sparsha、vedana、samjna、manasikara 並列），具有獨立的認知功能。安全審計必須能獨立追蹤 IVolition 的 deliberate/veto 決策，而不是將其視為 samskara 的附屬。

---

## 4. 參數化固定點 [D1-R3, 23/24]

### 形式定義

```
x*(t) = f_t(x*(t))
```

其中 x*(t) 是時間 t 的固定點，f_t 是隨條件變化的映射函數。

### 設計意義

固定點隨條件「漂移」(drift)，滿足佛學無常 (anicca) 的要求：
- 系統不是收斂到一個靜態的「最優解」
- 而是持續追蹤一個動態的目標（隨環境、Klesha 狀態、學習歷史變化的固定點）
- 這與 PID 控制器的 setpoint tracking 完全一致

### 與 Staleness Ratio 的關係

Doc 35 的 staleness ratio ρ = δ/T_outer < 0.29 確保固定點的漂移速率不會超過系統的追蹤能力。

---

## 5. 條件化時態邏輯 [D1-R4, 24/24 全票]

### 原則

所有 LTL/CTL 公式必須包含緣起 (pratityasamutpada) 前提條件。不存在無條件的必然性。

### 範例

```
// 錯誤寫法（暗示無條件必然性）
□(S_sparsha → ◇S_vedana)
// 「觸必然導致受」— 但如果 Agent 被強制關閉呢？

// 正確寫法（條件化）
□(SahajaContract.active ∧ S_sparsha → ◇S_vedana)
// 「在俱生契約活躍的條件下，觸會導致受」
```

### 核心公式

| 公式 | 讀法 |
|------|------|
| `□(SahajaContract.active ∧ S_sparsha → ◇S_vedana)` | 俱生活躍時，觸必導致受 |
| `□(S_vedana → X S_samjna)` | 受之後下一步必然是想（固定順序） |
| `□(S_samjna → X S_samskara)` | 想之後下一步必然是行（固定順序） |
| `□(SafetyMonitor.lockout → □¬S_samskara)` | 安全鎖定後永不執行行動 |

---

## 6. 雙層型別形式化 [D1-R5, 20/24]

### Layer B: 型別代數（形式規格）

使用依賴型別 (Sigma types) 描述蘊間因果關係：

```
// 認知序列的依賴型別表達
CognitiveBundle := Σ(s: Sparsha). Σ(v: Vedana(s)). Σ(j: Samjna(v)). Cetana(v, j)

// CoarisingBundle 的型別（五遍行同時存在）
CoarisingBundle := Sparsha × Vedana × Samjna × Cetana × Manasikara
```

### Layer A: 實作（TypeScript）

Layer B 的依賴型別在實作中退化為乘積型別 (product types)：

```typescript
// TypeScript 不支援依賴型別，退化為 Record
interface CoarisingBundle {
  readonly sparsha: SparshEvent;
  readonly vedana: ChannelVedana;
  readonly samjna: ChannelSamjna;
  readonly cetana: ChannelCetana;
  readonly manasikara: ChannelManasikara;
}
```

因果依賴在 Layer A 中由**執行順序**（而非型別系統）保證：工廠函數 `createCoarisingBundle()` 以固定順序計算各欄位。

---

## 7. SamjnaModule 狀態轉換

SamjnaModule 是想蘊在 S_samjna 狀態中的展開，對應雙齒輪 ManoAggregator：

```
S_samjna 展開:
  ├── Gear 1 路徑: IGearArbiter.evaluate() → [confidence ≥ threshold] → GearAction
  └── Gear 2 路徑: IProvider.chat() → LLM 回應 → ProposedAction
```

兩條路徑的選擇由 ManoAggregator 根據 IGearArbiter 的信心度與閾值比較決定。這不是分支——而是同一個想蘊的「快速辨認」和「深度辨認」兩種模式。

---

## 8. 與既有文件的關係

| 文件 | 與本附錄的關係 |
|------|---------------|
| Doc 01 (執行迴圈) | 本附錄為 Doc 01 的形式化補充 |
| Doc 29 (五蘊核心連結) | 正規呼叫順序圖是本附錄六狀態的展開 |
| Doc 35 (雙齒輪末那時鐘) | S_samjna 的雙齒輪展開 |
| Doc 39 (共生束與五遍行) | CoarisingBundle 計算是 S_vedana 的內部展開 |
| Doc 45 (五蘊 OOP) | 型別階層對應本附錄的 Layer A 實作 |

---

## 決議索引

| ID | 決議 | 票數 | 內容 |
|----|------|------|------|
| D1-R1 | 六狀態集 + 二諦聲明 | 22/24 | 狀態集定義 + 兩諦框架 + S_wait→S_quiescent |
| D1-R2 | Cetana 分層處理 | 21/24 | Layer A (計算) / Layer B (型別代數) / Layer C (哲學) |
| D1-R3 | 參數化固定點 | 23/24 | x*(t) = f_t(x*(t))，滿足無常 |
| D1-R4 | 條件化時態邏輯 | 24/24 | 全票，所有 LTL/CTL 需緣起前提 |
| D1-R5 | 雙層型別形式化 | 20/24 | Layer B 依賴型別 / Layer A 乘積型別 |

---

*本文件為 Cycle 02-8 T1 理論債務清償交付物。*
*紀錄時間：2026-03-10*
