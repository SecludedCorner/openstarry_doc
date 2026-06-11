# HMAC 合規 — Plan48 E-5 MUST（行程內範圍）

**約束**：Plan48 子項目 `C48-M3c`。
**範圍限定語（R3 D-12b）**：OpenStarry 的 HMAC 簽章以 **行程內範圍**
（within-process scope）運作；威脅模型假設攻擊者 **無法讀取行程內
記憶體**。跨行程 HMAC 認證 **不在 Plan48 範疇** 之內。

---

## 1. 標準對應

### 1.1 OWASP ASVS V2.10.1 — 密鑰管理

> "驗證密鑰在記憶體中存留時間不超過必要時長。"

**OpenStarry 實作**：
- `hmac-cleanup/index.ts` `captureHmacKey()` 讀取 env var 一次後立即
  將其歸零（`process.env[name] = ''` 再 `delete`）。
- 密鑰僅存於 closure 引用 `closureKey`，綁定於 shutdown 簽章回呼；
  無模組級變數保留。
- Shutdown 時 `binding.clear()` 覆寫並清除 closure 引用，完成
  ASVS V2.10.1 義務。

### 1.2 NIST SP 800-57 Part 1 §8.2.2 — 密鑰銷毀

> "當密鑰不再需要時，應予銷毀。"

**OpenStarry 實作**：
- `registerHmacCleanupShutdown()` 在 `SHUTDOWN_ORDER.HMAC_CLEAR_AND_SIGN`
 （400）掛入 cleanup hook — 於 audit-sink flush 之後、process exit 前。
  Hook 在 `finally` 區塊無條件執行 `binding.clear()`，使
  `onBeforeClear` callback 失敗亦不會洩漏密鑰。
- `binding.clear()` 先覆寫字串再捨棄引用；JavaScript 對 primitive string
  無保證零填充，因此採用「覆寫 + 釋引用」雙重防守。
- C48-M3d W2-R13 runtime 驗證 shutdown 後 `binding.cleared === true`。

---

## 2. 威脅模型與非目標

### 2.1 納入範疇（Plan48）

- 攻擊者無法直接讀取 OpenStarry 行程記憶體。
- 攻擊者在 process exit 後可能取得檔案系統唯讀存取 → 設定的
  secure-store 路徑之外不得留存明文密鑰。
- 攻擊者可生成子程序 / 兄弟程序讀取環境變數 → env var 在擷取後
  立即歸零。

### 2.2 不納入範疇（Plan48 外）

- **行程內記憶體讀取攻擊者**。Closure 密鑰受 V8/Node runtime 一般
  行為約束；可進行 heap-dump 或 debugger-attach 的攻擊者屬於
  D-12b 宣告範圍外。
- **跨行程 HMAC 認證**。OpenStarry 的 HMAC 使用限於單一 runner
  行程簽章 / 驗章自身 checkpoint blob；多行程 HMAC handshake 非
  Plan47 / Plan48 delta，明確排除。
- **HMAC 密鑰輪轉 runtime**。詳見
  `./hmac-key-rotation-architecture.md` — Plan48 只交付設計；
  runtime 輪轉依 R3 D-17(a) 延至後續 cycle。

---

## 3. 子項目對應

| 子項目 | 標準條款 | 實作產物 |
|--------|----------|----------|
| C48-M3a | ASVS V2.10.1 + NIST §8.2.2 | `hmac-cleanup/index.ts` `captureHmacKey` + `clear` |
| C48-M3b | NIST §8.2（ephemeral sources） | `hmac-cleanup/policy.ts` `resolveSecureStoreRoot` + `isPathInsideSecureStore` |
| C48-M3c | —（本文件）                  | `docs/EN/hmac-compliance.md` + TW 譯本 |
| C48-M3d | NIST §8.2.2（銷毀證據）        | W2-R13 shutdown 劇本 + `binding.cleared` 斷言 |

---

## 4. 驗證備註

- **L1 grep**：`captureHmacKey`、`registerHmacCleanupShutdown`、
  `isPathInsideSecureStore` 皆於 `apps/runner/src/hmac-cleanup/` 下。
- **L1' 文件**：本文件 + `hmac-cleanup/README.md` + CHANGELOG 條目 +
  與 `hmac-key-rotation-architecture.md` 雙向 cross-reference。
- **L2 單元測試**：`apps/runner/__tests__/hmac-cleanup/*.test.ts`。
- **L3 整合測試**：`plan48-integration.test.ts` 驗證端到端
  capture → zero env → sign → shutdown → clear 序列。
- **W2-R13 runtime**：shutdown 劇本斷言 `binding.cleared` 且
  shutdown 後 heap 狀態不含明文密鑰（best-effort：dumped string pool
  grep）。
