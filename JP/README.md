# OpenStarry: エージェント・オペレーティングシステム

**OpenStarry** は、AIエージェントの構築方法を根本から再定義するコアアーキテクチャです。モダンなオペレーティングシステムの設計哲学を参考にしつつ、東洋の「五蘊（ごうん）」思想を融合させ、高度にモジュール化され、安全で、擬人的な生命特性を備えたエージェント協調レイヤーの構築を目指しています。

私たちが構築するのは単なるチャットボットではありません。**デジタル種族のためのオペレーティングシステム**です。

---

## 🏗️ マクロシステムアーキテクチャ (Macro-System Architecture)

OpenStarry は三層の段階的なアーキテクチャ設計を採用し、生物とその生存環境の共生関係をシミュレートしています。

### 1. エージェント協調管理層 (Management Zone)
**位置づけ：システムのホスト環境と行政中枢。**
土壌と養分を提供する層です。環境の安定性と安全性を確保し、コンテナ隔離（Plumbing）、因果連鎖に基づくイベントスケジューリング（Orchestration）、セキュリティポリシー（Policy）、およびハードウェア抽象化層（HAL）を含みます。物理世界の信号をエージェントが理解可能なデータストリームに変換します。

### 2. Agent Core（自律生命ゾーン）
**位置づけ：純粋な「五蘊」計算ループ。**
「ヘッドレス（Headless）」かつ「ステートレス（Stateless）」な生命の内核です。唯一の責務は「受・想・行・識」の計算ループを維持すること。Core は本質的に空（くう）であり、異なるプラグインによって異なる生命の様態を現します。

### 3. 能力プラグイン層 (Capability Plugins)
**位置づけ：エージェントに個性、専門性、そして魂を与える機能コンポーネント。**
プラグインがエージェントの能力の境界を決定します。通信プロトコル（Protocol）、自己内省（Reflection）、状態記憶（Memory）プラグインなどが含まれます。これにより、同一の Core がいつでも「コードエキスパート」から「デバイス監視員」へと変化できます。

---

## 🔄 因果ライフサイクル (The Lifecycle)

OpenStarry では、タスクの実行は一つの生命の起滅として捉えられます：
1. **縁起 (Origination)**：環境層がニーズを検知します。
2. **調度 (Scheduling)**：管理層がニーズに応じて必要なプラグインをマッチングします。
3. **生起 (Arising)**：コンテナ層がコアをロードし、動的に能力を注入します。
4. **運行 (Operation)**：コアが「痛覚」を処理し、目標を達成します。
5. **寂滅 (Cessation)**：タスク完了後、経験がメモリに保存され、インスタンスは消滅します。

```mermaid
graph TD
    subgraph Host [🛡️ Management Zone (ホスト環境)]
        direction TB
        Orchestrator[スケジューリング層] --> Container[コンテナ層]
        Policy[セキュリティポリシー層] -.-> Container
        HAL[ハードウェア抽象化層] --> InputFlow((知覚フロー))
    end

    subgraph Runtime [⚡ 実行インスタンス]
        direction LR
        InputFlow --> Core

        subgraph Core [🧠 Agent Core (マイクロカーネル)]
            Loop[実行ループ]
            State[ステートマシン]
            Interceptor[異常インターセプター]
        end

        Core --> |1. Load| Plugins

        subgraph Plugins [🔌 能力プラグイン (五蘊)]
            Guide[識：Guide]
            Tool[行：Tools]
            LLM[想：Provider]
            Mem[記憶：Memory]
            Pain[痛覚：Reflex]
        end

        Plugins --> |2. Inject| Core
        Interceptor -.-> |3. Pain Signal| Guide
        Guide -.-> |4. Correction| Loop
    end
```

---

## 💻 コア設定例 (The Shape of an Agent)

OpenStarry の強力さは、その宣言的な設定にあります。以下は「痛覚」と「ファイル操作能力」を備えた標準的なエージェント定義です：

```jsonc
// agent.json
{
  "identity": { "id": "dev-bot-01", "name": "Resilient Developer" },
  "plugins": [
    // [想] 脳：認知エンジンを注入
    { "name": "@openstarry-plugin/provider-gemini" },

    // [行] 手足：ファイルシステム操作能力を注入
    { "name": "@openstarry-plugin/standard-function-fs" },

    // [受] 感覚器官：ターミナル入力をリッスン
    { "name": "@openstarry-plugin/standard-function-stdio" },

    // [識] 魂：痛覚メカニズムを注入（エラーへの対処方法を定義）
    { "name": "@openstarry-plugin/guide-pain-mechanism" }
  ],
  "policy": {
    // 管理層の戒律：連続3回のエラーで物理的なサーキットブレーカーを発動
    "safety": { "max_consecutive_errors": 3 }
  }
}
```

