# OpenStarry Agent Corps — Roles & Standard Operating Procedures

Established: 2026-02-09
Last Updated: 2026-02-16 (五種 SOP 重構)

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
| `security` | Pre-release security auditor, vulnerability assessment | sonnet | agent_dev/ (read-only), agent_test/ | share/test/reports/security_reviews/ |
| **Coordinator** | Main session — orchestration, sync, convergence, escalation | opus | all | share/test/reports/sys_summary/ |

---

## 2. RACI Matrix

**R** = Responsible (做), **A** = Accountable (負最終責任), **C** = Consulted (被諮詢), **I** = Informed (被通知)

| Phase | Coordinator | architect | dev-core | dev-plugin | qa | doc-keeper | researcher | security | User |
|-------|:-----------:|:---------:|:--------:|:----------:|:--:|:----------:|:----------:|:--------:|:----:|
| **Phase 0: Planning** | **R/A** | C | I | I | I | **R** (記錄) | **R** (預研) | I | **A** (核准) |
| **Phase 1: Design** | I | **R/A** | C | C | I | **R** (記錄設計決策) | C (提供研究) | I | I |
| **Phase 1.5: Baseline** | **R/A** | I | I | I | I | I | I | I | I |
| **Phase 2: Implement** | I | C (澄清 spec) | **R/A** (core) | **R/A** (plugin) | I | I | R (預研下輪) | I | I |
| **Phase 2.5: Sync** | **R/A** | I | I | I | I | I | I | I | I |
| **Phase 3: Verify** | I | **R** (code review) | I | I | **R/A** (測試) | I | I | I | I |
| **Phase 3.5: Security Audit** | I | I | I | I | I | I | I | **R/A** | I |
| **Phase 4: Converge** | **R/A** | I | I | I | I | **R** (更新文件) | I | I | **A** (裁決 PASS/FAIL) |
| **Rework** | **R** (分配) | C/R (若設計問題) | R (若 core 問題) | R (若 plugin 問題) | I | I | I | I | I |
| **Escalation** | **R** (提報) | C | C | C | C | I | I | I | **A** (裁決) |

---

## 3. 五種迭代 SOP

本節定義五種不同場景的操作流程（Standard、Release、Hotfix、DOC、Simulation）。

### SOP 總覽

| SOP | 用途 | Phases | Phase 4 產出 |
|-----|------|--------|-------------|
| **Standard** | 功能開發、重構、新 plugin | 0 → 1 → 1.5 → 2 → 2.5 → 3 → 4 | Snapshot |
| **Release** | 出新版本 release | 0 → 1 → 1.5 → 2 → 2.5 → 3 → **3.5** → 4 | Snapshot + Release + 版本號 |
| **Hotfix** | 修 bug、小修正 | 2 → 2.5 → 3 → (3.5) → 4 | 更新 Snapshot + Release |
| **DOC** | 僅改文件 | Write → Review → Done | 無（不需 build/test） |
| **Simulation** | 驗證 release 可用 | 環境準備 → 啟動 → 測試 → 記錄 | 驗證報告 |

### SOP 選擇指引（Decision Tree）

```
任務類型？
├── 只改文件（README/docs/memory）→ DOC SOP
├── 修 bug / 小修正 → Hotfix SOP（更新 release 必跑安檢）
├── 新功能 / 重構 / 新 plugin
│   ├── 需要出 release → Release SOP
│   └── 不需要出 release → Standard SOP
└── 驗證 release 是否能用 → Simulation SOP
```

---

### SOP 1: Standard（標準迭代）

**用途**: 功能開發、重構、新 plugin 等需要完整設計的任務
**Phases**: 0 → 1 → 1.5 → 2 → 2.5 → 3 → 4
**Phase 4 產出**: Snapshot（不出 release）

| Phase | 內容 | 參與者 |
|-------|------|--------|
| 0 | Planning — 研究、規劃 | Coordinator, researcher, doc-keeper |
| 1 | Design — Spec + Interface 凍結 | architect, doc-keeper |
| 1.5 | Baseline — 備份安全網 | Coordinator |
| 2 | Implementation | dev-core, dev-plugin |
| 2.5 | Sync — agent_dev → agent_test | Coordinator |
| 3 | Verification — QA + Code Review | qa, architect |
| 4 | Convergence — Snapshot | Coordinator, doc-keeper |

