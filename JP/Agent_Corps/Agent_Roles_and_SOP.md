# OpenStarry Agent Corps — 役割と標準操作手順 (Roles & SOP)

設立日：2026-02-09
最終更新日：2026-02-09 (PMP 監査 v2)

---

## 1. 代理人名簿 (Agent Roster)

| Agent | 役割 | モデル | 作業ディレクトリ | 出力ディレクトリ |
|-------|------|-------|-------------------|------------------|
| `architect` | アーキテクチャの守護者。設計仕様、コードレビュー、セキュリティ監査を担当 | sonnet | share/openstarry_doc/, agent_dev/ (読み取り専用) | share/test/reports/arch_reviews/ |
| `dev-core` | コア monorepo 開発 (sdk, core, shared, runner) | sonnet | agent_dev/openstarry/ | share/test/reports/dev_logs/ |
| `dev-plugin` | プラグインエコシステム開発 | sonnet | agent_dev/openstarry_plugin/ | share/test/reports/dev_logs/ |
| `qa` | 品質保証 — ビルド、テスト、純度検証 | sonnet | agent_test/ | share/test/reports/qa_results/ |
| `doc-keeper` | ドキュメント、意思決定の永続化、計画の追跡 | haiku | share/openstarry_doc/ | share/openstarry_doc/ |
| `researcher` | 技術的な事前調査、リファレンスプロジェクトの分析 | sonnet | share/ref/ | share/test/reports/research/ |
| **Coordinator** | メインセッション — オーケストレーション、同期、収束、エスカレーション | opus | すべて | share/test/reports/sys_summary/ |

---

## 2. RACI マトリックス

**R** = Responsible (実行責任者), **A** = Accountable (最終責任者), **C** = Consulted (相談先), **I** = Informed (報告先)

| Phase | Coordinator | architect | dev-core | dev-plugin | qa | doc-keeper | researcher | User |
|-------|:-----------:|:---------:|:--------:|:----------:|:--:|:----------:|:----------:|:----:|
| **Phase 0: Planning** | **R/A** | C | I | I | I | **R** (記録) | **R** (調査) | **A** (承認) |
| **Phase 1: Design** | I | **R/A** | C | C | I | **R** (設計決定の記録) | C (調査結果の提供) | I |
| **Phase 1.5: Baseline** | **R/A** | I | I | I | I | I | I | I |
| **Phase 2: Implement** | I | C (仕様の明確化) | **R/A** (コア) | **R/A** (プラグイン) | I | I | R (次ターンの調査) | I |
| **Phase 2.5: Sync** | **R/A** | I | I | I | I | I | I | I |
| **Phase 3: Verify** | I | **R** (コードレビュー) | I | I | **R/A** (テスト) | I | I | I |
| **Phase 4: Converge** | **R/A** | I | I | I | I | **R** (ドキュメント更新) | I | **A** (合否判定 PASS/FAIL) |
| **Rework** | **R** (割り当て) | C/R (設計上の問題の場合) | R (コアの問題の場合) | R (プラグインの問題の場合) | I | I | I | I |
| **Escalation** | **R** (提報) | C | C | C | C | I | I | **A** (裁決) |

---

## 3. 標準的な反復（イテレーション）サイクル

### 命名規則

各反復サイクルの ID 形式： `{YYYYMMDD}_cycle{N}`

例： `20260210_cycle1` 、 `20260210_cycle2` （同日の2回目）

そのターンのすべての報告書は、対応する cycle サブディレクトリに配置されます：
```
share/test/reports/arch_reviews/20260210_cycle1/Architecture_Spec_Plan05.1.md
share/test/reports/dev_logs/20260210_cycle1/dev-core_Plan05.1.md
share/test/reports/qa_results/20260210_cycle1/QA_Report_Plan05.1.md
share/test/reports/research/20260210_cycle1/Research_SessionIsolation.md
share/test/reports/sys_summary/20260210_cycle1/Summary.md
```

