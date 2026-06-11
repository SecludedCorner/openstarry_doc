<!-- Layer: 3-Architecture -->
<!-- Status: NEW -->
<!-- Cycle: 03-5, R1 -->
<!-- Author: SUSSMAN (#22), BABBAGE (#9) -->
<!-- Source: v0.40.0-alpha -->

# 59. ISeedKeyProvider 抽象分析與 DELTA_SCALING_FACTOR 機制定位

**版本**: 1.0 (Cycle 03-5, 2026-04-07)
**作者**: SUSSMAN (#22) — 程式結構與抽象分析, BABBAGE (#9) — 機制/策略分類
**來源**: Plan40 驗證 (15P/4D/1F), W2-R4 校準鎖定

---

## 1. 概述

本文件記錄 Cycle 03-5 對 Plan40 引入的兩個關鍵架構要素的抽象品質分析：

1. **ISeedKeyProvider** — HMAC 金鑰注入抽象介面
2. **DELTA_SCALING_FACTOR** — 校準縮放因子的機制定位

兩者皆為 Plan40 交付項目，經 TURING 三欄驗證 (15P/1D/0F) 確認實作正確。本分析聚焦於抽象品質與架構適切性。

---

## 2. ISeedKeyProvider 抽象品質

### 2.1 介面定義

**來源**: `openstarry_plugin/distributed-alaya/src/seed-signature.ts`

```typescript
export interface ISeedKeyProvider {
  getKey(): Buffer;
}
```

### 2.2 評分: A (教科書級依賴反轉)

| 評估維度 | 結果 | 說明 |
|----------|------|------|
| **最小性** (Minimality) | 優秀 | 僅一個方法 `getKey(): Buffer`，零多餘 API |
| **替換性** (Substitutability) | 優秀 | `DaemonKeyProvider` + `RandomKeyProvider` 完全可替換 |
| **集成方式** | 優秀 | 工廠模式在 `index.ts` 中根據 config 選擇實作 |
| **安全文件化** | 良好 | JSDoc 包含 "Implementations MUST NOT log the returned key" 協定約束 |
| **共同定位** (Co-location) | 優秀 | 介面、兩個實作、工廠全部位於同一模組 |

### 2.3 兩個實作的對稱性

| 實作 | 用途 | 金鑰來源 | 行數 |
|------|------|----------|------|
| `DaemonKeyProvider` (L44-50) | 生產環境 | 從 hex string 解碼 | ~7 |
| `RandomKeyProvider` (L57-63) | 獨立/測試 | `crypto.randomBytes(32)` | ~7 |

兩者回傳 `Buffer`，呼叫端無需知道哪個啟用。工廠 (`index.ts:66-68`) 根據 config 存在與否選擇 — 清潔的條件構造模式 (conditional construction pattern)。

### 2.4 Reader Pattern 分析

金鑰提供者在 plugin factory 時被呼叫一次：

```typescript
const signatureService = new SeedSignatureServiceImpl(keyProvider.getKey());
```

這是 **Reader Pattern** — 抽象屏障存在於配置邊界，而非每次呼叫點。金鑰材料在建構時解析一次，之後不再重複取得。這在架構上是正確的：金鑰解析是一次性的配置事件，不是重複的運行時操作。

### 2.5 結構隱憂: 雙路徑建構

`SeedSignatureServiceImpl` 建構函數 (L83-84) 接受 `secret?: Buffer` 並有 `randomBytes(32)` 後備方案。這意味著可繞過 `ISeedKeyProvider` 直接以原始 Buffer 建構。工廠 `createSeedSignatureService` (L105-107) 也繞過 ISeedKeyProvider。

**影響**: 兩條建構路徑共存 — 一條透過 ISeedKeyProvider（乾淨），一條透過原始 Buffer（遺留）。

**Plan41 處置**: W0 標記 `@deprecated`，引導後續遷移至單一 ISeedKeyProvider 路徑。[D4-Q9]

---

## 3. DELTA_SCALING_FACTOR 機制定位

### 3.1 程式碼位置

**來源**: `packages/core/src/execution/loop.ts` (L667-674)

```typescript
const DELTA_SCALING_FACTOR = 0.055;
const signMultiplier = (riskCat === 'destructive' || riskCat === 'state_modifying') ? -1 : 1;
const rawDelta = TOOL_CONFIDENCE_TABLE[riskCat] * signMultiplier * DELTA_SCALING_FACTOR;
```

### 3.2 機制 vs 策略分析

| 觀點 | 論據 |
|------|------|
| **支持機制** | 固定數學常數，源自校準流程 (W2-R3)。將 table-based 信心值轉換為有界 delta — 純數學變換。公式確定性: `(maxDelta / max(TABLE)) * HEADROOM = (0.05 / 0.85) * 0.935 = 0.055` |
| **反對機制** | 0.055 編碼了一個關於 delta 縮放激進程度的*策略決定*。若 HEADROOM (0.935) 或 maxDelta (0.05) 改變，0.055 必須跟著改變 |

### 3.3 SUSSMAN 裁定: 帶有策略衍生常數的機制

縮放因子正確放置在 Core 中，因為：

1. **位置正確**: 操作於執行迴路內部的 audit path — 這是轉換工具執行結果為信心 delta 的機制程式碼
2. **硬編碼正確** (C1): 非外部可配置 — 這適用於經校準的機制參數，不適用於使用者可調的策略旋鈕
3. **先例一致**: `TOOL_CONFIDENCE_TABLE` 已在 SDK 中，校準常數在 core/SDK 層有先例
4. **不違反 Tenet #7**: 不引入新的策略層級行為 — 參數化既有機制（delta 計算），而非決定*是否*計算

### 3.4 Scope 改進建議

常數宣告為 loop 迭代內的區域 `const` (L672)，非模組層級常數或命名匯出。這限制了可發現性。

**建議**: 提升為模組層級命名常數（例如 loop.ts 頂部附近的 `const DELTA_SCALING_FACTOR = 0.055 as const;`，或在 SDK 中 alongside TOOL_CONFIDENCE_TABLE）。

---

## 4. Plan40 架構影響評估

### 4.1 新耦合分析

| 變更 | 新耦合? | 評估 |
|------|---------|------|
| DELTA_SCALING_FACTOR in loop.ts | 否 | 使用既有 TOOL_CONFIDENCE_TABLE import |
| SEC-001 has() inside lock | 否 | 使用既有 registry.has() + registry.lock API |
| SEC-002 verify() in distributed-alaya | 否 | 使用既有 ISeedSignatureService.verify() |
| ISeedKeyProvider + DaemonKeyProvider | **最小** | 新介面，但僅在 plugin 邊界內消費 |
| HMAC key in daemon-entry.ts | **最小** | 新 env var 路徑 (OPENSTARRY_HMAC_KEY) |
| ipc-client.ts 型別修正 | 否 | 簽名變更 (Buffer -> string\|Buffer) |

### 4.2 模組數量變化

| 指標 | Plan39 | Plan40 | Delta |
|------|--------|--------|-------|
| 新模組 | 0 | 0 | 0 |
| 新介面 | 0 | 1 (ISeedKeyProvider) | +1 |
| 跨模組依賴 | 0 | 0 | 0 |

**結論**: Plan40 是一個**零新模組、零跨模組依賴**的交付。ISeedKeyProvider 是新介面，但完全封裝在 distributed-alaya plugin 內部。架構衝擊為零。

---

## 5. W2-R4 校準鎖定與機制確認

### 5.1 鎖定參數

| 參數 | 值 | 來源 |
|------|-----|------|
| DELTA_SCALING_FACTOR | **0.055** | W2-R3 校準 + W2-R4 驗證 |
| sigma | **0.02045** | W2-R4 實測 |
| F1/F2 | **0.997x** | 訊噪比達成 parity |
| Clamp rate | **0.0%** | 飽和消除 |

### 5.2 機制清單更新 (BABBAGE)

Plan40 確認的機制：

| # | 機制 | 分類 |
|---|------|------|
| 既有 | TOOL_CONFIDENCE_TABLE | Mechanism |
| 既有 | signMultiplier (-1/+1) | Mechanism |
| 新增 | DELTA_SCALING_FACTOR = 0.055 | Mechanism (策略衍生常數) |
| 新增 | ISeedKeyProvider.getKey() | Mechanism |
| 新增 | DaemonKeyProvider / RandomKeyProvider 工廠選擇 | Mechanism |

---

## 6. 與 Tenet 合規的關係

| Tenet | 影響 | 方向 |
|-------|------|------|
| #6 八識 | HMAC 強化 ISeedKeyProvider | 正面 |
| #7 微核心純淨 | DELTA_SCALING_FACTOR = 機制，非策略 | 中性 |
| #8 控制迴路 | 縮放改善訊號保真度 | 正面 |
| 其餘 | 無影響 | 中性 |

**結論**: Plan40 交付不影響 10/0/0 合規狀態。ISeedKeyProvider 強化安全架構，DELTA_SCALING_FACTOR 改善控制迴路精度。

---

## 7. 設計原則摘要

1. **介面最小化**: ISeedKeyProvider 示範了「最小可行介面」原則 — 單一方法、單一回傳型別
2. **Reader Pattern 適用性**: 一次性配置讀取比重複呼叫更適合金鑰場景
3. **機制常數應模組層級宣告**: 區域 const 降低可發現性
4. **抽象品質分級**: A (教科書級) — 可作為後續介面設計參考

---

*Architecture Documentation #59 — ISeedKeyProvider Abstraction Analysis*
*Cycle 03-5 Research Addition, 2026-04-07*
*SUSSMAN (#22), BABBAGE (#9)*
