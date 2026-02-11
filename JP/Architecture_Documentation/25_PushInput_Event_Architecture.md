# 25. pushInput 入力イベントアーキテクチャ (PushInput Event Architecture)

このドキュメントでは、 `IPluginContext.pushInput` の設計決定と動作メカニズムについて説明します。

---

## 1. 問題の背景

Plan02 以前、 stdio プラグインはホストから注入された `onInput` コールバックを介してユーザー入力をプッシュしていました。

```typescript
// 旧パターン（Plan01-02）
export function createStdioPlugin(opts: { onInput: (text: string) => void }): IPlugin {
  // opts.onInput はホスト（CLI）によって構築時に注入される
}
```

これには3つの問題がありました：

| 問題 | 影響 |
|------|------|
| **ホストがプラグインを認識しなければならない** | CLI は `ref.name === "standard-function-stdio"` を検知し、コールバックを注入する必要があった |
| **特殊処理** | stdio のロードパスが他のプラグインと異なり、統一性が損なわれていた |
| **完全な動態化が不可能** | 入力能力を必要とする Listener が追加されるたびに、ホストに特殊処理を追加しなければならなかった |

---

## 2. 解決策：IPluginContext.pushInput

SDK の `IPluginContext` に、標準化された入力プッシュメソッドを追加しました：

```typescript
export interface IPluginContext {
  bus: EventBus;
  workingDirectory: string;
  agentId: string;
  config: Record<string, unknown>;
  /** Push an input event into the agent's processing queue. */
  pushInput: (event: InputEvent) => void;  // ← 新設
}
```

コアはプラグインコンテキストを作成する際に、これを自動的に注入します：

```typescript
function getPluginContext(pluginConfig?: Record<string, unknown>): IPluginContext {
  return {
    bus,
    workingDirectory: process.cwd(),
    agentId: config.identity.id,
    config: pluginConfig ?? {},
    pushInput: (event) => core.pushInput(event),  // ← コアにバインド
  };
}
```

---

## 3. 入力イベントフロー

```
ユーザーがターミナルで "hello" と入力
    ↓
stdio リスナー (readline "line" イベント)
    ↓
ctx.pushInput({ source: "cli", inputType: "user_input", data: "hello" })
    ↓
core.pushInput(inputEvent)
    ↓
┌─ スラッシュコマンドか？ ──→ handleSlashCommand() ──→ ファストパス（LLM に入らない）
│
└─ 一般的な入力 ──→ queue.push(INPUT_RECEIVED) ──→ ExecutionLoop.processEvent()
```

### InputEvent 構造

```typescript
export interface InputEvent {
  source: string;                          // ソース識別子（"cli", "web", "api"）
  inputType: string;                       // タイプ（"user_input", "system_event"）
  data: string | Record<string, unknown>;  // 内容
  replyTo?: string;                        // 返信先（オプション）
}
```

---

## 4. プラグイン開発者への影響

### 以前（ホストの協力が必要）

```typescript
// プラグインはコンストラクタでコールバックを受け取る必要がある
export function createMyListener(opts: { onInput: Function }): IPlugin { ... }

// ホスト側で特殊処理が必要
const isMyPlugin = ref.name === "my-listener";
const opts = isMyPlugin ? { onInput: handleInput } : undefined;
```

### 以後（標準化）

```typescript
// プラグインは factory 内で ctx.pushInput を直接使用する
export function createMyListener(): IPlugin {
  return {
    manifest: { name: "my-listener", version: "1.0.0" },
    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      const listener: IListener = {
        id: "my-listener",
        name: "My Custom Listener",
        onEvent(event) { /* 出力イベントの処理 */ },
        async start() {
          // あらゆる入力ソースからプッシュ可能
          someInputSource.on("data", (text) => {
            ctx.pushInput({
              source: "my-source",
              inputType: "user_input",
              data: text,
            });
          });
        },
      };
      return { listeners: [listener] };
    },
  };
}

// ホスト側での特殊処理は不要
```

---

## 5. 適用シナリオ

`pushInput` は CLI に限定されません。エージェントに入力をプッシュする必要があるあらゆるプラグインで使用できます：

| プラグイン | 入力ソース | source 値 |
|------|---------|-----------|
| `standard-function-stdio` | ターミナル readline | `"cli"` |
| 将来：Web Listener | HTTP リクエスト | `"web"` |
| 将来：Discord Bot | Discord メッセージ | `"discord"` |
| 将来：File Watcher | ファイル変更イベント | `"file-watcher"` |
| 将來：Cron Scheduler | 定期実行トリガー | `"scheduler"` |

---

## 6. 設計上の決定

### なぜ EventBus で直接プッシュしないのか？

プラグインには既に `ctx.bus` がありますが、なぜ単に `bus.emit(INPUT_RECEIVED, payload)` としないのでしょうか？

それは、 `pushInput` が追加のロジックをカプセル化しているからです：
1. **スラッシュコマンドのファストパス**: `/help` や `/quit` などは EventQueue に入れず、直接処理します。
2. **イベント形式の標準化**: 自動的に `{ type: INPUT_RECEIVED, payload: inputEvent }` としてパッケージングします。
3. **将来の拡張性**: `pushInput` 内にレート制限、フィルタリング、認証などのロジックを追加できます。

EventBus を直接使用すると、これらのメカニズムをバイパスしてしまいます。