---

### Phase 0: Planning

**トリガー条件**： User からの指示、または前回の Phase 4 PASS 後の Coordinator による次回提案。

**参加者**： Coordinator (R/A), researcher (R), doc-keeper (R), User (A)

**ステップ**：
1. **Coordinator** は User の指示を受け取り、対応する Implementation Plan を特定します。
2. **Coordinator** は今回の cycle ディレクトリを作成します（すべての reports サブディレクトリ下）。
3. **researcher** はそのプランの技術方案を事前調査し（シフトレフト）、調査報告書を作成します。
4. **doc-keeper** は今回の反復計画を `share/openstarry_doc/Agent_Corps/Iteration_Log.md` に記録します。
5. **Coordinator** は調査報告書が整ったことを確認し、 architect に Phase 1 への移行を通知します。

**成果物**：
| 成果物 | パス | 責任者 |
|------|------|--------|
| 調査報告書 | `share/test/reports/research/{cycle_id}/Research_{Topic}.md` | researcher |
| 反復計画の記録 | `share/openstarry_doc/Agent_Corps/Iteration_Log.md` (追記) | doc-keeper |

**完了基準 (Exit Criteria)**：
- [ ] 調査報告書が存在し、内容が完全であること。
- [ ] 反復計画が記録されていること。
- [ ] Coordinator が Phase 1 への移行を確認していること。

---

### Phase 1: Design

**開始基準 (Entry Criteria)**： Phase 0 の完了基準をすべて満たしていること。

**参加者**： architect (R/A), doc-keeper (R), researcher (C)

**ステップ**：
1. **architect** は以下を読み込みます：
   - Implementation Plan： `share/openstarry_doc/Implementation_Plans/Plan{XX}_{Name}.md`
   - 調査報告書： `share/test/reports/research/{cycle_id}/Research_{Topic}.md`
   - 既存のアーキテクチャコード： `agent_dev/` （読み取り専用）
2. **architect** は Architecture_Spec を作成します。これには以下を含める必要があります：
   - **凍結された Interface 定義**（TypeScript 型定義）— dev-core と dev-plugin の共通の依拠となります。
   - 技術的制約と設計決定の理由。
   - 五蘊 (Five Aggregates) への帰属。
   - セキュリティ上の考慮事項。
3. **doc-keeper** は設計決定の理由を `share/openstarry_doc/Agent_Corps/Iteration_Log.md` に記録します。

**成果物**：
| 成果物 | パス | 責任者 |
|------|------|--------|
| Architecture_Spec | `share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md` | architect |
| 設計決定の記録 | `share/openstarry_doc/Agent_Corps/Iteration_Log.md` (追記) | doc-keeper |

**完了基準 (Exit Criteria)**：
- [ ] Architecture_Spec が存在すること。
- [ ] 仕様書に凍結された Interface 定義が含まれていること。
- [ ] 設計決定が記録されていること。
- [ ] Coordinator が仕様書の完成を確認し、 Phase 2 への移行を認めていること。

**重要**： Architecture_Spec 内の Interface 定義は、発行された時点で **凍結** されます。 Phase 2 期間中に修正が必要な場合は、エスカレーションプロセス（第6節参照）を経る必要があります。

---

### Phase 1.5: Baseline（安全網）

**開始基準 (Entry Criteria)**： Phase 1 の完了基準をすべて満たしていること。

**参加者**： Coordinator (R/A)

**目的**： Phase 2 の実装を開始する前に、現在の agent_dev の状態をバックアップします。 Phase 2 が失敗し修復不可能な場合、迅速にロールバックできます。

**実行**： `bash scripts/baseline.sh {cycle_id}`

**成果物**： `share/openstarry_code_iteration/{cycle_id}_baseline/`

**復元コマンド**： `bash scripts/restore.sh {cycle_id} baseline`

---

### Phase 2: Implementation

**開始基準 (Entry Criteria)**： Phase 1.5 の Baseline が確立されていること。

