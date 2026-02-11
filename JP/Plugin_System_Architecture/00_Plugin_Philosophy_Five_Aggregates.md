# 00. プラグインの哲学：五蘊の集合体 (Plugin as Five Aggregates)

> **[重要事項]** このドキュメントでは、 OpenStarry プラグインシステムの背後にある **アーキテクチャ上の理論的マッピング** について説明します。

## 核心的な哲学

**プラグインとは特定の型ではなく、一連の能力の集まりである。**

生命体が五蘊（ごうん）の集まりによって構成されているように、完全に機能するプラグインは、しばしば同時に複数の成分を必要とします。 OpenStarry のプラグインシステムでは、1つのプラグインパッケージが五蘊の成分を任意に組み合わせて提供することを可能にしています。

---

## 五蘊成分の定義 (The Aggregates)

`PluginHooks` において、私たちはこれらの成分を宣言するためにセマンティック（意味論的）なフィールドを使用します。これらはアーキテクチャ上で厳密に五蘊に対応しています。

```typescript
// PluginHooks 定義 ( @openstarry/sdk より)
interface PluginHooks {
  ui?: IUI[];          // 色蘊 (Rupa) - インターフェースと現れ。イベントを受信して出力を提示する
  listeners?: IListener[]; // 受蘊 (Vedana) - 感覚の監視。外部入力を受信する
  providers?: IProvider[]; // 想蘊 (Samjna) - 認知の脳。 LLM アダプター
  tools?: ITool[];     // 行蘊 (Samskara) - 実行ツール。ファイル/API/コードなど
  guides?: IGuide[];   // 識蘊 (Vijnana) - 魂の導きとスキル。 system prompt
  commands?: SlashCommand[];
  dispose?: () => Promise<void> | void;
}
```

## よくある集合パターン

### 1. 純粋な行蘊プラグイン (Pure Executor)
*   **成分：** `tools` のみ。
*   **例：** `fs-utils` 。

### 2. 完全な感覚プラグイン (Full Sensory Plugin)
*   **成分：** `listeners` + `tools` 。
*   **例：** `discord-connector` （メッセージの受信と送信の両方が可能）。

### 3. 魂の注入プラグイン (Soul Injector)
*   **成分：** `guides` （特定の `tools` を伴う場合がある）。
*   **例：** `expert-coder-skill` 。
    *   **識 (Guide):** エンジニアの人格を注入。
    *   **行 (Tool):** 特定の `linter` ツールを付属。

---

## アーキテクチャ上の利点

1.  **高い凝集性 (High Cohesion):** 関連する能力が同じパッケージ内にまとめられます。
2.  **原子的なロード (Atomic Loading):** ユーザーは1つの npm パッケージをインストールするだけで、特定の領域における完全な能力を獲得できます。
3.  **哲学的な一貫性:** これはコアの五蘊アーキテクチャと完璧に呼応しています。コアは五蘊の容器であり、プラグインは五蘊の供給者です。
