# OpenStarry Agent Corps — Roles & Standard Operating Procedures

Established: 2026-02-09
Last Updated: 2026-02-09 (PMP Audit v2)

---

## 1. Agent Roster

| Agent | Role | Model | Working Directory | Output Directory |
|-------|------|-------|-------------------|------------------|
| `architect` | Architecture guardian, design specs, code review, security audit | sonnet | share/openstarry_doc/, agent_dev/ (read-only) | share/test/reports/arch_reviews/ |
| `dev-core` | Core monorepo developer (sdk, core, shared, runner) | sonnet | agent_dev/openstarry/ | share/test/reports/dev_logs/ |
| `dev-plugin` | Plugin ecosystem developer | sonnet | agent_dev/openstarry_plugin/ | share/test/reports/dev_logs/ |
| `qa` | Quality assurance — build, test, purity verification | sonnet | agent_test/ | share/test/reports/qa_results/ |
| `doc-keeper` | Documentation, decision persistence, plan tracking | haiku | share/openstarry_doc/ | share/openstarry_doc/ |
| `researcher` | Technical pre-research, reference project analysis | sonnet | share/ref/ | share/test/reports/research/ |
| **Coordinator** | Main session — orchestration, sync, convergence, escalation | opus | all | share/test/reports/sys_summary/ |

---

## 2. RACI Matrix

**R** = Responsible (做), **A** = Accountable (負最終責任), **C** = Consulted (被諮詢), **I** = Informed (被通知)

| Phase | Coordinator | architect | dev-core | dev-plugin | qa | doc-keeper | researcher | User |
|-------|:-----------:|:---------:|:--------:|:----------:|:--:|:----------:|:----------:|:----:|
| **Phase 0: Planning** | **R/A** | C | I | I | I | **R** (記錄) | **R** (預研) | **A** (核准) |
| **Phase 1: Design** | I | **R/A** | C | C | I | **R** (記錄設計決策) | C (提供研究) | I |
| **Phase 1.5: Baseline** | **R/A** | I | I | I | I | I | I | I |
| **Phase 2: Implement** | I | C (澄清 spec) | **R/A** (core) | **R/A** (plugin) | I | I | R (預研下輪) | I |
| **Phase 2.5: Sync** | **R/A** | I | I | I | I | I | I | I |
| **Phase 3: Verify** | I | **R** (code review) | I | I | **R/A** (測試) | I | I | I |
| **Phase 4: Converge** | **R/A** | I | I | I | I | **R** (更新文件) | I | **A** (裁決 PASS/FAIL) |
| **Rework** | **R** (分配) | C/R (若設計問題) | R (若 core 問題) | R (若 plugin 問題) | I | I | I | I |
| **Escalation** | **R** (提報) | C | C | C | C | I | I | **A** (裁決) |

---

## 3. 標準迭代周期

### 命名規範

每輪迭代的 ID 格式：`{YYYYMMDD}_cycle{N}`

例：`20260210_cycle1`、`20260210_cycle2`（同天第二輪）

所有該輪的報告放在對應的 cycle 子目錄：
```
share/test/reports/arch_reviews/20260210_cycle1/Architecture_Spec_Plan05.1.md
share/test/reports/dev_logs/20260210_cycle1/dev-core_Plan05.1.md
share/test/reports/qa_results/20260210_cycle1/QA_Report_Plan05.1.md
share/test/reports/research/20260210_cycle1/Research_SessionIsolation.md
share/test/reports/sys_summary/20260210_cycle1/Summary.md
```

---

### Phase 0: Planning

**觸發條件**：User 下達指令，或上一輪 Phase 4 PASS 後 Coordinator 提議下一輪

**參與者**：Coordinator (R/A), researcher (R), doc-keeper (R), User (A)