**参加者**： dev-core (R/A), dev-plugin (R/A), architect (C), researcher (R 次ターンの調査)

**実行順序**：

```
Step 2a: dev-core が SDK Interface の変更を実装（ある場合）
         → pnpm build で検証
         ↓
Step 2b: dev-core (残りの実装) + dev-plugin (プラグインの実装) が並行して動作
         → 各自 pnpm build で検証
```

Architecture_Spec で SDK Interface の変更がない場合、 Step 2a はスキップされ、 dev-core と dev-plugin は直接並行して作業します。

**ステップ (dev-core)**：
1. 仕様書を読み込む： `share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md`
2. Interface の変更がある場合：まず `packages/sdk/` を修正し、 `pnpm build` を実行してコンパイルを確認します。
3. `packages/core/` 、 `packages/shared/` 、 `apps/runner/` 関連の変更を実装します。
4. `pnpm build` を実行して検証します。
5. 開発ログ (dev log) を記述します。

**ステップ (dev-plugin)**：
1. 仕様書を読み込む（上記と同じパス）。
2. **Step 2a の完了を待機します**（ SDK Interface の変更がある場合）。
3. プラグインの変更を実装します。
4. `pnpm build` を実行して検証します。
5. 開発ログ (dev log) を記述します。

**ステップ (researcher, 並行)**：
- 次の反復サイクルの技術方案を事前調査します（判明している場合）。

**成果物**：
| 成果物 | パス | 責任者 |
|------|------|--------|
| dev-core ログ | `share/test/reports/dev_logs/{cycle_id}/dev-core_{PlanName}.md` | dev-core |
| dev-plugin ログ | `share/test/reports/dev_logs/{cycle_id}/dev-plugin_{PlanName}.md` | dev-plugin |

**完了基準 (Exit Criteria)**：
- [ ] dev-core の `pnpm build` が PASS していること。
- [ ] dev-plugin の `pnpm build` が PASS していること（プラグインの変更がある場合）。
- [ ] 開発ログが記述されていること。
- [ ] Coordinator がすべてのビルド通過を確認し、 Phase 2.5 への移行を認めていること。

---

### Phase 2.5: Sync to Test Environment

**開始基準 (Entry Criteria)**： Phase 2 の完了基準をすべて満たしていること。

**参加者**： Coordinator (R/A)

**目的**： `agent_dev/` の最新コードを `agent_test/` に同期し、 QA が最新バージョンをテストできるようにします。

**クイック実行**： `bash scripts/sync-to-test.sh` （ openstarry_eco ルートから実行）

**同期 SOP（スクリプト内容）**：

```bash
# ステップ 1: agent_test の古いソースコードを消去（ node_modules は高速化のため保持）
rm -rf agent_test/openstarry/packages agent_test/openstarry/apps
rm -rf agent_test/openstarry/dist agent_test/openstarry/.turbo

# ステップ 2: ソースコードをコピー（ node_modules, dist, .turbo を除外）
cp -r agent_dev/openstarry/packages agent_test/openstarry/packages
cp -r agent_dev/openstarry/apps agent_test/openstarry/apps

# ルート階層の設定ファイルをコピー（変更がある場合）
cp agent_dev/openstarry/package.json agent_test/openstarry/package.json
cp agent_dev/openstarry/tsconfig*.json agent_test/openstarry/
cp agent_dev/openstarry/vitest*.* agent_test/openstarry/ 2>/dev/null
cp agent_dev/openstarry/pnpm-workspace.yaml agent_test/openstarry/ 2>/dev/null

# ステップ 3: プラグインを同期（同様の戦略）
for dir in agent_dev/openstarry_plugin/*/; do
  plugin=$(basename "$dir")
  rm -rf "agent_test/openstarry_plugin/$plugin/src" "agent_test/openstarry_plugin/$plugin/dist"
  cp -r "agent_dev/openstarry_plugin/$plugin/src" "agent_test/openstarry_plugin/$plugin/src"
  cp "agent_dev/openstarry_plugin/$plugin/package.json" "agent_test/openstarry_plugin/$plugin/package.json"
  cp "agent_dev/openstarry_plugin/$plugin/tsconfig.json" "agent_test/openstarry_plugin/$plugin/tsconfig.json" 2>/dev/null
done
cp agent_dev/openstarry_plugin/package.json agent_test/openstarry_plugin/package.json 2>/dev/null
cp agent_dev/openstarry_plugin/pnpm-workspace.yaml agent_test/openstarry_plugin/pnpm-workspace.yaml 2>/dev/null

# ステップ 4: 依存関係のインストール
cd agent_test/openstarry && pnpm install
cd agent_test/openstarry_plugin && pnpm install

# ステップ 5: 基本ビルドの検証
cd agent_test/openstarry && pnpm build
```

