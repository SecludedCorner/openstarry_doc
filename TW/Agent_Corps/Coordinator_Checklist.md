# Coordinator Quick-Reference Checklist

每輪迭代的操作清單。Coordinator（主 session）逐項執行。

---

## 開始新迭代

```
Cycle ID: {YYYYMMDD}_cycle{N}
Target Plan: Plan__________
```

### Phase 0: Planning
- [ ] 確認 User 指令，識別 target Plan
- [ ] 建立 cycle 目錄：
  ```bash
  CYCLE="{YYYYMMDD}_cycle{N}"
  mkdir -p share/test/reports/arch_reviews/$CYCLE
  mkdir -p share/test/reports/dev_logs/$CYCLE
  mkdir -p share/test/reports/qa_results/$CYCLE
  mkdir -p share/test/reports/research/$CYCLE
  mkdir -p share/test/reports/sys_summary/$CYCLE
  ```
- [ ] 派 **researcher** 預研 → 等待研究報告產出
- [ ] 派 **doc-keeper** 記錄迭代計畫到 Iteration_Log.md
- [ ] ✅ Exit: 研究報告存在 + 迭代計畫已記錄

### Phase 1: Design
- [ ] 通知 **architect**，提供：cycle_id + Plan 路徑 + 研究報告路徑
- [ ] 等待 Architecture_Spec 產出
- [ ] 確認 Spec 包含 **凍結的 Interface 定義**
- [ ] 確認 Spec 標註是否有 **SDK Interface 變更**（決定 Phase 2 順序）
- [ ] 派 **doc-keeper** 記錄設計決策
- [ ] ✅ Exit: Spec 存在 + Interface 凍結 + 決策已記錄

### Phase 1.5: Baseline (安全網)
- [ ] 建立 pre-Phase-2 baseline：
  ```bash
  bash scripts/baseline.sh $CYCLE
  ```
- [ ] 確認輸出 "Baseline Saved"
- [ ] ✅ 若 Phase 2 失敗需要 rollback：`bash scripts/restore.sh $CYCLE baseline`

### Phase 2: Implementation
- [ ] **SDK Interface 有變更？**
  - YES →
    1. 派 **dev-core** Step 2a，prompt 明確指定：
       「只實作 Architecture_Spec 中的 SDK Interface 變更（packages/sdk/），完成後 pnpm build 驗證。不要實作其他部分。」
    2. 等 dev-core Step 2a 完成（build pass）
    3. 並行派出：
       - **dev-core** Step 2b，prompt 明確指定：
         「SDK Interface 已在 Step 2a 完成，請實作 Spec 中 packages/core、packages/shared、apps/runner 的剩餘變更。」
       - **dev-plugin**，prompt 正常指定 Spec 路徑
  - NO → 直接派 **dev-core** + **dev-plugin** 並行
- [ ] 等待兩者 `pnpm build` PASS + dev log 寫入
- [ ] （可選）同時派 **researcher** 預研下一輪
- [ ] ⚠️ 若收到 `INTERFACE CHANGE NEEDED`：
  - 暫停 Phase 2
  - 通知 architect 評估 Spec Addendum
  - Addendum 發布後通知 dev-plugin 重讀
- [ ] ✅ Exit: 所有 build PASS + dev log 存在

### Phase 2.5: Sync
- [ ] 執行 sync 腳本：
  ```bash
  bash scripts/sync-to-test.sh
  ```
- [ ] 確認 sync 完成（最後輸出 "Sync Complete"）
- [ ] ⚠️ 若 sync 後 build 失敗 → 退回 Phase 2
- [ ] ✅ Exit: agent_test build PASS

### Phase 3: Verify
- [ ] 派 **qa** 跑測試（在 agent_test/）→ 等待 QA Report
- [ ] 派 **architect** 做 Code Review（讀 agent_dev/）→ 等待 Code Review Report
- [ ] （qa 和 architect 可並行）
- [ ] ✅ Exit: 兩份報告都已產出

### Phase 4: Converge
- [ ] 讀取 QA Report + Code Review Report
- [ ] 判定結果：

**→ 全部 PASS：**
- [ ] Snapshot（排除 node_modules/dist）：
  ```bash
  bash scripts/snapshot.sh $CYCLE
  ```
- [ ] 派 **doc-keeper** 更新 Plan ✅ + Iteration_Log
- [ ] 寫 Summary Report 到 `share/test/reports/sys_summary/$CYCLE/`
- [ ] **Lessons Learned 回顧**：
  - 本輪迭代中遇到哪些問題？（技術、流程、溝通）
  - 什麼做法有效？什麼做法需要改進？
  - 是否需要更新 Risk Register？（新風險、風險狀態變更）
  - 派 **doc-keeper** 追加到 `Lessons_Learned.md`
- [ ] 向 User 報告結果

**→ 任一 FAIL：**
- [ ] 分類每個 FAIL 項目：
  - `Code Fix` → 退回 Phase 2（僅修正部分）
  - `Design Fix` → 退回 Phase 1（architect 修訂 Spec）
  - `Plan Fix` → 退回 Phase 0（重新規劃）
- [ ] Rework 次數 +1（本輪已 rework ___/2 次）
- [ ] 若 rework > 2 → 升級到 User
- [ ] 寫 Rework 任務到 Summary Report
- [ ] 派 **doc-keeper** 記錄 FAIL 原因到 Iteration_Log + Lessons_Learned

---

## 升級 Checklist

當需要升級到 User 時：
- [ ] 收集各方意見
- [ ] 整理：問題描述 + 各方立場 + Coordinator 建議
- [ ] 呈報 User 裁決
- [ ] 派 **doc-keeper** 記錄裁決結果

---

## 故障恢復手冊

### 情境 A：Phase 2 實作把 agent_dev 改壞了（build 不過、邏輯錯亂）
```bash
# 恢復到 Phase 2 開始前的狀態
bash scripts/restore.sh $CYCLE baseline
```
恢復後從 Phase 2 重新開始。

### 情境 B：sync 腳本中途斷掉（agent_test 不完整）
```bash
# 重新執行即可（使用 staging 策略，斷掉時 agent_test 仍保持舊狀態）
bash scripts/sync-to-test.sh
```
若 agent_dev 本身也有問題 → 先恢復 agent_dev（情境 A），再重新 sync。

### 情境 C：Phase 4 錯誤地 PASS，snapshot 了壞程式碼
```bash
# 刪除壞的 snapshot
rm -rf share/openstarry_code_iteration/$CYCLE

# 恢復 agent_dev 到上一輪的好 snapshot
bash scripts/restore.sh {上一輪_cycle_id} snapshot
```
然後重新跑整輪迭代。

### 情境 D：Agent session 中途崩潰（檔案寫到一半）
1. 檢查 agent_dev 中的檔案狀態
2. 若程式碼不一致 → `bash scripts/restore.sh $CYCLE baseline`
3. 若只是報告沒寫完 → 重新派遣對應 agent 即可

### 情境 E：想退回到更早的版本
```bash
# 列出所有可用的 snapshot
ls share/openstarry_code_iteration/

# 恢復到指定版本
bash scripts/restore.sh {任意_cycle_id} snapshot
```

### 預防措施
- **Phase 1.5 Baseline 是必做步驟**，不可跳過
- 每輪 Phase 4 PASS 後的 snapshot 是永久保留的（除非手動刪除）
- 如果不確定 agent_dev 狀態，先跑 `cd agent_dev/openstarry && pnpm build` 驗證