**步驟**：
1. **Coordinator** 接收 User 指令，識別對應的 Implementation Plan
2. **Coordinator** 建立本輪 cycle 目錄（所有 reports 子目錄下）
3. **researcher** 預研該 Plan 的技術方案（Shift-Left），產出研究報告
4. **doc-keeper** 將本輪迭代計畫記錄到 `share/openstarry_doc/Agent_Corps/Iteration_Log.md`
5. **Coordinator** 確認研究報告已就緒，通知 architect 進入 Phase 1

**交付物**：
| 產出 | 路徑 | 負責人 |
|------|------|--------|
| 研究報告 | `share/test/reports/research/{cycle_id}/Research_{Topic}.md` | researcher |
| 迭代計畫記錄 | `share/openstarry_doc/Agent_Corps/Iteration_Log.md` (追加) | doc-keeper |

**Exit Criteria**：
- [ ] 研究報告已存在且內容完整
- [ ] 迭代計畫已記錄
- [ ] Coordinator 確認可進入 Phase 1

---

### Phase 1: Design

**Entry Criteria**：Phase 0 Exit Criteria 全部滿足

**參與者**：architect (R/A), doc-keeper (R), researcher (C)

**步驟**：
1. **architect** 讀取：
   - Implementation Plan：`share/openstarry_doc/Implementation_Plans/Plan{XX}_{Name}.md`
   - 研究報告：`share/test/reports/research/{cycle_id}/Research_{Topic}.md`
   - 現有架構程式碼：`agent_dev/`（唯讀）
2. **architect** 產出 Architecture_Spec，必須包含：
   - **凍結的 Interface 定義**（TypeScript types）— dev-core 和 dev-plugin 共同依據
   - 技術約束與設計決策理由
   - Five Aggregates 歸屬
   - 安全考量
3. **doc-keeper** 記錄設計決策理由到 `share/openstarry_doc/Agent_Corps/Iteration_Log.md`

**交付物**：
| 產出 | 路徑 | 負責人 |
|------|------|--------|
| Architecture_Spec | `share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md` | architect |
| 設計決策記錄 | `share/openstarry_doc/Agent_Corps/Iteration_Log.md` (追加) | doc-keeper |

**Exit Criteria**：
- [ ] Architecture_Spec 已存在
- [ ] Spec 包含凍結的 Interface 定義
- [ ] 設計決策已記錄
- [ ] Coordinator 確認 Spec 完整，可進入 Phase 2

**重要**：Architecture_Spec 中的 Interface 定義一經發布即**凍結**。Phase 2 期間若需修改，必須走升級流程（見第 6 節）。

---

### Phase 1.5: Baseline（安全網）

**Entry Criteria**：Phase 1 Exit Criteria 全部滿足

**參與者**：Coordinator (R/A)

**目的**：在 Phase 2 實作開始前，備份當前 agent_dev 狀態。若 Phase 2 失敗無法修復，可快速 rollback。

**執行**：`bash scripts/baseline.sh {cycle_id}`

**產出**：`share/openstarry_code_iteration/{cycle_id}_baseline/`

**恢復指令**：`bash scripts/restore.sh {cycle_id} baseline`

---

### Phase 2: Implementation

**Entry Criteria**：Phase 1.5 Baseline 已建立

**參與者**：dev-core (R/A), dev-plugin (R/A), architect (C), researcher (R 預研下輪)

**執行順序**：

```
Step 2a: dev-core 實作 SDK Interface 變更（若有）
         → pnpm build 驗證
         ↓
Step 2b: dev-core (剩餘實作) + dev-plugin (plugin 實作) 並行
         → 各自 pnpm build 驗證
```

若 Architecture_Spec 未變更 SDK Interface，則 Step 2a 跳過，dev-core 和 dev-plugin 直接並行。

**步驟（dev-core）**：
1. 讀取 Spec：`share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md`
2. 若有 Interface 變更：先修改 `packages/sdk/`，執行 `pnpm build` 確認編譯
3. 實作 `packages/core/`、`packages/shared/`、`apps/runner/` 相關變更
4. 執行 `pnpm build` 驗證
5. 寫 dev log

