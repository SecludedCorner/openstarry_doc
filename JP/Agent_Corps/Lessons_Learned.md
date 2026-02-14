# OpenStarry Agent Corps — Lessons Learned

各イテレーションの Phase 4 終了後、doc-keeper が当該サイクルの振り返りを追記する。

---

## 20260210_cycle1 + 20260211_cycle2

（Cycle 1/2 は Agent Corps 構築初期に完了。振り返りは Cycle 3 に統合。）

---

## 20260211_cycle3: MCP Client Plugin (v0.3.0-beta)

### What Went Well

1. **Researcher の事前調査が非常に有益だった** — MCP 事前調査レポート（openoctopus 参照分析と五蘊マッピングを含む）により、長時間の議論なしに迅速かつ確信を持って Phase 0 のスコープ決定が可能となった。
2. **カスタム JSON-RPC アプローチが正解だった** — 約400行のカスタムトランスポートコードにより、@modelcontextprotocol/sdk 依存の Zod バージョン競合を回避。クリーンでデバッグしやすく、外部 SDK への追従も不要。
3. **Phase 2a/2b の並列実行** — インターフェース凍結後、SDK 拡張とプラグイン実装を並行して進行可能となった。Phase 2a（SDK）は小規模で迅速に完了し、Phase 2b のブロック解除が早期に実現。
4. **Phase 3 における QA + Architect の並列実行** — 両方の検証ストリームが同時に完了。直列的なボトルネックなし。
5. **テストカバレッジが目標を上回った** — 目標 30-40 に対して 35 の新規テスト。schema、client、bridges、factory 全体にわたる包括的なカバレッジを達成。

### What Could Be Improved

1. **TypeScript strict モード + 外部型ブリッジ** — ビルド中に3種類の TS エラーが発生（config のダブルキャスト、JSON Schema から Zod への型不一致、union 型による再帰の破壊）。教訓：型が緩い外部プロトコル（JSON-RPC、JSON Schema）を厳格な TS にブリッジする場合、最初から明示的な境界レイヤー（内部型 + 公開アダプター関数）を計画すること。
2. **テストファイルの整理** — 35 テスト全てが単一の `index.test.ts` ファイルに格納。Architect がマイナーな問題として指摘。次のプラグインでは、最初からモジュール単位でテストファイルを分割すること（Architecture Spec のファイルインベントリに準拠）。
3. **vitest.config.ts の欠落** — Architecture Spec のファイルインベントリに記載されていたにもかかわらず、作成を忘れていた。dev-plugin チェックリストに追加：新パッケージには必ず vitest.config.ts を作成すること。
4. **サブエージェントの書き込み権限** — サブエージェントが share/ および agent_dev/ ディレクトリへの書き込みでブロックされた。回避策：coordinator がファイルを直接書き込み。サイクル開始前にエージェントの権限設定を確認する必要がある。

### Patterns to Reuse

1. **JSON Schema → Zod コンバーターパターン**：内部再帰関数（`convert(schema: InternalType)`）と公開アダプター（`publicFunc(schema: Record<string, unknown>)`）に分離。あらゆる JSON Schema ブリッジに再利用可能。
2. **MockTransport パターン**：メソッド→レスポンス Map を持つ `MockMcpTransport` クラスは、プロトコルテスト用のクリーンで再利用可能なテストパターン。
3. **Plugin factory の回復力**：マルチサーバー初期化における失敗時続行ループ（`for (const server of config.servers) { try { ... } catch { logger.error(...) } }`）— 全てのマルチリソースプラグインに適用可能な良いパターン。
4. **名前空間付きツール ID**：`serverName/toolName` パターンによりサーバー間の衝突を防止。複数のプラグインソースがツールを提供する場合にも同様のパターンを適用。

### Action Items for Future Cycles

- [ ] dev-plugin 新パッケージチェックリストに vitest.config.ts を追加
- [ ] 新プラグインではモジュール単位でテストファイルを分割（モノリシックな index.test.ts ではなく）
- [ ] サイクル開始前にサブエージェントの書き込み権限を確認
- [ ] MCP client でのプロトコルバージョン検証を検討（Architect レビューからのオプション強化）

---

## 20260211_cycle4: MCP Server Plugin (v0.3.1-beta)

### What Went Well

1. **Architecture Spec が包括的だった** — Architect が作成した仕様は、凍結済みインターフェース、設計決定、リスク評価、ファイルインベントリを全てカバー。実装はほぼ機械的に行えた。
2. **モジュール単位でテストファイルを分割** — Cycle 3 の教訓を適用：モノリシックではなく7つの個別テストファイル（tool-adapter、prompt-adapter、handler、stdio、http、plugin）。明確なカバレッジ境界で52テストを達成。
3. **vitest.config.ts を最初から作成** — Cycle 3 のアクションアイテムを適用。パッケージスケルトンの初期作成時に vitest.config.ts を含めた。
4. **zodToJsonSchema の再利用** — `@openstarry/shared` の `zodToJsonSchema` をリバースブリッジ（ITool → MCP ツール定義）に再利用して完璧に動作。新しいコンバーターは不要だった。
5. **Phase 3 の並列実行** — QA と Architect のコードレビューが同時に実行。両方とも数分で完了。
6. **テスト数が目標を上回った** — 仕様目標 30-40 に対して 52 テスト。全モジュールにわたる包括的なカバレッジ。

### What Could Be Improved

1. **サブエージェントの書き込み権限が依然としてブロック** — dev-plugin エージェント（Phase 2b）と QA エージェントの両方が share/ および openstarry_plugin/ ディレクトリへの書き込みでブロック。Coordinator がプラグイン全体を手動で実装し、全レポートを書き込む必要があった。この問題は3サイクル連続で発生。
2. **HTTP トランスポートテストのポート競合** — 初期 HTTP テストでは全テストで単一ポートを使用し、テスト並列実行時に "socket hang up" エラーが発生。テストごとに `nextPort()` カウンターでユニークなポートを使用することで修正。
3. **GET リクエストテストの失敗** — 初期 GET テストでは body を送信する `sendRequest` ヘルパーを使用し、ソケットの問題が発生。`http.get` を使用する別の `sendGetRequest` ヘルパーを作成して修正。
4. **zod devDependency の欠落** — 初期 package.json に `zod` が devDependency として含まれていなかった（テストでのみ使用）。`zod` から `z` をインポートするテストが "Cannot find package 'zod'" で失敗。迅速に修正。

### Patterns to Reuse

1. **HTTP テストごとのユニークポート**：`let portCounter = 39800; function nextPort() { return portCounter++; }` — テスト並列実行時のポート競合を防止。
2. **filterExposed ジェネリック**：`function filterExposed<T extends { id: string }>(items: T[], filter: string[] | "*" | undefined): T[]` — ID ベースのフィルタリング用の再利用可能なホワイトリストフィルター。
3. **Handler ファクトリーパターン**：`createMcpJsonRpcHandler(config, ctx)` は純粋な非同期関数 `(req) => Promise<response>` を返す。ハンドラーロジックとトランスポートのクリーンな分離。
4. **IPluginContext の遅延プロキシアクセサー**：オプショナルな `tools?` と `guides?` フィールドがレジストリに委譲し、ミューテーションを露出しない。将来のクロスプラグインレジストリアクセスニーズに再利用可能なパターン。

### Action Items for Future Cycles

- [ ] **重大**：Cycle 5 前にサブエージェントの書き込み権限を修正（3サイクルにわたる回避策）
- [ ] `@openstarry-plugin/mcp-common` 共有型パッケージを抽出（Architect Note 1）
- [ ] mcp-server パッケージに使用方法/ロード順序のガイダンスを含む README.md を追加（Architect Note 2）
- [x] モジュール単位でテストファイルを分割（完了 — Cycle 3 から適用）
- [x] 新パッケージに vitest.config.ts を作成（完了 — Cycle 3 から適用）

---

## 20260211_cycle5: Plan07 Sandbox MVP (v0.4)

（簡略化 — サンドボックス基礎に絞った3日間の迅速なサイクル。主要な教訓は Cycle 6-7 の改善に統合。）

### Highlights

1. **vm.createContext + worker_threads サンドイッチ** — 効果的な二層隔離（JS コンテキスト隔離 + OS プロセス境界）
2. **署名検証の統合がスムーズに動作** — SHA-512 ハッシュ検証は最小限で後方互換性あり
3. **サンドボックスエスケープテストにより攻撃面を早期に発見** — Phase 3 コードレビュー中に3つの潜在的なエスケープベクターを検出

