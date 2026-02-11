# OpenStarry — Plan Dependencies & Definition of Done

Established: 2026-02-09

---

## 1. Plan 相依關係圖

```
Plan01 (MVP Alpha Foundation)           ✅ v0.1-alpha  2026-02-04
  ↓
Plan02 (Event-Driven & Safety)          ✅ v0.1.1      2026-02-05
  ↓
Plan03 (Quality & Skill Parser)         ✅ v0.2-alpha  2026-02-05
  ↓
Plan04 (IUI Interface & Guide Plugin)   ✅ v0.2-alpha  2026-02-06
  ↓
Plan05 (Multi-Channel UI & Listener)    ✅ v0.2-beta   2026-02-07
  ↓
  ├── Plan05.1 (Session Isolation)      ⬜ → v0.2.1-beta
  ├── Plan05.2 (HTTP SSE)              ⬜ → v0.2.1-beta   ← 可與 05.1 並行
  ├── Plan05.5 (其他)                   ⬜ → v0.2.1-beta   ← 可與 05.1 並行
  │     ↓
  │   Plan06 (MCP Protocol)             ⬜ → v0.3       ← blocked by Plan05.1
  │     ↓
  │   Plan07 (Runtime Sandbox)          ⬜ → v0.4       ← blocked by Plan06
  │     ↓
  │   Plan08-09 (TUI Dashboard)         ⬜ → v0.5
  │
  └── (Extended Mode: 新 plugin、Web UI、安全稽核)
```

### 關鍵路徑（Critical Path）

```
Plan05.1 → Plan06 → Plan07 → Plan08-09
```

Plan05.1 (Session Isolation) 是關鍵路徑上的第一個未完成項目。Plan06 (MCP) 被它阻擋。

### 可並行的工作

| 可並行 | 說明 |
|--------|------|
| Plan05.1 + Plan05.2 + Plan05.5 | 三者可合併為一輪迭代（Implementation Cycle 1） |
| Plan06 的預研 + Plan05.x 的實作 | researcher 可在 dev agents 實作 05.x 時預研 Plan06 |

---

## 2. Definition of Done（完成定義）

### 2.1 通用 DoD（所有 Plan 適用）

每個 Plan 必須滿足以下**全部條件**才能標記 ✅：

| # | 條件 | 驗證方式 |
|---|------|---------|
| 1 | `pnpm build` 通過（monorepo + plugins） | qa 在 agent_test 驗證 |
| 2 | `pnpm test` 全部通過，且測試數 ≥ 前一輪基線 | qa 報告中的 regression check |
| 3 | `pnpm test:purity` 通過 | qa 報告中的 purity check |
| 4 | Architecture_Spec 中所有項目已實作 | architect Code Review PASS |
| 5 | 五蘊合規（新組件正確歸類） | architect Code Review 確認 |
| 6 | pushInput 模式合規 | architect Code Review 確認 |
| 7 | 無已知安全漏洞 | architect Code Review 確認 |
| 8 | dev log 已寫入 | Coordinator 確認檔案存在 |
| 9 | QA Report 和 Code Review 均為 PASS | Phase 4 判定 |
| 10 | doc-keeper 已更新 Plan 打 ✅ + Iteration_Log | Phase 4 收斂 |
| 11 | Snapshot 已建立 | `scripts/snapshot.sh` 完成 |
| 12 | Lessons Learned 已記錄 | Phase 4 回顧 |

### 2.2 各 Plan 專屬驗收條件

#### Plan05.1: Session Isolation
- [ ] WebSocket 連線支援 session token / auth
- [ ] 多個 client 連線互相隔離（session 資料不交叉）
- [ ] 單一 agent instance 可服務多個 session
- [ ] Session 生命週期管理（建立、維持、銷毀）
- [ ] 新增測試覆蓋 session isolation 場景

#### Plan05.2: HTTP SSE
- [ ] HTTP Server-Sent Events transport plugin 可用
- [ ] SSE 連線可接收即時串流回應
- [ ] 與現有 HTTP webhook listener 共存不衝突
- [ ] 新增測試覆蓋 SSE 場景

#### Plan06: MCP Protocol (Model Context Protocol)
- [ ] 實作 MCP 規格中的 tool、resource、prompt 三大能力
- [ ] MCP server mode：OpenStarry 作為 MCP server 被外部 client 連接
- [ ] MCP client mode：OpenStarry 連接外部 MCP server 取得工具
- [ ] 符合 MCP 官方規格（researcher 預研確認）
- [ ] 新增測試覆蓋 MCP 場景

#### Plan07: Runtime Sandbox
- [ ] Plugin 執行環境隔離（vm 或 WASM）
- [ ] 資源限制（CPU、記憶體、檔案存取）
- [ ] Plugin 簽章驗證機制
- [ ] 新增測試覆蓋 sandbox escape 場景

#### Plan08-09: TUI Dashboard
- [ ] 終端機互動式 Dashboard（Ink 或 Blessed）
- [ ] 即時顯示 Agent 狀態、對話、工具呼叫
- [ ] 支援多 session 切換
- [ ] 新增測試覆蓋 TUI 場景

---

## 3. 迭代排程建議

| 迭代 | 涵蓋 Plans | 目標版本 | 前置條件 |
|------|-----------|---------|---------|
| Cycle 1 | Plan05.1 + 05.2 + 05.5 | v0.2.1-beta | Plan05 ✅（已滿足） |
| Cycle 2 | Plan06 (MCP) | v0.3 | Plan05.1 ✅ |
| Cycle 3 | Plan07 (Sandbox) | v0.4 | Plan06 ✅ |
| Cycle 4 | Plan08-09 (TUI) | v0.5 | Plan07 ✅（或可提前） |
| Extended | 新功能、安全稽核 | v1.0 | 所有 Plan ✅ |

---

## 4. 使用方式

- **Coordinator** 開始新迭代前，查閱本文件確認：
  1. 前置 Plan 是否已完成 ✅
  2. 該 Plan 的專屬驗收條件
  3. 是否有可並行的工作
- **architect** 設計 Spec 時，參考專屬驗收條件確保覆蓋
- **qa** 驗證時，逐項檢查通用 DoD + 專屬條件
- **doc-keeper** Plan 打 ✅ 時，確認 DoD 全部滿足