**步驟（dev-plugin）**：
1. 讀取 Spec（同上路徑）
2. **等待 Step 2a 完成**（若有 SDK Interface 變更）
3. 實作 plugin 變更
4. 執行 `pnpm build` 驗證
5. 寫 dev log

**步驟（researcher，並行）**：
- 預研下一輪迭代的技術方案（若已知）

**交付物**：
| 產出 | 路徑 | 負責人 |
|------|------|--------|
| dev-core 日誌 | `share/test/reports/dev_logs/{cycle_id}/dev-core_{PlanName}.md` | dev-core |
| dev-plugin 日誌 | `share/test/reports/dev_logs/{cycle_id}/dev-plugin_{PlanName}.md` | dev-plugin |

**Exit Criteria**：
- [ ] dev-core `pnpm build` PASS
- [ ] dev-plugin `pnpm build` PASS（若有 plugin 變更）
- [ ] dev log 已寫入
- [ ] Coordinator 確認 build 均通過，可進入 Phase 2.5

---

### Phase 2.5: Sync to Test Environment

**Entry Criteria**：Phase 2 Exit Criteria 全部滿足

**參與者**：Coordinator (R/A)

**目的**：將 `agent_dev/` 的最新程式碼同步到 `agent_test/`，確保 QA 測試的是最新版本。

**快速執行**：`bash scripts/sync-to-test.sh`（從 openstarry_eco 根目錄執行）

**Sync SOP（腳本內容）**：

```bash
# Step 1: 清除 agent_test 舊的原始碼（保留 node_modules 加速）
rm -rf agent_test/openstarry/packages agent_test/openstarry/apps
rm -rf agent_test/openstarry/dist agent_test/openstarry/.turbo

# Step 2: 複製原始碼（排除 node_modules, dist, .turbo）
cp -r agent_dev/openstarry/packages agent_test/openstarry/packages
cp -r agent_dev/openstarry/apps agent_test/openstarry/apps

# 複製根層設定檔（如有變更）
cp agent_dev/openstarry/package.json agent_test/openstarry/package.json
cp agent_dev/openstarry/tsconfig*.json agent_test/openstarry/
cp agent_dev/openstarry/vitest*.* agent_test/openstarry/ 2>/dev/null
cp agent_dev/openstarry/pnpm-workspace.yaml agent_test/openstarry/ 2>/dev/null

# Step 3: 同步 plugin（同樣策略）
# 逐一複製每個 plugin 的 src, package.json, tsconfig
for dir in agent_dev/openstarry_plugin/*/; do
  plugin=$(basename "$dir")
  rm -rf "agent_test/openstarry_plugin/$plugin/src" "agent_test/openstarry_plugin/$plugin/dist"
  cp -r "agent_dev/openstarry_plugin/$plugin/src" "agent_test/openstarry_plugin/$plugin/src"
  cp "agent_dev/openstarry_plugin/$plugin/package.json" "agent_test/openstarry_plugin/$plugin/package.json"
  cp "agent_dev/openstarry_plugin/$plugin/tsconfig.json" "agent_test/openstarry_plugin/$plugin/tsconfig.json" 2>/dev/null
done
cp agent_dev/openstarry_plugin/package.json agent_test/openstarry_plugin/package.json 2>/dev/null
cp agent_dev/openstarry_plugin/pnpm-workspace.yaml agent_test/openstarry_plugin/pnpm-workspace.yaml 2>/dev/null

# Step 4: 安裝依賴（處理新增/變更的 dependencies）
cd agent_test/openstarry && pnpm install
cd agent_test/openstarry_plugin && pnpm install

# Step 5: 驗證基本 build
cd agent_test/openstarry && pnpm build
```

**Exit Criteria**：
- [ ] agent_test 原始碼已同步
- [ ] `pnpm install` 成功
- [ ] `pnpm build` 成功
- [ ] Coordinator 確認可進入 Phase 3