各 Phase 詳細定義見下方「Phase 詳細定義」。

---

### SOP 2: Release（發版）

**用途**: 出新版本 release
**Phases**: 0 → 1 → 1.5 → 2 → 2.5 → 3 → **3.5** → 4
**Phase 4 產出**: Snapshot + Release 目錄 + 版本號更新

與 Standard 相同，額外加：
- **Phase 3.5**: Security Audit（Critical/High BLOCKS release）
- **Phase 4**: 多出 release 目錄建立、版本號確認

各 Phase 詳細定義見下方「Phase 詳細定義」。

---

### SOP 3: Hotfix（快速修正）

**用途**: 修復已 release 版本的 bug、小改動、配置錯誤
**觸發**: 用戶回報 bug 或測試發現問題
**Phases**: 2 → 2.5 → 3 → (3.5) → 4

| Phase | 內容 | 參與者 |
|-------|------|--------|
| 2 | Fix — 在 agent_dev 直接修復 | dev-core 或 dev-plugin |
| 2.5 | Sync — agent_dev → agent_test | Coordinator |
| 3 | Verify — pnpm build + pnpm test | Coordinator（輕量驗證） |
| 3.5 | Security Audit — **只要更新 release 就必須跑** | security |
| 4 | Update — 更新 snapshot + release | Coordinator |

特點：
- **不需** Phase 0 研究、Phase 1 設計、Phase 1.5 baseline
- **不需** architect code review（除非改動涉及架構）
- Phase 3 由 Coordinator 直接跑 build+test，不需派 qa agent
- **安檢規則**: 只要 Hotfix 結果會更新到 release 目錄 → Phase 3.5 **必跑**

---

### SOP 4: DOC（文件更新）

**用途**: 僅修改文件，不改程式碼
**範圍**: share/openstarry_doc/、README.md、CLAUDE.md、memory 等
**Phases**: 寫 → 審 → 完成

| 步驟 | 內容 | 參與者 |
|------|------|--------|
| 1. Write | 撰寫/修改文件 | doc-keeper 或 Coordinator |
| 2. Review | 檢查內容正確性（optional） | Coordinator |
| 3. Done | 完成，記錄到 Iteration_Log | doc-keeper |

特點：
- **不需** build、test、sync、snapshot
- **不需** agent_test 驗證
- 若文件變更需包含在 release 中 → 直接更新 release 目錄的 openstarry_doc/

---

### SOP 5: Simulation（模擬操作驗證）

**用途**: 按照 README 流程實際跑一遍，驗證 release 從安裝到使用完全可行
**觸發**: Release SOP 或 Hotfix SOP 完成後；或用戶主動要求驗證
**需要用戶配合**: Provider 的 API Key、LM Studio URL 等憑證由用戶提供
**預估時間**: 30-45 分鐘（視 LLM 回應速度）

**流程**:

| 步驟 | 內容 | 參與者 |
|------|------|--------|
| 1. 環境準備 | 從 release 目錄：`pnpm install && pnpm build` | Coordinator |
| 2. 多配置啟動 | 依序測試各 config（至少 basic + lmstudio-auto + web） | Coordinator |
| 3. Provider 設定 | 用戶提供 credentials → `/provider login` + `/provider model` | **用戶** + Coordinator |
| 4. 斜線指令 | 測試 `/help`、`/provider status`、`/provider model`、`/metrics`、`/reset`、`/quit` | Coordinator |
| 5. 對話測試 | 至少 2 個 provider（1 本地 + 1 雲端）發送訊息、確認回應 | Coordinator |
| 6. 工具測試 | fs.read、fs.write、fs.list、fs.mkdir + 多工具並發限制 | Coordinator |
| 7. Daemon 工作流 | daemon start → ps → attach → 對話 → detach → daemon stop | Coordinator |
| 8. 傳輸層驗證 | stdio + WebSocket（web-agent） | Coordinator |
| 9. 錯誤恢復 | 無 provider 時提示、無效 model、/reset 恢復 | Coordinator |
| 10. 結果記錄 | 記錄通過/失敗項目，產出報告 | Coordinator |

