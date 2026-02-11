# 26. プラグインサービスとライフサイクル管理 (Plugin Service & Lifecycle Management)

このドキュメントでは、プラグイン間の **サービス共有メカニズム** と **動的なライフサイクル管理** を定義し、複雑なマルチプラグイン協調シナリオのためのアーキテクチャ基盤を提供します。

---

## 1. 問題の背景

### 1.1 既存メカニズムの限界

現在のアーキテクチャでは、五蘊プラグインはレジストリ (Registry) を介して間接的に協調します。

```
プラグイン A (Tool) ──登録──► ToolRegistry ◄──照会── コア
プラグイン B (Guide) ──登録──► GuideRegistry ◄──照会── コア
```

これは標準的な五蘊プラグインには十分ですが、以下のシナリオには対応できません。

| シナリオ | 問題 |
|------|------|
| **サービス共有** | Docker プラグインが WEB UI で使用するための API を提供する。 |
| **動的管理** | 実行時に WEB UI から他のプラグインを有効化/無効化する。 |
| **依存関係の調整** | プラグイン A が動作するために、プラグイン B の初期化が先に完了している必要がある。 |

### 1.2 目標とするシナリオ

```
エージェント (VM 環境)
│
├─ 複合 UI プラグイン
│   ├─ CLI UI ──────── ターミナルインターフェース
│   └─ WEB UI ──────── ブラウザ管理画面
│       ├─ プラグイン管理パネル ── プラグインの有効化/無効化
│       ├─ サービス状態監視 ── Docker/Nginx の状態照会
│       └─ 視覚的設定 ──── プラグイン設定の編集
│
├─ Docker プラグイン ────── コンテナ管理サービス
├─ Nginx プラグイン ─────── リバースプロキシサービス
└─ [プロジェクトプラグイン...] ── 動的な発見とロード
```

---

## 2. サービスの登録と発見メカニズム

### 2.1 IPluginContext の拡張

```typescript
export interface IPluginContext {
  // 既存のフィールド
  bus: EventBus;
  workingDirectory: string;
  agentId: string;
  config: Record<string, unknown>;
  pushInput: (event: InputEvent) => void;

  // === 新設：サービスメカニズム ===

  /**
   * 他のプラグインが使用できるようにサービスを登録する
   * @param id サービスの一意の識別子
   * @param service サービスインスタンス
   */
  registerService<T>(id: string, service: T): void;

  /**
   * 登録済みのサービスを取得する
   * @param id サービス識別子
   * @returns サービスインスタンス。存在しない場合は undefined
   */
  getService<T>(id: string): T | undefined;

  /**
   * プラグインマネージャー（特権プラグインのみ）
   * 管理権限が付与されていないプラグインの場合、このフィールドは undefined になる
   */
  pluginManager?: IPluginManager;
}
```

### 2.2 サービスプロバイダーの例

```typescript
// docker-plugin/src/index.ts

interface DockerService {
  listContainers(): Promise<Container[]>;
  startContainer(id: string): Promise<void>;
  stopContainer(id: string): Promise<void>;
}

export function createDockerPlugin(): IPlugin {
  return {
    manifest: {
      name: "docker-plugin",
      version: "0.1.0",
      provides: ["docker-service"],  // 提供するサービスを宣言
    },

    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      const dockerService: DockerService = {
        async listContainers() { /* ... */ },
        async startContainer(id) { /* ... */ },
        async stopContainer(id) { /* ... */ },
      };

      // サービスを登録
      ctx.registerService("docker", dockerService);

      return {
        tools: [createDockerTool(dockerService)],
      };
    },
  };
}
```

### 2.3 サービスコンシューマー（利用者）の例

```typescript
// web-ui-plugin/src/index.ts

export function createWebUIPlugin(): IPlugin {
  return {
    manifest: {
      name: "web-ui-plugin",
      version: "0.1.0",
      dependencies: ["docker-plugin"],  // 依存関係を宣言
    },

    async factory(ctx: IPluginContext): Promise<PluginHooks> {
      // Docker サービスを取得
      const docker = ctx.getService<DockerService>("docker");

      if (!docker) {
        throw new Error("Required service 'docker' not found");
      }

      return {
        ui: [createWebUI({
          docker,
          pluginManager: ctx.pluginManager
        })],
      };
    },
  };
}
```

---

## 3. プラグインライフサイクル管理

### 3.1 IPluginManager インターフェース

```typescript
export type PluginStatus =
  | "discovered"   // 発見されたが未ロード
  | "loading"      // ロード中
  | "running"      // 実行中
  | "stopped"      // 停止中
  | "error"        // エラー状態
  | "quarantined"; // 隔離中（ロード失敗）

export interface PluginDescriptor {
  name: string;
  version: string;
  path: string;
  status: PluginStatus;
  provides?: string[];      // 提供するサービス
  dependencies?: string[];  // 依存するプラグイン
  error?: string;           // エラーメッセージ（ある場合）
}

export interface IPluginManager {
  /**
   * 利用可能なプラグインを発見する
   * @param searchPaths 検索パスのリスト
   */
  discover(searchPaths: string[]): Promise<PluginDescriptor[]>;

  /**
   * プラグインを動的にロードする
   * @param name プラグイン名
   * @throws 依存関係が満たされていない場合やロードに失敗した場合
   */
  load(name: string): Promise<void>;

  /**
   * プラグインをアンロードする
   * @param name プラグイン名
   * @throws 他のプラグインがこのプラグインに依存している場合
   */
  unload(name: string): Promise<void>;

  /**
   * プラグインをリロードする（アンロード後に再ロード）
   */
  reload(name: string): Promise<void>;

  /**
   * プラグインの状態を取得する
   */
  getStatus(name: string): PluginStatus;

  /**
   * すべてのプラグインをリストアップする
   */
  list(): PluginDescriptor[];

  /**
   * プラグインの状態変化を監視する
   */
  onStatusChange(
    callback: (name: string, status: PluginStatus) => void
  ): () => void;
}
```

