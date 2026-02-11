# 05. プラグインインターフェースの定義 (Plugin Interface Definitions)

このドキュメントでは、各プラグインが「ヘッドレス・エージェントコア」と正しく対話するために実装する必要がある抽象インターフェースを定義します。具体的なコード実装例については、 `Implementation_Examples/` フォルダ内の対応するドキュメントを参照してください。

---

## 1. UI プラグインインターフェース

UI プラグインの核心は、「双方向通信インターフェース」のクライアントを実装することです。コアからのイベントを監視し、コアにユーザーコマンドを送信する必要があります。

### コア -> UI のイベント (UI プラグインが監視する必要があるもの)

*   `onNewMessage(message: object)`: コアが、ユーザーに直接表示するための最終的なテキストを生成しました。
    *   `message`: `{ content: string, format: 'markdown' | 'text' }`
*   `onToolCallRequest(request: object)`: コアの「セキュリティレイヤー」がツール呼び出しを傍受し、ユーザーの承認が必要です。
    *   `request`: `{ toolName: string, args: object, confirmationId: string, security_warning?: object }`
*   `onReadyForInput`: コアが新しい入力を受け入れる準備ができたことを UI に通知します。
*   `onAgentStateChange(state: string)`: コアの内部状態の変更 ( `thinking`, `executing_tool` など) をブロードキャストします。

### UI -> コアのコマンド (UI プラグインが送信する必要があるもの)

*   `submitUserInput(input: string)`: ユーザーが対話入力を送信し、実行ループをトリガーしました。
*   `provideConfirmation(confirmationId: string, approved: boolean)`: ツール呼び出しの承認に対するユーザーの応答。

### 具体的な実装例

*   `Implementation_Examples/UI_Plugin_Example.md` を参照してください。

---

## 2. ツール (Tool) プラグインインターフェース

ツールプラグインは比較的単純で、通常、メタデータと `execute` メソッドを含むオブジェクトまたはクラスをエクスポートするだけです。

*   `name: string`: ツールの一意の識別子。LLM が呼び出す際に使用されます。
*   `description: string`: ツールの機能の詳細な説明。LLM がその用途を理解するために使用されます。
*   `args: object`: ツールに必要なパラメータを記述します。JSON Schema 形式を推奨します。
*   `execute(args: object): Promise<any>`: ツールの実行ロジックの核となる部分です。

### 具体的な実装例

*   **ファイルシステム読み取りツール：** `Implementation_Examples/Tool_ReadFile_Example.md` を参照してください。
*   **コードインタープリタ・ツールセット：** `Implementation_Examples/Tool_CodeInterpreter_Example.md` を参照してください。

---

## 3. プロバイダー (Provider) プラグインインターフェース

プロバイダープラグインは、具体的な LLM API との通信を担当します。

*   `constructor(config: object)`: API クライアントを初期化します。
*   `generate(history: Array<object>, tools: Array<object>): Promise<object>`:
    *   コンテンツを生成するための核となるメソッドです。
    *   `history`: コアから渡される対話履歴。
    *   `tools`: 利用可能なツールの説明リスト。
    *   **戻り値の形式：** `{ is_final: boolean, tool_calls?: Array<object>, text_content?: string }`
        *   `is_final`: true の場合、これがユーザーへの最終的な回答であることを示します。
        *   `tool_calls`: LLM がツールの呼び出しを決定した場合、ツール呼び出しリクエストのリストが含まれます。
        *   `text_content`: 最終的な回答の場合、テキストコンテンツが含まれます。

### 具体的な実装例

*   `Implementation_Examples/Provider_Gemini_Example.md` を参照してください。

---

## 4. リスナー (Listener) プラグインインターフェース (受蘊)

リスナーは外部世界の信号を監視し、それをコアが理解できるイベントに変換する役割を担います。

*   `name: string`: リスナーの一意の識別子。
*   `onStart(context: IAgentContext): Promise<void>`:
    *   エージェントの起動時に呼び出されます。
    *   このメソッド内で HTTP サーバーの起動、WebSocket 接続、または `process.stdin` のマウントを行います。
    *   `context.emitInput(data)` を使用して、受信した信号をコアに渡します。
*   `onStop(): Promise<void>`:
    *   エージェントの停止時に呼び出されます。
    *   このメソッド内で接続を閉じ、リソースを解放します。

## 5. ガイド (Guide) プラグインインターフェース (識蘊)

ガイドはエージェントの魂と意識の設定を注入する役割を担います。

*   `systemPrompt: string`: エージェントの核心となる人格、価値観、行動指針を定義します。
*   `temperature?: number`: 思考の拡散の程度を定義します。
*   `memoryPolicy?: string`: 使用する記憶戦略を指定します (例: `sliding-window`, `vector-db`)。
*   `initialMemories?: string[]`: コンテキストの背景を構築するための初期記憶を注入します (Priming)。

---

## 6. ツールプラグインセットの例：コードインタープリタ

`opencode` のような機能を実現するために、エージェントには強力なコード実行およびファイル管理ツールのセットが必要です。 **このセットのすべてのツールは、実装時に安全でホストから隔離されたサンドボックス環境（Docker コンテナなど）で実行される必要があり、これが安全性を保証するための最優先事項です。**

### 4.1 コード実行ツール

```javascript
// 例：Python コードを実行するためのツールプラグイン
class Python_Interpreter_Tool {
  name = "python:execute";
  description = "Executes a block of Python code in a sandboxed environment. The environment has a temporary file system and can install packages if specified. Returns the stdout, stderr, and the final result.";
  args = {
    code: {
      type: "string",
      description: "The Python code to execute."
    },
    dependencies: {
      type: "array",
      description: "An optional list of pip packages to install before execution.",
      items: { type: "string" }
    }
  };

  /**
   * @returns {Promise<{stdout: string, stderr: string, result: any}>}
   */
  async execute(args) {
    // 実装の詳細：
    // 1. 新しくクリーンな Docker コンテナを起動します。
    // 2. dependencies が提供されている場合、コンテナ内で pip install を実行します。
    // 3. args.code をコンテナ内の .py ファイルに書き込みます。
    // 4. その .py ファイルを実行します。
    // 5. stdout, stderr、および考えられる戻り値をキャプチャします。
    // 6. コンテナを破棄します。
    // 7. キャプチャした出力を返します。
    const result = await this.runInSandbox(args.code, args.dependencies);
    return result;
  }

  async runInSandbox(code, deps) { /* ... sandboxing logic ... */ }
}
```

### 4.2 サンドボックス・ファイルシステム・ツール

これらのツールは、コードインタープリタが存在する **サンドボックス内部** のファイルシステムを操作します。

```javascript
// 例：サンドボックス内のディレクトリを一覧表示するためのツールプラグイン
class Sandbox_FS_List_Tool {
  name = "sandbox_fs:list";
  description = "Lists the files and directories inside the sandbox at a given path.";
  args = {
    path: { type: "string", description: "The path inside the sandbox." }
  };

  async execute(args) {
    // 実装の詳細：
    // 'docker exec' または同様のコマンドを介して、対応するサンドボックスコンテナ内で 'ls -l' または 'dir' を実行します。
    const result = await this.executeCommandInSandbox(`ls -l ${args.path}`);
    return result.stdout;
  }

  async executeCommandInSandbox(cmd) { /* ... sandboxing logic ... */ }
}
```
*他のファイルシステムツール (`read`, `write`, `mkdir`, `remove`) の実装もこれと同様です。*