**用戶需提供的項目**（視測試的 Provider 而定）：
- LM Studio: 本機 URL（預設 `http://127.0.0.1:1234/v1`）
- Ollama: 本機 URL（預設 `http://127.0.0.1:11434`）
- Gemini OAuth: 已存在的 token（`~/.openstarry/plugins/gemini-oauth/`）
- Gemini: API Key
- Claude: API Key
- ChatGPT: API Key

**可用配置**（configs/ 目錄）:

| Config | 用途 | 優先級 |
|--------|------|--------|
| basic-agent.json | 最小 CLI agent | 必測 |
| basic-agent-lmstudio-auto.json | 預設 LM Studio + Gemini OAuth | 必測 |
| web-agent.json | WebSocket + HTTP Web UI | 必測 |
| tui-agent.json | Ink-based TUI 介面 | 建議 |
| websocket-agent.json | Headless WebSocket | 建議 |
| mcp-agent.json | MCP server 模式 | 選測 |
| full-agent.json | 全功能（WebSocket + HTTP + TUI） | 選測 |

---

#### 驗證清單

**A. 環境準備（2 項）**
- [ ] `pnpm install` 成功
- [ ] `pnpm build` 成功（全部 workspace 編譯通過）

**B. 基礎啟動（每個 config 各驗證，至少測 3 個 config）**
- [ ] Agent 啟動無錯誤
- [ ] Banner 顯示正確（版本號、Provider 指令列表）
- [ ] 所有配置的 plugin 載入成功（0 errors）
- [ ] `/quit` 正常關閉

**C. Provider 驗證（至少 2 個：1 本地 + 1 雲端）**
- [ ] `/provider status` 顯示已登入的 provider
- [ ] `/provider login <provider>` 成功（如需手動登入）
- [ ] `/provider model <model_id>` 選擇模型成功
- [ ] 切換 provider 後對話正常

**D. 斜線指令（6 項）**
- [ ] `/help` — 列出所有已註冊指令（≥6 個）
- [ ] `/provider status` — 顯示 provider 狀態
- [ ] `/provider model` — 模型選擇
- [ ] `/metrics` — 顯示 metrics 快照
- [ ] `/reset` — 清除對話歷史
- [ ] `/quit` — 正常退出

**E. LLM 對話測試（每個 provider 各測 1 次）**
- [ ] 發送簡單訊息（如 "Hello, who are you?"）
- [ ] 收到完整回應（非截斷）
- [ ] 回應延遲合理（雲端 <30s，本地 <10s）

**F. 工具調用測試（4 項）**
- [ ] fs.read — 讀取檔案內容
- [ ] fs.write — 建立臨時檔案
- [ ] fs.list — 列出目錄
- [ ] fs.mkdir — 建立子目錄

**G. Daemon 工作流（6 項）**
- [ ] `daemon start --config <path>` — 背景啟動
- [ ] `ps` — 顯示執行中的 agent
- [ ] `attach <agent-id>` — 連線到 daemon
- [ ] attach 模式下發送訊息並收到回應
- [ ] detach 後 daemon 仍在執行
- [ ] `daemon stop` — 正常終止

**H. 傳輸層驗證（Web UI）**
- [ ] web-agent 啟動後 Web UI 可在瀏覽器開啟
- [ ] WebSocket 連線正常（瀏覽器 DevTools 可確認）
- [ ] 透過 Web UI 發送訊息並收到回應

**I. 錯誤恢復（3 項）**
- [ ] 無 provider 時顯示「No provider/model configured...」提示
- [ ] 無效 model 選擇時顯示錯誤訊息
- [ ] `/reset` 後可正常繼續對話

**J. 收尾（2 項）**
- [ ] `pnpm clean:all` 清除 build artifacts
- [ ] 所有 daemon 進程已停止

---

**結果**: PASS / FAIL（附失敗項目說明）
**失敗處理**: 回到 Hotfix SOP 修正 → 再跑 Simulation
**報告位置**: `share/test/reports/sys_summary/{YYYYMMDD}_Simulation_v{X.Y.Z-tag}.md`

---

### Phase 詳細定義（Standard / Release SOP 參考）

以下為 Standard SOP 和 Release SOP 使用的各 Phase 詳細定義。Hotfix SOP 的 Phase 2/2.5/3/4 也參考此處（簡化版）。

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

**參與者**：qa (R/A), architect (R), security (C — ENG-FAB-1)