**失敗處理**：若 sync 後 build 失敗，問題在 Phase 2，退回 Phase 2 修正（不進入 Phase 3）。

---

### Phase 3: Verification

**Entry Criteria**：Phase 2.5 Exit Criteria 全部滿足

**參與者**：qa (R/A), architect (R)

**可並行**：qa 和 architect 可同時進行，互不依賴。

**步驟（qa）**：
1. 在 `agent_test/openstarry/` 執行：
   - `pnpm build` — Build 驗證
   - `pnpm test` — 單元測試（基線：82 tests）
   - `pnpm test:purity` — 微核心純度檢查
2. 在 `agent_test/openstarry_plugin/` 執行：
   - `pnpm build` — Plugin build 驗證
   - `pnpm test` — Plugin 測試（若存在）
3. 產出 QA Report

**步驟（architect）**：
1. 讀取 Architecture_Spec：`share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md`
2. 讀取實作程式碼：`agent_dev/`（唯讀）
3. 逐項驗證：
   - Interface 實作是否符合凍結的 Spec
   - Five Aggregates 合規
   - 微核心純度
   - pushInput pattern
   - 安全漏洞
4. 產出 Code Review Report（PASS / FAIL / CONDITIONAL）

**交付物**：
| 產出 | 路徑 | 負責人 |
|------|------|--------|
| QA Report | `share/test/reports/qa_results/{cycle_id}/QA_Report_{PlanName}.md` | qa |
| Code Review | `share/test/reports/arch_reviews/{cycle_id}/Code_Review_{PlanName}.md` | architect |

**Exit Criteria**：
- [ ] QA Report 已存在
- [ ] Code Review Report 已存在
- [ ] Coordinator 已讀取兩份報告，進入 Phase 4

---

### Phase 4: Convergence

**Entry Criteria**：Phase 3 Exit Criteria 全部滿足

**參與者**：Coordinator (R/A), doc-keeper (R), User (A)

**步驟**：
1. **Coordinator** 讀取 QA Report 和 Code Review Report
2. **Coordinator** 彙整為 Summary Report
3. **Coordinator** 判定：

   **→ 若全部 PASS：**
   - Snapshot：`bash scripts/snapshot.sh {cycle_id}`（排除 node_modules/dist）
   - **doc-keeper** 更新：
     - Implementation Plan 中完成項打 ✅
     - Iteration_Log.md 追加本輪結果
   - **Coordinator** 向 User 報告結果，詢問下一步

   **→ 若任一 FAIL：** 進入 Rework 流程（見第 4 節）

**交付物**：
| 產出 | 路徑 | 負責人 |
|------|------|--------|
| Summary Report | `share/test/reports/sys_summary/{cycle_id}/Summary.md` | Coordinator |
| 更新的 Iteration Log | `share/openstarry_doc/Agent_Corps/Iteration_Log.md` | doc-keeper |
| 更新的 Plan（✅） | `share/openstarry_doc/Implementation_Plans/` | doc-keeper |

---

## 4. Rework 流程（Phase 4 FAIL）

### 4.1 問題分級

| 級別 | 描述 | 退回到 | 範例 |
|------|------|--------|------|
| **Code Fix** | 實作 bug、測試失敗、build 錯誤 | Phase 2（僅修正部分） | 型別錯誤、測試斷言失敗 |
| **Design Fix** | Spec 本身有缺陷或不完整 | Phase 1（architect 修訂 Spec） | Interface 設計不合理、遺漏邊界情況 |
| **Plan Fix** | Plan 需求本身有問題 | Phase 0（重新規劃） | 技術方案不可行、需求矛盾 |

### 4.2 Rework SOP

1. **Coordinator** 讀取 QA Report 和 Code Review Report 中的 FAIL 項目
2. **Coordinator** 對每個 FAIL 項目分級（Code Fix / Design Fix / Plan Fix）
3. **Coordinator** 產出 Rework Task，寫入 Summary Report：
   ```
   ## Rework Tasks
   - [ ] [FAIL-1] (Code Fix) 描述 → 指派給 dev-core/dev-plugin
   - [ ] [FAIL-2] (Design Fix) 描述 → 指派給 architect
   ```
