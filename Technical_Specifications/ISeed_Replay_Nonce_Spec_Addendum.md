<!-- Status: CURRENT -->
<!-- Applies to: v0.59.3-alpha -->
<!-- Last verified: 2026-06-15 (interface change + replay behavior covered by replay-nonce.test.ts, 6/6 passing) -->

# Spec Addendum — ISeed Replay-Nonce (2026-06-15)

## 授權

修改 **FROZEN 介面** `ISeed`（`packages/sdk/src/types/distributed-alaya.ts`）。該介面標注「Once published, this interface is FROZEN. Changes require a new Spec Addendum through the Coordinator.」本 Addendum 由 Master 於 2026-06-15 親手授權（決策見 coordinator memory）。

## 變更內容

`ISeed` 新增一個**選填**欄位：

```ts
nonce?: number;
```

- **選填、向後相容**：pre-addendum 的種子沒有 nonce，走原路徑（無防重放檢查），不破壞既有資料或測試。
- **純加性（BABBAGE BCT trivially satisfied）**：不移除、不改既有欄位語意。
- **不可變**：`SeedPatch`（`Pick<Partial<ISeed>, 'content'|'visibility'|'updatedAt'|'signature'>`）未把 nonce 納入 allowlist，故 `update()` 無法改 nonce——種一次定一次。

## 為什麼需要

宣言 #6 的跨進程種子傳播（v0.59 起 N=2 單機為真）原本明確標注「無 replay nonce」。威脅：能監看傳輸的攻擊者可把一則**簽章仍有效**的種子訊息錄下、反覆重送，接收端因簽章有效而反覆接受。此威脅在**跨主機走網路**時才實際成立；單機本地管道情境下，能讀管道者已在本機內。本 Addendum 為跨主機方向**預先補上防線**，並順帶接通一段早已寫好卻從未接線的代碼（見下）。

## 實作（讓欄位是活的，不是死欄位）

1. **簽章自動覆蓋**：`seed-signature.ts` 的 `seedCanonical()` 只排除 `signature` 欄位、其餘全部 hash——故 nonce **自動納入 HMAC 簽章範圍**。竄改 nonce 即破壞簽章，在 `verify()` 階段就被擋下（早於 nonce 檢查）。
2. **接通既有 `verifyNonce`**：`SeedSignatureServiceImpl.verifyNonce(agentId, nonce)`（SEC-001 / Plan46 W0）是一段**早已存在但從未被呼叫**的單調遞增 nonce 防重放邏輯。本 Addendum 在跨進程接收端 `DistributedAlayaImpl.acceptRemote()` 接通它：種子 nonce ≤ 該 agentId 上次接受的 nonce → 判為重放/亂序，fail-closed 拒收。
3. **指派**：`DistributedAlayaImpl.plant()` 在呼叫者未提供時，自動為本 agent 的種子蓋上嚴格遞增的 nonce（wall-clock ms 為基、必要時 +1 保證單調）。
4. **邊界**：in-process 傳播路徑（同進程，信任）**不**做 nonce 檢查；防重放只在跨進程 `acceptRemote` 收端，即威脅實際存在處。

## 證據

`openstarry_plugin/distributed-alaya/src/__tests__/replay-nonce.test.ts`（6/6 通過）：
- 蓋章種子首次接受、逐位元重放被拒（store 不被污染）
- 較低（亂序）nonce 被拒
- 竄改 nonce → HMAC verify 失敗（早於 nonce 檢查）
- 跨 agent 獨立計數（agent-a 的重放不擋 agent-b 首送）
- 無 nonce 種子走 legacy 路徑（可冪等重接、不報重放）
- `plant()` 自動蓋嚴格遞增 nonce

## 仍明確不宣稱（未竟）

跨主機 transport、N>2 gossip、late-joiner snapshot 跨進程交換、nonce 計數器跨進程重啟持久化（現為 per-run 記憶體態；agent 重啟後計數器歸零，跨主機/重啟韌性與 cross-host 工程一併處理）。

---
*ISeed 介面權威仍為 SDK 型別檔；本 Addendum 為其唯一一次 post-freeze 修訂的正式紀錄。*