---

## 🌟 10の核心宣言 (The Ten Tenets)

### 1. エージェントはOSプロセスである (Agent as OS Process)
エージェントは使い捨てのスクリプトではなく、永続的なライフサイクルを持ち、デーモン（Daemon）によって管理・監視・再起動される「デジタルエンティティ」です。独自のPID、独自の状態を持ち、まるで生きているプロセスのように振る舞います。

### 2. すべてはプラグインである (Everything is a Plugin)
システムのあらゆる器官は交換可能です。ツールはプラグイン、リスナーはプラグイン、LLMの頭脳もプラグインであり、記憶戦略や通信プロトコルさえもプラグインです。Core は空のソケットボードに過ぎず、すべての能力は外部から装着されます。

### 3. 五蘊集合アーキテクチャ (Five Aggregates Architecture)
システム設計は東洋哲学と深く融合しています。**Core は本質的に「空（Sunyata）」のコンテナです。** その生命特性はすべて五種のプラグイン（五蘊）によって賦与されます：
*   **色 (UI)**、**受 (Listener)**、**想 (Provider)**、**行 (Tool)**。
*   **識 (Guide):** 最も重要なコンポーネントです。Guide プラグインが記憶とペルソナを注入し、Core に「自己意識（Vijnana）」を与えます。Guide がなければ、Core はただの無意識な計算力に過ぎません。

### 4. ディレクトリ構造がプロトコルである (Directory as Protocol)
システムであれプロジェクトであれ、ローカルディスクであれUSBデバイスであれ、ディレクトリ構造が `plugins/`、`configs/` の標準規範に準拠していれば、システムは自動的にそれを認識してロードできます。物理的な構造がランタイムのロジックに直接マッピングされます。

### 5. ディレクトリ構造が権限である (Directory as Permission)
システム層とプロジェクト層は同型設計を採用しつつ、権限は厳密に隔離されます。プラグインの配置場所がその可視範囲を決定し、エージェントの実行場所がその権限境界を決定します。システム管理者がビジネスプラグインに直接触れることはできず、安全な隔離が保証されます。

### 6. 擬人化された認知フローと痛覚 (Anthropomorphic Cognitive Flow & Pain)
エラーはエージェントの「痛覚（ネガティブフィードバック）」に変換されます。システム内蔵のフィードバックループがランタイムエラーをコンテキストに注入し、エージェントに失敗からの自己省察と修正を促します。これは生物の試行錯誤による学習プロセスをシミュレートしています。

### 7. マイクロカーネルと絶対的な純粋性 (Microkernel & Absolute Purity)
Agent Core は厳格な**マイクロカーネルアーキテクチャ (Microkernel Architecture)** を採用しています。
*   **物理的隔離:** コンパイル済みの Core バイナリには**プラグインコードの混入が一切禁止**されています。
*   **絶対的純粋性:** Core は抽象インターフェース（SDK）にのみ依存し、具体的な能力を一切持ちません。すべての能力はランタイム時に外部プラグインから動的に注入されます。
*   **ヘッドレス設計 (Headless):** カーネルは非中央集権的で、特定の UI や IO デバイスに依存しません。これにより、エージェントの「魂」をあらゆる「器」に移植可能にしています — CLI から Web、Docker から IoT デバイスまで。
*   **意義:** ビルトインコードがなければ、ビルトインバグもありません。

### 8. 制御理論による閉ループモデル (Control-Theoretic Loop Model)
単なる実行ループではなく、制御ループです。システムはユーザーの目標を参照入力として、コンテキストを状態フィードバックとして、Tool Call を制御変数として扱います。エージェントの本質は、「目標と現状の誤差」を絶えず最小化するインテリジェントコントローラーです。

### 9. プラガブルな記憶戦略 (Pluggable Context Strategy)
記憶管理はハードコードされたロジックではなくなりました。開発者はエージェントの役割に応じて、記憶戦略（スライディングウィンドウ、動的要約、状態抽出）を動的に切り替えることができ、コストと記憶の深さを柔軟にバランスできます。