4. **Coordinator** 重建 Baseline：`bash scripts/baseline.sh {cycle_id}_rework{N}`
   （保留 rework 前的狀態，以便二次 rollback）
5. 依分級退回對應 Phase，**只有 FAIL 的部分需要重做**
6. 修正完成後，從修正 Phase 的下一個 Phase 重新往下走（Phase 2.5 Sync 必須重跑）

### 4.3 Rework 上限

- 同一輪迭代最多 **2 次 Rework**
- 超過 2 次 → **升級到 User** 裁決（見第 6 節）

---

## 5. Phase 2 並行策略：Interface 凍結機制

### 5.1 原則

Architecture_Spec 中定義的 **TypeScript Interface** 一經 Phase 1 完成即**凍結**。dev-core 和 dev-plugin 共同依據此凍結 Interface 開發。

### 5.2 執行順序

```
Architecture_Spec 是否包含 SDK Interface 變更？
│
├── 是 → dev-core 先行實作 SDK Interface（Step 2a）
│        → pnpm build 通過後
│        → dev-core (剩餘) + dev-plugin 並行（Step 2b）
│
└── 否 → dev-core + dev-plugin 直接並行
```

### 5.3 凍結期間的例外處理

若 dev-core 在實作中發現 Interface 必須修改：
1. dev-core **暫停實作**
2. dev-core 在 dev log 中記錄問題
3. **Coordinator** 通知 architect
4. architect 評估 → 發布 **Spec Addendum**（追加修訂）到同一 cycle 目錄
5. dev-plugin 讀取 Addendum 後調整
6. 若影響範圍過大 → 升級到 User

---

## 6. 升級與爭議處理

### 6.1 升級條件

| 情境 | 升級路徑 |
|------|---------|
| Agent 之間對 Spec 理解有分歧 | → Coordinator 仲裁 |
| Coordinator 無法仲裁（設計方向問題） | → User 裁決 |
| Rework 超過 2 次 | → User 裁決 |
| Phase 2 發現 Interface 需大幅修改 | → User 裁決 |
| 技術方案不可行（researcher 發現） | → User 裁決 |

### 6.2 升級流程

1. 發現方在報告中明確標記 `⚠️ ESCALATION NEEDED`
2. **Coordinator** 收集各方意見，整理為決策摘要
3. **Coordinator** 向 User 提出：
   - 問題描述
   - 各方立場
   - Coordinator 建議（若有）
4. **User** 裁決
5. **doc-keeper** 記錄裁決到 Iteration_Log.md

---

## 7. doc-keeper 全程參與時間表

| Phase | doc-keeper 職責 |
|-------|----------------|
| Phase 0 | 記錄：本輪迭代目標、涉及的 Plan、研究方向 |
| Phase 1 | 記錄：architect 的設計決策理由、Interface 凍結內容 |
| Phase 1.5 | 無主動任務（Coordinator 建 baseline） |
| Phase 2 | 待命：收到 Spec Addendum 時記錄變更 |
| Phase 3 | 待命：無主動任務 |
| Phase 4 PASS | 更新 Plan ✅、寫 Iteration 結果、更新 Roadmap |
| Phase 4 FAIL | 記錄 FAIL 原因和 Rework 決策 |

---

## 8. Communication Protocol（溝通協議）

Coordinator 派遣 agent 時，prompt 必須包含以下資訊，避免 agent 猜測或工作在錯誤的上下文。

### 8.1 必要欄位（所有派遣）

每次派遣 agent 的 prompt **必須包含**：

| 欄位 | 說明 | 範例 |
|------|------|------|
| `cycle_id` | 本輪迭代 ID | `20260210_cycle1` |
| `plan` | 目標 Plan 名稱 | `Plan05.1 Session Isolation` |
| `task` | 具體要做什麼（一句話） | 「預研 Session Isolation 的技術方案」 |
| `output_path` | 報告/產出要寫到哪裡 | `share/test/reports/research/20260210_cycle1/` |

