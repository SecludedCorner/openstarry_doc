# コーディネーター・クイックリファレンス・チェックリスト (Coordinator Checklist)

各反復サイクルにおける操作リスト。コーディネーター（メインセッション）が項目ごとに実行します。

---

## 新しい反復サイクルの開始

```
Cycle ID: {YYYYMMDD}_cycle{N}
Target Plan: Plan__________
```

### Phase 0: Planning
- [ ] User からの指示を確認し、ターゲットとなるプランを特定する。
- [ ] cycle ディレクトリを作成する：
  ```bash
  CYCLE="{YYYYMMDD}_cycle{N}"
  mkdir -p share/test/reports/arch_reviews/$CYCLE
  mkdir -p share/test/reports/dev_logs/$CYCLE
  mkdir -p share/test/reports/qa_results/$CYCLE
  mkdir -p share/test/reports/research/$CYCLE
  mkdir -p share/test/reports/sys_summary/$CYCLE
  ```
- [ ] **researcher** を派遣して事前調査を行う → 調査報告書の完成を待つ。
- [ ] **doc-keeper** を派遣して反復計画を `Iteration_Log.md` に記録する。
- [ ] ✅ Exit: 調査報告書が存在し、反復計画が記録されていること。

### Phase 1: Design
- [ ] **architect** に通知する。提供情報： cycle_id + プランのパス + 調査報告書のパス。
- [ ] Architecture_Spec の作成を待つ。
- [ ] 仕様書に **凍結された Interface 定義** が含まれていることを確認する。
- [ ] 仕様書に **SDK Interface の変更** の有無が明記されていることを確認する（ Phase 2 の順序を決定するため）。
- [ ] **doc-keeper** を派遣して設計決定を記録する。
- [ ] ✅ Exit: 仕様書が存在し、 Interface が凍結され、決定事項が記録されていること。

### Phase 1.5: Baseline (安全網)
- [ ] Phase 2 開始前の Baseline を作成する：
  ```bash
  bash scripts/baseline.sh $CYCLE
  ```
- [ ] "Baseline Saved" の出力を確認する。
- [ ] ✅ Phase 2 が失敗しロールバックが必要な場合： `bash scripts/restore.sh $CYCLE baseline`

### Phase 2: Implementation
- [ ] **SDK Interface の変更はあるか？**
  - はい →
    1. **dev-core** を Step 2a として派遣。プロンプトで以下を明確に指示：
       「 Architecture_Spec 内の SDK Interface の変更 (packages/sdk/) のみを実装せよ。完了後、 pnpm build で検証せよ。他の部分は実装しないこと。」
    2. dev-core の Step 2a 完了（ビルド通過）を待つ。
    3. 以下を並行して派遣：
       - **dev-core** を Step 2b として派遣。プロンプトで以下を明確に指示：
         「 SDK Interface は Step 2a で完了済み。仕様書に基づき、 packages/core, packages/shared, apps/runner の残りの変更を実装せよ。」
       - **dev-plugin** を派遣。通常通り仕様書のパスを指定。
  - いいえ → 直接 **dev-core** と **dev-plugin** を並行して派遣する。
- [ ] 両者の `pnpm build` PASS および開発ログの記述完了を待つ。
- [ ] （任意）同時に **researcher** を派遣して次ターンの事前調査を行う。
- [ ] ⚠️ `INTERFACE CHANGE NEEDED` を受け取った場合：
  - Phase 2 を一時停止する。
  - architect に仕様書の追加修正 (Spec Addendum) の評価を依頼する。
  - 修正発行後、 dev-plugin に再読み込みを通知する。
- [ ] ✅ Exit: すべてのビルドが PASS し、開発ログが存在すること。

### Phase 2.5: Sync
- 同期スクリプトを実行する：
  ```bash
  bash scripts/sync-to-test.sh
  ```
- 同期の完了を確認する（最後に "Sync Complete" と出力されること）。
- ⚠️ 同期後のビルドに失敗した場合 → Phase 2 に戻る。
- ✅ Exit: agent_test のビルドが PASS すること。