**完了基準 (Exit Criteria)**：
- [ ] agent_test のソースコードが同期されていること。
- [ ] `pnpm install` が成功していること。
- [ ] `pnpm build` が成功していること。
- [ ] Coordinator が Phase 3 への移行を確認していること。

**失敗時の処理**：同期後にビルドが失敗した場合、問題は Phase 2 にあります。 Phase 2 に戻って修正します（ Phase 3 には進みません）。

---

### Phase 3: Verification

**開始基準 (Entry Criteria)**： Phase 2.5 の完了基準をすべて満たしていること。

**参加者**： qa (R/A), architect (R)

**並行実行可能**： qa と architect は、互いに依存することなく同時に進行できます。

**ステップ (qa)**：
1. `agent_test/openstarry/` で以下を実行します：
   - `pnpm build` — ビルド検証。
   - `pnpm test` — ユニットテスト（ベースライン：82テスト）。
   - `pnpm test:purity` — マイクロカーネル純度チェック。
2. `agent_test/openstarry_plugin/` で以下を実行します：
   - `pnpm build` — プラグインビルド検証。
   - `pnpm test` — プラグインテスト（存在する場合）。
3. QA レポートを作成します。

**ステップ (architect)**：
1. 仕様書を読み込む： `share/test/reports/arch_reviews/{cycle_id}/Architecture_Spec_{PlanName}.md`
2. 実装コードを読み込む： `agent_dev/` （読み取り専用）。
3. 各項目を検証します：
   - Interface の実装が凍結された仕様書に従っているか。
   - 五蘊 (Five Aggregates) への適合。
   - マイクロカーネルの純度。
   - pushInput パターンの遵守。
   - セキュリティ上の脆弱性。
4. コードレビュー報告書（ PASS / FAIL / CONDITIONAL ）を作成します。

**成果物**：
| 成果物 | パス | 責任者 |
|------|------|--------|
| QA レポート | `share/test/reports/qa_results/{cycle_id}/QA_Report_{PlanName}.md` | qa |
| コードレビュー | `share/test/reports/arch_reviews/{cycle_id}/Code_Review_{PlanName}.md` | architect |

**完了基準 (Exit Criteria)**：
- [ ] QA レポートが存在すること。
- [ ] コードレビュー報告書が存在すること。
- [ ] Coordinator が両方のレポートを確認し、 Phase 4 への移行を認めていること。

---

### Phase 4: Convergence

**開始基準 (Entry Criteria)**： Phase 3 の完了基準をすべて満たしていること。

**参加者**： Coordinator (R/A), doc-keeper (R), User (A)

**ステップ：**
1. **Coordinator** は QA レポートとコードレビュー報告書を読み込みます。
2. **Coordinator** はそれらを Summary Report にまとめます。
3. **Coordinator** が判定を下します：

   **→ すべて PASS の場合：**
   - スナップショット： `bash scripts/snapshot.sh {cycle_id}` （ node_modules/dist を除外）。
   - **doc-keeper** が更新：
     - Implementation Plan の完了項目に ✅ を入れる。
     - Iteration_Log.md に今回の結果を追記する。
   - **Coordinator** は User に結果を報告し、次の方針を仰ぎます。

   **→ いずれかが FAIL の場合：** リワーク (Rework) フローに入ります（第4節参照）。

