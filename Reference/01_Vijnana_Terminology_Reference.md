<!-- Layer: 2-Philosophy -->
<!-- Status: NEW -->
<!-- Cycle: 02-11, D4-R3 -->
<!-- Author: NAGARJUNA (#7) -->

# S-3. 識蘊術語參照 (Vijnana Terminology Reference)

本文件定義「vijnana」(識) 在 OpenStarry 三層文件架構中的精確用法，消除因同一術語在不同語境下含義差異造成的歧義。本文件為專案標準術語參照 (project-standard terminology reference)。

> **Two Truths Declaration**: The mapping of vijnana (consciousness/discernment) to engineering constructs is a *samvriti-satya* (conventional truth) approximation. The engineering construct captures the Guide plugin's governance/routing function but does not represent the full *paramartha-satya* (ultimate truth) scope of vijnana-skandha, which encompasses all eight consciousnesses (asta-vijnana) in the Yogacara framework. This mapping is valid within the OpenStarry engineering context and should not be generalized to Buddhist philosophical discourse.

---

## 1. 三層定義 (Three-Tier Distinction)

| 層級 | 語境 | "Vijnana" 含義 | 範圍 | 梵文精度 |
|---|---|---|---|---|
| **Level 1 (API)** | 原始碼、SDK 介面、插件 manifest | Guide 插件 = 身份/路由層 | IGuide, PluginHooks.guide, skandha: "vijnana" | 零梵文解釋 |
| **Level 2 (Engineering)** | 架構文件、設計討論 | 第六意識 = 認知處理 | IInferenceProvider, cognitive loop | 二諦聲明必備 |
| **Level 3 (Philosophy)** | 研究論文、佛學分析 | 八識全體 (asta-vijnana) | 完整唯識學框架 | 完整唯識術語 |

### 使用規則

1. **Level 1 文件**: 使用 "vijnana" 僅指 Guide/Identity 插件層。不出現梵文解釋。
2. **Level 2 文件**: 必須包含二諦聲明 (Two Truths Declaration)。須指明是八識中的哪一識。
3. **Level 3 文件**: 允許完整唯識術語。須與下方快速參照表保持一致。

---

## 2. 第六識與第七識：雙重判準 (Mano-vijnana vs Manas — Dual Criteria)

此為本專案最重要的術語釐清：Guide 插件映射至**第六意識 (mano-vijnana)**，而非**第七識 (manas)**。

| | 第六意識 (Mano-vijnana) | 第七識 (Manas / 末那識) |
|---|---|---|
| **本質** | 主動辨別、概念處理 | 恆常自我引用、「我執」(atma-graha) |
| **OpenStarry 映射** | Guide 插件 (IGuide, IPersistentGuide) | 未實作 (Phase 6) |
| **關鍵特性** | clearDirectives() 可清除（無常） | 不可清除、持續運作 |

### 判準 A：不連續性 (Discontinuity)

Mano-vijnana 承認停止與清除——`clearDirectives()` 清除所有指令後，系統進入無概念狀態 (nirvikalpana)。Manas 則恆常運作，即使在深度睡眠/禪定中也不完全停止（唯識學特指「恆審思量」）。

### 判準 B：自我引用性 (Self-referentiality)

Mano-vijnana 處理外部對象與任務（CognitiveDirective 是任務導向的）。Manas 恆常執著「自我」(atma-graha)——這是一種指向自身的遞迴結構，與 Guide 的任務導向指令根本不同。

> IPersistentGuide (Plan35 W3) = mano-vijnana 的跨 session 持久化，不是 manas。"Persistent" 指資料持久化（工程機制），非 manas 的哲學恆常性。

---

## 3. 快速參照表 (Terminology Quick Reference)

| 術語 | Level 1 (API) | Level 2 (Engineering) | Level 3 (Philosophy) |
|---|---|---|---|
| vijnana | Guide 的 skandha 標籤 | 第六意識 | 八識全體 |
| mano-vijnana | IGuide, IPersistentGuide | 認知辨別 | 八識中第六 |
| manas | 未實作 | 未實作 | 第七識（我執） |
| alaya-vijnana | 未實作 | Management Zone (部分) | 第八識（藏識） |
| citta | API 不使用 | 一般「心」之討論 | 部分語境中等同 vijnana |
| pravrtti-vijnana | 不使用 | 「轉識」(前七識) | 與 alaya (第八) 對比 |
| panca-vijnana | Sensory 插件 (Listener, UI) | 五感輸入 | 前五識 |

---

## 4. smrti 映射修正

檔案持久化 (file persistence) 是一項工程機制，用於實現跨 session 的 vikalpana（概念建構）檢索。它不是佛學心所 smrti（念、記憶）的直接對應。

Guide-persistent 的功能對應表：

| Mano-vijnana 功能 | Guide-Persistent 對應 | 映射類型 |
|---|---|---|
| Vikalpana (概念建構) | CognitiveDirective content | 直接 |
| Manaskara (注意導向) | Directive priority ordering | 直接 |
| Nirvikalpana (無概念狀態) | clearDirectives() 後狀態 | 直接 |
| Prajna (辨別智) | 指令條件下的 LLM 回應 | 間接 |
| 跨 session 連續性 | 檔案持久化 | 工程機制，非 smrti |

---

## 5. 參考文件

- Doc 14 (Five Aggregates Philosophy) — 五蘊哲學基礎
- Doc 15 (Skandha Protocol Bridge) — 蘊與協議映射
- Doc 31 (Eight Consciousnesses Runtime) — 八識運行時
- Doc 40 (Tenet 6 Architecture Mapping) — Tenet 6 架構映射
- Doc 52 (Alaya Partial Mapping) — 阿賴耶部分映射

---

*Cycle 02-11 新增 | D4-R3 決議 | NAGARJUNA (#7) 撰寫*
