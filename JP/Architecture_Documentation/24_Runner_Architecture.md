# 24. Runner アーキテクチャ (Runner Architecture)

このドキュメントでは、 `apps/runner` の設計理念と責務の境界について説明します。

---

## 1. Runner とは何か？

Runner は OpenStarry の **ホスト引導プログラム** （Host Bootstrap Program）です。エージェントのエントリポイントですが、それ自体にはビジネスロジックは含まれていません。

```
apps/runner/src/bin.ts
```

Runner の責務は、以下の4つのステップのみです：
1.  `agent.json` 設定を読み取る（または組み込みのデフォルト値を使用する）
2.  `AgentCore` インスタンスを作成する
3. すべてのプラグインを動的に解決してロードする
4. エージェントを起動する

---

## 2. なぜ CLI と呼ばないのか？

Runner は以前 `apps/cli` と呼ばれていましたが、 Plan03 Phase C のリファクタリングで名称変更されました。その理由は以下の通りです：

| 問題 | 説明 |
|------|------|
| 名が体を表していない | CLI の対話体験（readline、カラー出力）は `standard-function-stdio` プラグインによって提供されており、 Runner 内にはありません。 |
| 密結合の示唆 | 「CLI」という名前は、それがコマンドライン専用であることを示唆しますが、 Runner は UI が何であるかを知りませんし、関心もありません。 |
| マイクロカーネル精神 | Runner は、 UI とは無関係な純粋なランチャーであるべきです。 |

---

## 3. Runner の依存関係

Runner は、3つのコアパッケージのみに依存しています：

```json
{
  "dependencies": {
    "@openstarry/core": "workspace:*",
    "@openstarry/sdk": "workspace:*",
    "@openstarry/shared": "workspace:*"
  }
}
```

**プラグインへの依存はゼロです。** すべての `@openstarry-plugin/*` パッケージは、実行時に `import()` を介して動的にロードされます。

---

## 4. プラグインはどうやって入力をプッシュするのか？

Runner が切り離される前、 stdio プラグインはホストから注入された `onInput` コールバックに依存していました。これにより、 Runner は stdio プラグインを認識し、特別な処理を行うことを強いられていました。

現在のアーキテクチャ：

```
SDK 層：IPluginContext.pushInput(event: InputEvent)
         ↓
Core 層：getPluginContext() が自動的に pushInput を注入 → core.pushInput
         ↓
プラグイン層：stdio リスナーがユーザー入力を受信 → ctx.pushInput({ source: "cli", data: text })
         ↓
Core 層：pushInput → スラッシュコマンドのファストパス / EventQueue → ExecutionLoop
```

**すべてのリスナープラグイン** は、 `ctx.pushInput` を介して入力をプッシュできます。 Runner はこのプロセスには関与しません。

---

## 5. デフォルト設定

`agent.json` が提供されない場合、 Runner は組み込みの `defaultConfig()` を使用します：

```typescript
function defaultConfig(): IAgentConfig {
  return {
    identity: { id: "openstarry-agent", name: "OpenStarry Agent", ... },
    cognition: { provider: "gemini-oauth", model: "gemini-2.0-flash", ... },
    plugins: [
      { name: "@openstarry-plugin/provider-gemini-oauth" },
      { name: "@openstarry-plugin/standard-function-fs" },
      { name: "@openstarry-plugin/standard-function-stdio" },
    ],
    guide: "default-guide",
  };
}
```

注意：デフォルト設定であっても、プラグインは完全なパッケージ名を使用して動的インポートによってロードされます。

---

## 6. プロセスレベルの責務

Runner は、どのプラグインの責任範囲にも属さない、少数のプロセスレベルの関心事を保持しています：

| 責務 | 説明 |
|------|------|
| `process.exit()` | `__QUIT__` イベントを監視し、プロセスを終了させます。 |
| `SIGINT / SIGTERM` | 終了信号を受信した際、エージェントを正常に終了させます。 |
| 設定のロード | `agent.json` の読み取り、 Zod による検証、デフォルト値へのフォールバックを行います。 |

---

## 7. 将来の進化

Runner は、いかなるプラグインも修正することなく、異なるホストに置き換えることができます：

| ホスト | 説明 |
|------|------|
| `apps/runner` | 現在の Node.js プロセスホスト。 |
| `apps/daemon` | 複数のエージェントインスタンスを管理する将来のデーモンプロセス。 |
| `apps/web-server` | HTTP を介して入力を受け取る将来の Web サービスホスト。 |

各ホストが行うことは同じです：設定の読み取り → コアの作成 → プラグインのロード → 起動。違いはプロセス管理と外部インターフェースのみです。