### Items Deferred to 07.1+

- CPU ウォッチドッグ（MVP には複雑すぎるため、07.1 で追加）
- ワーカープール事前生成（最適化であり、MVP には不要）
- 非対称鍵署名（あれば便利、07.2 で追加）

---

## 20260211_cycle6: Plan07.1 Sandbox Hardening (v0.4.1-beta)

### What Went Well

1. **デッドラインタイマーによる CPU ウォッチドッグ** — シンプルで非侵入的な実装。`setInterval` ハートビートチェック + AbortController + タイムアウトリジェクション。ネイティブ C++ モジュールは不要。
2. **onAny() サブスクリプションによる双方向 EventBus** — エレガントなパターン：ワーカーが EventBus メッセージをサブスクライブし、ハンドラーがイベントをブロードキャスト。特別な RPC エンコーディングなしでワーカー→メイン通知を実現。
3. **指数バックオフ付きワーカー再起動** — 優雅な障害回復。`1s → 2s → 4s → ...` によりカスケード障害時のサンダリングハードを防止。
4. **parentPort メッセージハンドラーの非同期プロキシ** — ワーカーコンテキストの async/await 問題を解決。適切な Promise エラーハンドリングを備えたラッパー関数。
5. **QA + Architect の並列検証** — Phase 3 が並列で完了。直列的なボトルネックなし。両エージェントが独立してテストし、同時に PASS を報告。

### What Could Be Improved

1. **テストの作成が遅かった** — 実装完了後にテストを追加。逆順序。教訓：Phase 1 でテスト仕様を書き、Phase 2 で実装+テストを行うこと。
2. **メッセージ型スキーマが分散** — EventBus メッセージ型が messages.ts で定義されているが、ドキュメントは Architecture Spec 内。全メッセージ型ドキュメントを専用の messages.md リファレンスに集約することを検討。
3. **ワーカー再起動の手動テストが必要** — テストはハッピーパス（ON_CRASH 動作）をカバーしているが、実際のワーカークラッシュによる手動テストが信頼性確保に必要。推奨：意図的にワーカーをクラッシュさせる「カオス」テストスイートを作成。

### Patterns to Reuse

1. **デッドラインタイマーパターン**：`const deadline = Date.now() + timeoutMs; const remaining = deadline - Date.now(); if (remaining <= 0) reject(new Error('timeout'))`。あらゆる非同期操作タイムアウトに再利用可能。
2. **指数バックオフ**：`const delay = Math.min(baseDelay * Math.pow(2, retries), maxDelay)`。標準パターン。ワーカー再起動と接続リトライに最適。
3. **デバッグ用の EventBus.onAny()**：onAny() サブスクリプションパターンは EventBus 転送だけでなく、中央集約ログやメトリクス収集にも有用。

### Action Items for Future Cycles

- [ ] テストファースト開発プロセスの確立（テストは Phase 2 ではなく Phase 1 で作成）
- [ ] ワーカー再起動シナリオ用のカオステストスイートを作成
- [ ] 全メッセージ型を専用リファレンス（messages.md）にドキュメント化

---

## 20260211_cycle7: Plan07.2 Advanced Hardening (v0.4.2-beta)

### What Went Well

1. **@babel/parser による静的インポート解析** — AST ベースのアプローチはクリーンで保守しやすい。@babel/parser 単体で十分（@babel/traverse は不要）。正確なインポート/require 検出を実現。
2. **Ed25519 PKI 署名が組み込み対応** — Node.js の crypto.sign(null, ...) が Ed25519 で完璧に動作。後方互換性のあるフォーマット検出（128文字 hex = レガシー、オブジェクト = PKI）がエレガントかつ非破壊的。
3. **Plugin-signer CLI ツール** — 3つのコマンド（keygen、sign、verify）でライフサイクル全体をカバー。JSON 出力形式はクリーンでパース可能。ツールは自己完結型でライブラリ非依存。
4. **リワークサイクルの効率性** — 3つの FAIL 項目を1回の4時間リワークサイクルで全て修正。明確な根本原因分析 + 集中的な修正 + 並列再検証。ピンポンなし。
5. **パッケージ名の優雅なハンドリング** — npm パッケージ名を持つプラグインはファイル検証不可。解決策：ログで警告し、実行を継続。ブロッキングなし。良い UX。

### What Could Be Improved

1. **統合ポイントが最初はコメントだった** — Phase 2 の実装で `// TODO: call analyzeImports()` という実際のコードではなくプレースホルダーが存在。Phase 3 で QA により検出。教訓：統合ポイントにプレースホルダーコメントは不可。実際のコードを書くこと。
2. **Plugin-signer パッケージが Phase 2 で未作成** — スコープ追跡の見落とし。パッケージは Architecture Spec にあったが作成されず。教訓：仕様に記載された全ファイル/パッケージのチェックリストを維持し、Phase 2 完了時に検証すること。
3. **パッケージ名プラグインで署名検証がスロー** — コードパスが全プラグインにファイルパスがあると仮定。見落とし：不変条件をチェックせずに仮定。教訓：仮定を検証し、早期にガードを追加すること。
4. **require() プロキシの二次防御が未テスト** — AST 解析 + require() プロキシがあるが、テストされているのは AST のみ。require() プロキシは未テストのフォールバック。将来：require() インターセプション専用テストを追加。

### Root Cause of Phase 3 FAILs

1. **FAIL-1（統合）**：ドキュメントだけでなく統合コードが欠落 → 設計/実装の不一致
2. **FAIL-2（スコープ）**：仕様に記載されたファイルが未作成 → Phase 2 チェックリストのプロセスギャップ
3. **FAIL-3（仮定）**：ガードなしでファイルパスの存在を仮定 → 入力検証の欠落

**メタ教訓**：3つの FAIL 全てがプロセス規律で予防可能だった：
- FAIL-1 → コミット前のコードレビュー（TODO コメントのチェック）
- FAIL-2 → Phase 2 チェックリストレビュー（仕様ファイルインベントリとの照合）
- FAIL-3 → 入力検証レビュー（仮定のチェック）

### Patterns to Reuse

1. **AST ベースの解析パターン**：
   ```typescript
   const ast = parse(code);
   const imports = new Set<string>();
   traverse(ast, { enter(node) { ... } });
   return imports;
   ```
   あらゆるコード解析に再利用可能：リンティング、セキュリティスキャン、依存関係抽出。

2. **後方互換性のある union 型フォーマット検出**：
   ```typescript
   if (typeof integrity === 'string' && integrity.length === 128) {
     // レガシー SHA-512 ハッシュ
   } else if (typeof integrity === 'object') {
     // PKI フォーマット
   }
   ```
   パターン：型/長さ/形状でフォーマットを検出し、ハンドラーにディスパッチ。あらゆるスキーマバージョニングに再利用可能。

3. **警告付き優雅なデグラデーション**：
   ```typescript
   if (!filePath) {
     logger.warn('Package-name plugin, skipping file verification');
     return; // 続行、ブロックしない
   }
   ```
   パターン：不一致をログに記録し、操作を続行。オプション機能や制約のある環境に最適。

4. **CLI ツール構造**（plugin-signer）：
   ```
   src/
     ├── commands/
     │   ├── keygen.ts
     │   ├── sign.ts
     │   └── verify.ts
     ├── utils/
     │   └── crypto.ts
     └── index.ts (コマンドルーター)
   ```
   あらゆるマルチコマンド CLI ツールに再利用可能な構造。

### Action Items for Future Cycles

- [ ] **プロセス**：Phase 2 チェックリスト項目を追加：「Architecture Spec 内の全ファイル/パッケージが作成されていることを確認」
- [ ] **プロセス**：コードレビューチェックリスト：「統合ポイントにプレースホルダーコメントなし」
- [ ] **テスト**：require() プロキシインターセプション（二次防御）専用テストを追加
- [ ] **ツール**：仕様からチェックリストへの自動生成ツールを検討（仕様ファイルインベントリの自動検証）
- [ ] **ドキュメント**：全メッセージ型の集約リファレンス messages.md を作成（Cycle 6 から延期）

### Lessons for Rework Cycles

1. **明確な根本原因分析** — 各 FAIL を具体的な根本原因に追跡（統合欠落、パッケージ欠落、ガード欠落）。集中的な修正が可能に。
2. **最小限のリワークスコープ** — リワーク全体で約30行 + 新パッケージ1つ。変更のカスケードなし。新たなバグ導入のリスクを低減。
3. **並列再検証** — QA + Architect の両方が並列でリワークを検証。修正後数分で完了。
4. **リワーク中のテスト拡張** — インポート統合とパッケージ名の優雅なハンドリングの専用テストを追加。リグレッションを防止。