**可並行**：qa 和 architect 可同時進行，互不依賴。ENG-FAB 驗證在兩者完成後執行。

**步驟（qa）**：
1. 在 `agent_test/openstarry/` 執行：
   - `pnpm build` — Build 驗證
   - `pnpm test` — 單元測試
   - `pnpm test:purity` — 微核心純度檢查
2. 在 `agent_test/openstarry_plugin/` 執行：
   - `pnpm build` — Plugin build 驗證
   - `pnpm test` — Plugin 測試（若存在）
3. **ENG-FAB-1 驗證**（Plan39 起強制）：
   - 比對 dev log 宣稱的新增/修改檔案 vs 實際 code（逐一確認檔案存在、功能對應）
   - 結果寫入 QA Report 的 `## ENG-FAB-1 Verification` 段落
4. 產出 QA Report

**步驟（architect）**：
1. 讀取 Architecture_Spec：`share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md`
2. 讀取實作程式碼：`agent_dev/`（唯讀）
3. 逐項驗證：
   - Interface 實作是否符合凍結的 Spec
   - Five Aggregates 合規
   - 微核心純度
   - pushInput pattern
   - 安全漏洞
   - **十大宣言合規檢查**（Tenet Compliance Check）
4. **ENG-FAB-2 驗證**（Plan39 起強制）：
   - 比對報告宣稱的修改 vs 實際 git diff / code diff
   - 確認報告中沒有造假（宣稱存在但實際不存在的檔案、功能、修改）
   - 結果寫入 Code Review Report 的 `## ENG-FAB-2 Verification` 段落
5. 產出 Code Review Report（PASS / FAIL / CONDITIONAL）

**交付物**：
| 產出 | 路徑 | 負責人 |
|------|------|--------|
| QA Report | `share/test/reports/qa_results/{cycle_id}/QA_Report_{PlanName}.md` | qa |
| Code Review | `share/test/reports/arch_reviews/{cycle_id}/Code_Review_{PlanName}.md` | architect |

**Exit Criteria**：
- [ ] QA Report 已存在
- [ ] Code Review Report 已存在
- [ ] ENG-FAB-1 驗證已執行（QA Report 中有結果）
- [ ] ENG-FAB-2 驗證已執行（Code Review Report 中有結果）
- [ ] Coordinator 已讀取兩份報告，進入 Phase 4

---

### Phase 3.5: Security Audit（安全稽核）— Release SOP / Hotfix SOP（更新 release 時）

**觸發條件**：Release SOP（必跑）或 Hotfix SOP（結果更新到 release 目錄時必跑）

**Entry Criteria**：Phase 3 Exit Criteria 全部滿足（QA PASS + Code Review PASS）

**參與者**：security (R/A), Coordinator (I)

**目的**：在 release 前進行全面對抗性安全稽核，覆蓋 8 大領域（Sandbox Isolation、Plugin Trust Boundaries、Transport Security、Session Security、Input Validation、Secrets Management、Dependency Risk、OWASP Compliance）。

**步驟**：
1. **Coordinator** 派遣 `security` agent，提供 cycle_id 和 release version
2. **security** 讀取 Architecture_Spec 和 Code Review 報告
3. **security** 逐一掃描 8 大領域，對 `agent_dev/` 和 `agent_test/` 進行全面稽核
4. **security** 產出 Security Audit Report（含 PASS/FAIL verdict）
5. **Coordinator** 讀取報告並判定結果

**交付物**：
| 產出 | 路徑 | 負責人 |
|------|------|--------|
| Security Audit Report | `share/test/reports/security_reviews/{cycle_id}/Security_Audit_{ReleaseName}.md` | security |

**Exit Criteria**：
- [ ] Security Audit Report 已存在
- [ ] Report 包含 8 大領域逐項 PASS/FAIL
- [ ] Report 包含明確的 PASS/FAIL verdict

**Severity 與 Release 影響**：

| Severity | Release 影響 |
|----------|-------------|
| Critical | **BLOCKS** release |
| High | **BLOCKS** release |
| Medium | Advisory（記錄，不阻擋） |
| Low / Info | Advisory |

**結果處理**：
- **PASS**（無 Critical/High findings）→ 進入 Phase 4
- **FAIL**（有 Critical/High findings）→ 進入 Security Rework 流程

