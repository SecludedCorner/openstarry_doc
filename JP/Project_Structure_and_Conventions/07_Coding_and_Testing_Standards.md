# 07. コーディングおよびテスト規範 (Coding & Testing Standards)

OpenStarry が産業用 OS のように安定して動作することを保証するために、以下の厳格な開発規範を策定しました。

## 1. TypeScript 厳格モード (Strictness)

すべての `tsconfig.json` はルートディレクトリの設定を継承し、 `strict: true` を維持しなければなりません。

*   **No Implicit Any:** 暗黙の `any` は禁止です。型が本当に不明な場合は、明示的に `unknown` を使用し、使用前に型を絞り込んでください (Type Narrowing) 。
*   **Null Checks:** `null` または `undefined` になる可能性のあるものはすべて処理しなければなりません。

```typescript
// ❌ 誤った例
function getName(user) {
  return user.name; 
}

// ✅ 正しい例
function getName(user: IUser): string | null {
  if (!user) return null;
  return user.name;
}
```

## 2. エラー処理の哲学：痛覚 (Error as Pain)

OpenStarry において、エラーは単なる例外ではなく、エージェントへのフィードバック信号です。

*   **エラーを握りつぶさない：** 完全に修正できる能力がない限り、低層でエラーを `catch` して上に投げないようなことはしないでください。
*   **標準エラーを使用する：** `@openstarry/shared` で定義されている `AgentPainException` またはそのサブクラスを優先的に使用してください。
*   **エラーを入力とする：** ツール実行レイヤーでキャッチされたエラーは、プロセスをクラッシュさせるのではなく、 `ToolResult` (status: error) としてフォーマットされるべきです。これにより、 LLM がエラーを認識し、自己修正を試みることができるようになります。

```typescript
// ✅ ツール内部の実装
try {
  // ... 実行ロジック
} catch (error) {
  // クラッシュさせるのではなく、例外を失敗した実行結果に変換する
  return {
    status: 'error',
    content: `操作に失敗しました: ${error.message}。パスが存在するか確認してください。`
  };
}
```

## 3. 非同期と並行処理

*   **Async/Await:** 「コールバック地獄」を避けるため、全面的に `async/await` を使用してください。
*   **Promise.all:** 依存関係のない並列操作（複数のファイルを同時に読み取るなど）については、効率向上のために並列処理を使用してください。

## 4. テスト規範 (Testing Strategy)

テストフレームワークとして **Jest** または **Vitest** を使用します。

### 4.1 ユニットテスト (Unit Tests)
*   **場所：** テストファイルはソースファイルと同じディレクトリに配置し、 `*.test.ts` と命名してください。
*   **原則：** 純粋なロジックをテストします。外部依存関係（ LLM API 、ファイルシステムなど）については、 **必ず** Mock を使用してください。
*   **Mocking:** 依存性の注入 (Dependency Injection) パターンを使用し、テスト時に `MockContext` や `MockFileSystem` を注入してください。

### 4.2 統合テスト (Integration Tests)
*   **場所：** `tests/integration/` ディレクトリに配置してください。
*   **原則：** 複数のコンポーネントの協調動作（例：コア + メモリ + MockLLM ）をテストします。

### 4.3 テストカバレッジ
*   コアロジック ( `packages/core/execution` ) は、 **90% 以上** のブランチカバレッジを要求します。
*   ツールプラグインは、「成功」と「失敗」の2つのパスをテストすることを要求します。

## 5. ログ規範 (Logging)

`console.log` を使用しないでください。必ず `context.logger` ( `@openstarry/sdk` 由来) を使用しなければなりません。

*   **DEBUG:** 詳細な内部状態の推移 (Loop tick, Context update) 。
*   **INFO:** 主要なライフサイクルイベント (Agent start, Tool call success) 。
*   **WARN:** 回復可能なエラー (Tool call failed but handled) 。
*   **ERROR:** 回復不能なシステムレベルのエラー (Plugin crash, Out of memory) 。