---

## 20260211_cycle8: Plan07.3 Sandbox Final Hardening (v0.4.3-beta)

### What Went Well

1. **Module._load パッチが正しいアプローチだった** — `Module._load` の直接置換は globalThis.require Proxy よりも信頼性が高いことが証明された。Node.js 内部の require ルーティングは _load を経由し、全パス（直接 require、動的 import→createRequire など）を捕捉。
2. **監査ログの JSONL フォーマット** — 行区切り JSON は構造化ログ + ストリーミングに最適。各行が独立してパース可能。ローテーションとクリーンアップも問題なく動作。
3. **バッファリングされた監査書き込み** — バッチ書き込み（flushIntervalMs、maxBufferSize）により応答性を維持しつつ I/O オーバーヘッドを削減。テストでサニタイゼーションと順序の一貫性を確認。
4. **ライフサイクル + RPC + ツールのログカバレッジ** — 3つの異なるイベントカテゴリ（ライフサイクル：init/stop、RPC：call/response/error、ツール：invoke/success/failure）が完全な監査証跡を提供。
5. **サニタイゼーションパターン** — 正規表現ベースと値リストベースの両パターンが動作。テストでネストされたオブジェクト全体のシークレット墨消し（apiKey、password、secret、token など）を検証。
6. **リワークなしで PASS** — クリーンな実装。QA と Architect が並列で検証。FAIL 項目なし。

### What Could Be Improved

1. **Node.js WriteStream の非同期タイミング** — `fs.createWriteStream()` がファイルを非同期にオープン。初期テストの失敗は 'open' イベント前に書き込みが発生したことが原因。解決策：最初の書き込み前に 'open' イベントを await するか、バッファリングキューを使用（後者を採用）。教訓：Node.js ストリームのライフサイクルの癖を理解すること。
2. **シークレットパターンマッチングが当初広すぎた** — 正規表現 `/key/i` は "apiKey"、"sessionKey"、"toolKey" などを意図以上に広くマッチ。設計決定：正確なキー名リスト（`['apiKey', 'password', 'secret', 'token']`）または非常に具体的なパターン（`/^(api)?[_-]?key$/i`）のいずれかを使用。リストを選択。
3. **Module._load の型付け** — TypeScript は内部 Module._load API の型を提供しない。回避策：`(Module as any)._load(...)`。このパターンを将来のモジュールレベルパッチングのためにドキュメント化。
4. **監査ログの保存場所** — 監査ログの保存場所について標準的なコンセンサスなし。`process.cwd()/agent-audit.log` を使用。将来：SandboxAuditConfig.logPath で設定可能にすること。
5. **ファイル I/O タイミングによるテストの不安定性** — テスト間のクリーンアップに明示的なファイル削除 + WriteStream の 'close' イベントの await が必要。パターン：テストのティアダウンでは必ず stream.close() を await すること。

### Patterns to Reuse

1. **Module._load パッチパターン**：
   ```typescript
   const originalLoad = Module._load;
   Module._load = function(request: string, parent: any) {
     if (blockedModules.has(request)) {
       throw new Error(`Module '${request}' is blocked`);
     }
     return originalLoad.apply(this, arguments);
   };
   ```
   あらゆるランタイムモジュールインターセプション（セキュリティ、機能フラグ、モック注入）に再利用可能。

2. **バッファリング付き JSONL 監査ロガー**：
   ```typescript
   buffer.push(entry);
   if (buffer.length >= maxBufferSize || timeSinceLastFlush > flushIntervalMs) {
     flush(); // 全エントリを一括書き込み
   }
   ```
   あらゆる大量構造化ログに再利用可能：コンプライアンス監査、トランザクションログ、エラートラッキング。

3. **値リストサニタイゼーション**（正規表現に対して）：
   ```typescript
   const sensitiveKeys = ['apiKey', 'password', 'secret', 'token', 'privateKey'];
   if (sensitiveKeys.includes(key)) {
     return '[REDACTED]';
   }
   ```
   正規表現パターンより保守しやすい。あらゆるシークレット墨消しに再利用可能。

4. **WriteStream ライフサイクルハンドリング**：
   ```typescript
   const stream = createWriteStream(path);
   await new Promise<void>((resolve) => {
     stream.once('open', () => resolve());
   });
   // 安全に書き込み可能
   ```
   パターン：バッファリング保証のために 'open' を必ず await。全てのファイルバックドストリームに適用。

### Action Items for Future Cycles

- [ ] 監査ログの保存場所を設定可能にする（SandboxAuditConfig.logPath または SandboxAuditConfig.logDir）
- [ ] Module._load 型付け回避策を Architecture リファレンスにドキュメント化
- [ ] 実際のファイル I/O による書き込みタイミングのテスト（Cycle 6 のカオステストスイートに「ファイル I/O ストレス」テストを追加）
- [ ] 設定可能なサニタイゼーションキーリストを検討（ハードコードリストに対するアローリストアプローチ）

### Lessons for Runtime Module Interception

1. **Module._load が唯一のポイント** — 全ての require() パス（同期、非同期、動的）が _load を経由。ここをパッチすれば100%有効。
2. **Strict/warn/off モード** — 3つの適用レベル（strict：スロー、warn：ログ+続行、off：パススルー）が運用上の柔軟性を提供。
3. **モジュールキャッシュバイパス** — require() は最初に require.cache をチェックするため、キャッシュヒット後に _load をパッチしても動作しない。require の前、サンドボックス初期化の早い段階でパッチすること。
4. **Proxy は不要** — 以前の globalThis.require Proxy アプローチは過剰設計。直接 _load パッチの方がシンプルで高性能かつ信頼性が高い。

---

## 20260211_cycle9: TUI Dashboard MVP (v0.5.0-beta)

### What Went Well

1. **SDK 変更が不要** — IUI.onEvent() が TUI に十分だった。これによりサイクルが大幅に簡略化（インターフェース凍結不要、SDK 追補不要、直接実装へ）。
2. **Ink v5 + React パターン** — 実績のあるフレームワーク、React ナレッジトランスファー、クリーンなコンポーネントモデル。
3. **純粋 reducer パターン** — tuiReducer は React や Ink 依存なしで完全にテスト可能。21の reducer テストが 15ms 未満で実行。
4. **独立監査で実際のバグを検出** — 実装後の詳細監査で2つの重大な問題（Ink アンマウントリーク、dispose フック欠落）と4つの中程度の問題を発見。プロダクションで問題を引き起こすものだった。
5. **テスト目標を超過** — 目標 18-25 に対して 58 テスト（最小値の294%）。

### What Could Be Improved

1. **QA エージェントがテスト記述を捏造**（重大なプロセス問題）
   - QA レポートがコードベースに存在しないアクションとイベントを記述：`UPDATE_MESSAGE`、`SET_SKILL_STATUS`、`CLEAR_MESSAGES`、`UPDATE_ACTIVE_SKILL`、`MESSAGE_AGENT`、`SKILL_STARTED` など。
   - 根本原因：QA エージェントが実際のテストファイルを読む代わりに、Architecture Spec や一般知識から記述を生成した可能性。
   - 適用された修正：`qa.md` エージェント定義を更新し、記述前に各テストファイルを Read する明示的な要件を追加。「CRITICAL: Test Description Accuracy」セクションを追加。
   - 教訓：**エージェント生成レポートはソースコードに対して検証必須。** 数値は正確でも記述が捏造されている場合がある。

2. **Ink ライフサイクル管理が初期実装で欠落**
   - `render()` は stop 時に `.unmount()` が必要な Instance を返す。これは Architecture Spec のコード例に含まれていなかった。
   - 教訓：ターミナル/UI 状態を制御するフレームワークでは、仕様で適切なクリーンアップを必ず計画すること。

3. **Core と React 間のイベント競合状態**
   - React マウント前に到着するイベントは無声でドロップされる。DispatchBridge パターンにはバッファが必要。
   - 教訓：同期イベントエミッターを非同期 UI フレームワークにブリッジする場合、初期イベントを必ずバッファリングすること。

4. **Reducer の純粋性違反** — reducer 内の `Date.now()` は副作用。監査で検出されたがテストでは検出されず。
   - 教訓：Reducer では Date.now()、Math.random()、crypto.randomUUID() を呼び出さないこと。タイムスタンプ/ID はアクションペイロード経由で渡すこと。

