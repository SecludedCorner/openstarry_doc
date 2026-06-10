# 66. HMAC 種子認證架構技術規範 (HMAC Seed Authentication Architecture)

本文件定義 OpenStarry 分散式阿賴耶 (distributed-alaya) 插件中，跨代理種子 (seed) 認證的安全架構。涵蓋 v0.39.0-alpha 的已知缺陷分析、威脅模型、修復方案比較，以及 Plan40 的具體設計。

---

## 1. 當前問題 (v0.39.0-alpha)

v0.39.0-alpha 的 HMAC 簽名系統存在結構性設計缺陷，導致跨代理認證功能完全失效。

### 1.1 F-2: HMAC 同義反覆驗證 (Tautological Verification)

`DistributedAlayaImpl.propagate()` 中的簽名驗證使用**發送者自身的 HMAC 密鑰**進行驗證：

```typescript
// distributed-alaya-impl.ts lines 112-114
const isValid = await this.signatureService.verify(signedSeed);
if (!isValid) continue;
```

`this.signatureService` 是發送者自己的 `SeedSignatureServiceImpl` 實例，以發送者的 HMAC 密鑰（`randomBytes(32)` 或由配置提供）初始化。該種子在第 94 行由 `this.signatureService.sign(matchedSeed)` 簽名，又在第 113 行由同一密鑰驗證。

**結果**：這是一個自一致性檢查 (self-consistency check)，不是跨代理認證 (cross-agent authentication)。`verify()` 對由同一代理在同一執行期簽名的種子**永遠返回 true**。

