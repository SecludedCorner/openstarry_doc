# OpenStarry 実装計画 02 — アーキテクチャの補完と品質向上

> **ステータス**: ✅ 完了 (2026-02-05)

## 背景

Plan01 で MVP v0.1 Alpha の基礎骨格（コンパイル可能、起動可能、指示受信可能）が完成しました。
しかし、テスターによる検証報告書（ `test/20260204/` ）および `openstarry_doc` 設計ドキュメントとの比較により、
いくつかの **実装と設計の乖離 (Gaps)** が特定されました。

本計画では Plan02 の改善範囲を定義し、 **フェーズ 2.3 / フェーズ 3.1 の欠落を補完し、設計ドキュメントの要件を達成すること** を目標とします。

---

## 一、テスト報告書で特定された問題

| # | ソース | 問題 | 深刻度 |
|---|------|------|--------|
| G1 | Developer_Handoff_Report / Engineering_Implementation_Gaps | CLI プラグインのロードがハードコード（ `switch-case` ）されており、サードパーティ製プラグインを動的に `import()` できない | 🔴 高 |
| G2 | Engineering_Implementation_Gaps | `ExecutionLoop.run(string)` が文字列を受け取っており、イベント駆動ではない。ドキュメントの「コアの唯一の入力ソースはイベントキューである」という記述と不一致 | 🔴 高 |
| G3 | QA_MultiInput_Verification_Report | ツール実行中の非同期競合リスク — 新しい入力と現在のループ状態で競合が発生する可能性がある | 🟡 中 |
| G4 | Developer_Handoff_Report / QA_Report | 並行処理のユニットテストが不足しており、 `EventQueue` の FIFO の安定性が検証されていない | 🟡 中 |

---

## 二、設計ドキュメントとの比較 — 未実装機能

| # | ドキュメントソース | 欠落している機能 | 所属フェーズ |
|---|----------|----------|-----------|
| D1 | `02_Headless_Agent_Core.md` | ExecutionLoop は `WAITING_FOR_EVENT` ステートマシンであるべきで、文字列を直接受け取るのではなく EventQueue からイベントを pull すべきである | 2.1 |
| D2 | `12_Capabilities_Injection_Mechanism.md` | PluginLoader は、 `agent.json` のパス解決に基づいた動的な `import()` ロードをサポートすべきである | 3.1 |
| D3 | `07_Safety_Circuit_Breakers.md` | リソースレベルの遮断： Token 予算上限、ループ回数上限 (Loop Cap) | 2.3 |
| D4 | `07_Safety_Circuit_Breakers.md` | 行動レベルの遮断：重複ツール呼び出し検知、エラー連鎖遮断 | 2.3 |
| D5 | `12_Error_Handling_and_Self_Correction.md` | 「エラーは痛覚」メカニズム：ツールエラーの標準化 + 挫折カウンター + 強制的な助けの要求 | 2.3 |
| D6 | `09_Observability_and_Tracing.md` | 構造化 JSON ログ（ agent_id, trace_id, module を含む） | 3.1 |
| D7 | `15_Testing_Strategy_and_Infrastructure.md` | ユニットテスト基盤 (Vitest) 、コア純度チェック、 MockHost | 1.3 |
| D8 | `01_Execution_Loop.md` | 出力ルーティング：イベントの `source` に基づいて正しいチャネルに応答を返す | 2.1 |

---

## 三、 Plan02 実装ステップ

### フェーズ A：イベント駆動への転換（ G1, G2, D1, D2, D8 に対応）

**目標**： ExecutionLoop が文字列を直接受け取るのをやめ、 EventQueue を監視するように変更。 CLI での動的なプラグインロードをサポート。

#### A1. ExecutionLoop のイベント化リファクタリング
- `ExecutionLoop` に `start()` メソッドを追加： `EventQueue.pull()` からイベントを継続的に取り出す
- イベントペイロードの標準化： `{ source: string, type: string, data: unknown }`
- `run(userInput)` を内部メソッドに格下げし、イベントトリガーからのみ呼び出されるようにする
- `isProcessing` ロックを追加し、複数イベントの同時処理を防止（ G3 の競合問題を解決）
- 出力ルーティング： `event.source` に基づいて返信チャネルを決定

#### A2. AgentCore の接続
- `AgentCore.processInput()` を、ループを直接呼び出すのではなく EventQueue にプッシュするように変更
- リスナープラグインは EventQueue にイベントをプッシュする（ AgentCore を直接呼び出さない）
- スラッシュコマンドのファストパスを維持（ LLM ループに入らない）

#### A3. CLI の動的プラグインロード
- `DynamicPluginLoader` を新設： `agent.json` の `plugins[].path` に基づいて `import()` で動的にロード
- 組み込みプラグインのファストパスを維持（パスがない場合は switch-case へフォールバック）
- `node_modules` のパッケージ名解決をサポート（ `import(pluginName)` ）

#### A4. イベントペイロードの標準化
- `InputEvent` 型を定義：
  ```typescript
  interface InputEvent {
    source: string;      // "cli", "webhook", "mcp"
    type: string;        // "user_input", "system_command"
    data: unknown;
    replyTo?: string;    // 返信チャネルの識別子
  }
  ```

---

### フェーズ B：安全遮断と自己修復（ D3, D4, D5 に対応）

**目標**：暴走や無限ループを防止するため、 `SafetyMonitor` モジュールを実装。