#### Security Rework 流程

```
Phase 3 PASS
  ↓
Phase 3.5: Security Audit
  ├── PASS → Phase 4 (release)
  └── FAIL (Critical/High)
        ↓
      分級: Code Fix → Phase 2 / Design Fix → Phase 1
        ↓
      修復 → Phase 2.5 sync → Phase 3 re-verify
        ↓
      Phase 3 PASS → Phase 3.5 Re-Audit (僅驗證 FAIL items)
        ├── PASS → Phase 4
        └── FAIL → rework +1 (上限 2 次，超過 → User 裁決)
```

**重要**：Security rework 與一般 rework **共用 2 次額度**。Re-Audit 時 security agent 僅需驗證先前 FAIL 的 findings 是否已修復，不需重跑全面稽核。

---

### Phase 4: Convergence

**Entry Criteria**：Phase 3 Exit Criteria 全部滿足（Standard SOP）；或 Phase 3.5 PASS（Release SOP / Hotfix SOP）

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

> **適用 SOP**: Standard、Release。Hotfix SOP 的 rework 直接重新修正後再跑 Phase 2.5+。

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

### 5.4 FROZEN Interface 修訂規則（Rule #45，2026-04-04 生效）

已凍結的 Interface 可修訂，若且唯若滿足以下**三項條件**：

1. **N=0 current consumers** — 目前沒有任何程式碼使用該 interface
2. **R3 辯論一致通過** — 研究團隊 R3 辯論對修訂方案達成一致（unanimous）
3. **即時 re-freeze + atomic commit** — 所有修訂在單一 git commit 完成，commit 後立即 re-freeze

修訂流程：
```
研究團隊 R3 辯論 → 修訂授權（N=0 + unanimous）
  → Phase 1 Architecture_Spec 納入 amendments
  → Phase 2 W0: atomic commit（所有 amendments 一次提交）
  → 即時 re-freeze（更新 FROZEN 標註 + @since 版本）
```

**重要**：FROZEN 不得成為達成十大核心宣言的障礙。N=0 的 interface 修訂是合理的演進，不是破壞凍結機制。

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
| Phase 3.5 | 待命：無主動任務（Release Cycle only） |
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

## 11. Baseline Rules（永久生效）

以下規則由 Master 批准，永久生效，適用於所有 SOP 和所有 Cycle。

### Rule #44 — 合規術語限制（2026-04-04 生效）

> 合規狀態描述**僅限**使用：**COMPLIANT** / **CONDITIONAL** / **NON-COMPLIANT**。
> 禁止使用 "Advancing"、"Progressing"、"Near-compliant" 或任何替代術語。

**適用範圍**：所有報告（QA Report、Code Review、Engineering Delivery、Iteration Log、Summary Report）。
**違規處理**：交付報告中出現非三元組術語，直接判定為違規。

### Rule #45 — FROZEN Interface 修訂規則（2026-04-04 生效）

> FROZEN interface 可修訂，若且唯若：
> 1. N=0 current consumers
> 2. R3 辯論一致通過
> 3. 即時 re-freeze + atomic commit

詳見 Section 5.4。

### Rule #46 — 證據格式標準化（2026-04-07 生效）

> 所有交付報告（QA Report、Code Review、Engineering Delivery）必須提供：
> 1. **原始日誌** — build logs、test output、security scan logs（無過濾）
> 2. **結構化摘要** — 表格或清單形式，便於快速驗證

**目的**：支援 ENG-FAB-1/2 造假檢測，允許研究團隊追蹤來源。

### Rule #47 — 追蹤驗證週期（2026-04-07 生效）

> 追蹤矩陣驗證遵循奇偶規則：
> - **奇數週期**（Cycle N 為奇數）：增量驗證（僅檢查本週期變更）
> - **偶數週期**（Cycle N 為偶數）：完整驗證（完整重新檢查前一週期）

**目的**：平衡驗證頻率與成本，確保無漏洞。

### Rule #48 — 製造相關檢測（Fabrication-Specific）（2026-04-07 生效）

> QA 報告必須包含 `## Fabrication Inspection` 段落，確認：
> - Artifact 中宣稱的每個檔案實際存在
> - 檔案內容與 artifact manifest 的雜湊值一致
> - 新增/修改的檔案在 code 中找到對應實作
> - 結果：PASS / FAIL

