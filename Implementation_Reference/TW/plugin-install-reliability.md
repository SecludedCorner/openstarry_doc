# Plugin 安裝可靠性

**狀態**：Plan49 C49-M1 — 根因確認並修復。
**生效版本**：v0.49.0-alpha（2026-04-24）。

## 1. 症狀

Plan49 之前，當 vitest 並行執行
`pnpm test apps/runner/__tests__/commands/plugin-install.test.ts` 與
`pnpm test apps/runner/__tests__/utils/plugin-installer.test.ts`
時（vitest 預設每個 test file 跑一個 thread），Windows 上偶發檔案系統錯誤
（EBUSY / EPERM / EEXIST）。

## 2. 根因

`plugin-installer.ts` 與 `plugin-lock.ts` 都 export **module-level 常量**
指向單一 user-global 檔案系統位置：

- `INSTALLED_DIR` → `~/.openstarry/plugins/installed/`
- `LOCK_FILE_PATH` → `~/.openstarry/plugins/lock.json`

vitest 並行跨 test file 時，兩個 thread 同時呼叫 `installPlugin()` 安裝
同一個 plugin，在同一個安裝目標目錄上競爭：

- Thread A：`rm(targetDir, …)` → `cp(packageDir, targetDir, …)`
- Thread B：`rm(targetDir, …)` → `cp(packageDir, targetDir, …)`

Windows 的 `rm` / `cp` 組合不是 race-safe — 一個 thread 的 open handle 會
擋住另一個 thread 的 delete 或 copy，產生 EBUSY / EPERM 偶發錯誤。
`syncPlugin()` 路徑（`plugin-scanner.ts`）也會命中同一個目標目錄競爭。

次要成因：npm-pack fallback tempdir 名稱是
`openstarry-install-${Date.now()}`（毫秒解析度）；兩個並行 thread 若在
同一毫秒進入 npm 路徑會取得相同 tempdir 路徑。

## 3. 修復（Plan49 C49-M1b）

### 3.1 測試可注入的安裝目錄

`InstallOptions` 新增 `installedDir?: string` 欄位。`installPlugin()`、
`uninstallPlugin()`、`installAll()` 現在依下列順序解析有效安裝目錄：

1. `options.installedDir`（若提供）。
2. `process.env.OPENSTARRY_INSTALL_DIR`（若設定）。
3. `DEFAULT_INSTALLED_DIR`（module-level 常量；production 預設行為不變）。

env var 的存在是為了讓 CLI 層的測試能隔離狀態，不必把 `installedDir`
option 穿過每個 CLI 呼叫。

`lockPath` 原本就支援 per-call 覆寫；同一個 env-var-then-module-default
fallback 模式新增 `OPENSTARRY_LOCK_PATH`。

### 3.2 碰撞免疫的 tmpdir 命名

npm-pack fallback 現在將 tempdir 命名為
`openstarry-install-${process.pid}-${Date.now()}-${random6}`。PID 單獨
消除跨 process 碰撞；隨機後綴涵蓋 process 內同毫秒的並行呼叫。

### 3.3 測試變更

- `apps/runner/__tests__/utils/plugin-installer.test.ts` 在 `beforeEach`
  建立 PID-scoped `tempDir`，每次呼叫都傳
  `{ lockPath, installedDir }`。
- `apps/runner/__tests__/commands/plugin-install.test.ts` 在 `beforeAll`
  設定 `OPENSTARRY_INSTALL_DIR` 與 `OPENSTARRY_LOCK_PATH`，`afterAll`
  還原，隔離 CLI 層的間接安裝呼叫。

## 4. 假設消除（Plan49 §2R1 §2.2.1）

Plan49 研究規格列舉八個假設（H1–H8）。觀察證據支持 H1（並行測試共享 FS
競爭）為根因，消除其他：

| ID | 假設 | 判定 |
|----|------|------|
| H1 | 並行 test file 共享 `INSTALLED_DIR` / `LOCK_FILE_PATH` | **確認 — 根因** |
| H2 | tmpdir 命名 `Date.now()` 碰撞（次要） | 貢獻因素；與 H1 一併修復 |
| H3 | `require.resolve` 子進程 timeout 不穩 | 未觀察到；timeout 僅在慢機器觸發 |
| H4 | `npm pack` retry flake | 重現時未觀察（用 workspace path） |
| H5 | `plugin-lock.ts` 非原子寫入 | 未觀察；寫入為整檔 write，Unix 有 atomic rename，Windows 上每測試序列化 |
| H6 | 單一 test file 內的 worker-thread race | 不適用（測試同步 await） |
| H7 | 檔案系統 journaling 延遲 | 在隔離目錄修復下無法重現 |
| H8 | npm hoist vs nested-import | 未觀察；Plan49 未調整 import 結構 |

## 5. CI 與 plugin 作者操作注意

- **Production 呼叫方**：無 API 破壞。`installPlugin` / `uninstallPlugin` /
  `installAll` 在未提供 option 或 env var 時，仍預設
  `~/.openstarry/plugins/installed/`。
- **測試作者**：任何會呼叫 `installPlugin` / `installAll` 的測試，必須傳
  `installedDir` + `lockPath`（或設 `OPENSTARRY_INSTALL_DIR` +
  `OPENSTARRY_LOCK_PATH`）。不要在並行 test file 間共享這些路徑。
- **CI flake gating（C49-M1d SHOULD）** — 由 Plan49 Follow-on B 交付：
  執行 `pnpm test:flake-gate`（預設 50 iter，可配置：
  `bash scripts/flake-gate.sh <N>`）。腳本每 iter 跑兩個 plugin-install
  測試檔；第一次失敗即中止並印出末 30 行診斷；零容忍語意。初始本地
  Windows smoke：5/5 iter × 17 tests PASS（Task #65）+ 2/2 iter
  flake-gate smoke（Task #68）。完整 50-iter gate 應在專案接入 CI pipeline
  後 wiring（目前 repo 無 CI）。

## 6. Rule #74 L1' 五項子檢

| # | 子檢 | 證據 |
|---|------|------|
| i | Code 檔案存在 | `apps/runner/src/utils/plugin-installer.ts`（`installedDir` option + env-var fallback + 碰撞免疫 tmpdir） |
| ii | Test 檔案存在 | `apps/runner/__tests__/utils/plugin-installer.test.ts`（`C49-M1c concurrent install isolation (regression)` 區塊 + 更新既有案例）+ `apps/runner/__tests__/commands/plugin-install.test.ts`（env-var 隔離） |
| iii | Doc 存在 | 本檔 + `docs/EN/plugin-install-reliability.md` |
| iv | CHANGELOG 引用 | `CHANGELOG.md` v0.49.0-alpha Plan49 C49-M1 條目 |
| v | Cross-ref 雙向 | `plugin-installer.ts` option JSDoc 引 Plan49 C49-M1；本檔交叉引用 `plugin-installer.ts`、`plugin-lock.ts`、regression test |

## 7. 參考

- Plan49 engineering spec（完整）：`share/research_team_suggestion/cycle03-13/deliver/O2_plan49_engineering_spec.md`
- Plan49 dev spec（精簡）：`share/research_team_suggestion/cycle03-13/todo/Plan49_dev_spec.md`
- Plan49 C49-M1 子項：M1a 根因、M1b 修復、M1c regression、M1d CI flake gating、M1e 本檔
