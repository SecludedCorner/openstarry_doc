# 13. 複合体と依存関係メカニズム (Composites & Dependencies)

このドキュメントでは、 **「複合体 (Composites)」** —— すなわち、低層の能力を自ら生成するのではなく、既存の Provider (想) と Tool (行) を組み合わせ、独自の Listener (受) を加えて新しいアプリケーションシナリオを創造するハードコードされたモジュールの構築方法を定義します。

## 1. 核心的な概念：能力依存 (Capability Dependency)

疎結合を維持するため、複合体 (Composite) はプラグイン A のコードに **直接依存すべきではありません** 。代わりに、プラグイン A が提供する **「能力タグ (Capability Tag)」** に依存すべきです。

*   ❌ **誤った依存：** `import { GoogleSearch } from 'openstarry-plugin-google'`
*   ✅ **正しい依存：** `dependencies: { "tools": ["search"] }`

これにより、ユーザーが Google Search をインストールしているか Bing Search をインストールしているかにかかわらず、 `search` 能力が提供されている限り、プラグイン C は動作できます。

## 2. 依存関係の宣言 (Manifest Declaration)

`plugin.json` または `package.json` に `dependencies` フィールドを追加し、他の **プラグイン ID** を指定します：

```json
{
  "id": "workflow-engine",
  "version": "1.0.0",
  "openstarry": {
    "type": "composite",
    "dependencies": {
      "plugins": ["@openstarry-plugin/standard-function-skill"] // ハード依存
    }
  }
}
```

## 3. インフラストラクチャへの依存 (Infrastructure Dependency)

特定の高度なプラグイン（ワークフローエンジンなど）は、システムの基礎となる解析能力に依存します。

*   **シナリオ：** Markdown スキルファイルを解析する必要がある。
*   **開発実装：**
    ```typescript
    initialize(context: IPluginContext) {
      // 調整レイヤーから注入された依存関係からパーサーを取得
      const parser = context.dependencies['@openstarry-plugin/standard-function-skill'].parser;
      
      if (!parser) throw new Error("Missing Skill Parser dependency");
    }
    ```

## 4. 調整レイヤーの責務 (Validation)

`PluginLoader` がプラグイン C をロードする際、 **依存関係チェック (Dependency Check)** を実行します：

1.  コアの `ProviderRegistry` に LLM が登録されているかチェックします。
2.  コアの `ToolRegistry` に `web-search` および `fs-write` タグを持つツールが存在するかチェックします。
3.  **結果：** チェックに失敗した場合、プラグイン C は **起動を拒否** し、不足しているプラグインをインストールするようユーザーに促すエラーを表示します。

## 4. 開発実装：能力をどのように呼び出すか？

プラグインのコード内では、 `AgentContext` （注意：初期化時の `PluginContext` ではなく、ランタイムイベントに付随する `AgentContext` ）を介して能力を取得します。

```typescript
// plugins/experimental/research-assistant/index.ts

export default class ResearchPlugin implements IPlugin {
  async initialize(context: IPluginContext) {
    // リスナーを登録し、タスクを待機
    context.registerListener({
      id: 'research-trigger',
      onEvent: async (event, agentOps: IAgentOperations) => {
        
        // 1. 脳 (想) を取得
        const llm = agentOps.getLLM();
        
        // 2. ツール (行) を取得
        // システムは 'web-search' タグが付いたツールを自動的に検索します
        const searchTool = agentOps.findToolByTag('web-search');
        
        if (!searchTool) {
          throw new Error("Missing required capability: web-search");
        }

        // 3. ロジックの実行
        const query = await llm.generate(`Extract keywords from: ${event.content}`);
        const results = await searchTool.call({ query });
        
        // 4. フィードバック
        agentOps.reply(`Search results: ${results}`);
      }
    });
  }
}
```

## 5. まとめ：エコシステムの階層

このメカニズムにより、エコシステムの階層が自然に形成されます：

*   **レベル 1 (Foundation)：** アトミックな能力を提供するプラグイン ( `fs`, `http`, `gemini` )。
*   **レベル 2 (Composites)：** レベル 1 の能力を組み合わせた複合プラグイン ( `researcher`, `coder`, `writer` )。
*   **レベル 3 (Agents)：** レベル 2 の複合体を組み合わせた完全なデジタル種。

これにより、開発者は積み木を組み立てるように複雑なアプリケーションを構築できます。