### 8.2 Phase 專屬欄位

| Phase | Agent | 額外必要欄位 |
|-------|-------|-------------|
| Phase 0 | researcher | `research_topic`, `reference_projects`（要掃描 share/ref/ 的哪些目錄） |
| Phase 0 | doc-keeper | `iteration_goals`（本輪目標摘要） |
| Phase 1 | architect | `plan_path`（Plan 檔案路徑）, `research_report_path`（研究報告路徑） |
| Phase 1 | doc-keeper | `spec_summary`（Spec 的關鍵設計決策摘要） |
| Phase 2a | dev-core | `spec_path`, 明確指示「僅實作 SDK Interface 變更」 |
| Phase 2b | dev-core | `spec_path`, 明確指示「SDK Interface 已完成，實作剩餘部分」 |
| Phase 2 | dev-plugin | `spec_path`, 若有 SDK 變更需指示「SDK Interface 已就緒」 |
| Phase 3 | qa | `cycle_id`（qa 不需要額外路徑，它在 agent_test/ 工作） |
| Phase 3 | architect | `spec_path`（Architecture_Spec 路徑，用於 code review 比對） |
| Phase 4 | doc-keeper | `qa_result`（PASS/FAIL）, `review_result`（PASS/FAIL）, `lessons_learned`（本輪回顧） |

### 8.3 Prompt 範本

**Phase 0 — researcher:**
```
Cycle: 20260210_cycle1
Plan: Plan05.1 Session Isolation
Task: 預研 Session Isolation 技術方案
Reference: share/ref/opencode/, share/ref/openclaw/, share/ref/openclaw研究/
Output: share/test/reports/research/20260210_cycle1/Research_SessionIsolation.md
重點研究：WebSocket auth 機制、multi-instance session 管理、session 生命週期
```

**Phase 2a — dev-core (SDK only):**
```
Cycle: 20260210_cycle1
Plan: Plan05.1 Session Isolation
Spec: share/test/reports/arch_reviews/20260210_cycle1/Architecture_Spec_Plan05.1.md
Task: 僅實作 Architecture_Spec 中的 SDK Interface 變更（packages/sdk/）。
      完成後執行 pnpm build 驗證。不要實作 core/shared/runner 的變更。
Output: share/test/reports/dev_logs/20260210_cycle1/dev-core_Plan05.1_step2a.md
```

**Phase 2b — dev-core (remaining):**
```
Cycle: 20260210_cycle1
Plan: Plan05.1 Session Isolation
Spec: share/test/reports/arch_reviews/20260210_cycle1/Architecture_Spec_Plan05.1.md
Task: SDK Interface 已在 Step 2a 完成。請實作 Spec 中 packages/core、packages/shared、
      apps/runner 的剩餘變更。完成後執行 pnpm build 驗證。
Output: share/test/reports/dev_logs/20260210_cycle1/dev-core_Plan05.1.md
```

---

## 9. Lessons Learned（經驗回顧）

### 9.1 時機

每輪迭代 **Phase 4 結束後**（無論 PASS 或 FAIL），Coordinator 執行回顧。

### 9.2 回顧流程

1. **Coordinator** 回顧本輪迭代，回答以下問題：
   - 什麼做得好？（流程、協作、品質）
   - 什麼做得不好？（卡住的地方、rework 原因、溝通問題）
   - 下次可以怎麼改善？（流程調整、prompt 優化、新增檢查項）
2. **Coordinator** 將回顧寫入 Summary Report 的 `## Lessons Learned` 段落
3. **doc-keeper** 將可持久化的改善項追加到 `share/openstarry_doc/Agent_Corps/Lessons_Learned.md`
4. 若發現需要修改 SOP/Agent 定義/Checklist → Coordinator 在下一輪 Phase 0 執行