#### B1. SafetyMonitor モジュール
- **場所**： `packages/core/src/security/safety-monitor.ts`
- **責務**：
  - `beforeLLMCall()`: Token 予算のチェック
  - `afterToolExecution()`: 重複呼び出し検知 + エラー率の計算
  - `onLoopTick()`: ループ回数上限のチェック

#### B2. リソースレベルの遮断
- `MAX_LOOP_TICKS`：単一タスクの最大ループ回数（デフォルト 50 、 `policy` で設定可能）
- `MAX_TOKEN_USAGE`： Token 累積消費上限（プロバイダーが返す `usage` に依存）
- トリガー後、状態を `SAFETY_LOCKOUT` に切り替え

#### B3. 行動レベルの遮断
- `ToolCallFingerprint` 履歴キュー： hash(toolName + args)
- 同じ失敗指紋が N 回（例：3回）連続した場合 → システムプロンプトを強制注入：「 STOP and analyze why 」
- スライディングウィンドウによるエラー率（直近10回の操作中8回失敗 → `EMERGENCY_HALT` ）

#### B4. 挫折カウンターと自己修復
- 連続失敗カウンター
- 閾値（デフォルト 5 ）を超えた場合 → システムプロンプトを強制注入：「 ask the user for help 」
- ツールエラーを `{ code, message, suggestion }` 構造に標準化

---

### フェーズ C：テスト基盤（ G4, D7 に対応）

**目標**： Vitest テスト環境を構築し、コアロジックをカバー。

#### C1. テストフレームワークの設定
- ルートディレクトリに Vitest 設定を追加
- 各パッケージに `test` スクリプトを追加

#### C2. コア・ユニットテスト
- `EventBus.test.ts`：複数ハンドラー、ワイルドカード、エラー隔離
- `EventQueue.test.ts`： FIFO 順序、並行 push/pull 、負荷テスト
- `StateManager.test.ts`： add/clear/snapshot/restore
- `ContextManager.test.ts`：スライディングウィンドウ切り詰めロジック
- `SecurityLayer.test.ts`：パス検証の正確性
- `SafetyMonitor.test.ts`：各段階の遮断トリガー条件

#### C3. 統合テスト
- `MultiSourceEvent.test.ts`：複数ソースからのイベント同時注入のシミュレーション
- `ExecutionLoop.test.ts`：プロバイダーを mock し、完全なループフローを検証
- `PluginLoader.test.ts`：動的ロードの検証

#### C4. コア純度チェック
- `dependency-cruiser` または ESLint ルールを追加
- `packages/core` が `plugins/*` や `apps/*` をインポートしていないことを確認

---

### フェーズ D：可観測性の向上（ D6 に対応）

**目標**：構造化 JSON ログの実装。

#### D1. Logger のアップグレード
- `packages/shared/src/logger` を JSON 形式で出力するように更新
- `agent_id`, `trace_id` フィールドを追加
- ログレベルによるフィルタリングをサポート（環境変数 `LOG_LEVEL` ）

---

## 四、実装の優先順位

```
フェーズ A (イベント駆動への転換)     ← 🔴 最高優先度。テスト報告の核心的問題
  └→ A1 ExecutionLoop のイベント化
  └→ A2 AgentCore の接続
  └→ A3 動的プラグインロード
  └→ A4 イベントペイロードの標準化

フェーズ B (安全遮断)                ← 🔴 高。フェーズ 2.3 の未完了項目
  └→ B1 SafetyMonitor
  └→ B2 リソースレベルの遮断
  └→ B3 行動レベルの遮断
  └→ B4 挫折カウンター

フェーズ C (テスト基盤)              ← 🟡 中。フェーズ 1.3 の未完了項目
  └→ C1 Vitest 設定
  └→ C2 コア・ユニットテスト
  └→ C3 統合テスト
  └→ C4 純度チェック

フェーズ D (可観測性)                ← 🟢 低。他のフェーズと並行可能
  └→ D1 Logger のアップグレード
```

---

## 五、完了後の予定状態

Plan02 完了後、プロジェクトは以下の状態に達します：

- **フェーズ 1.3** ✅ テスト基盤 + CI ルール
- **フェーズ 2** ✅ 完全な意識カーネル（安全遮断、イベント駆動、自己修復を含む）
- **フェーズ 3.1** ✅ 動的なプラグインロード + イベントの標準化
- `openstarry_doc` におけるフェーズ 1〜3.1 のすべての設計要件が実装済み

**フェーズ 3 完了まで残り**：
- 3.2 `openstarry plugin sync` コマンド
- 3.2 `guide-mcp` + `standard-function-skill` プラグイン
- 3.4 `openstarry create-plugin` スキャフォールディング + `MockHost`

**フェーズ 4 完了まで残り**：
- 4.2 エンドツーエンドの LLM 通話検証（ OAuth ログイン後の手動テストが必要）

---

## 六、検証方法

1. `pnpm install && pnpm build` — コンパイル通過
2. `pnpm test` — すべてのユニットテスト + 統合テストが通過
3. EventQueue 負荷テスト通過（1000イベントが正しい FIFO 順で処理されること）
4. 動的ロード： `agent.json` でカスタムプラグインパスを設定 → CLI が正常にロードすること
5. 安全遮断：連続失敗をシミュレート → SafetyMonitor が正しく halt をトリガーすること
6. 複数入力シミュレーション：2つのイベントソースを同時に注入 → 競合なく順番に処理されること
