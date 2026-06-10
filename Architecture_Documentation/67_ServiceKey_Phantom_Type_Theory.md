<!-- Layer: 3-Architecture -->
<!-- Status: NEW -->
<!-- Cycle: 03-6, R1 -->
<!-- Author: BABBAGE (#9), SUSSMAN (#22) -->
<!-- Source: v0.41.0-alpha Plan41 W1 ServiceKey<T> -->

# 64. ServiceKey\<T\> Phantom Type 型別理論分析

**版本**: 1.0 (Cycle 03-6, 2026-04-09)
**作者**: BABBAGE (#9) — 型別理論形式分析, SUSSMAN (#22) — 程式抽象品質
**來源**: Plan41 W1 Typed Service Registry (~105 LOC), SUSSMAN R1 報告
**決議參照**: D4-Q3 (03-5, typed registry 採用, 10-0)

---

## 1. 概述

Plan41 W1 將 OpenStarry 的 service registry 從 untyped string-keyed API 遷移到 phantom type-keyed API。本文件分析 ServiceKey\<T\> 的型別理論基礎、設計選擇理由，以及遷移方法論。

---

## 2. Phantom Type Pattern 在 TypeScript 中的實現

### 2.1 定義

**Phantom type** 是一個型別參數，出現在型別定義中但不出現在任何運行時值中。它僅在編譯時存在，用於在型別層面區分結構上相同但語義上不同的值。

### 2.2 ServiceKey\<T\> 實現

**來源**: `packages/sdk/src/types/service.ts:32-35`

```typescript
export class ServiceKey<T extends IPluginService> {
  readonly _phantom?: T;
  constructor(readonly name: string) {}
}
```

### 2.3 型別層面解構

| 組件 | 目的 | 運行時存在? |
|------|------|:----------:|
| `T extends IPluginService` | 約束型別參數為有效服務型別 | 否 |
| `readonly _phantom?: T` | 攜帶型別參數，建立名義型別區分 | 否 (optional, never assigned) |
| `constructor(readonly name: string)` | 運行時鍵值 (實際用於查找) | 是 |

**運行時成本**: 零。ServiceKey 在運行時就是一個持有 string 的物件。`_phantom` 欄位因為是 optional 且從不賦值，不佔用記憶體。

---

## 3. 為何選擇 Phantom Type

### 3.1 候選方案比較

| 方案 | 型別安全 | 運行時成本 | 遷移成本 | 可擴展性 |
|------|:--------:|:----------:|:--------:|:--------:|
| **Raw string** (舊方案) | 無 | 零 | N/A (現狀) | 高 |
| **Branded string** (`type ServiceKey = string & { __brand: T }`) | 部分 | 零 | 低 | 中 |
| **Symbol** (`const KEY = Symbol('service')`) | 無 (Symbol 不攜帶型別) | 極低 | 中 | 低 |
| **Phantom type class** (採用方案) | **完整** | **零** | **中** | **高** |

### 3.2 Branded String 的不足

```typescript
type BrandedKey<T> = string & { readonly __brand: T };
```

Branded string 的問題：
1. **Intersection 語義怪異**: `string & { __brand: T }` 在 TypeScript 中是合法但語義模糊 — string 是原始型別，不能有屬性
2. **No constructor**: 無法封裝建構邏輯，string literal 可以 `as BrandedKey<T>` 任意轉型
3. **Structural leakage**: 兩個不同 branded key 如果 brand 的型別結構相同，TypeScript 可能視為相容

### 3.3 Symbol 的不足

```typescript
const COGNITION_CONFIG = Symbol('cognition-config');
```

Symbol 的問題：
1. **No type parameter**: JavaScript Symbol 不攜帶泛型型別參數
2. **Map key limitation**: 以 Symbol 為 key 的 Map 無法推斷 value 型別
3. **Serialization**: Symbol 不可序列化，影響跨程序通訊

### 3.4 Phantom Type 的優勢

1. **完整的型別推斷鏈**: `get<T>(key: ServiceKey<T>): T | undefined` — 回傳型別自動推斷
2. **名義型別區分**: `ServiceKey<ICognitionConfigService>` 和 `ServiceKey<IDistributedAlaya>` 是不相容型別
3. **運行時透明**: 在運行時就是 string lookup，零 overhead
4. **可擴展**: 新增服務只需新增一個 `new ServiceKey<T>(name)` 實例

---

## 4. 型別層面服務註冊安全保證

### 4.1 IServiceRegistry 型別簽名

```typescript
interface IServiceRegistry {
  get<T extends IPluginService>(key: ServiceKey<T>): T | undefined;
  has(key: ServiceKey<IPluginService>): boolean;
  unregister(key: ServiceKey<IPluginService>): boolean;
}
```

### 4.2 型別安全保證矩陣

| 操作 | 舊 API (string) | 新 API (ServiceKey\<T\>) | 保證 |
|------|:---:|:---:|---|
| `get('cognition-config')` | 回傳 `unknown` | N/A (不允許 string) | 強制使用型別鍵 |
| `get(SERVICE_KEYS.COGNITION_CONFIG)` | N/A | 回傳 `ICognitionConfigService \| undefined` | 型別自動推斷 |
| 錯誤的型別轉型 `as any` | 常見 | **消除** | `as any` 不再需要 |
| 註冊/取得型別不匹配 | 靜默錯誤 | **編譯時錯誤** | 型別不相容時拒絕編譯 |

### 4.3 消除的不安全模式

Plan41 W1 遷移消除了以下不安全模式：

| # | 模式 | 舊程式碼 | 新程式碼 |
|---|------|----------|----------|
| 1 | `as any` in distributed-alaya | `ctx.services.get('distributed-alaya') as any` | `ctx.services.get(SERVICE_KEYS.DISTRIBUTED_ALAYA)` |
| 2 | `as T \| undefined` in agent-core | `registry.get(name) as T \| undefined` | `registry.get(key)` (自動推斷) |
| 3 | String literal typo | `get('cogntion-config')` (不會被抓到) | `SERVICE_KEYS.COGNTION_CONFIG` (編譯時錯誤) |
| 4 | Wrong type assumption | `get('x') as IWrongType` | 型別由 key 決定，不需 assertion |

**Finding F-4 (MEDIUM)** 和 **Finding F-5 (MEDIUM)** (Cycle 03-5) 的 `as any` 和 unsafe cast 發現類別在 Plan41 W1 後**消除**。

---

## 5. SERVICE_KEYS 集中註冊表

### 5.1 當前註冊

```typescript
export const SERVICE_KEYS = {
  COGNITION_CONFIG: new ServiceKey<ICognitionConfigService>('cognition-config'),
  DISTRIBUTED_ALAYA: new ServiceKey<IDistributedAlaya & IPluginService>('distributed-alaya'),
} as const;
```

### 5.2 設計決策

| 決策 | 選擇 | 理由 |
|------|------|------|
| 集中 vs 分散 | **集中** | 所有鍵在單一位置，可發現性高 |
| `as const` | **是** | 防止意外修改 |
| Intersection type `IDistributedAlaya & IPluginService` | **是** | IDistributedAlaya 不直接 extend IPluginService，需明確 intersection |

### 5.3 新服務註冊流程

```
1. 在 SERVICE_KEYS 中新增: new ServiceKey<INewService>('new-service')
2. 在 Plugin factory 中: ctx.services.register(SERVICE_KEYS.NEW_SERVICE, impl)
3. 在消費端: const svc = ctx.services.get(SERVICE_KEYS.NEW_SERVICE)
   → 型別自動推斷為 INewService | undefined
```

### 5.4 has() / unregister() 的型別簡化

`has()` 和 `unregister()` 接受 `ServiceKey<IPluginService>` (non-generic) 而非 `ServiceKey<T>`。

**理由** (SUSSMAN): 這些方法只需要 `.name` string，不回傳 typed value。Generic 參數會是 unused — 這是可接受的實用簡化。

---

## 6. 遷移方法論

### 6.1 原子遷移 (Atomic Migration)

Plan41 W1 採用**原子遷移** — 舊 API 和新 API 不共存：

```
Step 1: 定義 ServiceKey<T>, SERVICE_KEYS, 新 IServiceRegistry 簽名
Step 2: 同時更新所有 8 個消費檔案
Step 3: 移除舊 get<T>(string) 簽名
```

### 6.2 為何不並行 API

| 方案 | 優點 | 缺點 |
|------|------|------|
| 並行 API (舊+新共存) | 漸進遷移、低單次風險 | 維護兩套 API、混用風險、遷移可能永遠不完成 |
| **原子遷移** (一次切換) | 無混用風險、乾淨切割 | 單次觸及多檔案 |

**採用原子遷移**: 在 monorepo 中，所有消費端共享相同的程式碼庫和版本週期。原子遷移避免了 API 分叉的維護成本。8 個檔案的修改量可控。[D4-Q3, Master letter S5-2: 拒絕兩套 API 共存]

### 6.3 遷移影響範圍

| 套件 | 修改檔案數 | 修改 LOC |
|------|:---:|:---:|
| SDK (`packages/sdk/`) | 2 | ~35 |
| Core (`packages/core/`) | 2 | ~15 |
| Plugin (distributed-alaya, comm-proxy, workflow) | 3 | ~25 |
| Service executor | 1 | ~5 |
| **合計** | **8** | **~80** |

每個檔案的修改為 1-10 LOC — 寬但淺 (wide but shallow)。

---

## 7. 型別理論觀察 (BABBAGE)

### 7.1 Phantom Type 與 Dependent Type 的關係

在完整的 dependent type 系統（如 Idris, Agda）中，`get(key)` 的回傳型別可以*直接*依賴 `key` 的值。TypeScript 不是 dependently typed — phantom type 是在 TypeScript 的結構型別系統中*模擬*依賴型別的慣用模式。

```
Dependent type (理想):  get : (k : Key) → Lookup(k)
Phantom type (實際):    get<T>(key: ServiceKey<T>): T | undefined
```

兩者在實踐效果上等價 — 回傳型別由輸入鍵決定 — 但 phantom type 的保證是淺層的（只在 TypeScript 編譯時，不在 JavaScript 運行時）。

### 7.2 名義 vs 結構型別

TypeScript 預設為結構型別 (structural typing)。ServiceKey\<T\> 使用 `_phantom?: T` 欄位引入名義區分 (nominal distinction)：

```typescript
ServiceKey<ICognitionConfigService> ≠ ServiceKey<IDistributedAlaya>
```

即使兩者的運行時結構相同（都是 `{ name: string }`），phantom type 讓 TypeScript 將它們視為不同型別。

### 7.3 io-ts / fp-ts 先例

此模式在 TypeScript 生態中有成熟的先例：

| Library | Phantom Type 用法 |
|---------|------------------|
| io-ts | `Type<A, O, I>` — A 是 phantom 解碼型別 |
| fp-ts | `HKT<URI, A>` — URI 是 phantom higher-kinded type |
| Angular | `InjectionToken<T>` — T 是 phantom 注入型別 |
| OpenStarry | `ServiceKey<T>` — T 是 phantom 服務型別 |

---

## 8. 邊界案例與限制

### 8.1 ServiceKey\<any\> 退化

`service-executor.ts:30` 使用 `ServiceKey<any>`：

```typescript
const key = new ServiceKey<any>(name);
```

這在動態 service name 場景下是必要的 — 當 name 在運行時才確定時，無法在編譯時推斷 T。

**分類**: INFO (已接受的實用妥協, Finding F-8)。影響範圍: 僅 service executor 的動態調用路徑。

### 8.2 Intersection Type 的必要性

`SERVICE_KEYS.DISTRIBUTED_ALAYA` 使用 `IDistributedAlaya & IPluginService` 而非單純 `IDistributedAlaya`：

**原因**: `IDistributedAlaya` 不直接 extend `IPluginService`。Plugin 在註冊時透過 wrapper 添加 `name`/`version` 欄位。Intersection type 表達了「這個服務同時滿足 IDistributedAlaya 和 IPluginService 介面」的事實。

### 8.3 Future-proofing

新增服務時的型別安全保證：

```typescript
// 編譯時阻止:
const wrong: IDistributedAlaya = ctx.services.get(SERVICE_KEYS.COGNITION_CONFIG);
// Error: Type 'ICognitionConfigService | undefined' is not assignable to 'IDistributedAlaya'

// 編譯時允許:
const right: ICognitionConfigService | undefined = ctx.services.get(SERVICE_KEYS.COGNITION_CONFIG);
// OK: type correctly inferred
```

---

## 9. 與 Tenet 合規的關係

| Tenet | 影響 | 說明 |
|-------|------|------|
| #7 微核心純淨 | 正面 | 消除 `as any` 不安全轉型 |
| #9 型別安全 | **正面** | 編譯時保證服務型別正確 |
| #6 八識架構 | 中性 | ServiceKey 是機制改進，不改變八識映射 |

**合規影響**: 強化 10/0/0。ServiceKey\<T\> 是純正面改進 — 消除已知不安全模式，不引入新風險。

---

## 10. 設計原則摘要

| 原則 | 實現 |
|------|------|
| **Phantom type 模擬依賴型別** | `ServiceKey<T>` 的 T 只在編譯時存在 |
| **零運行時成本** | `_phantom?: T` 從不賦值、不佔記憶體 |
| **集中註冊、分散使用** | `SERVICE_KEYS` 集中定義，各模組透過 import 使用 |
| **原子遷移** | 無並行 API 期，一次切換完成 |
| **約束繼承** | `T extends IPluginService` 防止無效參數化 |
| **實用妥協** | `ServiceKey<any>` 在動態路徑允許，但範圍受限 |

---

*Architecture Documentation #64 — ServiceKey\<T\> Phantom Type Theory*
*Cycle 03-6 Research Addition, 2026-04-09*
*BABBAGE (#9), SUSSMAN (#22)*
*Decision Reference: D4-Q3 (03-5)*
