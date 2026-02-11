# OpenStarry 実装計画 03 — 品質補強とアーキテクチャの純化

> **ステータス**: ✅ 完了 (2026-02-05)

## 背景

Plan01 で MVP の骨格が完成し、 Plan02 でイベント駆動アーキテクチャと安全遮断が補完されました。
テスターは 2026-02-05 にすべてのテスト通過を確認し、次のステップを提案しました。
本計画は、設計ドキュメントの乖離分析結果に基づき、 **v0.1 → v0.2 への品質アップグレードとアーキテクチャの純化** に焦点を当てます。

---

## 目標

1. **品質補強**：設定ミスによるクラッシュ、ツールのフリーズ、デバッグの困難さを回避し、既存システムをより堅牢にする。
2. **能力拡張**： Markdown スキル解析を実装し、エージェントが人格や行動規範を動的にロードできるようにする。
3. **アーキテクチャの純化**： Runner からプラグインを完全に切り離し、すべてのプラグインを動的ロード可能にし、完全なパッケージ名に統一する。

---

## フェーズ A：品質補強（ v0.2 基盤） ✅ 完了済み

### A1. agent.json の Zod による実行時検証 ✅

- パス： `packages/shared/src/utils/config-schema.ts`
- Zod を使用して `AgentConfigSchema` を定義し、すべての設定フィールドをカバー。
- `apps/runner/src/bin.ts` の `loadConfig()` 内で schema 検証を実行。
- 検証失敗時に分かりやすいエラーメッセージを出力。

### A2. ツール呼び出しのタイムアウト (Timeout) ✅

- パス： `packages/core/src/execution/loop.ts` の `executeTool()`
- `Promise.race([tool.execute(...), timeoutPromise])` でラップ。
- タイムアウト値は `config.policy.toolTimeout` （デフォルト 30000ms）から取得。

### A3. TraceID メカニズム ✅

- `processEvent()` の開始時に `traceId` を生成。
- ロガーとイベントペイロードに注入し、一連の処理サイクルを紐付ける。

### A4. CI 純度チェック ✅

- `scripts/check-purity.sh` + `pnpm test:purity`
- core が plugin/apps を参照しない、 sdk が core/shared を参照しないことを保証。

---

## フェーズ B：能力拡張 ✅ 完了済み

### B1. standard-function-skill プラグイン ✅

- パス： `openstarry_plugin/standard-function-skill/`
- `.md` スキルファイルを読み込み、 YAML Frontmatter + Markdown Body を解析。
- Guide プラグインとして GuideRegistry に登録。
- **動的ロード**： Runner にハードコードせず、 `agent.json` で設定：
  ```json
  {
    "name": "@openstarry-plugin/standard-function-skill",
    "config": { "skillPath": "./skills/my-agent.md" }
  }
  ```
- スキルファイルの例： `openstarry_plugin/standard-function-skill/examples/coder.md`
- 7つのユニットテストで frontmatter 解析をカバー。

### B2. system_prompt の外部ファイル参照 ✅

- `IAgentConfig` に `guideFile?: string` フィールドを追加。
- AgentCore 起動時に外部 `.md` ファイルを読み込み、 FileGuide として登録。
- `guide` （ Guide ID ）または `guideFile` （ファイルパス）の2つの方式が共存。

---

## フェーズ C：アーキテクチャの純化 ✅ 完了済み

### C1. IPluginContext.pushInput ✅

**問題点：** stdio プラグインがユーザー入力をプッシュするために、ホストから注入された `onInput` コールバックに依存していた。これにより、 CLI が stdio プラグインを認識して特殊な処理を行う必要があり、完全な動的ロードの目標に反していた。

**解決策：**
- SDK の `IPluginContext` に `pushInput(event: InputEvent)` メソッドを追加。
- コアの `getPluginContext()` で `pushInput → core.pushInput` を自動注入。
- すべてのリスナープラグインが標準化された context を通じて入力をプッシュするようにし、ホストのコールバックへの依存を排除。

### C2. BUILTIN_FACTORIES の削除 ✅

**問題点：** Runner の `BUILTIN_FACTORIES` に3つのプラグインのインポートがハードコードされており、プラグインを追加するたびに Runner を修正する必要があった。

**解決策：**
- `BUILTIN_FACTORIES` およびすべての `@openstarry-plugin/*` インポートを完全に削除。
- プラグインの解決を以下の2段階に限定：
  1. `ref.path` → ファイルパスから直接ロード。
  2. `import(ref.name)` → ワークスペース / node_modules から完全なパッケージ名で動的ロード。
- Runner の `package.json` はいかなるプラグインパッケージにも依存しなくなった。

### C3. 完全なパッケージ名の統一 ✅

- `agent.json` 内のすべてのプラグインで完全なパッケージ名を使用： `@openstarry-plugin/xxx`
- defaultConfig のプラグインリストでも完全なパッケージ名を使用。
- 短縮名と完全な名前の混在を解消。

### C4. apps/cli → apps/runner ✅

- 名称を `apps/runner` に変更し、パッケージ名を `@openstarry/runner` に変更。
- 「純粋なランチャー」としての役割（設定読み込み → コア構築 → プラグイン動的ロード → 起動）を反映。
- 具体的なプラグインを一切認識しない。

---

## 実装順序（すべて完了済み）

```
A1 agent.json Zod 検証          ✅
A2 ツール呼び出し Timeout        ✅
A3 TraceID メカニズム            ✅
A4 CI 純度チェック               ✅
B1 standard-function-skill      ✅
B2 system_prompt 外部ファイル参照 ✅
C1 IPluginContext.pushInput     ✅
C2 BUILTIN_FACTORIES の削除      ✅
C3 完全なパッケージ名の統一      ✅
C4 apps/cli → apps/runner       ✅
```

---

## 検証結果

1. `pnpm build` — ✅ 全8つのワークスペース・プロジェクトのコンパイル通過。
2. `pnpm test` — ✅ 56のテスト通過（7つのテストファイル）。
3. `pnpm test:purity` — ✅ 純度チェック通過。
4. Runner のプラグイン依存ゼロ — ✅ `apps/runner/package.json` は core/sdk/shared にのみ依存。