**成果物**：
| 成果物 | パス | 責任者 |
|------|------|--------|
| Summary Report | `share/test/reports/sys_summary/{cycle_id}/Summary.md` | Coordinator |
| 更新された反復ログ | `share/openstarry_doc/Agent_Corps/Iteration_Log.md` | doc-keeper |
| 更新されたプラン（ ✅ ） | `share/openstarry_doc/Implementation_Plans/` | doc-keeper |

---

## 4. リワーク (Rework) フロー（ Phase 4 FAIL 時）

### 4.1 問題の分類

| レベル | 内容 | 戻り先 | 例 |
|------|------|--------|------|
| **Code Fix** | 実装バグ、テスト失敗、ビルドエラー。 | Phase 2（部分的な修正のみ） | 型エラー、テストのアサーション失敗。 |
| **Design Fix** | 仕様書自体に欠陥がある、または不完全。 | Phase 1（ architect が仕様を改訂） | Interface 設計の不備、エッジケースの漏れ。 |
| **Plan Fix** | プランの要件自体に問題がある。 | Phase 0（再計画） | 技術方案が実現不可能、要件の矛盾。 |

### 4.2 リワーク SOP

1. **Coordinator** は、 QA レポートおよびコードレビュー報告書内の FAIL 項目を確認します。
2. **Coordinator** は各 FAIL 項目を分類します（ Code Fix / Design Fix / Plan Fix ）。
3. **Coordinator** はリワークタスクを作成し、 Summary Report に記述します：
   ```
   ## Rework Tasks
   - [ ] [FAIL-1] (Code Fix) 内容の説明 → dev-core/dev-plugin に割り当て
   - [ ] [FAIL-2] (Design Fix) 内容の説明 → architect に割り当て
   ```
4. **Coordinator** は Baseline を再作成します： `bash scripts/baseline.sh {cycle_id}_rework{N}` （二次ロールバックに備え、リワーク前の状態を保存）。
5. 分類に基づき、対応する Phase に戻ります。 **FAIL した部分のみをやり直します** 。
6. 修正完了後、修正した Phase の次の Phase から再びフローを進めます（ Phase 2.5 の同期は必須です）。

### 4.3 リワークの上限

- 同一反復サイクル内でのリワークは、最大 **2回** までとします。
- 2回を超えた場合 → **User へのエスカレーション** を行い、裁決を仰ぎます（第6節参照）。

---

## 5. Phase 2 並行戦略： Interface 凍結メカニズム

### 5.1 原則

Architecture_Spec で定義された **TypeScript Interface** は、 Phase 1 が完了した時点で **凍結** されます。 dev-core と dev-plugin は、この凍結されたインターフェースを共通の基盤として開発を行います。

### 5.2 実行順序

```
Architecture_Spec に SDK Interface の変更が含まれているか？
│
├── はい → dev-core が先に SDK Interface を実装（Step 2a）
│        → pnpm build が通過した後
│        → dev-core (残り) + dev-plugin が並行作業（Step 2b）
│
└── いいえ → dev-core + dev-plugin が直接並行作業を開始
```

### 5.3 凍結期間中の例外処理

実装中に dev-core が Interface の修正が不可欠であると判断した場合：
1. dev-core は **実装を一時停止** します。
2. dev-core は開発ログに問題を記録します。
3. **Coordinator** が architect に通知します。
4. architect が評価を行い、同じ cycle ディレクトリに **Spec Addendum** （追加修正事項）を発行します。
5. dev-plugin は Addendum を読み込んだ後、調整を行います。
6. 影響範囲が大きすぎる場合 → User へエスカレーションします。

---

## 6. エスカレーションと紛争解決

