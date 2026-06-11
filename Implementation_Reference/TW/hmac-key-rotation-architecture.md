# HMAC 密鑰輪轉 — 架構（僅設計，依 R3 D-17a）

**Plan48 範疇**：本文件僅為 **設計規格**。Runtime 密鑰輪轉 **不在
Plan48 實作範圍** 內，延至後續 cycle。R3 D-17(a) UNANIMOUS（23/0）。

**範圍限定語（D-12b）**：行程內範圍亦適用於輪轉 — 輪轉後的密鑰繼承
`./hmac-compliance.md` 宣告之 in-process-memory 威脅模型。

---

## 1. 密鑰生命週期

### 1.1 目前狀態（Plan48）

```
    [init]
      ↓
    [bound-to-closure]        ← captureHmacKey()
      ↓
    [consumed-for-shutdown-sign]
      ↓
    [cleared]                 ← binding.clear()
```

單一轉移；無輪轉。

### 1.2 未來狀態（輪轉設計）

```
    [init]
      ↓
    [bound-to-closure-N]
      ↓ ── [sign N events] ──
      ↓
    [rotate-trigger-fires]
      ↓
    [bound-to-closure-N+1]    ← 透過同一 captureHmacKey()
      ↓                          表面擷取新密鑰
    ...
      ↓
    [cleared-all]
```

輪轉新增 `[rotate-trigger-fires]` 轉移，不向其他子系統釋放控制即
swap closure 引用。

---

## 2. 輪轉觸發條件（已考量）

| 觸發   | 說明                                        | 取捨                             |
|--------|---------------------------------------------|----------------------------------|
| 牆時 TTL | 每 `N` 小時輪轉。                          | 簡單；可能過度輪轉。             |
| 使用次數 | 簽章 `K` 次後輪轉。                        | 限制洩漏窗口。                   |
| 外部訊號 | 由管理員指令 / signal 顯式觸發。            | Human-in-the-loop；最安全。      |
| 行程重啟 | 目前狀態 — 輪轉等於行程重啟。               | 最小；Plan48 預設。              |

未來實作 MAY 組合觸發條件（例：「牆時 OR 使用次數，取先到者」）。

---

## 3. 儲存需求

- **無明文持久化** 輪轉後密鑰於 `OPENSTARRY_SECURE_STORE` 設定之
  secure-store 根目錄外。
- 輪轉 MUST 使用與 `captureHmacKey()` 相同的 secure-store 整合契約 —
  輪轉設計不得引入第二條密鑰來源 code path。
- 封套儲存格式此處刻意不指定；實作 Plan 再選定（如 sealed box、KMS
  reference）。

---

## 4. 稽核需求

每次輪轉 MUST 透過 audit-sink 發布一筆 journal 條目
（`capability_denied` / `ws_connection_denied` 非適當通道；輪轉實作
時新增 `hmac_key_rotated` 事件型別）。audit-trail 檔案應記錄：

- `timestamp`
- `trigger`（`wallclock_ttl` | `usage_count` | `external` | `restart`）
- `reason`（短句 free-text）
- `previous_key_id`（短 hash，非密鑰本身）
- `new_key_id`（短 hash）

---

## 5. 威脅模型限定語（D-12b 重申）

行程內範圍對輪轉後密鑰亦適用，與初始密鑰相同。跨行程攻擊者明確
不在範疇。

---

## 6. 向前相容約束

- **MR-6 保持**：輪轉設計不得要求 Core 政策修改。所有輪轉邏輯
  駐留於 runner 層，與 `hmac-cleanup/` 同模組家族。
- **介面穩定**：`captureHmacKey()` 回傳型別（`HmacCleanupBinding`）
  必須維持為主要 handle；輪轉可於 binding 新增選用
  `rotate(): Promise<HmacCleanupBinding>` 方法，但 `sign` / `clear`
  不得破壞。

---

## 7. Plan48 明確排除

- 無 runtime 輪轉 code。
- 無 scheduler / timer 整合。
- 無 key-store connector / adapter。
- 無輪轉測試（現有 cleanup 測試僅涵蓋 C48-M3）。

Plan48 aggregate PASS 與未來輪轉工作相互獨立。

---

## 8. 交叉引用

- `./hmac-compliance.md` — C48-M3c 標準對應（ASVS + NIST）。
- `apps/runner/src/hmac-cleanup/README.md` — C48-M3 實作表面。
- `share/research_team_suggestion/cycle03-12/deliver/O5_Plan48_scope_and_engineering_spec.md` §4 — 約束設計規格範疇。
