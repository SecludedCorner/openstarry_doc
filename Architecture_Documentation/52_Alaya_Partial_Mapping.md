<!-- Layer: 2-Philosophy -->
<!-- Status: NEW -->
<!-- Cycle: 02-11, D4-R2 -->
<!-- Author: NAGARJUNA (#7) -->

# 56. 阿賴耶識部分映射分析 (Alaya-vijnana Partial Mapping Analysis)

本文件系統性地將 AgentCore (v0.34.0-alpha) 的現有實作對照唯識學 (Yogacara) 阿賴耶識 (Alaya-vijnana, 第八識) 的八項核心功能，明確界定映射邊界。

> **Two Truths Declaration**: The mapping of Alaya-vijnana to AgentCore is a *samvriti-satya* (conventional truth) approximation for engineering purposes. The engineering construct captures structural prerequisites (container, neutrality, continuity) that coincide with Alaya properties, but does not represent the full *paramartha-satya* (ultimate truth) meaning, which includes vasana (perfuming), vipaka (maturation), and the manas-alaya interdependence that constitutes Alaya's essential karmic-causal character. From the ultimate truth perspective, calling AgentCore "partially Alaya" is as misleading as calling a parking garage "partially a home" because both have walls and a roof. This mapping is valid within the OpenStarry engineering context and should not be generalized to Buddhist philosophical discourse.

---

## 1. 阿賴耶識八功能 (Eight Functions of Alaya-vijnana)

唯識學中，阿賴耶識（第八識、藏識）執行以下八項核心功能：

| # | 功能 | 梵文 / 中文 | 說明 |
|---|---|---|---|
| 1 | 能藏 (Container) | Neng-cang | 儲存一切經驗的種子 (bija) |
| 2 | 所藏 (Stored-by) | Suo-cang | 被第七識 (manas) 執取為「我」的對象 |
| 3 | 執藏 (Attached-to) | Zhi-cang | 被 manas 恆常執著為自我 |
| 4 | 薰習 (Perfuming) | Vasana / Xun-xi | 新經驗創造新種子 |
| 5 | 異熟 (Maturation) | Vipaka / Yi-shu | 種子因緣成熟為果報 |
| 6 | 無覆無記 (Unobscured-neutral) | Wu-fu-wu-ji | 道德中性，非善非惡 |
| 7 | 恆轉 (Continuous flow) | Heng-zhuan | 一期生命中永不停歇 |
| 8 | 攝受 (Appropriation) | She-shou | 維持個體身份的連續性 |

---

## 2. 映射評估 (v0.34.0-alpha)

| # | 功能 | AgentCore 對應 | 覆蓋 | 證據 |
|---|---|---|---|---|
| 1 | **能藏** | 狀態持久層、PluginHooks store、plugin registry | **PRESENT** | [Code: packages/core/src/agent-core.ts#PluginHooks] |
| 2 | **所藏** | 未實作 — 需 manas 層 | **ABSENT** | Phase 6; AC-7 |
| 3 | **執藏** | 未實作 — 需 manas 層 | **ABSENT** | Phase 6; AC-7 |
| 4 | **薰習** | 無 vasana 引擎，無經驗→種子機制 | **ABSENT** | Deferred: D-30-5 (VasanaEngine) |
| 5 | **異熟** | 無種子成熟管線 | **ABSENT** | Phase 6+ |
| 6 | **無覆無記** | Core 道德/策略中性；零 policy 常數 (MR-6, Plan33) | **PRESENT** | [Source: compliance_report_v0.33.md] |
| 7 | **恆轉** | ExecutionLoop 持續運行；daemon 管理生命週期 (Tenet #1) | **PRESENT** | [Code: packages/core/src/execution-loop.ts] |
| 8 | **攝受** | IAgentConfig identity 欄位；agent.json 身份持久化 | **PARTIAL** | 身份持久但缺乏深層自我引用 |

---

## 3. 覆蓋率摘要

| 類別 | 功能 | 數量 | 百分比 |
|---|---|---|---|
| **PRESENT** (結構性) | 能藏、無覆無記、恆轉 | 3/8 | **37.5%** |
| **PARTIAL** | 攝受 | 1/8 | — |
| **ABSENT** (動態性) | 薰習、異熟、所藏、執藏 | 4/8 | 50% |

### 映射邊界聲明

> AgentCore 在 v0.34.0-alpha 實作了阿賴耶識的**結構性前提** (structural prerequisites) —— 容器功能、道德中性、持續運行 —— 但缺乏**動態性功能** (dynamic functions) —— 薰習、異熟 —— 以及**關係性功能** (relational functions) —— manas-alaya 的互依。
>
> 正確表述為：**「37.5% 結構性前提覆蓋，0% 動態阿賴耶功能覆蓋。」**
>
> 「37.5% 結構性」不應被誤讀為「37.5% 的阿賴耶已實作」。Present 的三項功能是許多系統共有的架構特性（中性容器 + 持續運行），它們是阿賴耶的必要條件但非充分條件。沒有薰習 (vasana) 和異熟 (vipaka)，就沒有業因果律 (karmic causality)；沒有業因果律，就不存在唯識學意義上的阿賴耶識。

---

## 4. 分散式阿賴耶願景 (Phase 6+)

研究路線圖標識 "Alaya distributed" (AC-7) 為 Cycle 03+ / Phase 6 項目。在完整願景中，阿賴耶識映射至 Management Zone 的分散式協調層，而非單一 AgentCore 實例。

當前狀態：Management Zone 提供編排與容器管理，但未實作阿賴耶的動態功能。分散式種子庫 (distributed seed-store) 尚不存在。

---

## 5. 術語參照

詳見 [Reference/01 Vijnana Terminology Reference](../Reference/01_Vijnana_Terminology_Reference.md) 之 Level 3 定義。

| 術語 | 本文用法 |
|---|---|
| Alaya-vijnana | 第八識，藏識，唯識學完整意義 |
| Manas | 第七識，末那識，恆審思量、我執 |
| Vasana | 薰習，經驗印記 |
| Vipaka | 異熟，種子成熟 |
| Bija | 種子，經驗的潛在形式 |

---

## 6. 參考文件

- Doc 31 (Eight Consciousnesses Runtime) — 八識運行時架構
- Doc 40 (Tenet 6 Architecture Mapping) — 八識俱轉架構映射
- Calibration_Reports/04 (Lyapunov Phase 0 Report) — 系統穩定性分析
- [Source: 玄奘《成唯識論》 — 阿賴耶識八功能]

---

*Cycle 02-11 新增 | D4-R2 決議 | NAGARJUNA (#7) 撰寫*