### 6.1 エスカレーション条件

| 状況 | エスカレーションパス |
|------|---------|
| エージェント間で仕様の解釈に相違がある | → Coordinator による仲裁 |
| Coordinator が仲裁できない（設計の方向性の問題） | → User による裁決 |
| リワークが2回を超えた | → User による裁決 |
| Phase 2 で Interface の大幅な修正が必要と判明 | → User による裁決 |
| 技術方案が実現不可能である（ researcher による発見） | → User による裁決 |

### 6.2 エスカレーションフロー

1. 発見した側が、レポート内に `⚠️ ESCALATION NEEDED` と明確に記します。
2. **Coordinator** は各方面の意見を収集し、意思決定サマリーとしてまとめます。
3. **Coordinator** は User に対して以下を提示します：
   - 問題の説明
   - 各方面の立場
   - Coordinator からの提案（ある場合）
4. **User** が裁決を下します。
5. **doc-keeper** は裁決内容を Iteration_Log.md に記録します。

---

## 7. doc-keeper の全プロセスへの関与スケジュール

| Phase | doc-keeper の責務 |
|-------|----------------|
| Phase 0 | 記録：今回の反復目標、関連するプラン、調査の方向性。 |
| Phase 1 | 記録： architect の設計決定理由、凍結されたインターフェースの内容。 |
| Phase 1.5 | 能動的なタスクなし（ Coordinator が Baseline 作成）。 |
| Phase 2 | 待機： Spec Addendum 受領時に変更内容を記録。 |
| Phase 3 | 待機：能動的なタスクなし。 |
| Phase 4 PASS | プランの更新 (✅)、反復結果の記述、ロードマップの更新。 |
| Phase 4 FAIL | FAIL の原因とリワーク決定事項の記録。 |

---

## 8. 通信プロトコル (Communication Protocol)

Coordinator がエージェントを派遣する際、エージェントが推測で動いたり、誤ったコンテキストで作業したりするのを防ぐため、プロンプトには以下の情報を含める必要があります。

### 8.1 必須フィールド（すべての派遣に共通）

各エージェントへの派遣プロンプトには、 **必ず以下を含める** 必要があります：

| フィールド | 説明 | 例 |
|------|------|------|
| `cycle_id` | 今回の反復サイクル ID | `20260210_cycle1` |
| `plan` | 対象となるプラン名 | `Plan05.1 Session Isolation` |
| `task` | 具体的に何をするか（一言で） | 「 Session Isolation の技術方案を事前調査する」 |
| `output_path` | 報告書や成果物をどこに書き出すか | `share/test/reports/research/20260210_cycle1/` |

### 8.2 フェーズ固有のフィールド

| Phase | Agent | 追加の必須フィールド |
|-------|-------|-------------|
| Phase 0 | researcher | `research_topic`, `reference_projects` （ share/ref/ 内のどのディレクトリをスキャンするか）。 |
| Phase 0 | doc-keeper | `iteration_goals` （今回の目標サマリー）。 |
| Phase 1 | architect | `plan_path` （プランファイルへのパス）、 `research_report_path` （調査報告書へのパス）。 |
| Phase 1 | doc-keeper | `spec_summary` （仕様書の主要な設計決定サマリー）。 |
| Phase 2a | dev-core | `spec_path` 、「 SDK Interface の変更のみを実装せよ」という明確な指示。 |
| Phase 2b | dev-core | `spec_path` 、「 SDK Interface は完了。残りの部分を実装せよ」という指示。 |
| Phase 2 | dev-plugin | `spec_path` 、 SDK 変更がある場合は「 SDK Interface 準備完了」という指示。 |
| Phase 3 | qa | `cycle_id` （ qa は agent_test/ で作業するため、追加パスは不要）。 |
| Phase 3 | architect | `spec_path` （コードレビューの比較対象となる仕様書へのパス）。 |
| Phase 4 | doc-keeper | `qa_result` (PASS/FAIL), `review_result` (PASS/FAIL), `lessons_learned` 。 |