### 10. フラクタル社会構造 (Fractal Social Structure)
システムは自己相似性を持ちます。複雑なエージェントは複数のサブエージェントで構成し、外部には統一されたMCPインターフェースを公開できます。このフラクタル設計により、無限の階層を持つ協調ネットワークの構築が可能になり、「一から万物を生む」デジタル社会を実現します。

---

## 📚 ドキュメントナビゲーション (Documentation Map)

### 1. システムアーキテクチャドキュメント (Architecture Documentation)
*システムのビジョン、役割、マクロレベルの起動フローを定義します。*
* [00_設計哲学 (OpenStarry Design Philosophy)](./Architecture_Documentation/00_OpenStarry_Design_Philosophy.md)
* [01_アーキテクチャ概要 (Architecture Overview)](./Architecture_Documentation/01_Architecture_Overview.md)
* [02_ヘッドレスエージェントコア (Headless Agent Core)](./Architecture_Documentation/02_Headless_Agent_Core.md)
* [03_エージェント設計とテンプレートサービス (Agent Design & Template Service)](./Architecture_Documentation/03_Agent_Design_and_Template_Service.md)
* [04_プラグインインフラストラクチャ (Plugin Infrastructure)](./Architecture_Documentation/04_Plugin_Infrastructure.md)
* [05_Linux設計原則からの着想 (Linux Design Principles Inspiration)](./Architecture_Documentation/05_Linux_Design_Principles_Inspiration.md)
* [06_プラグインインターフェース例 (Plugin Interface Examples)](./Architecture_Documentation/06_Plugin_Interface_Examples.md)
* [07_サポートエンジンエコシステム (Supporting Engines Ecosystem)](./Architecture_Documentation/07_Supporting_Engines_Ecosystem.md)
* [08_コマンドとツール設計 (Command & Tool Design)](./Architecture_Documentation/08_Command_And_Tool_Design.md)
* [09_通信プロトコル戦略 (Communication Protocol Strategy)](./Architecture_Documentation/09_Communication_Protocol_Strategy.md)
* [10_ブートストラップとプラグインローディング (Bootstrapping & Plugin Loading)](./Architecture_Documentation/10_Bootstrapping_And_Plugin_Loading.md)
* [11_エージェントマネージャーツール設計 (Agent Manager Tool Design)](./Architecture_Documentation/11_Agent_Manager_Tool_Design.md)
* [12_ワークフローエンジンツール設計 (Workflow Engine Tool Design)](./Architecture_Documentation/12_Workflow_Engine_Tool_Design.md)
* [13_オーケストレーターデーモン設計 (Orchestrator Daemon Design)](./Architecture_Documentation/13_Orchestrator_Daemon_Design.md)
* [14_システムブートシーケンス (System Boot Sequence)](./Architecture_Documentation/14_System_Boot_Sequence.md)
* [15_起動とタスクフロー (System Startup & Task Flow)](./Architecture_Documentation/15_System_Startup_and_Task_Flow.md)
* [16_プラグインタイプの哲学的マッピング (Plugin Types Philosophical Mapping)](./Architecture_Documentation/16_Plugin_Types_Philosophical_Mapping.md)
* [17_ホストブートストラップパターン (Host Bootstrapping Pattern)](./Architecture_Documentation/17_Host_Bootstrapping_Pattern.md)
* [18_プラグインローディングプロトコル (Plugin Loading Protocol)](./Architecture_Documentation/18_Plugin_Loading_Protocol.md)
* [19_エージェント協調層 (Agent Coordination Layer)](./Architecture_Documentation/19_Agent_Coordination_Layer.md)
* [20_依存性注入と制御ループ (Dependency Wiring & Control Loop)](./Architecture_Documentation/20_Dependency_Injection_and_Control_Loop.md)
* [21_プラグインインターフェース詳細解析 (Plugin Interface Deep Dive)](./Architecture_Documentation/21_Plugin_Interface_Deep_Dive.md)
* [22_エージェント協調層：正規化とアダプテーション (Agent Coordination Layer: Normalization)](./Architecture_Documentation/22_Agent_Coordination_Layer_Normalization.md)
* [23_動的プラグインローディングと命名規則 (Dynamic Plugin Loading & Naming)](./Architecture_Documentation/23_Dynamic_Plugin_Loading_and_Naming.md)
* [24_Runner アーキテクチャ (Runner Architecture)](./Architecture_Documentation/24_Runner_Architecture.md)
* [25_PushInput イベントアーキテクチャ (PushInput Event Architecture)](./Architecture_Documentation/25_PushInput_Event_Architecture.md)
* [26_プラグインサービスとライフサイクル管理 (Plugin Service & Lifecycle Management)](./Architecture_Documentation/26_Plugin_Service_And_Lifecycle_Management.md)
* [27_システムトポロジーと管理層アーキテクチャ (System Topology & Management Zone)](./Architecture_Documentation/27_System_Topology_and_Management_Zone.md)