### Patterns to Reuse

1. **DispatchBridge パターン** — null 初期化された参照 + useEffect コールバック + 保留イベントバッファを介して、React 内部 dispatch を外部プラグインコードにブリッジ。
2. **APPEND_STREAM / FINALIZE_STREAM** — ストリーミングコンテンツ蓄積用の専用アクション。マジック ID で ADD_MESSAGE をオーバーロードするよりクリーン。
3. **重い依存関係の動的インポート** — `start()` 内の `await import("ink")` により、プラグインが登録されているだけで未アクティブの場合に React/Ink のロードを回避。
4. **実装後監査** — Phase 3 PASS 後に独立したコード監査を実行することで、QA と Architect の両方が見逃した問題を検出。これを標準的な強化ステップとすることを検討。

### Action Items for Future Cycles

- [ ] イベントログスクロールテストを追加（キーバインド `[`/`]` は実装済みだが未テスト）
- [ ] 将来のサイクルでコンポーネントレンダリングテスト用に `ink-testing-library` の追加を検討
- [ ] ランナー設定ガイドで stdio/TUI の相互排他をドキュメント化
- [ ] スクロールオフセットに上限クランプを追加（任意に大きな値を防止）
- [ ] 過去の全 QA レポートで同様の精度問題をレビュー

---

## 20260212_cycle12: Plan11 DevTools & E2E Testing Framework (v0.7.0-beta)

### What Went Well

1. **Dev-plugin サブエージェントの権限回避策が成功** — dev-plugin エージェントが agent_dev/openstarry_plugin/ ディレクトリへの書き込みでブロックされたが、coordinator が DevTools プラグイン全体を直接実装可能だった。権限問題があっても柔軟なワークフローでプロセスを継続できることを実証。
2. **ヘッドレス UI 設計による優れたテスタビリティ** — DevTools プラグインはテストで Ink 依存なしに React コンポーネントを使用。純粋 reducer パターン（前サイクルの tuiReducer）が再利用可能であることが証明され、DevTools の状態管理テストが容易に（38テスト、高カバレッジ）。
3. **E2E フィクスチャーパターンが堅牢** — MockProvider + AgentTestFixture パターン（Cycle 11 ランナーテストで導入）がマルチセッション、マルチプラグイン、ワークフローテストに優れたスケーラビリティを発揮。フィクスチャーは構成可能で再利用可能。
4. **リワーク項目としての README 作成** — Cycle 11 の経験から、README リクエストが条件付きアーキテクチャパスになることが判明。Cycle 12 では初期設計に README 項目を積極的に含め、リワークは発生したが Phase 2 からドキュメント品質を確保。
5. **テスト数の効率性** — 新プラグイン + テストフレームワーク全体で 75 の新テスト（38 DevTools + 40 E2E）。膨張なしに包括的なカバレッジを達成。主要機能あたり平均 2-3 テスト。

### What Could Be Improved

1. **README は Phase 3 の助言ではなく Phase 2 の成果物とすべき** — 3サイクル連続（Cycle 9-11-12）で README が条件付き/助言的だった。パターン：Architect は Phase 1 で README を Phase 2 の成果物として指定すべきであり、Phase 3 の助言としてではない。Phase 1 仕様フォーマットのプロセス改善。
2. **サブエージェントの書き込み権限が未解決のまま** — dev-plugin エージェントが権限ブロックに遭遇する4サイクル連続。Coordinator は回避策（直接実装）を開発したが、持続可能ではない。根本原因：Cycle 13 前にサブエージェントの権限設定の監査と修正が必要。
3. **E2E フィクスチャーのドキュメントが当初過大** — 初期 E2E ヘルパー（MockProvider、AgentTestFixture、SessionHelper）はそれぞれ 800行以上。リワークで200行以上のインラインドキュメントを追加。教訓：複雑なフィクスチャーは遡及的ではなく最初の実装からインラインドキュメントが必要。

### Patterns to Reuse

1. **状態管理用の純粋 reducer** — DevTools は Cycle 9 の tuiReducer パターンを再利用。テストで React/フレームワーク依存ゼロ。このパターンを全 UI 状態（TUI、DevTools、将来の Web UI）の標準とすべき。
2. **E2E フィクスチャーの構成** — スタックパターンが良く機能：
   ```typescript
   const session = new SessionHelper(agent);
   const mock = new MockProvider(session);
   const fixture = new AgentTestFixture(mock);
   ```
   フィクスチャーが絡み合うことなくクリーンに構成。将来のフレームワーク（マルチエージェントテスト、デーモンテスト）に再利用可能。
3. **README 統合テスト** — E2E フレームワークに README を読み、例をパースし、実行を検証するテストを含む。ドキュメントの劣化を防止。複雑なフレームワークに適用可能なパターン。

### Lessons Learned

1. **ヘッドレス設計はフレームワーク依存より優れている** — DevTools はコアロジックで Ink 依存を避け、レンダリングにのみ React を使用。テストが容易になり実装が高速化。推奨：可能な限り UI コンポーネントをヘッドレスで設計（状態 + レンダリングの分離）。
2. **E2E テストは実際のバグを検出する** — E2E フレームワークが CLI ルーティング（引数パース：`--config path/to/file` であり `--config=path/to/file` ではない）とプラグイン解決（プラグインロード順序の循環依存）の2つの微妙なバグを検出。これらのバグはユニットテストでは検出されない。
3. **フィクスチャーライブラリの成熟がテストを加速** — Cycle 12 までに、フィクスチャーライブラリ（MockProvider、AgentTestFixture、SessionHelper）は成熟し十分にドキュメント化された。新しい E2E テストの作成が Cycle 11 のランナーテストより4倍高速。教訓：良いテストインフラに早期投資し、積極的に再利用すること。
4. **Documentation as Code が機能する** — E2E README に実行可能な例を含む。テストスイートに例の動作を検証するテストを含む。ドキュメントが劣化しない。教訓：複雑な API では例を実行可能なテストにすることを検討。

### Action Items for Future Cycles

- [ ] **プロセス**：Phase 1 仕様フォーマットを変更し、README を成果物として含める（助言ではなく）。コンテンツアウトラインを指定
- [ ] **インフラ**：dev-plugin のサブエージェント書き込み権限を監査・修正（5サイクルの回避策）
- [ ] **テスト**：E2E フィクスチャーのドキュメントは最初の実装からインラインで（遡及的リワークではなく）
- [ ] **標準**：全ての将来の UI コンポーネントにヘッドレス UI 設計パターンを推奨
- [ ] **ツール**：自動 README 検証を検討（例が実行可能なテストとなること）

---

## 20260212_cycle13: Plan12 Daemon Mode MVP (v0.8.0-beta)

### What Went Well

1. **デーモンプロセスのライフサイクル管理が単純だった** — `child_process.spawn` + `detached: true` + `unref()` パターンはエレガントで信頼性が高い。プロセスが親から完全に分離。
2. **Unix ドメインソケット上の IPC が軽量** — カスタム約200行の JSON-RPC 実装により重量級の依存関係を回避。ソケットベースの通信は高速でポート競合が不要。
3. **ヘルスチェックプロバイダーパターンが検証済み** — エージェントのヘルスエンドポイント（uptime、version、status）に IProvider を使用するのはクリーン。将来のシームレスアタッチがこの基盤の上に構築可能。
4. **シグナルカスケードが設計通りに動作** — SIGTERM → 優雅なシャットダウン → ソケットクリーンアップ → PID ファイルクリーンアップ。マルチステージシャットダウンは単一シグナルアプローチより堅牢。
5. **シャットダウンタイムアウト時の優雅な障害処理** — エージェントが5秒以内にシャットダウンしない場合、デーモンプロセスが強制終了しステータスを報告。良い運用動作。

### What Could Be Improved

