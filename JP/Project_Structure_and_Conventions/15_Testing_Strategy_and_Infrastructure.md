# 15. テスト戦略とインフラストラクチャ (Testing Strategy and Infrastructure)

このドキュメントでは、 OpenStarry プロジェクトのテスト階層戦略、自動化インフラストラクチャ、およびコアの純粋性検証メカニズムについて詳細に説明します。本プロジェクトはマイクロカーネル・アーキテクチャを採用しているため、テストの重点はコアの安定性とプラグインの隔離性の確保にあります。

## 1. テスト階層ピラミッド (Testing Pyramid)

私たちは古典的なテストピラミッドモデルを採用していますが、エージェントシステムに合わせて調整を行っています：

### 1.1 ユニットテスト (Unit Tests) - 礎石
*   **範囲：** `packages/core`, `packages/sdk`, `packages/shared` 内のすべての関数とクラスを対象とします。
*   **ツール：** `Vitest` ( Monorepo と TypeScript へのネイティブなサポートがあるため)。
*   **原則：**
    *   **Mock Everything:** コアロジックのテスト時には、すべての I/O 操作を Mock 化しなければなりません。
    *   **Pure Functions:** 入出力のテストが容易な純粋関数の作成を推奨します。
*   **場所：** ソースコードと同じディレクトリに配置し、 `*.test.ts` と命名します。

### 1.2 統合テスト (Integration Tests) - コアの対話
*   **範囲：** コアと模擬プラグイン (Mock Plugins) との間の対話をテストします。
*   **シナリオ：**
    *   コアは SDK 仕様に準拠した Mock プラグインを正しくロードできるか？
    *   コアはプラグインがスローした異常を正しく処理できるか？
    *   イベントバス (Event Bus) のメッセージは正しくルーティングされているか？
*   **ツール：** `Vitest` + 自社開発の `MockHost` 環境。

### 1.3 システム/エンドツーエンドテスト (E2E Tests) - 現実世界のシミュレーション
*   **範囲：** 実際の要求（ `apps/runner` を使用）を使用して、実際のエージェントインスタンスを起動します。
*   **シナリオ：**
    *   **"Hello World" テスト：** エージェントが起動し、 Stdio リスナーをロードし、入力を受け取って、 Echo を返します。
    *   **記憶テスト：** 3ターンの対話を行い、 Context の状態が正しく更新されるかチェックします。
    *   **ツール呼び出しテスト：** `fs` ツールを使用してファイルを書き込み、ファイルが存在するか検証します。
*   **戦略：** **「リプレイ (Replay)」メカニズム** を使用します。実際の LLM 応答を一度記録し、それを CI で再生することで、 Token の消費を抑えつつテストの確実性を保証します。

## 2. コアの純粋性強制メカニズム (Core Purity Enforcement)

OpenStarry は `packages/core` がいかなる具体的な実装にも依存してはならないことを強調しているため、 CI 段階で静的解析を導入しています：

### 2.1 依存関係の境界チェック
*   **ツール：** `dependency-cruiser` または `eslint-plugin-import` 。
*   **規則：**
    *   `packages/core` は `plugins/*` を **import してはなりません** 。
    *   `packages/core` は `apps/*` を **import してはなりません** 。
    *   `packages/core` が依存できるのは `packages/sdk` と `packages/shared` のみです。
*   **実行タイミング：** 毎回の `git push` 前の Git Hook および CI プロセス。

### 2.2 ビルド成果物のチェック
*   コンパイル後の `dist/` ディレクトリをチェックし、 `langchain` や `puppeteer` などの大規模なサードパーティライブラリが誤ってコアバンドルに同梱されていないか確認します。コアは極めて軽量である必要があります。

## 3. 継続的インテグレーション・パイプライン (CI Pipeline)

GitHub Actions を使用して自動化パイプラインを構築します：

```yaml
name: OpenStarry CI
on: [push, pull_request]

jobs:
  quality-gate:
    steps:
      - name: Linting
        run: pnpm lint
      - name: Type Checking
        run: pnpm tsc --noEmit
      - name: Purity Check
        run: pnpm check-dependency-boundaries

  test-core:
    needs: quality-gate
    steps:
      - name: Unit Tests
        run: pnpm test:unit --filter "@openstarry/core"

  test-integration:
    needs: test-core
    steps:
      - name: Integration Tests
        run: pnpm test:integration

  build-dry-run:
    steps:
      - name: Build All
        run: pnpm build
```

## 4. プラグインテスト規範

サードパーティまたは公式プラグインの開発者向け：

*   **MockHost の提供：** SDK は、実際のエージェント環境をシミュレートする `TestAgentHost` クラスを提供します。開発者はシステム全体を起動することなく、プラグインの `onStart`, `onStop` およびツール呼び出しロジックをテストできます。
*   **契約テスト (Contract Testing)：** プラグインが `ITool` または `IProvider` インターフェースの入出力仕様を厳格に遵守していることを保証します。