### 3.2 ロード順序とトポロジカルソート

プラグインが依存関係を宣言している場合、 PluginManager はトポロジカルソートを行う必要があります。

```
依存グラフ：
  web-ui-plugin
       │
       ├──► docker-plugin
       │
       └──► nginx-plugin

ロード順序：
  1. docker-plugin
  2. nginx-plugin
  3. web-ui-plugin
```

**実装ロジック：**

```typescript
function topologicalSort(plugins: PluginDescriptor[]): string[] {
  const graph = new Map<string, string[]>();
  const inDegree = new Map<string, number>();

  // 依存グラフの構築
  for (const p of plugins) {
    graph.set(p.name, p.dependencies ?? []);
    inDegree.set(p.name, (p.dependencies ?? []).length);
  }

  // Kahn's algorithm
  const queue = [...inDegree.entries()]
    .filter(([_, deg]) => deg === 0)
    .map(([name]) => name);

  const result: string[] = [];

  while (queue.length > 0) {
    const current = queue.shift()!;
    result.push(current);

    for (const [name, deps] of graph) {
      if (deps.includes(current)) {
        inDegree.set(name, inDegree.get(name)! - 1);
        if (inDegree.get(name) === 0) {
          queue.push(name);
        }
      }
    }
  }

  // 循環依存の検知
  if (result.length !== plugins.length) {
    throw new Error("Circular dependency detected");
  }

  return result;
}
```

---

## 4. マルチチャネル UI の協調

### 4.1 シナリオの説明

同じエージェントが同時に複数の UI チャネルを持つ場合があります。

```
ユーザー A ──► CLI UI ──┐
                        ├──► エージェントコア ──► ツール
ユーザー B ──► WEB UI ──┘
```

### 4.2 イベントブロードキャストメカニズム

すべての UI は `UIRegistry` を介してイベントを受信します。

```typescript
// transport/bridge.ts
function broadcast(event: AgentEvent): void {
  for (const ui of uiRegistry.list()) {
    ui.onEvent(event);  // CLI と WEB の両方が受信する
  }
}
```

### 4.3 入力ソースの識別

`InputEvent.source` を使用して入力ソースを区別します。

```typescript
interface InputEvent {
  source: string;      // "cli" | "web" | "api" | ...
  inputType: string;
  data: unknown;
  replyTo?: string;    // 特定の返信先用
}
```

### 4.4 ターゲット指定の返信（将来の拡張）

返信を特定の UI にのみ送信する必要がある場合：

```typescript
interface AgentEvent {
  type: string;
  timestamp: number;
  payload?: unknown;
  targetUI?: string;   // 指定されている場合、その UI のみが受信する
}
```

---

## 5. 権限管理

### 5.1 特権プラグイン

すべてのプラグインが他のプラグインを管理できるべきではありません。 **特権プラグイン** のみが `pluginManager` にアクセスできます。

```json
// agent.json
{
  "plugins": [
    {
      "name": "@openstarry-plugin/web-ui",
      "privileged": true  // 管理権限を付与
    },
    { "name": "@openstarry-plugin/docker" }
  ]
}
```

### 5.2 コアの実装

```typescript
function getPluginContext(
  pluginRef: PluginRef,
  isPrivileged: boolean
): IPluginContext {
  return {
    bus,
    workingDirectory: process.cwd(),
    agentId: config.identity.id,
    config: pluginRef.config ?? {},
    pushInput: (event) => core.pushInput(event),
    registerService: (id, svc) => serviceRegistry.register(id, svc),
    getService: (id) => serviceRegistry.get(id),
    pluginManager: isPrivileged ? pluginManager : undefined,
  };
}
```

---

## 6. 実装ロードマップ

| フェーズ | 内容 | 優先度 |
|-------|------|--------|
| **フェーズ 1** | ServiceRegistry (registerService / getService) | 高 |
| **フェーズ 2** | PluginManifest 依存関係宣言 + トポロジカルソート | 高 |
| **フェーズ 3** | IPluginManager 基本実装 (load/unload) | 中 |
| **フェーズ 4** | プラグイン発見メカニズム（プロジェクトフォルダのスキャン） | 中 |
| **フェーズ 5** | 権限管理と特権プラグイン | 中 |
| **フェーズ 6** | マルチチャネルターゲット指定返信 | 低 |

---

## 7. 既存ドキュメントとの関係

| ドキュメント | 補足説明 |
|------|----------|
| `19_Agent_Coordination_Layer.md` | 「プラグイン交換機」の概念を拡張し、サービス登録を追加。 |
| `13_Composite_Plugins_and_Dependencies.md` | 具体的な API 実装を提供。 |
| `20_Dependency_Injection_and_Control_Loop.md` | 動的注入メカニズムを追加。 |
| `21_Plugin_Interface_Deep_Dive.md` | IPluginContext を拡張。 |

---

## 8. 設計原則

1. **コアを純粋に保つ** — サービス登録メカニズムはコアレイヤーにあり、具体的なプラグイン実装には依存しない。
2. **段階的な採用** — 既存のプラグインを修正する必要はなく、新しいメカニズムはオプションの拡張である。
3. **明示的な依存関係** — 暗黙的な結合を避けるため、マニフェストを通じて宣言する。
4. **セキュリティ第一** — プラグイン管理には明示的な認可が必要である。