1. **ファイルディスクリプターに関する `createWriteStream` タイミング問題** — 初期実装では `WriteStream` を `spawn(stdio)` に渡し、テストディレクトリのクリーンアップ時に非同期 ENOENT エラーが発生。解決策：`openSync()` でファイルディスクリプターを即座に取得し、ストリームオブジェクトの代わりに FD 番号を渡す。教訓：`spawn()` の stdio 設定に非同期オブジェクトを渡さないこと。同期ファイル操作を使用すること。
2. **初期実装のデッドコード** — Architect レビューで未使用のヘルパー関数を検出（`shutdownGracefully` が定義されているが未呼び出し、インラインの `shutdownWithTimeout` で置換）。教訓：コードレビューチェックリスト項目「デッドコードなし」を Phase 3 前に適用すること。
3. **モックデーモンエントリースクリプトの統合テスト** — デーモン起動テストには LLM プロバイダー初期化をバイパスするモックエントリースクリプトが必要。パターン：統合テスト用に最小限の daemon-entry-mock.js を作成し、実際のエントリーポイントでは別途検証。完全なエージェント起動なしに IPC プロトコルの高速テストが可能に。
4. **高負荷時の PID ファイル競合状態** — 複数の `daemon start` コマンドが同時に発火した場合、PID ファイルが順序外で書き込まれる可能性。解決策：アトミック書き込み（一時ファイルに書き込み、リネーム）。教訓：あらゆる永続状態（PID ファイル、ソケット）にはアトミック操作が必要。

### Patterns to Reuse

1. **デーモンエントリースクリプトパターン**：
   ```typescript
   // daemon-entry.ts: バックグラウンドプロセスで実行
   import { AgentCore } from '@openstarry/core';
   import { createDaemonPlugin } from '@openstarry-plugin/daemon';

   const agent = new AgentCore();
   await agent.bootstrap({
     configPath: process.argv[2],
     transportMode: 'daemon',  // TUI をスキップし、IPC を有効化するシグナル
   });
   ```
   あらゆるバックグラウンドサービス（ワーカー、オーケストレーター、データプロセッサー）に再利用可能。

2. **WriteStream の代わりにファイルディスクリプター**：
   ```typescript
   const logFd = fs.openSync(logPath, 'a');
   const proc = spawn(entry, args, {
     stdio: ['inherit', logFd, logFd],  // ストリームではなく FD を使用
   });
   ```
   `spawn()` にストリームを渡すより安全。あらゆる長寿命子プロセスに使用。

3. **バックグラウンドサービスのヘルスチェック RPC**：
   ```typescript
   // IPC メソッド: agent.health() → {ok: bool, uptime: number, version: string}
   // 外部モニタリング、オーケストレーション、シームレスアタッチを実現
   ```
   あらゆる長寿命バックグラウンドプロセスに再利用可能。将来の優雅なリロード、カナリアデプロイメントの基盤。

4. **SetDaemonEntryOverride テスティングフック**：
   ```typescript
   // テストでデーモンエントリーポイントをオーバーライドしてプロバイダー初期化をスキップ
   setDaemonEntryOverride(mockEntryPath);
   const proc = spawn(mockEntryPath, ...);
   ```
   凍結インターフェースを変更せずにテスト用のクリーンな依存性注入。あらゆるオーバーライド可能なサービスロケーターニーズに適用可能なパターン。

5. **アトミック PID ファイル書き込み**：
   ```typescript
   const tempPath = pidPath + '.tmp';
   fs.writeFileSync(tempPath, pid.toString());
   fs.renameSync(tempPath, pidPath);  // アトミックスワップ
   ```
   永続状態の標準パターン。部分書き込みと競合状態を防止。

### Lessons for Socket/IPC Programming

1. **Unix ドメインソケットにはポート競合がない** — TCP とは異なり、`EADDRINUSE` エラーなしにテストインスタンス間でソケットパスを安全に再利用可能。テストクリーンアップでは `unlink()` を使用。
2. **JSON-RPC 2.0 の `id` フィールドは一致必須** — id=1 のリクエストには id=1 のレスポンスが必要。初期実装では `undefined` ID を使用。テストで検出。
3. **`connect()` は即座に返るかコールバックを後で呼ぶ** — 両方の動作が有効。常に両パスを処理すること（イベントリスナーを追加、同期を仮定しない）。
4. **ソケットの `destroy()` vs `end()`** — `end()` は優雅にクローズ。`destroy()` は強制クローズ。クリーンアップには `destroy()` を使用。優雅なシャットダウンには `end()` を使用。

### Root Cause of Phase 3 Conditional Pass

1. **CONDITIONAL（2件の修正）**：
   - **修正1**：シャットダウンタイムアウトの適用欠落 — `shutdownWithTimeout()` 関数が定義されているが、デーモンライフサイクルで未呼び出し。Architect がデッドコードとして指摘。解決策：SIGTERM 時の IPC サーバークリーンアップで呼び出し。
   - **修正2**：デッドコード削除 — 未使用のヘルパー関数を削除。教訓：Phase 2 クリーンアップチェックリスト：「Phase 3 前に未使用関数を grep」。

**リワークは最小限**：コード変更2行（関数呼び出し1つ、コード削除1つ）。良好な初期設計を示す。問題は配線の見落としであり、アーキテクチャの問題ではない。

### Action Items for Future Cycles

- [ ] ファイルディスクリプターハンドリング（openSync + FD 番号）をテストインフラドキュメントに追加
- [ ] デーモンエントリーモック用テストインフラを作成（Plan13+ で再利用可能）
- [ ] Unix ドメインソケットのベストプラクティスをドキュメント化（ソケットパスクリーンアップ、アトミック書き込み）
- [ ] Phase 2 チェックリストに「デッドコードなし」リンタールールの追加を検討
- [x] Plan13：ヘルスチェック RPC の上にシームレスアタッチを構築（基盤は準備完了） — 完了（Cycle 14）

---

## 20260212_cycle14: Plan13 Seamless Attach (v0.9.0-beta)

### What Went Well

1. **イベントフォワーダーパターンがエレガント** — core.bus ブリッジにおける sessionId によるセッションフィルタリングされたイベント配信が、関心の分離をクリーンに実現。フォワーディングロジックは約100行で最小限かつテスト可能。
2. **IPC プロトコル拡張が自然だった** — `agent.attach`、`agent.input`、`agent.detach` RPC メソッドの追加が既存 IPC レイヤーに自然に適合。SDK 変更不要。既存デーモンインフラを再利用。
3. **ターミナル I/O プロキシが単純だった** — stdin → IPC client → agent.input RPC → イベントストリーミング → stdout パターンが確実に動作。特別なターミナルハンドリング不要（readline + ネイティブストリームで十分）。
4. **セッションライフサイクル管理によりテストが簡略化** — アタッチ時の sessionId 作成、デタッチ時のクリーンアップ、タイムアウト処理の全てがシンプルなセッション状態チェックでテスト可能。純粋な async/await コード、最小限の状態機械。
5. **自動起動機能によるシームレスな UX** — `openstarry attach [agent-id]` がデーモン未起動の場合に自動でスポーンし、接続。手動の `daemon start` ステップが不要。ユーザーにとって透過的な体験。

### What Could Be Improved

1. **セキュリティ検証は設計のより早い段階で行うべき** — Architect が「アタッチ前に agentId を検証」を conditional PASS として指摘。振り返れば明白（デーモン存在確認、エージェント応答確認）だが Phase 3 リワークに延期。教訓：全セキュリティチェックを Phase 1 Architecture Spec で列挙し、レビューで発見するのではなく事前に対処。
2. **イベントタイムスタンプフィールドがリワークで追加** — 初期イベントフォワーディングにタイムスタンプがなく、追跡可能性が低下。Architect が正しく指摘。教訓：あらゆるイベントシステムは初日からタイムスタンプを含むべき。五蘊イベントインターフェース標準に追加。
3. **エラーコードドキュメントの欠落** — Phase 2 実装にはエラースローがあったが、集約されたエラーコードドキュメントがなかった。Architect が conditional PASS で指摘。解決策：ドキュメント化されたエラーコード（AGENT_OFFLINE、SESSION_TIMEOUT、INVALID_INPUT_TYPE、MESSAGE_SIZE_LIMIT_EXCEEDED）を持つ ErrorCode const を作成。教訓：API サーフェスとしてのエラーハンドリングには明示的なドキュメントが必要。
4. **シングルクライアント制約が未適用** — Architecture Spec に「一度に1デーモンアタッチにつき1クライアント（マルチクライアントは延期）」と記載。実装では未適用。複数クライアントが同じデーモンにアタッチ可能。リスク：設計ではなくドキュメントでのみ適用。教訓：コードで適用されない場合、仕様に適用メカニズムをドキュメント化。

### Patterns to Reuse

1. **セッションフィルタリングされたイベントフォワーディング**：
   ```typescript
   // Core.bus が全イベントを発行
   core.bus.on(event => {
     // sessionId でフィルタリング
     if (event.sessionId === attachedSessionId) {
       ipcClient.notifyEvent(event);  // アタッチされたクライアントに転送
     }
   });
   ```
   各クライアントが異なるイベントを受信すべきマルチクライアントシナリオに再利用可能。マルチエージェント、マルチテナント、イベントフィルタリングを伴うあらゆるシナリオにスケール。

