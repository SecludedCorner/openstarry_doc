# 07. 管理層とオーケストレーター技術仕様 (Management Zone & Orchestrator Spec)

本ドキュメントは、OpenStarry システムにおける「管理層 (Management Zone)」の技術実装標準を定義します。管理層は Agent のライフサイクル管理、リソース分離、およびクロス Agent スケジューリングを担当します。

## 1. デーモンアーキテクチャ (Daemon Architecture)

OpenStarry Daemon (`openstarryd`) は、すべての Agent のホストとして機能する常駐システムサービスです。

### 1.1 コア責務
*   **Process Management:** Agent インスタンスの起動、停止、再起動。
*   **Resource Monitoring:** CPU、メモリ、およびトークン消費量の監視。
*   **Registry Service:** ローカルプラグインおよび Agent のレジストリの維持。

### 1.2 内部 API (Local Control Plane)
Daemon は CLI ツールから呼び出すための軽量なコントロールプレーン (Control Plane) を公開する必要があります。

```typescript
interface IDaemonControlPlane {
  // Agent インスタンスを起動
  spawnAgent(manifestPath: string, options?: ISpawnOptions): Promise<string>; // returns PID

  // 強制終了
  killAgent(pid: string, signal: 'SIGTERM' | 'SIGKILL'): Promise<void>;

  // ステータス照会
  listAgents(): Promise<IAgentStatus[]>;
}
```

---

## 2. コンテナレイヤー仕様 (Container Layer Spec)

セキュリティと分離を確保するため、Agent はホストプロセスで直接実行するのではなく、制御されたコンテナ環境内で実行する必要があります。

### 2.1 分離レベル (Isolation Levels)
管理層は複数の分離ドライバー (Driver) をサポートする必要があります：

1.  **Process Driver（デフォルト）:** Node.js の `child_process` または Worker Threads を使用。開発環境に適しています。
2.  **Docker Driver（プロダクション）:** 各 Agent に対して Docker コンテナを起動。サーバーデプロイに適しています。
3.  **WASM Driver（実験的）:** WebAssembly サンドボックス内で Core を実行。エッジコンピューティングに適しています。

### 2.2 依存性注入 (DI) プロトコル
コンテナレイヤーは、Agent の起動前に必要な環境変数とプラグインパスを準備する責任を負います。

*   **環境変数の注入:** `OPENSTARRY_AGENT_ID`, `OPENSTARRY_HOME`
*   **プラグインマウント:** `agent.json` で定義されたプラグインパスをコンテナ内の読み取り可能なパスにマッピング。

---

## 3. オーケストレーションとスケジューリング (Orchestration)

スケジューラーはクロス Agent の因果チェーン (Causality Chain) の処理を担当します。

### 3.1 イベントトリガー (Event Triggers)
スケジューラーは「境界イベント」をリッスンし、ルールに基づいて Agent を起動します。

```yaml
# orchestration.yaml
rules:
  - on: "git:commit_pushed"
    if: "branch == 'main'"
    action: "spawn(code-reviewer-agent)"

  - on: "alert:server_down"
    action: "spawn(devops-agent)"
```

### 3.2 ステートプロジェクション (State Projection)
スケジューラーは各アクティブ Agent の `IInternalBus` からステートスナップショットを定期的にプルし、グローバルダッシュボード (Global Dashboard) に集約します。

---

## 4. ハードウェア抽象化レイヤー (HAL - Hardware Abstraction Layer)

Agent を IoT デバイスで実行できるようにするため、管理層は標準化された HAL を提供する必要があります。

```typescript
interface IHAL {
  getCameraStream(deviceId: string): ReadableStream;
  getSensorData(sensorType: string): Promise<number>;
  controlActuator(deviceId: string, command: any): Promise<void>;
}
```

すべての物理操作は HAL インターフェースを経由する必要があります。Agent が `/dev/*` デバイスファイルに直接アクセスすることは厳禁です。
