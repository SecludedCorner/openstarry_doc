# 02. Core ソースコードディレクトリ構造仕様 (Core Source Code Structure)

このドキュメントでは、 `packages/core/src/` 内部のコード組織方式を定義します。この構造は、「高度な疎結合、テスト可能性、および擬人化アーキテクチャへの適合」という開発目標を実現することを目指しています。

## ソースコードディレクトリのツリー図

```text
src/
├── agents/             # エージェント実体クラス (Agent Entities)
├── execution/          # 実行、スケジューリング、およびイベントフロー (The Loop)
├── memory/             # コンテキスト管理と記憶戦略 (Working Memory)
├── infrastructure/     # プラグインのロード、登録、および隔離 (Plugin Infra)
├── security/           # セキュリティ傍受と遮断器 (Guardrails)
├── state/              # ステートスナップショットと永続化 (State Manager)
├── transport/          # コア通信ブリッジレイヤー (Communication Bridge)
├── types/              # TypeScript 強型定義
└── index.ts            # ライブラリエントリファイル
```

---

## モジュール責務の詳細（五蘊哲学へのマッピングを含む）

### 1. `execution/` (魂魄：実行システム) —— [マッピング：行蘊 Samskara / 識蘊 Vijnana]
エージェントの生命の中枢であり、「実行ループ」と「制御理論」モデルを実装します。
*   **`loop/`**: `tick()` メカニズムを実装し、イベントキューからの取り出し、 LLM の呼び出し、およびアクション（意志と形成）のトリガーを担当します。
*   **`queue/`**: 非同期イベントの優先順位を管理します。

### 2. `memory/` (記憶：コンテキスト戦略) —— [マッピング：想蘊 Samjna]
`10_Context_Management_Strategy` で定義されたロジックを実装します。
*   **`context/`**: 受信した情報の識別、関連付け、および圧縮（認知とパターン認識）を担当します。

### 3. `infrastructure/` (器官：プラグイン管理) —— [マッピング：色蘊 Rupa]
「すべてはプラグインである」物理的なロードを実装します。
*   **`loader/` & `registry/`**: エージェントの「身体部品」のスキャンと保存を担当します。プラグインがロードされると、その具体的な機能（色）を分解してシステムに登録します。

### 4. `security/` (超我：セキュリティレイヤー) —— [マッピング：受蘊 Vedana の防御面]
「意思決定」と「実行」の間に位置するファイアウォールです。
*   **`guardrails/`**: 異常をキャッチし、それをエージェントの「負の感覚（痛覚）」に変換することで、自己修復をトリガーします。

### 5. `transport/` (感官：通信ブリッジ) —— [マッピング：受蘊 Vedana の感知面]
`02_Communication_Interface` で定義された API を実装します。
*   **`bridge/`**: 外部からの刺激（目、耳、身などのリスナー）に対応し、外部情報を内部の感覚に変換します。

---

## 命名規則 (Naming Conventions)

1.  **クラス (Classes):** PascalCase を使用。例： `AgentCore`, `ExecutionLoop` 。
2.  **インターフェース (Interfaces):** `I` で始める。例： `IPlugin`, `ITool` 。
3.  **ファイル拡張子:** 
    *   ロジック実装： `.ts`
    *   ユニットテスト： `.test.ts` （ソースファイルと同じディレクトリに配置）
4.  **エントリ：** 各ディレクトリには公開 API をエクスポートするための `index.ts` を含め、外部からの参照を簡潔に保ちます。

---

## コード進化の原則

*   **副作用なし (No Side Effects):** `infrastructure/` と `transport/` を除き、他のモジュールは可能な限り純粋なロジックを維持し、ネットワークやファイルシステムを直接操作しないようにします。
*   **依存性の注入 (Dependency Injection):** `AgentCore` クラスは初期化時に `IContextManager`, `IStateManager` のインスタンスを受け取るようにし、テスト時に Mock バージョンに容易に置き換えられるようにします。