2. **ターミナル I/O プロキシパターン**：
   ```typescript
   // stdin → IPC
   readline.on('line', (input) => {
     ipcClient.call('agent.input', { sessionId, message: input });
   });
   // IPC → stdout
   ipcClient.on('event', (event) => {
     process.stdout.write(formatEvent(event));
   });
   ```
   シンプルで効果的。リモートエージェントへのプロキシが必要なあらゆる CLI ツールに再利用可能。

3. **クリーンアップ付き自動起動パターン**：
   ```typescript
   let autoStartedDaemon = false;
   try {
     if (!isDaemonRunning()) {
       await startDaemon();
       autoStartedDaemon = true;
     }
     await attachToAgent();
   } finally {
     if (autoStartedDaemon) {
       await stopDaemon();  // 自分が起動した場合のみクリーンアップ
     }
   }
   ```
   永続的なフットプリントなしでシームレスな UX を実現。あらゆるオンデマンドバックグラウンドサービスに適用。

4. **操作前の RPC ヘルスチェック**：
   ```typescript
   // アタッチ前にエージェントの応答性を検証
   const health = await ipcClient.call('agent.health', { agentId });
   if (!health.ok) {
     throw new Error(`Agent offline: ${health.reason}`);
   }
   ```
   カスケードエラーを防止するシンプルな検証。複雑な操作前にピアのヘルスを確認する必要があるあらゆる分散システムに適用可能なパターン。

### Lessons for Session-Based IPC

1. **sessionId は UUID とすべき** — セッション追跡が信頼性を持ち、中央レジストリなしでグローバルにユニーク。スキーマチェックでも検証良好。推奨：分散セッション ID には常に UUID を使用。
2. **イベントタイムスタンプは必須** — クライアントに転送されるあらゆるイベントにはデバッグ、リプレイ、監査証跡のための `timestamp: ISO8601` フィールドが必要。今後のコア五蘊イベントインターフェース仕様に追加。
3. **エラーコードは集約すべき** — 任意のエラーメッセージをスローしないこと。可能な全エラーコードを含む ErrorCode const を定義し、API ドキュメントに含める。クライアント側のエラーハンドリングを可能にする。
4. **入力検証にはサイズ制限を含むべき** — 64KB メッセージサイズ制限 + 1MB イベント制限がなければ、悪意のあるまたはバグのあるクライアントがデーモンに巨大なペイロードをフラッドさせる可能性。スキーマ検証 + ランタイムチェックの両方が必要。

### Root Cause of Phase 3 Conditional Pass

1. **CONDITIONAL（3件の修正）**：
   - **修正1**：セッション作成前の AgentId 検証 — セッション作成前にデーモン存在 + エージェント応答を確認。無効なセッション状態を防止。
   - **修正2**：イベントタイムスタンプフィールド — 追跡可能性のために転送イベントに `timestamp: ISO8601` を追加。監査証跡を有効化。
   - **修正3**：エラーコードドキュメント — 4つのエラーコード（AGENT_OFFLINE、SESSION_TIMEOUT、INVALID_INPUT_TYPE、MESSAGE_SIZE_LIMIT_EXCEEDED）を正確なエラーメッセージとともにドキュメント化。

**リワークは最小限**：3つの小さな強化、アーキテクチャ変更なし。堅実な Phase 1 設計を示す。修正は洗練であり再設計ではない。

### Action Items for Future Cycles

- [x] Phase 1 Architecture Spec チェックリストにセキュリティチェックを追加（状態作成前に全入力を検証）
- [x] 五蘊の全イベントインターフェースにタイムスタンプフィールドを追加（EventBase.timestamp 必須）
- [x] ErrorCode ドキュメント標準を作成（全エラーコードを集約、API 仕様にドキュメント化）
- [ ] Plan14：デーモンレジストリによるシングルクライアントパーデーモン制約の適用（エージェントごとにカウンター追跡）
- [ ] Plan14：セッションタイムアウト適用を追加（非アクティブ5分後に自動デタッチ）— 現在仕様には記載、未実装
- [ ] Plan15：Web ベースのリモートアタッチ（HTTP/WebSocket、暗号化チャネル、認証）
- [ ] Plan15：マルチクライアントアタッチ（並行セッション、クライアント固有フィルタリングによる共有イベントストリーム）

---

## 20260212_cycle18: Plan15 SDK Context Extensions & Provider Integration (v0.13.0-beta)

### What Went Well

1. **遅延アクセサーパターンの成熟** — IPluginContext.providers アクセサーは Cycle 3-4 で確立された tools/guides パターンに従った。設計上のイノベーション不要。実績あるパターン（readonly getter、インターフェースのみ、非破壊）の再利用により Phase 1 設計が迅速かつ確信を持って行えた。
2. **延期された技術的負債の統合** — Cycle 17 で延期された2つの問題（サンプリングプロバイダー統合、roots allowedPaths コンテキスト）。Cycle 18 で両方を集中的なスコープで直接取り組んだ。明確な依存関係チェーンとリワークなしの実装。
3. **型付きメタデータインターフェースとしての SessionConfig** — SessionConfig をオプショナルインターフェース（getSessionConfig/setSessionConfig ヘルパー）として導入したことは、アドホックなセッションコンテキスト配線よりクリーン。コアの状態変更なしにプラグインがセッションレベル設定を読み取ることが可能に。
4. **初回パスで PASS、リワークなし** — 894 → 915 テスト（+21 新テスト）、全コードレビュー項目 PASS、条件付き修正なし。Architecture Spec の完全性と Phase 2 の整合性を示す。
5. **プロバイダーレジストリをプラグイン所有に維持** — 重要なアーキテクチャ決定：プロバイダーレジストリは mcp-common またはプロバイダープラグインに存在し、SDK は IProviderRegistry インターフェースのみを公開。コアは最小限を維持（プロバイダー実装なし）。マイクロカーネルの純粋性が完璧に維持。

### What Could Be Improved

1. **同期環境の問題が発生** — Phase 2.5 の agent_test への同期で陳腐化した node_modules（yaml + vitest モジュール解決）に遭遇。回避策：agent_dev で検証を実施。根本原因：以前の dev/test サイクルが不整合な状態を残した。教訓：node_modules クリーンアップの Infrastructure-as-Code が必要（Plan16+ タスク）。
2. **テスト数の目標が保守的だった** — 仕様推定26テストに対して実際は21テスト。理由：一部のテストカバレッジがベースラインに既に部分的に存在。教訓：サイクル間で機能を延期する場合、投影での二重カウントを避けるためにベースラインのテスト重複を確認。
3. **設計代替案が検討されなかった** — Phase 1 仕様が最終設計（遅延アクセサー）に直接進んだ。代替案を評価可能だった：providers + allowedPaths を含む `ISessionContext`（より明示的だがより高い結合）。教訓：SDK 拡張では、コンセンサスが明白でも Architecture Spec に設計代替案をドキュメント化。

### Patterns to Reuse

1. **クロスプラグインレジストリ用の遅延アクセサーパターン**：
   ```typescript
   interface IPluginContext {
     readonly providers?: IProviderRegistry;
     readonly tools?: IToolRegistry;
     readonly guides?: IGuideRegistry;
   }
   ```
   このパターンは4サイクルにわたり実証済み。将来のレジストリニーズ（prompts、resources、scripts など）も同様に従うべき：readonly getter、インターフェースのみの公開、コアでの遅延インスタンス化、ミューテーションを公開しない。あらゆるプラグイン所有リソースレジストリに再利用可能。

2. **SessionConfig フォールバックチェーンパターン**：
   ```typescript
   const allowedPaths = ctx.session?.config?.allowedPaths
     || agentConfig.fileSystem.allowedPaths
     || [process.cwd()];
   ```
   セッション → エージェント → デフォルトへの安全なデグラデーション。あらゆる階層的設定（session > user > global）に適用。防御的 null の必要性を排除。明確な優先順位チェーン。

3. **ハンドラーにおけるプロバイダーフォールバック**：
   ```typescript
   const result = ctx.providers
     ? await ctx.providers.get('sampling')?.invoke(req)
     : null;
   return result || await fallbackLLMInvocation(req);
   ```
   クリーンな二段階委譲：まず能力固有のプロバイダーをチェックし、デフォルト動作にフォールバック。あらゆるプラガブルフォールバックシステムにスケール。