### 9.3 Lessons Learned 格式

追加到 `share/openstarry_doc/Agent_Corps/Lessons_Learned.md`：
```
## {cycle_id}

### What went well
- ...

### What went wrong
- ...

### Action items
- [ ] [改善項描述] → 影響的文件/流程
```

### 9.4 風險登記冊更新

若回顧中發現新風險或已知風險的機率/影響需調整：
- **Coordinator** 更新 `share/openstarry_doc/Agent_Corps/Risk_Register.md`

---

## 10. 故障恢復

### 10.1 恢復腳本

| 腳本 | 用途 | 指令 |
|------|------|------|
| `scripts/baseline.sh` | Phase 2 前建立安全網 | `bash scripts/baseline.sh {cycle_id}` |
| `scripts/restore.sh` | 從 baseline 或 snapshot 恢復 agent_dev | `bash scripts/restore.sh {cycle_id} baseline\|snapshot` |
| `scripts/sync-to-test.sh` | agent_dev → agent_test 同步 | `bash scripts/sync-to-test.sh` |
| `scripts/snapshot.sh` | Phase 4 PASS 後存檔 | `bash scripts/snapshot.sh {cycle_id}` |

### 10.2 恢復場景

| 場景 | 恢復方式 |
|------|---------|
| Phase 2 把 agent_dev 改壞了 | `restore.sh {cycle_id} baseline` → 重跑 Phase 2 |
| Sync 中途中斷 | 重跑 `sync-to-test.sh`（staging 策略保護 agent_test） |
| 錯誤的 PASS + 壞 snapshot | 刪除壞 snapshot → `restore.sh {上一輪} snapshot` |
| Agent session 崩潰 | 檢查檔案 → 若不一致則 restore baseline → 重派 agent |
| Rework 後又失敗 | `restore.sh {cycle_id}_rework{N} baseline` |

### 10.3 迭代狀態判斷

若主 session 崩潰，新 session 可透過檢查 cycle 目錄中的報告判斷當前 Phase：

| 存在的報告 | 代表完成到 |
|-----------|-----------|
| `research/{cycle_id}/` 有報告 | Phase 0 完成 |
| `arch_reviews/{cycle_id}/Architecture_Spec_*` | Phase 1 完成 |
| `openstarry_code_iteration/{cycle_id}_baseline/` | Phase 1.5 完成 |
| `dev_logs/{cycle_id}/` 有報告 | Phase 2 完成 |
| `qa_results/{cycle_id}/` 有報告 | Phase 3 完成 |
| `sys_summary/{cycle_id}/` 有報告 | Phase 4 完成 |

詳細恢復步驟見 `share/openstarry_doc/Agent_Corps/Coordinator_Checklist.md` 故障恢復手冊。

---

## 11. 設計原則

1. **Roles ≠ Tasks** — 每個 Agent 是永久角色，不為單一任務而建
2. **All Tier 1** — 6 個 Agent 全部是核心團隊，無階層
3. **Extensible** — 既定 Plan 完成後，同一批 Agent 自然承接延伸任務
4. **Decision Persistence** — 所有決策必須寫入檔案，不允許只存在記憶中
5. **Quality Gates** — 每個 Phase 有明確的 Entry/Exit Criteria，不滿足不推進
6. **Interface Freeze** — Spec 發布後 Interface 凍結，變更需走升級流程

---

## 12. Extended Mode（既定 Plan 完成後）

觸發條件：`share/openstarry_doc/Implementation_Plans/` 中所有 Plan 已完成 ✅

Coordinator 評估後詢問 User 是否繼續：
- **researcher** → 深度分析 ref 專案，產出新 plugin 構想 + 可行性報告
- **architect** → 全面安全稽核 + 沙盒強化方案
- **dev-core/dev-plugin** → 實作新功能（Web UI、新 plugin 等）
- **qa** → 驗證延伸功能
- **doc-keeper** → 記錄所有延伸工作的決策與成果
