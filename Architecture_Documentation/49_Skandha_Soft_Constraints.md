<!-- Layer: 2-Philosophy -->

# 49. 蘊歸屬軟約束框架 (Skandha Soft Constraints — σ-Constraint Framework)

`[Cycle 02-8 新增: T3 理論債務清償]`

> Cycle 02-8 D3a-R1 ~ D3c-R1 共 3 項決議。本文件形式化 Plugin 蘊歸屬 (skandha) 的軟約束系統——`checkSkandhaCorrespondence()` 的完整規格、三種執行策略、約束傳播規則。
>
> **核心學者**: LINNAEUS (#13), DARWIN (#6), SUSSMAN (#22), KERNEL (#10), GUARDIAN (#11)
> **依賴文件**: #33 多值蘊設計, #45 五蘊 OOP 架構, #04 插件基礎設施

---

## 1. 概述

Plugin 的 `manifest.skandha` 宣告其蘊歸屬，但實際的 PluginHooks 註冊可能與宣告不一致。例如，一個宣告 `skandha: 'vedana'` 的 plugin 可能註冊了 `tools` hook（行蘊），或者一個宣告 `skandha: ['samjna', 'vijnana']` 的 plugin 可能沒有註冊任何識蘊相關的 hook。

σ-constraint 框架定義了如何偵測、報告、以及處理這些不一致。

### 設計原則

1. **軟約束**: 蘊歸屬不一致**不**阻止 plugin 載入（預設行為）
2. **可觀測**: 所有不一致都會被記錄
3. **可升級**: 部署者可選擇更嚴格的執行等級

---

## 2. σ-Constraint 定義

### 2.1 約束類型

一個 σ-constraint 是一個三元組：

```
σ = (condition, severity, message)
```

其中 `condition` 是布林表達式，`severity` 是違反時的嚴重程度，`message` 是人類可讀的描述。

### 2.2 17 項已識別約束

| # | 約束 | 條件 | 預設嚴重度 |
|---|------|------|-----------|
| σ-1 | vedana 宣告需有 vedana hook | manifest.skandha 含 'vedana' → hooks 含 vedana 相關 | INFO |
| σ-2 | samjna 宣告需有 provider/arbiter hook | manifest.skandha 含 'samjna' → hooks 含 provider/arbiter | INFO |
| σ-3 | samskara 宣告需有 tool hook | manifest.skandha 含 'samskara' → hooks 含 tools | INFO |
| σ-4 | rupa 宣告需有 ui/listener hook | manifest.skandha 含 'rupa' → hooks 含 ui/listeners | INFO |
| σ-5 | vijnana 宣告需有 guide/auditor/monitor hook | manifest.skandha 含 'vijnana' → hooks 含相關 | INFO |
| σ-6 | tool hook 未宣告 samskara | hooks 含 tools → manifest.skandha 含 'samskara' | WARN |
| σ-7 | ui hook 未宣告 rupa | hooks 含 ui → manifest.skandha 含 'rupa' | WARN |
| σ-8 | listener hook 未宣告 rupa | hooks 含 listeners → manifest.skandha 含 'rupa' | WARN |
| σ-9 | provider hook 未宣告 samjna | hooks 含 provider → manifest.skandha 含 'samjna' | WARN |
| σ-10 | auditor hook 未宣告 vijnana | hooks 含 auditor → manifest.skandha 含 'vijnana' | WARN |
| σ-11 | monitor hook 未宣告 vijnana | hooks 含 monitors → manifest.skandha 含 'vijnana' | WARN |
| σ-12 | guide hook 未宣告 vijnana | hooks 含 guide → manifest.skandha 含 'vijnana' | WARN |
| σ-13 | 空 skandha 宣告 | manifest.skandha 為空陣列 | WARN |
| σ-14 | 無任何 hook 的 plugin | hooks 全部為空/undefined | INFO |
| σ-15 | skandha 未宣告但有 hook | 至少一個 hook 非空但 skandha 缺失 | WARN |
| σ-16 | klesha hook 未宣告 vijnana | hooks 含 klesha → manifest.skandha 含 'vijnana' | WARN |
| σ-17 | volition hook 未宣告 samskara+vijnana | hooks 含 volition → manifest.skandha 含 'samskara' 或 'vijnana' | WARN |

### 2.3 約束分類

| 類型 | 方向 | 約束 | 含義 |
|------|------|------|------|
| **Undeclared Hook** (未宣告 hook) | 宣告 → hook | σ-1 ~ σ-5 | 宣告了蘊歸屬但沒有對應的 hook 註冊 |
| **Overclaimed Skandha** (過度宣告) | hook → 宣告 | σ-6 ~ σ-12 | 有 hook 但沒有宣告對應的蘊歸屬 |
| **Structural** (結構性) | — | σ-13 ~ σ-17 | 結構完整性問題 |

---

## 3. 執行策略 [D3a-R1, 23/24]

### 3.1 三級策略

| 級別 | 名稱 | 行為 | 適用場景 |
|------|------|------|---------|
| **L1** | Silent | 僅內部記錄，不輸出 | 生產環境（關注效能） |
| **L2** | Warn+Info (預設) | Overclaimed → `logger.warn()` / Undeclared → `logger.info()` | 開發/測試環境 |
| **L3** | EventBus | L2 + `skandha:mismatch` 事件發射 | 需要 programmatic 反應 |
| **L4** | Reject | L3 + 拒絕載入 | 嚴格環境（由獨立策略模組決定） |

### 3.2 預設行為

```
預設級別: L2 (Warn+Info)
```

- **Undeclared hook** (σ-1~σ-5): `logger.info('Plugin ${name} declares ${skandha} but no matching hook registered')`
- **Overclaimed skandha** (σ-6~σ-12): `logger.warn('Plugin ${name} registers ${hookType} but does not declare ${expectedSkandha}')`

---

## 4. 配置與控制 [D3b-R1, 24/24 全票]

### 雙層配置

1. **`checkSkandhaCorrespondence()` 永遠執行**: 不提供開關——檢查本身是零成本的
2. **輸出由 logger 配置控制**: 如果 logger level 設為 ERROR，則 WARN/INFO 自然被抑制

```typescript
// 無專用 config flag
// 由 logger 級別控制輸出
// logger.level = 'error'  → L2 的 warn/info 被抑制
// logger.level = 'debug'  → 全部可見
```

### 設計理由

- 檢查的成本是 O(|hooks|) ≈ O(1)（plugin 的 hook 數量有限），不值得提供開關
- 避免「我關掉了約束檢查所以我的 plugin 是正確的」的虛假安全感
- Logger 配置已經提供了足夠的粒度控制

---

## 5. L4 策略隔離 [D3c-R1, 22/24]

### 核心原則

L3（EventBus 發射）在 `PluginLoader.load()` 中執行。L4（拒絕載入）**僅**在獨立的策略模組 (policy module) 中執行，**永不**在 PluginLoader 中。

```
PluginLoader.load()
  ├── checkSkandhaCorrespondence()  → L2 log + L3 EventBus 發射
  └── [不做 L4 拒絕]

PolicyGateway (獨立模組)
  ├── 訂閱 'skandha:mismatch' 事件
  └── 根據策略決定是否 unload plugin → L4
```

### 設計理由

| 理由 | 說明 |
|------|------|
| Tenet #7 | PluginLoader 是機制 (mechanism)，策略決策不屬於 Core |
| 可組合性 | PolicyGateway 本身是 plugin，可替換、可配置 |
| 漸進式嚴格 | 從 L2 → L3 → L4 的升級路徑清晰，不需要修改 Core |

---

## 6. 約束傳播規則

### 6.1 組合 Plugin

組合 plugin (composite plugin) 的 skandha 是其子 plugin 的聯集：

```
composite.skandha = union(child₁.skandha, child₂.skandha, ...)
```

約束檢查在組合層面和子 plugin 層面**分別**執行。

### 6.2 跨蘊 Plugin

跨蘊 plugin（如 ILoopQualityMonitor: `skandha: ['vedana', 'samjna', 'vijnana']`）的約束是各蘊約束的合取：

```
σ-跨蘊 = σ-vedana ∧ σ-samjna ∧ σ-vijnana
```

每個宣告的蘊都必須有對應的 hook 或功能。

---

## 7. 與蘊歸屬永久原則的關係

σ-constraint 框架是 Cycle 02-6 D1-R6 蘊歸屬 5 項永久原則的執行層：

| 原則 | σ-constraint 實現 |
|------|------------------|
| 功能分析為唯一依據 | hook → skandha 方向的約束 (σ-6~σ-12) |
| 心所不決定蘊歸屬 | σ-constraint 只檢查 hook-skandha 對應，不涉及心所 |
| 跨蘊允許 | 多值 skandha 的合取約束 |
| 既有決策有效 | 既有 plugin 的 skandha 宣告不受新約束影響 |

---

## 決議索引

| ID | 決議 | 票數 | 內容 |
|----|------|------|------|
| D3a-R1 | 預設執行級別 L2 | 23/24 | Warn+Info 作為預設 |
| D3b-R1 | 雙層配置 | 24/24 | 全票，永遠執行+logger 控制 |
| D3c-R1 | L4 在獨立策略模組 | 22/24 | L3 在 PluginLoader，L4 在 PolicyGateway |

---

*本文件為 Cycle 02-8 T3 理論債務清償交付物。*
*紀錄時間：2026-03-10*
