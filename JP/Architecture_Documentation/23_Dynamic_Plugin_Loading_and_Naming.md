# 23. 動的プラグインロードと命名規則 (Dynamic Plugin Loading & Naming Convention)

このドキュメントでは、OpenStarry のプラグイン解決メカニズムとパッケージ命名規則について説明します。

---

## 1. 設計原則

Runner（ `apps/runner` ）は **純粋なランチャー** であり、具体的なプラグインを一切認識しません。Provider、Tool、Listener、Guide を含むすべてのプラグインは、 `agent.json` を通じて設定され、実行時に動的にロードされます。

これは以下を意味します：
- Runner の `package.json` は、いかなる `@openstarry-plugin/*` パッケージにも **依存しません** 。
- Runner のソースコードは、いかなるプラグインモジュールも **インポートしません** 。
- プラグインを追加するために Runner のコードを修正する必要はありません。

---

## 2. パッケージ命名規則

すべての公式プラグインは、一貫して `@openstarry-plugin/` スコープを使用します。

```
@openstarry-plugin/{plugin-name}
```

| 完全なパッケージ名 | タイプ | 説明 |
|-----------|------|------|
| `@openstarry-plugin/provider-gemini-oauth` | Provider (想) | Gemini LLM + OAuth 認証 |
| `@openstarry-plugin/standard-function-fs` | Tool (行) | ファイルシステム操作 |
| `@openstarry-plugin/standard-function-stdio` | Listener + Guide (受 + 識) | CLI ターミナル I/O + デフォルト人格 |
| `@openstarry-plugin/standard-function-skill` | Guide (識) | Markdown スキルファイルのロード |

### agent.json での使用例

```json
{
  "plugins": [
    { "name": "@openstarry-plugin/provider-gemini-oauth" },
    { "name": "@openstarry-plugin/standard-function-fs" },
    { "name": "@openstarry-plugin/standard-function-stdio" },
    {
      "name": "@openstarry-plugin/standard-function-skill",
      "config": { "skillPath": "./skills/coder.md" }
    }
  ]
}
```

---

## 3. プラグイン解決戦略（2段階）

Runner の `resolvePlugins()` は、 `agent.json` 内の各プラグインエントリに対して、以下の順序で試行します。

### 戦略 1：ファイルパスによるロード

`ref.path` に値がある場合、そのファイルパスを直接 `import()` します。

```json
{
  "name": "my-custom-plugin",
  "path": "../my-plugins/custom-tool/dist/index.js"
}
```

**適用シナリオ：**
- ローカルで開発中のプラグイン（まだ npm に公開されていないもの）
- pnpm ワークスペース外にあるサードパーティ製プラグイン
- 他の人から提供された単一の `.js` プラグインファイル

### 戦略 2：パッケージ名による動的ロード

`import(ref.name)` を試行し、Node.js のモジュール解決メカニズムによってパッケージを検索します。

```json
{
  "name": "@openstarry-plugin/standard-function-fs"
}
```

**解決パス：**
- pnpm ワークスペースのリンク（開発環境）
- `node_modules/` （本番環境、npm install 後）

---

## 4. プラグインファクトリパターン

各プラグインパッケージは、ファクトリ関数 (factory function) をエクスポートする必要があります。

```typescript
// 名前付きエクスポート
export function createMyPlugin(): IPlugin { ... }

// またはデフォルトエクスポート
export default function createMyPlugin(): IPlugin { ... }
```

Runner は、 `mod.default ?? mod.createPlugin` を試行してファクトリ関数を取得します。

ファクトリ関数は外部パラメータを受け取りません。プラグインの設定は、ファクトリ段階で `IPluginContext.config` （ `agent.json` の `plugins[].config` 由来）を介して注入されます。

---

## 5. 歴史的な経緯

| バージョン | ロード方式 |
|------|---------|
| Plan01 | `BUILTIN_FACTORIES` を CLI 内にハードコード + 短縮名 |
| Plan02 | `ref.path` による動的ロードを追加（3段階戦略） |
| Plan03 Phase C | `BUILTIN_FACTORIES` を削除。全面的な動的ロードへ移行し、完全なパッケージ名に統一。 |