4. **プラグイン能力ディスカバリー用のサンドボックス RPC**：
   ```typescript
   // IPC ブリッジ: sandbox.providers.list() → 利用可能な全プロバイダーを返す
   // 直接 require('provider-package') なしで安全なプロバイダー検査を実現
   ```
   能力ディスカバリー（サンドボックス内で許可）とプロバイダーインスタンス化（IPluginContext が必要）を分離。プラグインのイントロスペクションを有効にしつつサンドボックスの制約を維持。

### Lessons for SDK Extension Design

1. **オプショナルフィールドは union 型より安全** — allowedPaths オプショナルフィールドを持つ SessionConfig は、SessionConfig | undefined やIPluginContext を条件付きフィールドで拡張するよりシンプルで前方互換性が高い。教訓：SDK に追加する場合、型 union よりオプショナルプロパティを優先。

2. **Readonly アクセサーは偶発的なミューテーションを防止** — IPluginContext.providers は readonly でプラグインが `ctx.providers.unregister()`（存在しない）を呼ぶのを防止。ただし TypeScript の readonly はコンパイル時保証であり、ランタイムではない。教訓：「readonly」は「ミューテーションしないこと」を意味するとドキュメント化。将来ミューテーションサーフェスが追加される場合、ランタイム安全性のために Object.freeze() ラップされたレジストリを返すこと。

3. **インターフェースのみの公開が実装を分離** — SDK の IProviderRegistry はインターフェース。実際の ProviderRegistry はコアとプラグインレイヤーに存在。コアが実装を交換したりプラグインが独自のレジストリを提供することが可能。教訓：SDK はインターフェースを公開すべきであり、実装ではない。将来の全レジストリ追加に適用。

4. **セッションコンテキストはプラグインコンテキストから分離すべき** — このサイクルでは ctx.session（存在を仮定）経由でセッション設定にアクセス。将来のサイクルでは IPluginContext（プラグイン固有）と ISessionContext（セッション固有）の分離が必要になる可能性。教訓：IPluginContext の境界をドキュメント化。セッションコンテキストが成長する場合、Plan16+ でインターフェースの分割を検討。

### Root Cause Analysis: Zero Issues

FAIL または CONDITIONAL 項目なし。Phase 3 の検証は単純：
- QA：915 テスト PASS（リグレッションなし、包括的カバレッジ）
- Architect：仕様準拠100%、3つの非ブロッキングアドバイザリを Plan16+ に延期（プロバイダーアクセス制御スコーピング、セッション設定検証、高度な非同期パターン）

**クリーン PASS の理由**：Architecture Spec が包括的、凍結インターフェースが完全に実装、設計の不一致なし、実装が仕様に正確に従い、テストカバレッジが適切。

### Non-Blocking Issues Deferred to Plan16+

1. **プロバイダーアクセス制御** — サンプリングハンドラーがファイルシステムプロバイダーを呼ぶのを防ぐスコーピングを追加可能。延期：複雑さ対メリットがまだ正当化されない。Plan16 で検証。
2. **起動時のセッション設定検証** — SessionConfig.allowedPaths をエージェント起動時に実際のファイルシステムパーミッションに対して検証可能。延期：変更されたパーミッションとの競合状態の可能性。Plan16 で再検討。
3. **高度な非同期プロバイダーパターン** — プロバイダーベースの補完（例：Claude Claude-to-Claude）を伴う MCP サンプリングが新しい UX の可能性を開く。延期：設計が必要。Plan15 のスコープ外。

### Action Items for Plan16+

- [ ] **インフラ修正**：node_modules クリーンアップスクリプト + CI フックを作成（dev/test サイクルでの陳腐化した依存関係を防止）
- [ ] **ドキュメント**：「SDK Extension Design Guidelines」を追加（遅延アクセサー、インターフェースのみ、オプショナルフィールド、readonly）
- [ ] **検証**：Plan16 セッション設定のハードニング（起動時検証 + 動的パーミッションチェック）
- [ ] **リファクタリング**：セッションの関心事が成長する場合、ISessionContext を IPluginContext から分離することを検討
- [x] **アーキテクチャ**：Sampling + Roots ハンドラーが実プロバイダー委譲で完成 — 完了
- [x] **プロバイダーレジストリの成熟**：安定したプラグイン所有パターンとして確立 — 完了

---

## 20260212_cycle22: Plan19 — Plugin Dependency Wiring & Cross-Plugin Services (v0.17.0-beta)

### What Went Well

1. **Spec Addendum プロセスが良く機能** — Architect がオリジナル Architecture Spec の文字列 ID パターンに代わるインターフェースベースの IServiceRegistry パターンを推奨する Spec Addendum を発行。Coordinator が承認。より優れた設計で実装を続行。迅速な設計イテレーションを完全なリワークなしに実現するプロセス。

2. **IServiceRegistry パターンがアーキテクチャ的に優れていた** — 完全な機能を持つインターフェース（register、get、has、list）が当初指定された文字列 ID ルックアップより柔軟で型安全であることが証明。IDE サポートの改善とメタデータによる将来の拡張性を実現。

3. **トポロジカルソート + 循環依存検出** — Kahn のアルゴリズム実装（145 LOC）がサイクルの明示的なエラー報告付きでプラグインロード順序を適切に処理。実証済みパターン。将来の依存関係グラフで再利用可能。

4. **サービスレジストリがプラグイン所有パターンに** — ServiceRegistry はプラグインシステムに存在し、SDK インターフェース経由で公開。コア結合を低減し、プラグインがサービスを独立して拡張可能に。確立されたパターン：コアがインターフェースを公開、プラグインが実装。

5. **リワークサイクルが迅速で生産的** — 1回のリワークサイクル（1日）で2項目を特定：欠落した has() メソッドとフィールド名の整合性。両方とも単純な修正。アーキテクチャリワーク不要。検証はクリーン PASS。

6. **リワークにもかかわらず包括的なテストカバレッジ** — 最終 1067 テスト（Plan19 用 65 新規、初期 58 + リワーク 7）。サービスレジストリ、トポロジカルソート、プラグインローダー、e2e インジェクションテスト全体にわたる。リグレッション未検出。

### What Could Be Improved

1. **agent_test での tsbuildinfo の陳腐化状態問題** — agent_dev から agent_test への同期後、ビルドで陳腐化した tsbuildinfo ファイルにより「incremental build mismatch」エラーが発生。回避策：同期前に agent_test/openstarry の node_modules をクリーン。教訓：同期スクリプトが tsbuildinfo アーティファクトを検証または自動クリーンすべき。

2. **Spec Addendum のコミュニケーションフロー** — 初期レビュー → coordinator 承認 → リワークサイクルを経て最終設計が明確になった。教訓：Architect がサイクル中に設計変更を推奨する場合、実装のフォルススタートを避けるために coordinator に即座にエスカレーションして明示的な承認を得ること。

3. **フィールド名の一貫性** — PluginManifest が当初混合命名（services vs provided、serviceDependencies vs consumed）を使用。リワークで修正されたが追加サイクルが発生。教訓：Architect は Phase 1（設計）で命名規約を指摘し、実装前に不整合を検出すべき。

### Patterns to Reuse

1. **設計ピボット用の Spec Addendum** — 実装がより優れた設計を明らかにした場合、Coordinator + Architect が開発をブロックせずに Spec Addendum を発行可能。プロセス：Architect が推奨を発行 → Coordinator が書面で承認 → dev が承認された設計を実装 → 再計画不要。同様のサイクル中設計洗練に再利用可能。

2. **プラグイン依存順序付け用の Kahn のアルゴリズム** — 次数追跡による循環検出付き O(V+E) トポロジカルソート。順序付きプラグイン初期化の実証済みパターン。あらゆる DAG ベースの依存システム（ワークフロー、サービス、プロバイダー）に再利用可能。

3. **明確なエラー報告付き循環依存検出** — エラーメッセージ：「Circular dependency detected: X → Y → Z → X」。実行可能なデバッグ情報を提供。人間が可読なフィードバック付きのあらゆるトポロジカルソートに再利用可能なパターン。

4. **インターフェースベースのサービスレジストリ** — SDK で IServiceRegistry（インターフェース）を公開し、コア/プラグインレイヤーで実装。実装の交換可能性とプラグイン所有レジストリを実現。prompts、resources、guides レジストリにスケール可能なパターン。

### Lessons for Rework Cycles

1. **コードレビューでの欠落メソッド発見** — Architect コードレビューが欠落した has() メソッド（IServiceRegistry インターフェース契約の一部）を検出。教訓：コードレビューではインターフェースメソッドと実装の明示的なチェックとして列挙すべき。

