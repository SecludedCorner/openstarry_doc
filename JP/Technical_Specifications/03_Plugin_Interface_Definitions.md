# 03. プラグインインターフェース技術仕様 (Plugin Interface Specifications)

本ドキュメントは、OpenStarry プラグインシステムの技術的な契約を正確に定義します。プラグイン開発者は、マイクロカーネルとのシームレスな統合を確保するために、以下のインターフェースを実装する必要があります。

## 1. ベースプラグインライフサイクル (Base Plugin)

すべてのプラグインは、ベースライフサイクルメソッドを継承または実装する必要があります。

```typescript
interface IOpenStarryPlugin {
  id: string;
  version: string;
  author?: string;

  // 初期化時に Core API プロキシを注入
  initialize(host: ICoreHost): Promise<void>;

  // コアのシャットダウンまたはプラグインのアンインストール時に呼び出し
  shutdown(): Promise<void>;
}
```

---

## 2. 五蘊プラグインインターフェース (The Five Aggregates Interfaces)

### 2.1 UI (色蘊 - Rupa)
ユーザーインターフェースコンポーネントを定義します。
```typescript
interface IUIComponent {
  id: string;
  type: 'terminal' | 'web_view' | 'widget';
  render(data: any): void;
  onInput(callback: (val: string) => void): void;
}
```

### 2.2 Listener (受蘊 - Vedana)
外部イベントのセンサーを定義します。
```typescript
interface IEventListener {
  id: string;
  start(): Promise<void>;
  stop(): Promise<void>;
  // 外部シグナルを標準 Event にカプセル化してコアキューにプッシュ
  emit(event: IOpenStarryEvent): void;
}
```

### 2.3 Provider (想蘊 - Samjna)
認知バックエンド（LLM）を定義します。
```typescript
interface ILLMProvider {
  id: string;
  generateResponse(messages: IMessage[], tools?: ITool[]): Promise<ILLMResult>;
  streamResponse(messages: IMessage[]): AsyncIterable<string>;
}
```

### 2.4 Tool (行蘊 - Samskara)
具体的な実行能力を定義します。
```typescript
interface IAgentTool {
  name: string;
  description: string;
  parameters: JSONSchema7; // 標準 JSON Schema を使用してパラメータを定義
  execute(args: any, context: IExecutionContext): Promise<IToolResult>;
}
```

### 2.5 Guide (識蘊 - Vijnana)
これは Agent に意識とロジックを付与する重要なインターフェースです。「ペインインタープリテーション（痛覚解釈）」の処理が許可される唯一の場所です。

```typescript
interface IAgentGuide {
  id: string;

  // ペルソナとコアロジックを注入
  getSystemInstructions(): string;

  // ペインインタープリテーション Hooks:
  // コアが標準化されたエラーを返した際、Guide はそれを LLM が理解できる意識シグナルに変換する責任を持つ
  interpretPain?(error: IStandardError): string;

  // リフレクションモードに入るべきかどうかを判断
  shouldReflect(lastHistory: IMessage[]): boolean;
}
```

---

## 3. セキュリティと分離仕様

1.  **サンドボックス制限**: プラグインコードは `process.env` への直接アクセスや `fs.rm` の実行を許可されていません。すべての機密操作は `ICoreHost` が提供するプロキシ API を介して実行する必要があります。
2.  **リソース制限**: コアは各プラグインのメモリおよび CPU 消費を監視します。閾値を超えたプラグインは強制的に `shutdown` され、`ERROR` ステートに遷移します。
3.  **ステートレス原則**: プラグインは可能な限りステートレス (Stateless) を維持する必要があります。すべての永続化要件は、コアの `StateManager` サービスを介して実現する必要があります。