### 2. コアコンポーネント詳細解説 (Agent Core Components Deep Dive)
*カーネルの内部に踏み込み、具体的な技術メカニズムと理論モデルを研究します。*
* [00_コア哲学 (Core Philosophy)](./Agent_Core_Components_Deep_Dive/00_Core_Philosophy.md)
* [01_実行ループ (Execution Loop)](./Agent_Core_Components_Deep_Dive/01_Execution_Loop.md)
* [02_通信インターフェース (Communication Interface)](./Agent_Core_Components_Deep_Dive/02_Communication_Interface.md)
* [03_セキュリティ層 (Security Layer)](./Agent_Core_Components_Deep_Dive/03_Security_Layer.md)
* [04_ステートマネージャー (State Manager)](./Agent_Core_Components_Deep_Dive/04_State_Manager.md)
* [05_プラグインインフラストラクチャ統合 (Plugin Infrastructure Integration)](./Agent_Core_Components_Deep_Dive/05_Plugin_Infrastructure_Integration.md)
* [06_状態永続化メカニズム (State Persistence Mechanism)](./Agent_Core_Components_Deep_Dive/06_State_Persistence_Mechanism.md)
* [07_安全サーキットブレーカー (Safety Circuit Breakers)](./Agent_Core_Components_Deep_Dive/07_Safety_Circuit_Breakers.md)
* [08_安全実装 (Safety Implementation)](./Agent_Core_Components_Deep_Dive/08_Safety_Implementation.md)
* [09_可観測性とトレーシング (Observability and Tracing)](./Agent_Core_Components_Deep_Dive/09_Observability_and_Tracing.md)
* [10_コンテキスト管理戦略 (Context Management Strategy)](./Agent_Core_Components_Deep_Dive/10_Context_Management_Strategy.md)
* [11_プラグインランタイム隔離 (Plugin Runtime Isolation)](./Agent_Core_Components_Deep_Dive/11_Plugin_Runtime_Isolation.md)
* [12_エラーハンドリングと自己修正 (Error Handling & Self Correction)](./Agent_Core_Components_Deep_Dive/12_Error_Handling_and_Self_Correction.md)
* [13_エージェントコアとしての制御システム (Agent Core as Control System)](./Agent_Core_Components_Deep_Dive/13_Agent_Core_as_Control_System.md)
* [14_エージェントコア哲学：五蘊 (Agent Core Philosophy: Five Aggregates)](./Agent_Core_Components_Deep_Dive/14_Agent_Core_Philosophy_Five_Aggregates.md)
* [16_OpenStarry 標準プロトコル (OpenStarry Standard Protocol)](./Agent_Core_Components_Deep_Dive/16_OpenStarry_Standard_Protocol.md)

### 3. プロジェクト構造と規約 (Project Structure and Conventions)
*物理レイアウト、ソースコード構成、開発フロー、インストール規範を定義します。*
* [00_ロードマップとマイルストーン (Roadmap & Milestones)](./Project_Structure_and_Conventions/00_Roadmap_and_Milestones.md)
* [01_Monorepo トップレベル構造 (Monorepo Top Level Structure)](./Project_Structure_and_Conventions/01_Monorepo_Top_Level_Structure.md)
* [02_コアソースコード構造 (Core Source Code Structure)](./Project_Structure_and_Conventions/02_Core_Source_Code_Structure.md)
* [03_共有コンポーネントと SDK 構造 (Shared & SDK Structure)](./Project_Structure_and_Conventions/03_Shared_and_SDK_Structure.md)
* [04_標準エージェントディレクトリ構造 (Standard Agent Directory Anatomy)](./Project_Structure_and_Conventions/04_Standard_Agent_Directory_Anatomy.md)
* [05_エージェントマニフェスト仕様 (Agent Manifest Specification)](./Project_Structure_and_Conventions/05_Agent_Manifest_Specification.md)
* [06_プラグインディレクトリ規約 (Plugin Directory Conventions)](./Project_Structure_and_Conventions/06_Plugin_Directory_Conventions.md)
* [07_コーディングとテスト基準 (Coding & Testing Standards)](./Project_Structure_and_Conventions/07_Coding_and_Testing_Standards.md)
* [08_システムとプロジェクトのランタイムレイアウト (System & Project Runtime Layouts)](./Project_Structure_and_Conventions/08_System_and_Project_Runtime_Layouts.md)
* [09_CLI 設計と管理コマンド (CLI Design & Management Commands)](./Project_Structure_and_Conventions/09_CLI_Design_and_Management_Commands.md)
* [10_ビルドと配布戦略 (Build & Distribution Strategy)](./Project_Structure_and_Conventions/10_Build_and_Distribution_Strategy.md)
* [11_サードパーティプラグインのインストール (Third-Party Plugin Installation)](./Project_Structure_and_Conventions/11_Third_Party_Plugin_Installation.md)
* [12_能力注入メカニズム (Capabilities Injection Mechanism)](./Project_Structure_and_Conventions/12_Capabilities_Injection_Mechanism.md)
* [13_複合プラグインと依存関係 (Composite Plugins & Dependencies)](./Project_Structure_and_Conventions/13_Composite_Plugins_and_Dependencies.md)
* [14_Markdown スキル仕様 (Markdown Skill Specification)](./Project_Structure_and_Conventions/14_Markdown_Skill_Specification.md)

