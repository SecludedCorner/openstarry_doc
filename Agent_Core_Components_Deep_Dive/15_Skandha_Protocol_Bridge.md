<!-- Layer: 2-Philosophy -->
<!-- Status: NEW -->
<!-- Cycle: 02-11, D4-R2 -->
<!-- Authors: NAGARJUNA (#7) + SCRIBE (#2) -->

# 15. 蘊與協議橋接 (Skandha Protocol Bridge)

本文件橋接 Doc 14（五蘊哲學映射）與 Doc 16（OpenStarry 標準協議），定義五蘊 (skandha) 宣告在插件協議層的具體語義。當插件在 manifest 中宣告 `skandha` 屬性時，該宣告如何映射至 PluginHooks 的實際欄位、runtime 路由行為、以及 sigma 約束檢查。

> **Two Truths Declaration**: The mapping of Five Aggregates (panca-skandha) to PluginHooks fields is a *samvriti-satya* (conventional truth) approximation for engineering purposes. The engineering construct captures the functional differentiation of cognitive faculties but does not represent the full *paramartha-satya* (ultimate truth) meaning, which includes the emptiness (sunyata) of all aggregates and their interdependent co-arising. This mapping is valid within the OpenStarry engineering context and should not be generalized to Buddhist philosophical discourse.

---

## 1. 蘊標籤與 PluginHooks 欄位映射

每個 skandha 標籤對應 PluginHooks 中的一組特定欄位。插件透過 manifest 的 `skandha` 屬性宣告其歸屬，系統透過 PluginHooks 註冊確認實際能力。

| 蘊 (Skandha) | 中文 | PluginHooks 欄位 | 協議語義 |
|---|---|---|---|
| **rupa** (色) | 色蘊 | `ui`, `listeners` | 感官通道：輸入/輸出介面註冊。UI 為外顯形相，Listener 為感官根門 (indriya)。 |
| **vedana** (受) | 受蘊 | `vedana` (VedanaSensor/Registry) | 回饋信號發射：苦/樂/捨三受評估。Sensor 產生原始感受，Registry 聚合多維信號。 |
| **samjna** (想) | 想蘊 | `provider`, `arbiter`, `contextManager` | 認知推理：推理請求/回應、齒輪仲裁、上下文管理。Provider 為深層認知引擎。 |
| **samskara** (行) | 行蘊 | `tools`, `volition` | 行動執行：工具調用、意志審議 (IVolition)。Tool 為行動能力，Volition 為決策閘門。 |
| **vijnana** (識) | 識蘊 | `guide`, `auditor`, `monitors`, `klesha` | 身份治理：指令注入、閾值審計、品質監控、煩惱增益。Guide 定義行為框架。 |

---

## 2. 蘊宣告方向性：Overclaim vs Underclaim 不對稱

Doc 49 (Skandha Soft Constraints) 定義了 sigma 約束框架。在協議橋接語境下，兩類不一致具有不同嚴重性：

### 2.1 Undeclared Hook (σ-1 ~ σ-5)

**方向**: 宣告 → hook（宣告了蘊歸屬但無對應 hook）

**語義**: 插件聲稱擁有某蘊的能力，但未提供實作。這是**誇大宣告** (overclaim)——對系統的風險較低，因為缺少 hook 不會影響其他組件。預設嚴重度 INFO。

| 約束 | 條件 |
|---|---|
| σ-1 | `vedana` 宣告 → 需有 vedana hook |
| σ-2 | `samjna` 宣告 → 需有 provider/arbiter hook |
| σ-3 | `samskara` 宣告 → 需有 tool hook |
| σ-4 | `rupa` 宣告 → 需有 ui/listener hook |
| σ-5 | `vijnana` 宣告 → 需有 guide/auditor/monitor hook |

### 2.2 Overclaimed Skandha (σ-6 ~ σ-12)

**方向**: hook → 宣告（有 hook 但未宣告對應蘊歸屬）

**語義**: 插件提供了能力但未在 manifest 中宣告。這是**低報宣告** (underclaim)——對系統風險較高，因為蘊歸屬資訊用於文件生成、分類排序、以及 CoarisingBundle 組裝。未宣告的能力可能導致組裝遺漏。預設嚴重度 WARN。

| 約束 | 條件 |
|---|---|
| σ-6 | tool hook → 需宣告 `samskara` |
| σ-7 | ui hook → 需宣告 `rupa` |
| σ-8 | listener hook → 需宣告 `rupa` |
| σ-9 | provider hook → 需宣告 `samjna` |
| σ-9b | (保留編號，PROC-SPEC-2 追溯) |
| σ-10 | auditor hook → 需宣告 `vijnana` |
| σ-11 | monitor hook → 需宣告 `vijnana` |
| σ-12 | guide hook → 需宣告 `vijnana` |

### 2.3 結構性約束 (σ-13 ~ σ-17)

| 約束 | 條件 | 嚴重度 |
|---|---|---|
| σ-13 | 空 skandha 宣告 | WARN |
| σ-14 | 無任何 hook 的 plugin | INFO |
| σ-15 | skandha 未宣告但有 hook | WARN |
| σ-16 | klesha hook 未宣告 vijnana | WARN |
| σ-17 | volition hook 未宣告 samskara+vijnana | WARN |

---

## 3. 多蘊插件與共生束 (Multi-Skandha & CoarisingBundle)

### 3.1 多蘊宣告

插件可宣告多個蘊歸屬：`skandha: ['samskara', 'vijnana']`。此時 sigma 檢查針對每個宣告的蘊獨立執行。

### 3.2 CoarisingBundle 映射

Doc 39 (CoarisingBundle & Five Universals) 定義了五遍行心所的共生束。在協議層，CoarisingBundle 確保特定 hook 組合在同一認知週期內同步觸發。蘊宣告的正確性直接影響 CoarisingBundle 的組裝邏輯：

- 未宣告 `vedana` 的 Sensor 不會被納入受蘊共生束
- 未宣告 `samjna` 的 Provider 不會參與想蘊協調

### 3.3 三級關鍵性模型

蘊宣告與 Tenet #7 三級關鍵性模型 (Required / Optional-degraded / Optional-no-effect) 交互：

| 關鍵性 | 蘊缺席行為 | 範例 |
|---|---|---|
| Required | `throw Error()` | contextManager (想蘊) |
| Optional-degraded | 中性值回退 | vedana sensor (受蘊) |
| Optional-no-effect | 功能不可用 | guide-persistent (識蘊) |

---

## 4. 執行等級與協議行為

Doc 49 定義了四級執行策略 (L1-L4)。在協議層的對應行為：

| 等級 | 協議行為 |
|---|---|
| L1 (Silent) | sigma 不一致僅記錄，不影響插件載入或路由 |
| L2 (Warn+Info, 預設) | overclaim 產生 `logger.warn()`，undeclared 產生 `logger.info()` |
| L3 (EventBus) | 額外發射 `skandha:mismatch` 事件，供外部監控消費 |
| L4 (Reject) | 由獨立策略模組決定拒絕載入（永不在 PluginLoader 中） |

---

## 5. 參考文件

| 文件 | 關係 |
|---|---|
| Doc 14 (Five Aggregates Philosophy) | 哲學基礎：五蘊定義與分類原則 |
| Doc 16 (Standard Protocol) | 協議規格：PluginHooks 完整欄位定義 |
| Doc 33 (Multivalue Skandha Design) | 多值蘊設計：多蘊宣告語法 |
| Doc 39 (CoarisingBundle & Five Universals) | 共生束：同步觸發邏輯 |
| Doc 45 (Five Skandha OOP Architecture) | OOP 架構：介面與蘊的對應 |
| Doc 49 (Skandha Soft Constraints) | sigma 約束：17 項約束完整規格 |

---

*Cycle 02-11 新增 | D4-R2 決議 | NAGARJUNA (#7) + SCRIBE (#2) 共同撰寫*