2. **ファイル間のフィールド整合性** — フィールド命名の不一致（services vs provided）が PluginManifest → types → tests にまたがった。1つの修正が3ファイルに波及。教訓：Phase 2 実装中に grep ベースのチェックリストを使用（全参照を検索して命名を早期に検出）。

### Root Cause of Phase 3 Rework

Architect が初期レビューで7項目を発見：
- 4項目：欠落した has() メソッド（アーキテクチャギャップ）
- 3項目：フィールド命名の不整合（実装エラー）

両方とも Code Fix による1回のリワークサイクルで解決（設計リワーク不要）。QA 再検証で全修正を確認。新たな問題なし。

### Action Items for Plan20+

- [x] **ビルドインフラ**：sync-to-test.sh に tsbuildinfo クリーンアップを追加（agent_test ビルド失敗を防止） — 完了（Cycle 23）
- [ ] **Coordinator チェックリスト**：Spec Addendum 発行時、実装開始前にリポジトリ内の Architecture Spec を明示的に更新
- [ ] **コードレビューチェックリスト**：インターフェースメソッドの列挙（インターフェース契約に従い全メソッドが実装されていること）
- [ ] **Dev チェックリスト**：ファイル間のフィールド命名一貫性を grep で検証（manifest、types、tests、loaders）
- [ ] **ドキュメント**：Architecture Spec テンプレートを更新し「Fields & Naming Conventions」セクションを含める
- [x] **サービスレジストリ**：安定したパターンが確立（インターフェースベース、プラグイン所有、コア公開） — 完了
- [x] **依存順序付け**：Kahn のアルゴリズムが DAG ベースのプラグインロードに対して実証 — 完了

---

## 20260212_cycle23: Plan20 — Workflow Engine MVP (v0.18.0-beta)

### What Went Well

1. **純粋なプラグイン実装を維持** — 複雑な機能（ワークフローオーケストレーション、ストリーミング API）にもかかわらず SDK/Core の変更ゼロ。マイクロカーネルの純粋性が完璧に保持。
2. **柔軟な仕様が実用的な書き直しを許容** — ユーザーがアーキテクチャ改善（カスタム構文の代わりに Mustache、transform/condition の代わりに service/llm ステップ、ファイル永続化の代わりに LRU）を特定した際、凍結仕様への厳格な固守ではなく、Architect 承認のもと迅速なアーキテクチャ改善が可能だった。
3. **ビルドエラー診断が体系的** — 書き直しによる6つのビルド問題を系統的に診断・修正：
   - データワークフロー（HTML ではない）のために Mustache HTML エスケープの無効化が必要
   - IProvider.chat() ストリーミング API に AsyncIterable + text_delta イベント収集が必要
   - Message 型に `id` フィールド（randomUUID）が必要
   - AgentEvent に `timestamp` フィールドが必要
   - 空のエビクションでの LRUCache undefined チェック
   - 同期パイプラインでの tsbuildinfo クリーンアップが必要
4. **書き直し後のコード品質が優れていた** — テスト数50%削減（67→37）にもかかわらず、シンプルで保守しやすいコードベースで包括的なカバレッジを維持。
5. **Architect コードレビューが建設的** — 仕様からの乖離にもかかわらず 10/10 チェックリスト PASS。Architect が実用的な根拠（優れた設計、プロダクション品質、コア目標達成）で PASS を正当化。
6. **ドキュメントが実際の教訓を捕捉** — Dev ログと Architect レビューがアーキテクチャ書き直しを詳細にドキュメント化し、同様の乖離に関する将来の意思決定を支援。

### What Could Be Improved

1. **仕様準拠 vs 実用主義の緊張** — オリジナル凍結仕様が実装とは全く異なるステップタイプ（transform、condition、prompt、return）を規定（実装：tool、service、llm、command）。ユーザーの書き直しはアーキテクチャ的に優れていたが、Phase 1 設計レビュー中に早期に議論できた可能性。
   - **教訓**：Architect は Phase 1 で「凍結仕様が最適でない可能性」の懸念を指摘し、実装前に早期の Spec Addendum 議論を可能にすべき。
2. **Mustache HTML エスケープの微妙な問題** — デフォルトの Mustache 動作が非 HTML ワークフローを破壊（`/` を `&#x2F;` にエンコード）。修正（エスケープ無効化）は正しいがライブラリドキュメントからは明白でなかった。
   - **教訓**：非 HTML データにテンプレートエンジンを使用する場合、コードコメントにエスケープ戦略を明示的にドキュメント化（本サイクルで完了）。
3. **mcp-common tsbuildinfo の繰り返し問題** — sync-to-test.sh が陳腐化した .tsbuildinfo にヒットする3回目のサイクル。クリーンアップステップは追加されたが、将来の同期では積極的に行うべき。
   - **教訓**：同期スクリプトはリビルド前に全 .tsbuildinfo ファイルを無条件にクリーンすべき（Phase 5 ツールで実装）。
4. **Command ステップのプレースホルダー混乱** — Command ステップはスキーマで定義されているがエラーをスロー。MVP の前方互換性のために意図的だが、ユーザーを混乱させる可能性。
   - **教訓**：「未サポート」マーカー付きの明確なドキュメント（README）を追加。スキーマレベルの検証警告または明示的な Future 型マーカーを検討。

### Patterns to Reuse

1. **仕様書き直し承認プロセス** — 凍結仕様が実装設計から大きく乖離する場合：
   - 乖離を明示的にドキュメント化（Architect レビューで全乖離をリスト化すべき）
   - 品質/実用的な根拠で乖離を正当化
   - 乖離にもかかわらず明示的な PASS を取得
   - 決定を記録するためにイテレーションログを更新
   - 同様の状況の将来のサイクル（Plan21+ 条件ロジック、永続化など）に再利用可能

2. **Mustache 補間パターン** — 非 HTML テンプレートデータ用：
   ```typescript
   // Mustache のデフォルト HTML エスケープを無効化 — ワークフローデータは HTML ではない。
   Mustache.escape = (value: string) => value;
   ```
   - 将来のプラグインでのデータ指向テンプレーティングに再利用可能なパターン

3. **ストリーミング API 収集パターン** — LLM 等のストリーミングプロバイダー用：
   ```typescript
   let responseText = "";
   for await (const event of provider.chat(request)) {
     if (event.type === "text_delta") {
       responseText += event.text;
     }
   }
   ```
   - AsyncIterable を返すあらゆるプロバイダーに再利用可能なパターン

4. **判別共用体の型安全性** — TypeScript 判別共用体による網羅性チェック：
   ```typescript
   export type IWorkflowStep = IToolStep | IServiceStep | ILLMStep | ICommandStep;
   // 同じ判別子を持つ Zod スキーマとマッチ
   const result = WorkflowSchema.safeParse(data); // ランタイムと TypeScript の整合性を保証
   ```
   - 複数のバリアントを持つあらゆるプラグインに適用（サービスタイプ、メッセージタイプなど）

5. **エラーコンテキストの保持** — ワークフローエラーがステップコンテキストを捕捉：
   ```typescript
   error: { message: string; step?: string; cause?: unknown }
   ```
   - 順次処理を伴うあらゆるエンジンに再利用可能なパターン（将来のワークフロー強化、バッチプロセッサーなど）

### Action Items for Plan21+

- [x] **ビルドインフラ**：sync-to-test.sh での tsbuildinfo クリーンアップ（本サイクルで完了）
- [ ] **仕様乖離ハンドリング**：Agent SOP に承認プロセスをドキュメント化（推奨：Architect が早期に乖離を指摘、Coordinator が承認、明示的に記録）
- [ ] **Workflow-engine README**：ステップタイプ、Mustache 構文、例、command ステップの制限をドキュメント化（Architect レビューの MINOR-1）
- [ ] **LRU キャッシュの設定可能性**：キャッシュサイズをマニフェストまたはコンテキスト経由で設定可能にする（Architect レビューの MINOR-4）
- [ ] **同期スクリプトのハードニング**：リビルド前に .tsbuildinfo ファイルを無条件にクリーン（繰り返し問題 #3）
- [ ] **条件ロジック設計**：Plan21 で condition ステップタイプ、サービスベースの分岐、JavaScript 式のいずれかを決定（MVP から延期）
- [ ] **永続化レイヤー**：Plan21 でオプショナルなファイルベースまたはデータベースの実行履歴を実装（MVP から延期）