---

## 9. 経験の教訓 (Lessons Learned)

### 9.1 タイミング

各反復サイクルにおける **Phase 4 の終了後** （合否にかかわらず）、 Coordinator は振り返りを実施します。

### 9.2 振り返りプロセス

1. **Coordinator** は今回の反復を振り返り、以下の問いに答えます：
   - 何がうまくいったか？（プロセス、協調、品質）。
   - 何がうまくいかなかったか？（詰まった点、リワークの原因、通信の問題）。
   - 次回はどう改善できるか？（プロセスの調整、プロンプトの最適化、チェック項目の追加）。
2. **Coordinator** は振り返り内容を Summary Report の `## Lessons Learned` セクションに記述します。
3. **doc-keeper** は永続化可能な改善項目を `share/openstarry_doc/Agent_Corps/Lessons_Learned.md` に追記します。
4. SOP 、エージェントの定義、チェックリストの修正が必要と判明した場合 → Coordinator が次回の Phase 0 で実行します。

---

## 10. 故障復旧 (Fault Recovery)

### 10.1 復旧スクリプト

| スクリプト | 用途 | コマンド |
|------|------|------|
| `scripts/baseline.sh` | Phase 2 開始前に安全網を構築 | `bash scripts/baseline.sh {cycle_id}` |
| `scripts/restore.sh` | Baseline またはスナップショットから agent_dev を復元 | `bash scripts/restore.sh {cycle_id} baseline\|snapshot` |
| `scripts/sync-to-test.sh` | agent_dev から agent_test への同期 | `bash scripts/sync-to-test.sh` |
| `scripts/snapshot.sh` | Phase 4 PASS 後のアーカイブ保存 | `bash scripts/snapshot.sh {cycle_id}` |

### 10.2 復旧シナリオ

| シナリオ | 復旧方法 |
|------|---------|
| Phase 2 で agent_dev を壊してしまった | `restore.sh {cycle_id} baseline` → Phase 2 をやり直す。 |
| 同期が途中で中断した | `sync-to-test.sh` を再実行。 |
| 誤った PASS 判定で壊れたスナップショットを作成した | 不正なスナップショットを削除 → `restore.sh {前回サイクル} snapshot` 。 |
| エージェントセッションがクラッシュした | ファイルをチェック → 不整合があれば restore baseline → エージェントを再派遣。 |
| リワーク後に再び失敗した | `restore.sh {cycle_id}_rework{N} baseline` 。 |

---

## 11. 設計原則

1. **Roles ≠ Tasks** — 各エージェントは恒久的な役割であり、単一のタスクのために作成されるものではありません。
2. **All Tier 1** — 6つのエージェントはすべてコアチームであり、階層はありません。
3. **Extensible** — 規定のプランが完了した後、同じエージェントたちが自然に拡張タスクを引き継ぎます。
4. **Decision Persistence** — すべての決定事項はファイルに記述されなければなりません。記憶の中だけに存在させることは許されません。
5. **Quality Gates** — 各フェーズには明確な開始/完了基準があり、それを満たさなければ次に進めません。
6. **Interface Freeze** — 仕様書の発行後、インターフェースは凍結されます。変更にはエスカレーションが必要です。

---

## 12. 拡張モード (Extended Mode)

トリガー条件： `share/openstarry_doc/Implementation_Plans/` 内のすべてのプランが完了したとき ✅

Coordinator は評価を行い、続行するか User に尋ねます：
- **researcher** → リファレンスプロジェクトを深く分析し、新しいプラグインの構想と実現可能性レポートを作成。
- **architect** → 全面的なセキュリティ監査とサンドボックス強化案の作成。
- **dev-core/dev-plugin** → 新機能（ Web UI 、新しいプラグインなど）の実装。
- **qa** → 拡張機能の検証。
- **doc-keeper** → すべての拡張作業の決定事項と成果を記録。
