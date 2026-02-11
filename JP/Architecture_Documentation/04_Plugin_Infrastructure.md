# 04. プラグインインフラストラクチャ (Plugin Infrastructure)

このドキュメントでは、OpenStarry のプラグインシステムアーキテクチャを定義します。私たちは **アグリゲートパターン (Aggregate Pattern)** を採用しており、1つのプラグインパッケージが同時に複数の能力を提供することを可能にしています。

> **設計上の注釈 (Architectural Note):**
> このアーキテクチャは「五蘊 (Five Aggregates)」の思想を深く取り入れており、プラグインの5つの基本形態を定義しています。
> 1.  **UI (色蘊 Rupa):** 物理的な現れとインターフェース。
> 2.  **リスナー (受蘊 Vedana):** 感知のインターフェース (Input)。
> 3.  **プロバイダー (想蘊 Samjna):** 認知のエンジン (Processing)。
> 4.  **ツール (行蘊 Samskara):** 実行の能力 (Output/Action)。
> 5.  **ガイド (識蘊 Vijnana):** 魂の導き (Identity/Protocol/Logic)。コア自体は空の容器であり、ガイドこそがそれに「識別」と「自己意識」を与えます。

---

## プラグイン構造の仕様

OpenStarry において、 **プラグイン (Plugin)** は機能単位であり、以下のコンポーネントを任意に組み合わせて含めることができます。

### `plugin.json` 定義

```json
{
  "id": "openstarry-full-stack",
  "components": {
    "ui": { ... },        // [色]
    "listeners": [ ... ], // [受]
    "providers": [ ... ], // [想]
    "tools": [ ... ],     // [行]
    "guides": [           // [識]
      { "name": "expert-persona", "entry": "./skills/expert.md" },
      { "name": "mcp-protocol", "entry": "./protocols/mcp-logic.js" }
    ]
  }
}
```

---

## プラグインロードプロセス (Loading Mechanism)

`PluginLoader` がプラグインをロードする際、「解体と登録」のプロセスを実行します。

1.  **マニフェストの読み取り:** `plugin.json` を読み取ります。
2.  **成分の解体 (Destructuring):**
    *   `components.tools` をスキャンし、 **ToolRegistry** に登録します。
    *   `components.listeners` をスキャンし、 **ListenerRegistry** に登録して起動します。
    *   `components.providers` をスキャンし、 **ProviderRegistry** に登録します。
3.  **依存性の注入:** 各成分に、必要となる Core API を注入します。

これは、開発者が「機能パッケージ」の形式でプラグインを公開し（例: `npm install @openstarry-plugin/whatsapp`）、コアがその中の手足や耳目を自動的に解体して適切な場所に配置することを意味します。

### ハードコードされたファクトリに代わる動的ロード (Dynamic Loading Replaces Hardcoded Factories)

初期のバージョンでは、コアは `BUILTIN_FACTORIES` マップを介して組み込みプラグインのロードロジックをハードコードしていました。この方式は完全に廃止され、統合された **2段階の動的解決 (Two-Tier Resolution)** に置き換えられました。

1.  **`ref.path` 優先：** `agent.json` 内のプラグイン参照に `path` フィールド（ローカルファイルシステムパスを指す）が含まれている場合、ローダーはそのパスを介してプラグインモジュールを直接ロードします。
2.  **`import(ref.name)` バックアップ：** `path` がない場合、ローダーは `import(ref.name)` を使用して NPM パッケージ名に従って動的ロードを行います（例: `import('@openstarry-plugin/fs')`）。

この統合メカニズムにより、「組み込みプラグイン」と「サードパーティプラグイン」の境界がなくなり、すべてのプラグインが同じロードパスとライフサイクル管理を享受できます。

---

## グローバル・プラグイン・レジストリ (The Global Plugin Registry)

これらの能力がコア内部のレジストリに入る前に、調整レイヤー (Daemon) による発見とインデックス作成を経る必要があります。これはシステムレベルのサービスです。

### 1. 責務 (Responsibilities)

*   **スキャンと発見 (Scanning):** 起動時に `~/.openstarry/plugins/` (システム層) および `./.openstarry/plugins/` (プロジェクト層) をスキャンします。
*   **インデックス作成 (Indexing):** すべてのプラグインのマニフェストを読み取り、 `Plugin ID -> Physical Path` のマッピングテーブルを作成します。
*   **能力の解決 (Capability Resolution):** 設計レイヤーが「誰が `send_message` 能力を持っているか？」を問い合わせた際、レジストリは対応するプラグインリストを返す役割を担います。

### 2. 発見プロセス

1.  **起動:** デーモンが `PluginRegistryService` を起動します。
2.  **巡回:** サービスはすべての既知のパスを巡回し、 `package.json` または `plugin.json` を探します。
3.  **登録:** 有効なプラグインのメタデータをメモリ内データベースに保存します。
4.  **提供:** 具体的なエージェントが起動する際、ローダーはこのサービスに指定されたプラグインのパスを要求し、その後にロードを行います。

---

## 標準能力レジストリ (The Registry)

Core 内部では、いくつかのグローバルレジストリが維持されています。

*   `core.registry.tools`: 利用可能なすべてのツール関数を保存します。
*   `core.registry.listeners`: すべてのアクティブなリスナーインスタンスを管理します。
*   `core.registry.providers`: 利用可能な LLM バックエンドを保存します。

この設計は、エンジニアリング上の直観性を保証すると同時に、低層のロジックにおいて五蘊アーキテクチャの完全性を維持しています。片。