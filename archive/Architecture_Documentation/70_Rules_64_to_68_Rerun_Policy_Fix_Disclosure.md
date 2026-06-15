# 70. Rules #64-#68: Re-Run Policy, Fix Disclosure, Two-Path Config Verification

`[Cycle 03-9 新增]` `[Master Ratified 2026-04-15]`

> **來源**: Cycle 03-9 R3 Debate D1 (Re-Run Policy), D3 (NC4 Ruling), D6 (Config Propagation Forward Prevention)
> **核心學者**: KERNEL (#10), BABBAGE (#9), GUARDIAN (#11), SUSSMAN (#22)
> **規則範圍**: Rules #64, #65, #66, #67, #68

---

## 1. 概述

Rules #64-#68 是 Cycle 03-9 新增的 5 條規則，針對三個關鍵治理缺口：

| 缺口 | 新規則 | 功能 |
|------|--------|------|
| 重跑缺乏正式政策 | #64, #65, #66 | 定義允許條件、fix-between-reruns、強制揭露 |
| 造假判定基準不完整 | #67 | 建立 fix disclosure 作為反造假正面信號 |
| Config 傳遞路徑驗證不全 | #68 | 強制雙路徑驗證（Path A + Path B） |

觸發來源：W2-R9 的 3 次重跑揭露了系統性治理缺口，加上 Fix 12d 發現 plugin-resolver 自 runner 建立以來就未傳遞 config 給 factory（24 版本空窗）。

---

## 2. Rule #64 — Re-Run Conditions

### 規則內容

> 重跑條件：基礎設施失敗（infrastructure failure，如 API 限速、沙箱問題、網路中斷）或系統性失敗（systematic failure，如運行中發現 code bug）。兩者皆須在重跑前完成根因識別並記錄。

### 設計動機

W2-R9 有 3 次重跑，但當時無正式政策區分「可重跑」vs「不可重跑」的失敗類型。若無規則，重跑可能被濫用作為「試到對為止」的手段，破壞觀測獨立性（Tenet #8 control theory）。

### 允許條件的精確定義

| 條件類型 | 範例 | 是否允許重跑 |
|---------|------|:-----------:|
| Infrastructure failure | API 429 rate limit | ✅ 允許 |
| Infrastructure failure | 沙箱路徑錯誤 | ✅ 允許 |
| Infrastructure failure | 網路中斷 | ✅ 允許 |
| Systematic failure | Plugin resolution bug (Fix 12b) | ✅ 允許（附 fix） |
| Systematic failure | StateTracker persistence bug (Fix 12c) | ✅ 允許（附 fix） |
| Systematic failure | Config propagation bug (Fix 12d) | ✅ 允許（附 fix） |
| 非結構性失敗 | sigma 不如預期 | ❌ 不允許 |
| 非結構性失敗 | Agreement rate 不如預期 | ❌ 不允許 |
| 非結構性失敗 | 希望數據更漂亮 | ❌ 不允許 |

### 根因識別要求

重跑前必須：
1. 記錄失敗模式（logs, error stack, partial data）
2. 識別根因（code trace 或環境診斷）
3. 分類為 infrastructure OR systematic
4. 若 systematic：識別具體 fix 需求

### 最大重跑次數

**硬上限：4 次總嘗試**（1 次初次 + 3 次重跑）。若 4 次皆失敗，升級 Master。

### 合規依據

- **Tenet #8**: 觀測程序必須定義明確
- **MR-1**: 合規標準，非任意判斷

---

## 3. Rule #65 — Fix-Between-Reruns Rule

### 規則內容

> Fix-between-reruns 允許條件：(a) 根因已記錄；(b) Fix 符合 Rule #62 分類（Tier 1/2/3）；(c) Fix scope 限於已識別問題。**不允許**：(a) 推測性變更；(b) Scope expansion；(c) 改動 locked parameters。

### 設計動機

W2-R9 的 3 次 fix（12b/12c/12d）都是在重跑之間做的。這是否合規？R3 debate D1 確認：只要每個 fix 都有明確根因 + Rule #62 分類 + scope 受限，就合規。

### Rule #62 分類強制要求

每個 fix-between-reruns 必須標示：

| Tier | 類型 | W2-R9 範例 |
|------|------|-----------|
| Tier 1 | Bug-fix-ratification | Fix 12b, 12c, 12d 全為 Tier 1 |
| Tier 2 | Design-change | （R9 無此例） |
| Tier 3 | Scope-expansion | （R9 無此例） |

### 禁止的 Fix 類型

不論分類，以下**絕對禁止**：
- 改動 locked parameters（DELTA_SCALING_FACTOR, MIN_N, UP/DOWN 閾值等）
- 推測性變更（未經根因識別的嘗試性修改）
- Scope expansion（原本計畫外的功能）

### 合規依據

- **Rule #62**: Spec amendment 三級分類
- **MR-10**: 造假/錯誤必須補回
- **MR-12**: 回補優先原則

---

## 4. Rule #66 — Re-Run Disclosure Mandatory

### 規則內容

> 所有重跑必須在測試報告中揭露。揭露內容包括：失敗描述、根因、所應用的 fix（如有）、Rule #62 分類。**省略 = 造假（Rule #58 違反）**。

### 設計動機

若允許重跑但不強制揭露，系統無法區分「一次成功的試驗」vs「失敗 3 次後的成功」。後者的統計性質可能大相逕庭（survivorship bias）。

### 揭露要求（最小集）

| 欄位 | 內容 | 強制性 |
|------|------|:------:|
| Run # | 第幾次嘗試（1=初次，2-4=重跑） | MUST |
| Failure description | 觀察到的失敗模式 | MUST |
| Root cause | 根因（code trace 或環境診斷） | MUST |
| Fix applied | 如有 code fix，記錄變更 | 條件 |
| Rule #62 classification | Tier 分類 | 若有 fix 則 MUST |
| Data fate | 本次 run 的數據是 DISCARD / PARTIAL / OFFICIAL | MUST |

### 省略 = 造假

Rule #58 的 H0=fabrication 框架適用：若測試報告未揭露重跑，這等於對「該 run 是首次即成功」的事實作出不實陳述。H0 不可被 REJECTED，fabrication 成立。

### 配合 Rule #67

揭露重跑本身（Rule #66）與揭露 post-delivery fixes（Rule #67）同屬「透明化 = 反造假」原則。

### 合規依據

- **Rule #58**: H0 = fabrication，omission = 不實陳述
- **MR-11**: 十大宣言是核心價值，透明度是其表現

---

## 5. Rule #67 — Fix Disclosure as Positive Signal

### 規則內容

> 在 post-delivery 透明揭露的 fixes、附根因記錄、並符合 Rule #62 分類者，是「反造假」的**正面證據**（evidence AGAINST fabrication），加強 H0=fabrication 的 REJECTION。

### 設計動機

歷史上，後交付 fixes 常被視為「交付品質不足」的負面信號。但 Plan44 的 4 個 fixes（12a-12d）全數通過 L1-L4 驗證，每個都有清晰根因與分類。R3 D3-Q16 決策：這些 fix 的**存在**反而是 H0 REJECTION 的強證據。

### 判定邏輯

| 條件 | 是否為正面信號 |
|------|:-------------:|
| Fix 已揭露（Rule #66） | 必要條件 |
| 根因已記錄 | 必要條件 |
| Rule #62 分類正確 | 必要條件 |
| 每個 fix 獨立通過 L1-L4 | 必要條件 |
| 符合以上 → **正面信號** | ✅ 是 |
| 缺任一條件 → 不構成正面信號 | ❌ 否 |

### 反面範例（NOT 正面信號）

- Fix 存在但未揭露 → Rule #66 違反 = 造假
- Fix 揭露但無根因 → 推測性變更 = 無法判定
- Fix 根因錯分類 → Tier 誤判，可能掩蓋更深問題

### Plan44 的 4 fixes 為何是正面信號

| Fix | 揭露 | 根因 | Tier | L1-L4 |
|-----|:----:|:----:|:----:|:-----:|
| 12a | ✅ delivery_report §12a | ✅ 開發過程文件未同步 | N/A (doc) | PASS |
| 12b | ✅ delivery_report §12b | ✅ pnpm strict mode | Tier 1 | PASS |
| 12c | ✅ delivery_report §12c | ✅ 每 cycle 重建 core | Tier 1 | PASS |
| 12d | ✅ delivery_report §12d | ✅ 24 版本空窗 | Tier 1 | PASS |

所有條件滿足 → 4 fixes 集體構成 H0 REJECTION 的強證據。

### 合規依據

- **Rule #58**: 三層清白證明
- **Rule #62**: Spec amendment 分類
- **MR-10**: 錯誤必須補回；補回是德行，非瑕疵
- **MR-11**: 十大宣言是一種抉擇，選擇透明 = 選擇對齊

---

## 6. Rule #68 — Two-Path Config Verification

### 規則內容

> 所有 plugin config 變更必須測試**雙路徑**：Path A（module factory `factory(ref.config)`）及 Path B（`IPluginContext.config`）。忽略任一路徑 = 配置覆蓋不完整。

### 設計動機

GUARDIAN 的追溯調查揭露：v0.14.0-beta 至 v0.43.0-alpha 的 24 個版本中，plugin-resolver 的 module factory 從未接收 config（Path A 斷裂）。所幸無 plugin 使用此路徑，Path B（IPluginContext.config）一直正常。

但此 24 版本空窗表明：**單路徑驗證不足**。任何新 plugin 若意外依賴 Path A，bug 可能潛伏數年。

### 兩條路徑定義

#### Path A: Module-Level Factory Config

```
agent.json → PluginRef.config → resolvePlugin(ref) → factory(ref.config) → IPlugin
```

- `plugin-resolver.ts` 調用 module 的 `factory()` 函數建構 `IPlugin`
- Fix 12d 前：`factory()` 無參數
- Fix 12d 後：`factory(ref.config)`

#### Path B: IPluginContext.config

```
agent.json → IAgentConfig.plugins[].config → agent-core.ts getPluginContext(ref?.config) → IPluginContext.config
```

- `agent-core.ts` 獨立查詢 `PluginRef` 並傳遞給 `getPluginContext()`
- 提供給 `IPlugin.factory(ctx)` 內部使用
- **此路徑自始至終正常**

### 驗證要求

| 測試類型 | Path A | Path B |
|---------|:------:|:------:|
| 單元測試 | 必要 | 必要 |
| 工廠級整合測試（Rule #63） | 必要 | 必要 |
| E2E runner 測試 | 至少其中一個 | 至少其中一個 |

### ENG-FAB 更新

**F-7（v1.4 新增）**：整合測試涵蓋 plugin config propagation through runner's plugin-resolver（Path A + Path B）。

### Plan45 W0-3 實作

Plan45 W0 Wave 3 交付：`apps/runner/__tests__/integration/plugin-config-propagation.test.ts`，最小驗證範例：

```typescript
test("Path A: factory receives ref.config via plugin-resolver", async () => {
  const mockConfig = { testFlag: true };
  const agentConfig = { plugins: [{ name: "test-plugin", config: mockConfig }] };
  const resolved = await resolvePlugins(agentConfig, false, projectRoot);
  expect(resolved[0].factoryConfigReceived).toEqual(mockConfig);
});

test("Path B: IPluginContext.config populated from PluginRef.config", async () => {
  const mockConfig = { testFlag: true };
  const agentConfig = { plugins: [{ name: "test-plugin", config: mockConfig }] };
  const core = await buildCore(agentConfig);
  expect(core.pluginContext.config).toEqual(mockConfig);
});
```

兩 tests 必須同時通過。

### 合規依據

- **Rule #58**: L1/L2/L3/L4 完整性（L4 Config Activates 之擴展）
- **Rule #63**: Config→Factory→Runtime behavior 驗證
- **MR-7**: Code 完成才算 COMPLIANT（單路徑不算完成）
- **MR-11**: 十大宣言是核心價值，完整性是其體現

---

## 7. 規則之間的關係

```
┌─────────────────────────────────────────────────────────┐
│                  Cycle 03-9 新規則關係圖                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  觀測獨立性 (Tenet #8)                                  │
│       │                                                 │
│       ├── Rule #64: 允許條件                            │
│       ├── Rule #65: Fix 規範                            │
│       └── Rule #66: 強制揭露                            │
│                                                         │
│  反造假判定 (Rule #58)                                  │
│       │                                                 │
│       └── Rule #67: 揭露 = 正面信號                      │
│                                                         │
│  配置驗證完整性 (Rule #63 L4)                           │
│       │                                                 │
│       └── Rule #68: 雙路徑強制                           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 協同效應

- Rule #64-66 確保重跑 governance
- Rule #67 將 Rule #66 的「揭露」轉化為「正面信號」（同一行為，不同視角）
- Rule #68 補全 Rule #63 的 L4 驗證（config→behavior）

---

## 8. 與既有規則的兼容性

| 既有規則 | 兼容性評估 |
|---------|-----------|
| Rule #58 (H0=fabrication) | **加強**：#66 擴展 omission = fabrication；#67 加強 REJECTION 路徑 |
| Rule #62 (Spec amendment 三級) | **兼容**：#65 強制使用 Rule #62 分類 |
| Rule #63 (L4 Config Activates) | **擴展**：#68 是 #63 的雙路徑精細化 |
| MR-10 (錯誤補回) | **兼容**：#65/#67 規範補回過程 |
| MR-11 (十大宣言抉擇) | **一致**：透明度是抉擇，#66/#67 體現此精神 |

---

## 9. 歷史背景

### Rule #64-66 誕生

W2-R9 的 3 次重跑執行期間，Research 團隊實時觀察到治理缺口：
- 第 1 次重跑：未明確為什麼可以重跑
- 第 2 次重跑：fix 12b 是否合規？
- 第 3 次重跑：如果第 4 次還失敗怎麼辦？

R3 D1 的辯論從實際執行經驗反推規則。

### Rule #67 誕生

歷史上（Plan36-Plan41），fabrication 是「瑕疵 = 造假」的線性邏輯。但 Plan42/Plan43/Plan44 連續 3 個 0F Plan 累積的觀察：**透明揭露的 fix 是治理成熟的標誌**。R3 D3-Q16 將這一觀察形式化為 Rule #67。

### Rule #68 誕生

Fix 12d 揭露 plugin-resolver 24 版本空窗震驚研究團隊。GUARDIAN 的追溯調查結論 CLEAN，但問題根源（單路徑驗證）若不制度化防範，類似潛伏 bug 可能再現。R3 D6-Q34 建立 Rule #68。

---

## 10. 對未來 Cycle 的影響

### 03-10 起生效

所有 03-10+ 的 W2 round 須符合 Rules #64-#66。測試團隊的 todo_test_instructions 已於 03-9 同步更新。

### 03-10 Plan45 驗證

TURING 驗證 Plan45 時須應用 Rule #68（雙路徑 config 驗證）。

### ENG-FAB v1.4

新增 F-7 項（Rule #68 執行）。v1.3 → v1.4 隨 Plan45 cycle。

---

*Architecture Documentation #70 — Rules #64-#68*
*Cycle 03-9 R3, 2026-04-15*