[程式碼: openstarry_plugin/distributed-alaya/src/distributed-alaya-impl.ts#propagate]

### 1.2 SEC-002: exchangeSeeds() 完全繞過驗證

`exchangeSeeds()` (lines 153-197) 呼叫 `accept()` 時**完全沒有任何 verify() 呼叫**。種子從 `query()` 直接傳遞到 `accept()`，中間不經過任何密碼學檢查。

| 程式碼路徑 | 驗證方式 | 實際安全效果 |
|:---|:---|:---|
| `propagate()` | `this.signatureService.verify()` (發送者密鑰) | 同義反覆 -- 自簽自驗 |
| `exchangeSeeds()` | **無** | 零密碼學驗證 |
| `accept()` | 檢查 `seed.signature` 是否非 null | 僅存在性檢查，非有效性檢查 |

[程式碼: openstarry_plugin/distributed-alaya/src/distributed-alaya-impl.ts#exchangeSeeds]
[程式碼: openstarry_plugin/distributed-alaya/src/bija-store.ts#accept]

### 1.3 根因分析

HMAC 系統的根本問題在於**密鑰分發模型**：每個代理在初始化時自行生成隨機 HMAC 密鑰，密鑰從不共享。在此模型下，接收方無法獨立驗證發送方的簽名，因為接收方不持有發送方的密鑰。

程式碼註釋明確承認此限制：

> "Signature verification is performed by the caller... using the sender's ISeedSignatureService -- since the HMAC key is agent-local and cannot be re-verified by the receiver."
> -- bija-store.ts line 112

然而，此註釋造成了**虛假的安全敘事** (false security narrative)：未來維護者讀到「Signature verification is performed by the caller」時，會合理地認為跨代理認證正在發生。實際上並沒有。

### 1.4 綜合影響

HMAC 系統提供**零跨代理認證**。兩條路徑（propagate 的同義反覆驗證 + exchangeSeeds 的完全缺失驗證）從不同角度描述了**同一架構缺陷**。

**嚴重性：HIGH**

- 不降級為 MEDIUM 的理由：缺陷是結構性的；代碼註釋製造虛假安全感；從單進程到多進程的轉換會立即使漏洞可被利用。
- 不升級為 CRITICAL 的理由：在 v0.39.0-alpha 部署模型（單進程）下不可被利用；distributed-alaya 插件無外部消費者 (F-1)；修復方案已知且直觀。

[來源: R2_06_GUARDIAN_reviews_SUSSMAN.md#1.4 Severity Assessment]

---

## 2. 威脅模型

### 2.1 信任假設

當前簽名模型依賴以下假設：

1. 所有代理實例共享同一進程位址空間（進程內模型）
2. 沒有代理可以直接篡改另一代理的 BijaStore 或 SeedSignatureService（語言級隔離）
3. `DistributedAlayaImpl` 是 `accept()` 的唯一呼叫者（呼叫者紀律假設）

### 2.2 依部署模型的攻擊面

#### 場景 A：單進程（v0.39.0-alpha 現行模型）

| 攻擊向量 | 可利用？ | 原因 |
|:---|:---|:---|
| 以任意簽名字串偽造種子 | **否** | 攻擊者需繞過 TypeScript 類型系統並直接呼叫 BijaStore 的 `accept()`。進程內隔離依賴模組封裝。 |
| 簽名後修改種子 payload | **否** | propagate() 中的同義反覆 verify() 能捕捉 payload 修改（發送者密鑰仍正確用於自一致性）。但這是**意外防禦**，非設計意圖。 |
| 透過 exchangeSeeds() 注入無簽名種子 | **否** | `accept()` 檢查 `!seed.signature`（存在性閘門）。來自 `query()` 的種子在 `plant()` 時已簽名。 |
| 透過 exchangeSeeds() 注入偽造簽名種子 | **否** | 進程內，`query()` 返回合法植入種子的副本。無程式碼路徑允許攻擊者在不直接操作記憶體的情況下將偽造種子注入查詢結果集。 |

**場景 A 有效可利用性：無。**

實際安全邊界是 JavaScript 執行期的物件引用隔離，而非 HMAC 簽名。

#### 場景 B：多進程（Plan40+）

| 攻擊向量 | 可利用？ | 原因 |
|:---|:---|:---|
| 以任意簽名字串偽造種子 | **是** | 惡意進程可發送帶有 `signature: "anything"` 的 JSON 訊息到 `accept()`。接收進程無密鑰可驗證發送者的 HMAC。 |
| IPC 中間人攻擊 | **是** | 若傳播跨越進程邊界，攔截者可修改種子 payload 並以任意字串替換簽名。`accept()` 僅檢查存在性。 |
| 重放攻擊 | **是** | 已捕獲的傳播訊息可被重放。`accept()` 中無 nonce、無時間戳驗證。向量時鐘合併會默默接受重放的種子。 |
| 透過 exchangeSeeds() 的身份偽造 | **是** | `exchangeSeeds()` 執行零驗證。從 `query()` 返回偽造種子的惡意對等方，其種子會被直接接受。 |

**場景 B 有效可利用性：高 (HIGH)。**

#### 場景 C：多機器（假設性）

場景 B 的所有攻擊均適用，再加上網路層攻擊。傳播通道缺乏 TLS（AT-4a 已延遲處理）使 HMAC 缺口雪上加霜。

**場景 C 有效可利用性：嚴重 (CRITICAL)。**

[來源: R2_06_GUARDIAN_reviews_SUSSMAN.md#1.3 Threat Model]

### 2.3 複合威脅：SEC-001 TOCTOU + F-2 HMAC

最令人擔憂的複合場景：

1. AT-7b 殘餘風險（SEC-001 TOCTOU）允許影子代理 (shadow agent) 註冊
2. 影子代理擁有自己的 HMAC 密鑰（每代理隨機生成）
3. 影子代理呼叫 `propagate()` 傳播偽造種子
4. 接收代理的 `accept()` 僅檢查簽名存在性 -- 通過
5. 偽造種子進入合法代理的 BijaStore

此複合攻擊需要：(a) 多進程部署、(b) SEC-001 TOCTOU 利用、(c) F-2 HMAC 缺口。三者在引入多進程時**同時**變為可行。

**結論**：SEC-001 修復（`has()` 移入寫鎖內）與 F-2 修復（共享或非對稱密鑰）**必須在同一 Plan 交付**，以防止此複合向量開啟。

[來源: R2_06_GUARDIAN_reviews_SUSSMAN.md#3.2 Combined Threat]

---

## 3. 方案比較

R3 D5-Q1 辯論中，研究團隊評估了三個修復方案：

### 3.1 方案總覽

| 面向 | Option A: 共享 HMAC 密鑰 | Option B: 成對密鑰 | Option C: PKI |
|:---|:---|:---|:---|
| **信任粒度** | 叢集全域 | 成對 | 每代理 |
| **密鑰管理** | O(1) | O(n^2) | O(n) |
| **洩漏爆炸半徑** | 全部 -- 一個密鑰洩漏 = 全部代理受影響 | 隔離 -- 一對洩漏 = 兩個代理 | 每代理 -- 一個密鑰 = 一個代理 |
| **計算成本** | HMAC (低) | HMAC (低) | 非對稱簽名 (中) |
| **實作複雜度** | ~12-15 LOC | O(n^2) 密鑰分發 | PKI 生命週期管理 |
| **適用場景** | 單機多進程 | 多機（中等規模） | 多機（生產環境） |

### 3.2 Option A: Daemon 分發共享 HMAC 密鑰（推薦方案）

- Daemon 啟動時生成叢集全域 HMAC 密鑰
- 透過 spawn 時的 IPC payload 分發給每個代理
- 所有代理以同一密鑰簽名；任何代理皆可驗證任何種子
- **優點**：最少程式碼變更（~12-15 LOC），無 PKI 基礎設施需求
- **缺點**：一個代理洩漏密鑰則全叢集受影響；無每代理歸屬
- **架構一致性**：Daemon 已作為身份的信任錨點 (AT-7 closure)，將密鑰分發角色擴展給 Daemon 在架構上是一致的

### 3.3 Option B: 成對共享密鑰 (Daemon-Brokered)

- Daemon 為每個代理對生成唯一共享密鑰
- 將 secret(A,B) 分發給 Agent A 和 Agent B
- 驗證使用成對專屬密鑰
- **優點**：洩漏隔離；成對歸屬
- **缺點**：O(n^2) 密鑰分發；更複雜的密鑰管理

### 3.4 Option C: 非對稱簽名 (PKI)

- 每個代理生成金鑰對；公鑰註冊到 Daemon
- 種子以私鑰簽名；以發送者公鑰驗證（從註冊表取得）
- **優點**：無共享密鑰；最強安全模型；每代理歸屬
- **缺點**：較高計算成本；PKI 生命週期管理

### 3.5 決議

**D5-Q1 決議（5-0 全票通過）**：Plan40 採用 Option A（Daemon 分發共享 HMAC 密鑰）。此為多進程部署的門檻條件 (gate)。密鑰注入機制必須透過可替換介面 (ISeedKeyProvider) 實作，為未來 PKI 遷移預留路徑。

[來源: R3_D5_architecture.md#D5-Q1]

---

## 4. Plan40 設計：Option A 實作

### 4.1 密鑰生成

Daemon 在啟動時生成 32 位元組的叢集密鑰：

```typescript
// Daemon init
import { randomBytes } from 'node:crypto';
const clusterHmacKey = randomBytes(32);
```

此密鑰在 Daemon 的整個生命週期內保持不變。密鑰輪換 (key rotation) 延遲至 Plan41+。

### 4.2 密鑰分發

密鑰透過 spawn 時的 IPC payload 分發給每個代理：

```typescript
// Extend IPC spawn message schema
interface AgentSpawnPayload {
  agentId: string;
  // ... existing fields
  hmacKey: Buffer;  // +1 field
}
```

Daemon 在 fork 代理進程時，將 `clusterHmacKey` 包含在 spawn payload 中。代理進程從初始化配置中讀取此密鑰。

### 4.3 ISeedKeyProvider 介面

密鑰注入透過可替換介面實作，使未來可無縫遷移至 PKI 模型：

```typescript
interface ISeedKeyProvider {
  getSigningKey(agentId: string): Buffer;
  getVerificationKey(agentId: string): Buffer;
}
```

**Option A 實作**：`getSigningKey()` 與 `getVerificationKey()` 皆返回同一叢集密鑰。

**未來 PKI 實作**：`getSigningKey()` 返回代理私鑰，`getVerificationKey()` 返回發送者公鑰。

### 4.4 SeedSignatureService 變更

插件工廠不再自行生成隨機密鑰，改為從 spawn 配置接收密鑰：

```typescript
// Before (v0.39):
const secret = config.hmacSecret ?? randomBytes(32);  // agent-local key

// After (v0.40):
const secret = config.hmacKey;  // Daemon-distributed cluster key
// config.hmacKey is received from spawn payload
```

現有建構子已接受可選密鑰參數，因此這主要是配置穿透 (config passthrough) 的變更。

### 4.5 驗證流程變更

修復後，跨代理驗證變為有意義的：

| 步驟 | v0.39 (修復前) | v0.40 (修復後) |
|:---|:---|:---|
| 代理 A 簽名種子 | 以 A 的私有密鑰簽名 | 以叢集共享密鑰簽名 |
| 代理 A 傳播至 B | `propagate()` 以 A 的密鑰自驗 (同義反覆) | `propagate()` 以共享密鑰驗證 (有意義) |
| 代理 B 接收 | `accept()` 僅檢查簽名存在 | `accept()` 可獨立驗證簽名有效性 |
| exchangeSeeds() | 零驗證 | 兩個方向均呼叫 `verify()` |

### 4.6 實作明細

| 組件 | 變更 | LOC |
|:---|:---|:---|
| Daemon：啟動時生成叢集密鑰 | `randomBytes(32)` in Daemon init | +2 |
| Daemon：spawn payload 中包含密鑰 | 擴展 IPC 訊息結構 | +3 |
| Agent：從 spawn payload 接收密鑰 | 從初始化配置讀取 | +2 |
| 插件工廠：使用接收密鑰而非自行生成 | 配置參數穿透 | +3 |
| `propagate()` 與 `accept()` 註釋更新 | 自一致性 -> 跨代理驗證 | +2-3 |
| **總計** | | **~12-15** |

[來源: plan40_engineering_recommendation.md#W3-C]

---

## 5. 配套安全修復

F-2 HMAC 修復必須與以下安全修復在同一 Wave (W3) 作為原子交付：

### 5.1 SEC-001: TOCTOU 修復 (~3-5 LOC)

**問題**：`onAgentSpawned()` 中的 `has()` 檢查在寫鎖之外。

**修復**：將 `has()` 檢查移入寫鎖內，使用 `try/finally` 模式防止鎖洩漏。

```typescript
// 修復後 (registry-bridge.ts)
this.registry.lock.acquireWrite().then(() => {
  try {
    if (this.registry.has(agentId)) { /* reject duplicate */ return; }
    this.registry.register(entry);
  } finally { this.registry.lock.releaseWrite(); }
});
```

**檔案**：`registry-bridge.ts`

[來源: R1_GUARDIAN_security_engfab.md#SEC-001]

### 5.2 SEC-002: exchangeSeeds() 簽名驗證 (~8-10 LOC)

**問題**：`exchangeSeeds()` 在兩個方向的 `accept()` 呼叫前均未執行 `verify()`。

**修復**：在 `exchangeSeeds()` 的兩個交換方向中，於 `accept()` 前新增 `verify()` 呼叫。搭配 HMAC Option A（共享密鑰），此驗證對跨代理種子變為有意義。

**檔案**：`distributed-alaya-impl.ts`

[來源: R1_GUARDIAN_security_engfab.md#SEC-002]

### 5.3 原子交付要求

**綁定約束 C4**：SEC-001 (TOCTOU) + SEC-002 (verify) + HMAC Option A **必須在同一 Wave 交付**。

理由：
- 部分交付會產生不完整的安全狀態
- SEC-001 + F-2 的複合威脅（第 2.3 節）要求兩者同時關閉
- 三個修復共同針對同一威脅面（多代理場景下的種子完整性）

[來源: plan40_engineering_recommendation.md#Binding Constraints C4]

---

## 6. 驗收標準

| ID | 標準 | 驗證方式 |
|:---|:---|:---|
| AC-W3-1 | SEC-001：`has()` 檢查在 `onAgentSpawned()` 的寫鎖內 | 程式碼審查 + GUARDIAN 審計 |
| AC-W3-2 | SEC-001：`try/finally` 模式防止鎖洩漏 | 程式碼審查 |
| AC-W3-3 | SEC-002：`verify()` 在 `exchangeSeeds()` 的兩個方向均被呼叫 | 程式碼審查 + GUARDIAN 審計 |
| AC-W3-4 | HMAC：Daemon 啟動時生成 `randomBytes(32)` 密鑰 | 程式碼審查 |
| AC-W3-5 | HMAC：Spawn payload 包含 HMAC 密鑰 | 程式碼審查 + IPC 訊息結構 |
| AC-W3-6 | HMAC：SeedSignatureService 從 spawn 配置接收密鑰（非自行生成） | 程式碼審查 |
| AC-W3-7 | HMAC：密鑰注入使用可替換抽象 (ISeedKeyProvider 或等價物) | 介面審查 |
| AC-W3-8 | HMAC：`propagate()` 與 `accept()` 註釋更新以反映跨代理驗證模型 | 程式碼審查 |
| AC-W3-9 | SEC-001 + SEC-002 + HMAC 以原子提交交付（同一 Wave） | Git log |
| AC-W3-10 | 建置通過 (`npm run build`) | 建置日誌 |
| AC-W3-11 | 現有測試通過 (`npm test`) | 測試日誌 |

[來源: plan40_engineering_recommendation.md#Section 3 Acceptance Criteria]

---

## 7. 未來演進路徑

HMAC 認證架構依循**漸進式信任升級**路線：

```
Phase 1 (Plan40)         Phase 2 (Plan41+)        Phase 3 (Plan42+)
────────────────         ─────────────────        ─────────────────
共享 HMAC 密鑰      →    成對密鑰 / 密鑰輪換  →   PKI (非對稱簽名)
Option A                 Option B                  Option C
O(1) 密鑰管理            O(n^2) 密鑰管理           O(n) 密鑰管理
叢集全域信任              成對信任隔離              每代理信任歸屬
單機多進程                多機中等規模              多機生產環境
```

### 7.1 已延遲項目

| 項目 | 優先級 | 延遲至 | 理由 |
|:---|:---|:---|:---|
| HMAC 密鑰輪換 | P2 | Plan41+ | Plan40 Option A 的已知限制 |
| 成對密鑰分發 (Option B) | P2 | Plan41+ | 需要 O(n^2) 密鑰管理基礎設施 |
| PKI 非對稱簽名 (Option C) | P3 | Plan42+ | 需要完整 PKI 生命週期管理 |
| IPC 加密 (AT-4a) | P3 | Plan42+ | 僅在多機器部署時相關 |
| 重放防護 (nonce/timestamp) | P2 | Plan41+ | 搭配密鑰輪換一併處理 |

### 7.2 ISeedKeyProvider 的遷移路徑

ISeedKeyProvider 介面的設計確保每個階段的遷移僅需替換 provider 實作，無需修改 SeedSignatureService 核心邏輯：

| 階段 | ISeedKeyProvider 實作 | getSigningKey() | getVerificationKey() |
|:---|:---|:---|:---|
| Phase 1 | SharedKeyProvider | 返回叢集共享密鑰 | 返回叢集共享密鑰 |
| Phase 2 | PairKeyProvider | 返回 secret(A,B) | 返回 secret(A,B) |
| Phase 3 | PkiKeyProvider | 返回代理私鑰 | 返回發送者公鑰 |

---

## 8. 正面發現

### 8.1 常數時間比較 (Constant-Time Comparison)

`seed-signature.ts` lines 48-52 正確實作了常數時間比較：

- 長度檢查優先（防止長度預言機攻擊）
- 逐位元組 XOR 搭配累加器（防止計時預言機攻擊）
- 基於累加差異返回布林值

建議未來替換為 Node.js `crypto.timingSafeEqual`（優先級 P3），但當前手動實作功能正確。

[程式碼: openstarry_plugin/distributed-alaya/src/seed-signature.ts#SeedSignatureServiceImpl]

### 8.2 F-8 所有權強制執行

agentId 強制執行在多層正確實作：

- `plant()`：BijaStoreImpl 檢查 `seed.agentId !== this.agentId`
- `propagate()`：DistributedAlayaImpl 額外檢查（縱深防禦）
- `accept()`：有意跳過 F-8（接收的種子合法屬於其他代理）
- `update()`：SeedPatch 排除 `skandha` 和 `agentId`（編譯期 + 執行期雙重強制）

---

## 研究來源

- [來源: R3_D5_architecture.md#D5-Q1] -- F-2 HMAC 方案辯論與決議
- [來源: R2_06_GUARDIAN_reviews_SUSSMAN.md] -- F-2 安全深度分析、威脅模型、方案比較
- [來源: R1_SUSSMAN_KERNEL_architecture.md#F-2] -- 跨代理簽名驗證缺口原始發現
- [來源: R1_GUARDIAN_security_engfab.md#SEC-001, SEC-002] -- TOCTOU 與 exchangeSeeds() 驗證缺失
- [來源: plan40_engineering_recommendation.md#W3] -- Plan40 W3 安全修復實作規範
