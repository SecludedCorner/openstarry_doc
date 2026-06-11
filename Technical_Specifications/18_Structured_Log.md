# 結構化日誌 — Plan48 F-L3-SEC-2（Doc 78 候選）

**約束子項目**：C48-M1g（L1' 文件同步）。
**層級**：Runner（非 Core；MR-6 保持）。

自建、零外部相依的結構化日誌模組，用於 OpenStarry runner 的政策路徑
觀測性。詳見 `apps/runner/src/structured-log/README.md` 實作介面。

## JSON-line schema

每條輸出皆為一個 JSON 物件：

| 欄位 | 型別 | 備註 |
|------|------|------|
| `timestamp` | string | ISO-8601，於 emit 時 `isoTimestamp()` 產生。 |
| `level` | string | `DEBUG` / `INFO` / `WARN` / `ERROR` / `FATAL` 之一。 |
| `event` | string | 短 dotted 識別碼（例：`runner.boot`）。 |
| `payload` | any \| null | 安全序列化（circular / BigInt / 超長字串皆防護）。 |

## 設定

| 環境變數 | 預設 | 效果 |
|----------|------|------|
| `LOG_LEVEL` | `INFO` | 低於此級別之訊息被濾除。 |
| `OPENSTARRY_LOG_PATH` | *(stderr)* | JSONL 絕對路徑。 |
| `OPENSTARRY_LOG_BUFFER_MAX` | `1024` | Ring buffer 容量。 |

## 背壓契約（C48-M1d）

溢位時以 FIFO 丟棄最舊條目；並發出一次性 `W_AUDIT_OVERFLOW` WARN
直寫 sink，使此背壓訊號本身不會被丟棄。`resetOverflowReported()`
可重新啟用訊號。

## Shutdown flush（C48-M1e）

`registerStructuredLogShutdown()` 於 `SHUTDOWN_ORDER.FLUSH_STRUCTURED_LOG`
（200）掛入 hook。Registry 序列化等待各 hook，故結構化日誌於
audit-sink flush（300）及 HMAC clear（400）之前完全排空。

## 安全序列化邊角情境

`safe-stringify.ts` 涵蓋 HERACLITUS MRB-12 §12.2 JSON edge-case 矩陣：

| 情境 | 行為 |
|------|------|
| 循環引用 | 替換為 `"[Circular ~]"`。 |
| BigInt | 轉為 `"<digits>n"` 字串。 |
| 超長字串 | 於 16 KiB 截斷，尾綴 `"...[truncated]"`。 |
| Error 實例 | `{ name, message, stack }`。 |
| Symbol 互動 | 捕獲 → `{ "__serialization_error": ... }`。 |

## 與 Plan47 snapshot-hmac 日誌之關係

Plan47 `snapshot-hmac.ts` 未倚賴結構化日誌模組。

> **[2026-06-11 修復稽核更正]** 原文宣稱「Plan48 將 writer 引入 runner
> bootstrap、plugin-install flow 及 SIGTERM cascade」**與當時事實不符**：
> Plan48 交付後，structured-log / audit-sink 模組在任何 production 路徑
> 均無 import（僅 unit test 覆蓋）。實際接線於 v0.58.0-alpha 完成
> （`apps/runner/src/observability.ts`，由 start 指令啟用）：
> - structured-log：`OPENSTARRY_LOG_PATH` 設定時記錄 runner:started /
>   plugin:loaded / runner:shutdown 生命週期 JSONL（opt-in，預設關閉）
> - audit-sink：`OPENSTARRY_AUDIT=1` 時訂閱 `capability_denied`
>   （生產者 = Plan46 tool-filter-proxy）並落盤 audit-trail.jsonl
> - 關機 flush 經共用 registry（structured-log order 200 → audit-sink 300）
> - plugin-install flow 接線**未實作**（保留為未來工作）
> - hmac-cleanup（C48-M3）**仍為 library-only**，見該模組 README 誠實標記

既有 callers（例：checkpoint 指令）仍以 `console.*` 做 human-facing
CLI 輸出，不強制轉用 structured log。