### Phase 3: Verify
- [ ] **qa** を派遣してテストを実行（ agent_test/ 内） → QA レポートを待つ。
- [ ] **architect** を派遣してコードレビューを実施（ agent_dev/ を読み取り） → コードレビュー報告書を待つ。
- [ ] （ qa と architect は並行実行可能）
- [ ] ✅ Exit: 両方の報告書が作成されていること。

### Phase 4: Converge
- [ ] QA レポートとコードレビュー報告書を確認する。
- [ ] 結果を判定する：

**→ すべて PASS の場合：**
- [ ] スナップショットを作成（ node_modules/dist を除外）：
  ```bash
  bash scripts/snapshot.sh $CYCLE
  ```
- [ ] **doc-keeper** を派遣してプランの更新 (✅) および反復ログへの追記を行う。
- [ ] Summary Report を `share/test/reports/sys_summary/$CYCLE/` に作成する。
- [ ] **教訓 (Lessons Learned) の振り返り**：
  - 今回の反復でどのような問題が発生したか？（技術、プロセス、通信）
  - どのような手法が有効だったか？何が改善されるべきか？
  - リスク登録簿 (Risk Register) の更新は必要か？（新しいリスク、リスク状態の変化）
  - **doc-keeper** を派遣して `Lessons_Learned.md` に追記する。
- [ ] User に結果を報告する。

**→ いずれかが FAIL の場合：**
- [ ] 各 FAIL 項目を分類する：
  - `Code Fix` → Phase 2 に戻る（部分的な修正のみ）。
  - `Design Fix` → Phase 1 に戻る（ architect が仕様を改訂）。
  - `Plan Fix` → Phase 0 に戻る（再計画）。
- [ ] リワーク回数を +1 する（今回のサイクルで ___/2 回のリワーク）。
- [ ] リワーク回数が 2回を超えた場合 → User へエスカレーションする。
- [ ] リワークタスクを Summary Report に記述する。
- [ ] **doc-keeper** を派遣して FAIL の原因を反復ログおよび教訓に記録する。

---

## エスカレーション・チェックリスト

User へのエスカレーションが必要な場合：
- [ ] 各方面の意見を収集する。
- [ ] 整理：問題の説明 + 各方面の立場 + コーディネーターの提案。
- [ ] User に呈報し、裁決を仰ぐ。
- [ ] **doc-keeper** を派遣して裁決結果を記録する。

---

## 故障復旧マニュアル

### ケース A： Phase 2 の実装で agent_dev を壊してしまった（ビルド不能、ロジック崩壊）
```bash
# Phase 2 開始前の状態に復元
bash scripts/restore.sh $CYCLE baseline
```
復元後、 Phase 2 からやり直す。

### ケース B：同期スクリプトが途中で止まった（ agent_test が不完全）
```bash
# 再実行するだけでよい（ステージング戦略により、中断時は agent_test は旧状態を維持）
bash scripts/sync-to-test.sh
```
agent_dev 自体にも問題がある場合 → まず agent_dev を復元し（ケース A ）、その後再同期する。

### ケース C： Phase 4 で誤って PASS と判定し、壊れたコードをスナップショットした
```bash
# 壊れたスナップショットを削除
rm -rf share/openstarry_code_iteration/$CYCLE

# agent_dev を前回の正常なスナップショットに復元
bash scripts/restore.sh {前回_cycle_id} snapshot
```
その後、反復サイクル全体をやり直す。

### ケース D：エージェントセッションが途中でクラッシュした（ファイルの書き込み途中など）
1. agent_dev 内のファイル状態を確認する。
2. コードに不整合がある場合 → `bash scripts/restore.sh $CYCLE baseline`
3. 報告書が書きかけなだけの場合 → 対応するエージェントを再派遣する。

### ケース E：より古いバージョンに戻したい
```bash
# 利用可能なすべてのスナップショットを表示
ls share/openstarry_code_iteration/

# 指定したバージョンに復元
bash scripts/restore.sh {任意の_cycle_id} snapshot
```

### 予防措置
- **Phase 1.5 の Baseline 作成は必須ステップ**であり、スキップ不可。
- Phase 4 PASS 後のスナップショットは永久に保存される（手動削除時を除く）。
- agent_dev の状態が不明な場合は、まず `cd agent_dev/openstarry && pnpm build` で検証すること。
