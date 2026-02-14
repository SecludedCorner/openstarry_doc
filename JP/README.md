# OpenStarry: エージェント・オペレーティングシステム

**OpenStarry** は、AIエージェントの構築方法を再定義するコア・アーキテクチャです。現代のオペレーティングシステムの設計哲学を参考にし、東洋の「五蘊（ごうん）」思想を融合させることで、高度にモジュール化され、安全で、擬人化された生命特性を持つエージェント調整層を目指しています。

私たちは単なるチャットボットを作るのではなく、**デジタル種のためのOS**を構築しています。

---

## 🏗️ システム・マクロ・アーキテクチャ

OpenStarryは、生物とその生存環境の共生関係を模倣した、3層の漸進的なアーキテクチャを採用しています：

### 1. エージェント調整管理層 (Management Zone)
**位置付け：システムのホスト環境と行政中枢。**
土壌と養分を提供する役割。環境の安定性と安全性を確保し、コンテナ隔離 (Plumbing)、因果関係に基づくイベントスケジューリング (Orchestration)、安全ポリシー (Policy)、およびハードウェア抽象化層 (HAL) を含みます。

### 2. エージェント・コア (Autonomous Life Zone)
**位置付け：純粋な「五蘊」計算サイクル。**
「無頭 (Headless)」かつ「無状態 (Stateless)」の生命核。唯一の責務は「受・想・行・識」の計算サイクルを維持することです。コア自体は空であり、プラグインによって異なる生命形態を見せます。

### 3. 能力プラグイン層 (Capability Plugins)
**位置付け：エージェントに個性、専門性、魂を与える機能コンポーネント。**
プラグインがエージェントの能力の境界線を決定します。通信プロトコル、自己反省、状態記憶プラグインなどを含みます。

---

## 🔄 因果ライフサイクル (The Lifecycle)

OpenStarryでは、タスクの実行は生命の誕生と消滅として扱われます：
1. **縁起 (Origination)**：環境層が要求を検知。
2. **スケジューリング (Scheduling)**：管理層が要求に合わせてプラグインをマッチング。
3. **生起 (Arising)**：コンテナ層がコアをロードし、動的に能力を注入。
4. **運行 (Operation)**：コアが「痛覚」を処理し、目標を達成。
5. **寂滅 (Cessation)**：タスク完了、経験を記憶に保存し、インスタンスを破棄。

```mermaid
graph TD
    subgraph Host [🛡️ Management Zone (Host Environment)]
        direction TB
        Orchestrator[調整層] --> Container[コンテナ層]
        Policy[安全ポリシー層] -.-> Container
        HAL[ハードウェア抽象化層] --> InputFlow((感知フロー))
    end

    subgraph Runtime [⚡ Running Instance]
        direction LR
        InputFlow --> Core
        
        subgraph Core [🧠 Agent Core (Microkernel)]
            Loop[実行ループ]
            State[状態マシン]
            Interceptor[異常インターセプト]
        end

        Core --> |1. Load| Plugins
        
        subgraph Plugins [🔌 Capability Plugins (The 5 Aggregates)]
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

OpenStarryの強みは宣言的な設定にあります。以下は「痛覚」と「ファイル操作能力」を備えた標準的なエージェント定義です：

```jsonc
// agent.json
{
  "identity": { "id": "dev-bot-01", "name": "Resilient Developer" },
  "plugins": [
    // [想] 脳：認知エンジンを注入
    { "name": "@openstarry-plugin/provider-gemini" },
    
    // [行] 手足：ファイルシステム操作能力を注入
    { "name": "@openstarry-plugin/standard-function-fs" },
    
    // [受] 感覚：ターミナル入力を監視
    { "name": "@openstarry-plugin/standard-function-stdio" },
    
    // [識] 魂：痛覚メカニズムを注入 (エラーへの対処方法を定義)
    { "name": "@openstarry-plugin/guide-pain-mechanism" }
  ],
  "policy": {
    // 管理層の戒律：3回連続のエラーで物理的な遮断を発動
    "safety": { "max_consecutive_errors": 3 } 
  }
}
```

---

## 🌟 十大コア宣言

1. **エージェントはOSプロセスである**: エージェントはPIDを持ち、デーモンによって管理される生命体のようなプロセスです。
2. **すべてはプラグインである**: コアはただのソケットボードであり、すべての能力は外部から装着されます。
3. **五蘊集約アーキテクチャ**: コアの本質は「空 (Sunyata)」であり、生命特性は五蘊プラグインによって与えられます。
4. **ディレクトリ構造はプロトコルである**: 物理的なディレクトリ構造がそのまま実行時の論理にマッピングされます。
5. **ディレクトリ構造は権限である**: システム層とプロジェクト層の隔離により、安全な実行境界を保証します。
6. **擬人化された認知フローと痛覚**: エラーを「痛覚」として処理し、自己反省と修正を促します。
7. **マイクロカーネルと絶対的な純粋性**: コアはプラグインコードを含まず、抽象インターフェース (SDK) にのみ依存します。
8. **制御理論の閉ループモデル**: エージェントの本質は、目標と現状の誤差を最小化するインテリジェント・コントローラです。
9. **プラグイン可能な記憶戦略**: 役割に応じて記憶の深さとコストを柔軟に調整できます。
10. **フラクタル社会構造**: 子エージェントを組み合わせることで、無限層の協力ネットワークを構築可能です。

---

## 🛠️ クイックスタート

準備はできましたか？ **[Developer_Guide_Standalone_Execution.md](./Implementation_Examples/Developer_Guide_Standalone_Execution.md)** を参照して、最初のエージェントを実行してみましょう。