### Rule #49 — 稽核清單（Audit Checklist）生活文件（2026-04-07 生效）

> `share/openstarry_doc/Agent_Corps/Audit_Checklist.md` 是生活文件，隨著每個 Cycle 更新：
> - Phase 3 驗證項目清單
> - ENG-FAB-1/2/3 檢查點
> - Tenet Compliance Check 項目
> - 期望值與實際值對應

**維護者**：Coordinator（每週期更新）

### Rule #50 — CV-6 飽和度閾值（2026-04-07 生效）

> **FAIL 條件**：若稽核週期中，confidence saturation 達到或超過 10%
> - 定義：saturation = (confidence_exceed_max / total_samples) ≥ 0.10
> - 涉及組件：ExecutionLoop、ILoopQualityMonitor、loop_quality auditor

**影響**：觸發 Phase 4 FAIL，進入 Design Rework（Plan 級別調整）

### Rule #51 — CV-6 警告閾值（2026-04-07 生效）

> **WARNING 條件**：若稽核週期中，negative frequency 達到或超過 25%
> - 定義：neg_freq = (negative_vedana_events / total_vedana_events) ≥ 0.25
> - 涉及組件：VedanaRegistry、Klesha dispatcher、IVedanaSensor

**影響**：記錄在 Iteration Log，不阻擋 release，但標記為注意項

### Rule #52 — 快照完整性（Snapshot Completeness）（2026-04-07 生效）

> 快照管理遵循規則：
> - **DEFER 一次**：允許（進入下個週期補償）
> - **DEFER 兩次連續**：組件被標記為 BLOCKED（無法進行後續開發）
> - **重置機制**：成功完成快照後，DEFER 計數器重置

**應用**：Late-Joiner Snapshot、Distributed State Snapshot

### Rule #53 — ENG-FAB 強制失敗（2026-04-07 生效）

> 以下情況導致 **Plan 不被接受**（即使 QA/Code Review 通過）：
> - ENG-FAB-1 驗證 FAIL（artifact 與 code 不符）
> - ENG-FAB-2 驗證 FAIL（代碼造假）
> - ENG-FAB-3 追蹤 FAIL（artifact hash 不一致）
> 
> 修復後需重新提交 Phase 4，不計入 rework cycle 額度（ENG-FAB 失敗為計劃審批問題，非實作問題）。

---

### Rule #54 — A-9 整合驗證（2026-04-09 生效）

> **Components 必須確實被呼叫，不能是死代碼。**
>
> **驗證方式**: Phase 3 code review 必須確認：
> - A-9 fabrication reporter 在 ExecutionLoop init 時被實例化
> - DEV-1b callback 在至少 1 個 authority transfer 場景被呼叫
> - Audit events 實際到達 AuditTrail（trace log 驗證）
> 
> **失敗分類**: 若代碼未被呼叫（dead code），分類為 Design Fix（→Phase 1 重新檢視）

**根據**: Plan42 RC-1 發現 payload extraction 不完整，因為 integration 未被驗證。此規則確保所有宣稱的 components 確實被使用。

---

### Rule #55 — 破壞性操作絕不委派（永久）

> **狀態修改操作必須絕不委派給動態 arbiters 或 observables。**
>
> **破壞性操作定義**:
> - Seed 狀態突變
> - Gear switch（authority transfer）
> - Authority 委派
> - Audit trail 寫入
> - Key rotation
> 
> **模式**: 使用靜態 dispatch 或顯式 permission lattice 進行狀態修改。Observe mode 必須是唯讀的。
>
> **驗證**: Phase 3 架構審查必須確認 observe path 零狀態突變。

**根據**: Plan42 RC-2 發現 priority deadlock，因為全局 queue 試圖同時服務唯讀（observe）和寫入（state-mod）路徑。此規則強制執行嚴格分離。

---

### Rule #56 — 分階段 Authority 轉移（2026-04-09 生效）

