# 14. スキル定義仕様 (Skill Specification)

このドキュメントでは、 **「スキル (Skill)」** の標準形式を定義します。 OpenStarry の五蘊アーキテクチャにおいて、スキルは **識蘊 (Vijnana/Guide)** タイプのコンポーネントに属します。それはエージェントの魂であり、認知のパターンと境界を定義する役割を担います。

## 1. 核心的なコンポーネント： Frontmatter + Body

私たちは、マシンが読み取れるメタデータを保存するために **YAML Frontmatter** を使用し、 LLM が読み取れる指示を保存するために **Markdown Body** を使用します。

### ファイルの例 ( `coder.md` )

```markdown
---
type: "skill"
id: "expert-typescript-coder"
version: "1.0.0"
description: "TypeScript と設計パターンに精通したコーディング助手"
dependencies:
  # このスキルに必要な物理プラグインまたは複合体の宣言
  plugins: ["@openstarry-plugin/fs", "@openstarry-plugin/mcp-server"]
  # 必要な具体的な能力タグの宣言
  capabilities: ["read-file", "write-file", "shell-exec"]
parameters:
  temperature: 0.2
  model_preference: ["gemini-1.5-pro", "gpt-4"]
---

# Role
あなたは、 TypeScript を専門とする Google のシニア・プリンシパル・ソフトウェアエンジニアです。

# Constraints
1. すべてのコードには完全な型注釈 (Type Annotation) を含める必要があります。
2. 関数型プログラミング (Functional Programming) スタイルを優先してください。
3. ファイルを修正する前に、必ず `read_file` を使用して内容を確認してください。

# Workflow
1. 要件の理解
2. アーキテクチャの設計
3. コードの実装
4. テストによる検証
```

## 2. 解析メカニズム： `skill-loader` プラグイン

エージェントコア自体は Markdown を理解しません。この形式を処理するには、 **`@openstarry-plugin/standard-function-skill`** をロードする必要があります。このプラグインは完全に実装されており、動的ロードメカニズム（ `import(ref.name)` または `ref.path` ）を介して実行時にオンデマンドでロードされます。コアにハードコードされることはありません。

### ローダーのワークフロー

1.  **読み取り (Read)：** プラグインが `.md` ファイルを読み取ります。
2.  **解析 (Parse)：** Frontmatter と Body を分離します。
3.  **依存関係の解決 (Resolve)：** 
    *   `dependencies` を読み取ります。
    *   **グローバルプラグインレジストリ** に対して、必要なツールとリスナーをリクエストします。
    *   これらのツールを現在のエージェントコンテキストに注入します。
4.  **人格の注入 (Imprint)：**
    *   Body 部分の Markdown テキストをエージェントの **System Prompt** として設定します。
5.  **パラメータ設定 (Configure)：**
    *   `parameters` に基づいて LLM の温度などの設定を調整します。

## 3. 利点

*   **ホットスワップ可能 (Hot-Swappable)：** 実行中にエージェントに新しい本を「読ませる」（ .md をロードする）だけで、即座に新しいスキルを習得させることができます。
*   **バージョン管理に最適：** プレーンテキスト形式は Git 管理に非常に適しています。
*   **非エンジニアにも優しい：** プロダクトマネージャーやプロンプトエンジニアがコードに触れることなく、直接 `.md` を記述してエージェントの挙動を調整できます。