### 4. プラグインシステムアーキテクチャ (Plugin System Architecture)
*プラグインシステムの具体的な応用、コンセプト、仕様です。*
* [00_プラグイン哲学：五蘊 (Plugin Philosophy: Five Aggregates)](./Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md)
* [01_MCP プラグイン例 (MCP Plugin Example)](./Plugin_System_Architecture/01_MCP_Plugin_Example.md)
* [02_MCP プロトコル統合 (MCP Protocol Integration)](./Plugin_System_Architecture/02_MCP_Protocol_Integration.md)
* [03_開発者ツール例 (Developer Tools Example)](./Plugin_System_Architecture/03_Developer_Tools_Example.md)
* [04_Web インタラクション例 (Web Interaction Example)](./Plugin_System_Architecture/04_Web_Interaction_Example.md)
* [05_高度な UI とデバイス例 (Advanced UI & Device Example)](./Plugin_System_Architecture/05_Advanced_UI_And_Device_Example.md)
* [06_データバリデーション例 (Data Validation Example)](./Plugin_System_Architecture/06_Data_Validation_Example.md)

### 5. 実装例とガイド (Implementation Examples)
*手を動かしてコードを書き、事例から実践を学びます。*
* [コンテキスト戦略：スライディングウィンドウ (Context Strategy: Sliding Window)](./Implementation_Examples/Context_Strategy_SlidingWindow.md)
* [開発者ガイド：スタンドアロン実行 (Developer Guide: Standalone Execution)](./Implementation_Examples/Developer_Guide_Standalone_Execution.md)
* [OpenClaw 協調層 (OpenClaw Coordination Layer)](./Implementation_Examples/openclaw_Coordination_Layer.md)
* [OpenClaw UI チャンネルアダプター (OpenClaw UI Channel Adapters)](./Implementation_Examples/openclaw_UI_Channel_Adapters.md)
* [OpenCode コードインタプリタスイート (OpenCode Code Interpreter Suite)](./Implementation_Examples/opencode_Code_Interpreter_Suite.md)
* [Provider: Gemini 例](./Implementation_Examples/Provider_Gemini_Example.md)
* [Tool: コードインタプリタ例](./Implementation_Examples/Tool_CodeInterpreter_Example.md)
* [Tool: ファイル読み込み例](./Implementation_Examples/Tool_ReadFile_Example.md)
* [Transport: WebSocket プラグイン](./Implementation_Examples/Transport_Plugin_Websocket.md)
* [UI プラグイン例 (UI Plugin Example)](./Implementation_Examples/UI_Plugin_Example.md)
* [USB プラグアンドプレイ・エージェントシナリオ (USB Plug-and-Play Agent Scenario)](./Implementation_Examples/USB_Plug_and_Play_Agent_Scenario.md)
* [擬人化痛覚メカニズムデモ (Pain Mechanism Demo)](./Implementation_Examples/Pain_Mechanism_Demo.md)


---

## 🛠️ クイックスタート

準備はできましたか？**[Developer_Guide_Standalone_Execution.md](./Implementation_Examples/Developer_Guide_Standalone_Execution.md)** を参照して、最初のエージェントを実行しましょう。