> **Authority 轉移策略必須分階段漸進: P2 (shadow) → P3 (low-risk) → P4 (state_mod).**
>
> **階段定義**:
> - **P2 (Shadow)**: 唯讀信心追蹤，observe mode 專用
> - **P3 (Low-Risk)**: 無狀態修改決策（報告、指標、建議）
> - **P4 (State-Mod)**: Authority 修改 seed 狀態、執行 gear switches、轉移委派
> 
> **政策應用**: 導入 ExecutionLoop 的新政策必須從 P2 開始，經過 1 cycle 測試後升級到 P3，然後升級到 P4。
>
> **範例**: CV-5 dynamic routing 開始為 observe-only (P2)，在 W2 升級為 low-risk recommender，不允許在 P2/P3 修改狀態。
>
> **驗證**: Phase 3 code review 必須確認政策在正確的 phase level 實現。

**根據**: Plan42 RC-3 發現 hardcoded gear=1 threshold，因為參數化未分階段。此規則確保政策常數逐漸導入，從安全開始並根據證據升級。

---

## 12. ENG-FAB 反造假機制（Plan39 起強制）

Master 指示：**開發團隊應善用現有機制自行攔截造假，不能只依賴研究團隊事後發現。**

### 12.1 雙邊機制

| 措施 | 開發團隊（出件自檢） | 研究團隊（收件驗證） |
|------|:------------------:|:------------------:|
| **ENG-FAB-1** 驗證腳本 | Phase 3：qa 或 security 執行，比對報告宣稱 vs 實際 code | 收到後再次驗證 |
| **ENG-FAB-2** 強制 diff | Phase 3：architect 比對報告宣稱 vs code diff | 審閱時比對 |
| **ENG-FAB-3** traceability（Plan40 起） | doc-keeper 維護追蹤矩陣 | 追蹤驗證 |

### 12.2 Phase 3 攔截流程

```
dev-core/dev-plugin 寫 code
  → qa 跑 build + test + purity
  → architect code review + Tenet Compliance Check
  → 產出交付報告（dev logs、QA Report、Code Review）
  → 【ENG-FAB-1】qa/security 比對報告宣稱 vs 實際 code    ← 攔截點
  → 【ENG-FAB-2】architect 比對報告宣稱 vs code diff       ← 攔截點
  → 通過 → Phase 4
  → 發現造假 → 修正報告 / 修正 code → 重新驗證
```

### 12.3 ENG-FAB-1 驗證項目

QA Report 必須包含 `## ENG-FAB-1 Verification` 段落，逐一確認：
- 報告宣稱新增的每個檔案是否實際存在
- 報告宣稱修改的每個檔案是否有對應的實際變更
- 報告宣稱的功能是否可在程式碼中找到對應實作
- 結果：逐項 PASS / FAIL

### 12.4 ENG-FAB-2 驗證項目

Code Review Report 必須包含 `## ENG-FAB-2 Verification` 段落，確認：
- 報告宣稱的修改 vs 實際 code diff 一致
- 沒有宣稱存在但實際不存在的檔案、功能、或修改
- 結果：PASS / FAIL

---

## 13. 設計原則

1. **Roles ≠ Tasks** — 每個 Agent 是永久角色，不為單一任務而建
2. **All Tier 1** — 7 個 Agent 全部是核心團隊，無階層
3. **Extensible** — 既定 Plan 完成後，同一批 Agent 自然承接延伸任務
4. **Decision Persistence** — 所有決策必須寫入檔案，不允許只存在記憶中
5. **Quality Gates** — 每個 Phase 有明確的 Entry/Exit Criteria，不滿足不推進
6. **Interface Freeze** — Spec 發布後 Interface 凍結，變更需走升級流程（Rule #45 例外）
7. **Anti-Fabrication** — 交付物必須對源碼交叉驗證（ENG-FAB），不能僅依賴報告宣稱
8. **Compliance Triad** — 合規狀態僅限 COMPLIANT / CONDITIONAL / NON-COMPLIANT（Rule #44）

---

## 14. Extended Mode（既定 Plan 完成後）

觸發條件：`share/openstarry_doc/Implementation_Plans/` 中所有 Plan 已完成 ✅

Coordinator 評估後詢問 User 是否繼續：
- **researcher** → 深度分析 ref 專案，產出新 plugin 構想 + 可行性報告
- **architect** → 全面安全稽核 + 沙盒強化方案
- **dev-core/dev-plugin** → 實作新功能（Web UI、新 plugin 等）
- **qa** → 驗證延伸功能
- **doc-keeper** → 記錄所有延伸工作的決策與成果
