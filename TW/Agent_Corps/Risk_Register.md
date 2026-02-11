# OpenStarry Agent Corps — Risk Register

Established: 2026-02-09

**評分標準**：機率 (L/M/H) × 影響 (L/M/H) = 風險等級 (Low/Medium/High/Critical)

---

## 技術風險

### R-T01: Agent Context Window 溢出
- **機率**: H（大型功能實作時幾乎必然）
- **影響**: M（agent 產出不完整或遺漏細節）
- **風險等級**: High
- **觸發場景**: dev-core 需讀取大量既有程式碼 + Spec + 寫入大量變更
- **緩解策略**:
  - Coordinator 將大功能拆成多個小任務分次派遣
  - Agent prompt 中明確指定只讀取必要的檔案
  - 複雜變更分成 Step 2a / 2b / 2c 多步驟
- **應對計畫**: 若 agent 產出不完整，Coordinator 檢視已完成部分，派遣新 agent session 繼續

### R-T02: pnpm install 或 build 在 agent_test 失敗
- **機率**: M
- **影響**: M（Phase 2.5/3 卡住）
- **風險等級**: Medium
- **觸發場景**: 依賴衝突、workspace protocol 問題、Windows 路徑問題
- **緩解策略**:
  - sync 腳本複製 pnpm-lock.yaml 確保依賴一致
  - Phase 2 exit criteria 要求 build pass，理論上 sync 後也應 pass
- **應對計畫**: 若 sync build 失敗但 agent_dev build 通過 → 檢查 sync 腳本是否遺漏檔案

### R-T03: Windows 環境相容性
- **機率**: M
- **影響**: M（腳本或工具不如預期運作）
- **風險等級**: Medium
- **觸發場景**: bash 腳本在 Windows Git Bash 中行為不同、路徑分隔符、symlink 不支援
- **緩解策略**:
  - 腳本使用 `$(cd ... && pwd)` 取得絕對路徑
  - 避免 symlink，使用 cp 複製
  - 腳本使用 `2>/dev/null || true` 處理非關鍵錯誤
- **應對計畫**: 手動執行失敗的步驟，記錄問題供後續修正腳本

### R-T04: Plugin 依賴 SDK 版本不同步
- **機率**: M
- **影響**: H（plugin build 失敗、runtime 型別不匹配）
- **風險等級**: High
- **觸發場景**: dev-core 改了 SDK interface，dev-plugin 使用舊版 SDK
- **緩解策略**:
  - Interface Freeze 機制確保 Spec 凍結
  - Phase 2 Step 2a 先完成 SDK → build → 再派 dev-plugin
  - workspace protocol (`workspace:*`) 自動連結本地版本
- **應對計畫**: Spec Addendum 流程處理必要的 interface 變更

---

## 流程風險

### R-P01: Coordinator 主 Session 崩潰
- **機率**: M（長時間作業、token 用盡）
- **影響**: H（迭代中斷，狀態可能不一致）
- **風險等級**: High
- **緩解策略**:
  - 所有狀態透過檔案持久化（報告 = 隱式狀態標記）
  - SOP 8.3 定義了狀態判斷方法
  - 每個 Phase 的產出都是獨立檔案，不依賴 session 記憶
- **應對計畫**: 新 session 讀取 SOP + 檢查 cycle 目錄 → 判斷 Phase → 繼續

### R-P02: Rework 死循環
- **機率**: L
- **影響**: H（迭代永遠完不成）
- **風險等級**: Medium
- **觸發場景**: 設計缺陷導致反覆 FAIL，每次 rework 引入新問題
- **緩解策略**:
  - Rework 上限 2 次，超過強制升級到 User
  - 每次 rework 前重建 baseline
- **應對計畫**: User 裁決是否放棄此 Plan 或大幅重新設計

### R-P03: Agent 產出品質不一致
- **機率**: M
- **影響**: M（垃圾進垃圾出，Phase 3 才發現）
- **觸發場景**: Agent prompt 不夠明確、agent 誤解 Spec
- **緩解策略**:
  - Communication Protocol 定義必要的 prompt 欄位
  - Phase 3 雙重驗證（qa + architect）
  - dev log 記錄所有變更供 review
- **應對計畫**: Rework 流程處理

### R-P04: 報告或文件內容衝突
- **機率**: L
- **影響**: L（混淆但不致命）
- **觸發場景**: 多個 agent 同時寫入相鄰檔案、doc-keeper 和 Coordinator 同時更新 Iteration_Log
- **緩解策略**:
  - 每個 agent 有明確的寫入目錄，不重疊
  - doc-keeper 是 Iteration_Log 的唯一寫入者
- **應對計畫**: Coordinator 發現衝突時手動合併

---

## 外部風險

### R-E01: Claude Pro Token 額度用盡
- **機率**: M（大型迭代可能消耗大量 token）
- **影響**: H（整個迭代被迫暫停）
- **緩解策略**:
  - doc-keeper 使用 haiku（低成本）
  - 只在需要時派遣 agent，避免不必要的調用
  - 大功能拆成多輪小迭代
- **應對計畫**: 暫停迭代，等待額度恢復，從 SOP 8.3 狀態判斷恢復

### R-E02: Reference 專案版本過時
- **機率**: L
- **影響**: L（researcher 報告基於舊版資訊）
- **觸發場景**: share/ref/ 中的 openclaw/opencode/openoctopus 不再更新
- **緩解策略**: researcher 在報告中標註參考版本和日期
- **應對計畫**: 需要時手動更新 share/ref/

---

## 風險總覽

| ID | 風險 | 等級 | 狀態 |
|----|------|:----:|:----:|
| R-T01 | Context Window 溢出 | High | 監控中 |
| R-T02 | agent_test build 失敗 | Medium | 監控中 |
| R-T03 | Windows 相容性 | Medium | 監控中 |
| R-T04 | SDK 版本不同步 | High | 已緩解（Interface Freeze） |
| R-P01 | 主 Session 崩潰 | High | 已緩解（狀態持久化） |
| R-P02 | Rework 死循環 | Medium | 已緩解（上限 2 次） |
| R-P03 | Agent 產出品質不一致 | Medium | 已緩解（Communication Protocol） |
| R-P04 | 報告內容衝突 | Low | 已緩解（寫入目錄分離） |
| R-E01 | Token 額度用盡 | Medium | 監控中 |
| R-E02 | Reference 專案過時 | Low | 接受 |
