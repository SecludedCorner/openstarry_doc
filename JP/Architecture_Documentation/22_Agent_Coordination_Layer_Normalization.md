# 22. エージェント調整レイヤー：帰一化と適応 (Agent Coordination Layer: Normalization & Adaptation)

## 1. 核心的な哲学

変化が激しく断片化された AI エコシステムにおいて、 **OpenStarry エージェント調整レイヤー (Agent Coordination Layer)** の核心的な価値は、「万能アダプター」としての役割を果たすことにあります。 **帰一化 (Normalization)** と **適応 (Adaptation)** のメカニズムを通じて、エージェントコア（カーネル）を純粋かつ安定した状態に保ちながら、あらゆる外部 LLM やツールと柔軟に接続できるようにします。

私たちはカーネルを外部世界に適応させることを拒否します。代わりに、外部世界をカーネルの標準プロトコルに適応させます。

---

## 2. 課題：バベルの塔効果 (The Tower of Babel Effect)

現在の LLM エコシステムは「戦国時代」にあり、各メーカーによる「ツール呼び出し (Function Calling)」の実装はそれぞれ異なります。

*   **OpenAI:** `tools` フィールドを使用し、 `tool_calls` 配列を返します。
*   **Google Gemini:** `function_declarations` を使用し、構造化された `functionCall` オブジェクトまたは入れ子になった `parts` を返します。
*   **Anthropic:** XML タグまたは特定の `tools` 構造を使用します。
*   **ローカル LLM (Llama/Mistral):** ツールをトリガーするために特定のプロンプト形式（ `[INST]` など）が必要な場合があります。

これらの差異を `Agent Core` で直接処理しようとすると、カーネルは肥大化し、メンテナンスが困難になります。新しいモデルがリリースされるたびにカーネルをアップグレードする必要があり、これはマイクロカーネルアーキテクチャの「安定性」の原則に反します。

---

## 3. 解決策：調整レイヤーによる双方向の翻訳

調整レイヤーは、システム内部の唯一の共通言語として **OpenStarry 標準プロトコル (Standard Protocol)** を導入しています。

### A. 帰一化 (Normalization) - カーネルの視点
`Agent Core` にとって、世界は標準化されています。
*   **ツールの定義：** 常に標準の JSON Schema（ `ITool` インターフェースに基づく）です。
*   **思考結果：** 常に標準の `ProviderResponse` オブジェクトであり、 `content` （テキスト）と `toolCalls` （アクション）を含みます。

コアは、背後にあるのが GPT-4 なのか Gemini 1.5 なのかを知る必要はありません。標準プロトコルのみが見えています。

### B. 適応 (Adaptation) - プラグインの視点
複雑さはエッジ（端）、すなわち **Provider プラグイン** に押し出されます。各 Provider プラグインは本質的に **アダプター (Adapter)** です。

1.  **アップストリーム適応 (Upstream Adaptation):**
    *   **入力：** コアが提供する標準の `ITool[]` リスト。
    *   **変換：** プロバイダーは、それをターゲット LLM が理解できる形式（例：Gemini の `v1beta.FunctionDeclaration`）に翻訳します。
    *   **送信：** LLM API に送信します。

2.  **ダウンストリーム適応 (Downstream Adaptation):**
    *   **入力：** LLM API から返される異質な生データ (Raw Response)。
    *   **変換：** 生データを解析し（XML、JSON、特殊フィールドの処理）、OpenStarry の標準 `ProviderResponse` にマッピングし直します。
    *   **返却：** 処理のためにコアに渡します。

---

## 4. アーキテクチャの利点

### 1. カーネルの極限までの簡素化 (Microkernel Purity)
コアの `Execution Loop` ロジックは非常にシンプルになります。
```typescript
// モデルが何であれ、コアのロジックは不変です
const response = await provider.generate(prompt, context);
if (response.toolCalls) {
    executeTools(response.toolCalls);
}
```

### 2. プラグインのホットスワップ (Hot-Swappable Providers)
いつでも `provider-openai` を `provider-gemini` に、あるいは `provider-local-llama` に交換できます。そのプラグインが `IProvider` インターフェースを遵守し、適応作業を完了している限り、エージェントの行動ロジックを修正する必要は全くありません。

### 3. 将来の互換性 (Future Proofing)
新しい AI モデルや新しい対話モードが登場した際、OS のカーネルを書き直す必要はなく、新しいアダプタープラグインを開発するだけで済みます。

---

## 5. 結論

OpenStarry の調整レイヤーは単なるコネクタではありません。それは **エントロピー減少 (Entropy Reduction)** のメカニズムです。外部世界の混乱を秩序化し、内部システムの秩序へと変換します。これが、私たちのエージェントが変化し続ける環境の中で安定した「精神」構造を維持できる理由です。
