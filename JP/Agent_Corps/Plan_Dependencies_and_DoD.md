# OpenStarry — プランの依存関係と完了定義 (Plan Dependencies & DoD)

設立日：2026-02-09

---

## 1. プラン依存関係図 (Plan Dependencies)

```
Plan01 (MVP Alpha Foundation)           ✅ v0.1-alpha  2026-02-04
  ↓
Plan02 (Event-Driven & Safety)          ✅ v0.1.1      2026-02-05
  ↓
Plan03 (Quality & Skill Parser)         ✅ v0.2-alpha  2026-02-05
  ↓
Plan04 (IUI Interface & Guide Plugin)   ✅ v0.2-alpha  2026-02-06
  ↓
Plan05 (Multi-Channel UI & Listener)    ✅ v0.2-beta   2026-02-07
  ↓
  ├── Plan05.1 (Session Isolation)      ⬜ → v0.2.1-beta
  ├── Plan05.2 (HTTP SSE)              ⬜ → v0.2.1-beta   ← 05.1 と並行可能
  ├── Plan05.5 (その他)                   ⬜ → v0.2.1-beta   ← 05.1 と並行可能
  │     ↓
  │   Plan06 (MCP Protocol)             ⬜ → v0.3       ← Plan05.1 によりブロック
  │     ↓
  │   Plan07 (Runtime Sandbox)          ⬜ → v0.4       ← Plan06 によりブロック
  │     ↓
  │   Plan08-09 (TUI Dashboard)         ⬜ → v0.5
  │
  └── (Extended Mode: 新しいプラグイン、 Web UI 、安全監査)
```

### クリティカルパス (Critical Path)

```
Plan05.1 → Plan06 → Plan07 → Plan08-09
```

Plan05.1 (Session Isolation) は、クリティカルパス上の最初の未完了項目です。 Plan06 (MCP) はこれによってブロックされています。

### 並行可能な作業

| 並行可能 | 説明 |
|--------|------|
| Plan05.1 + Plan05.2 + Plan05.5 | これら3つは、1つの反復サイクル（実装サイクル 1）に統合可能です。 |
| Plan06 の事前調査 + Plan05.x の実装 | researcher は、開発エージェントが 05.x を実装している間に Plan06 を調査できます。 |

---

## 2. 完了定義 (Definition of Done)

### 2.1 共通 DoD（すべてのプランに適用）

各プランは、 ✅ をマークする前に以下の **すべての条件** を満たす必要があります：

| # | 条件 | 検証方法 |
|---|------|---------|
| 1 | `pnpm build` が通過していること (monorepo + plugins) | qa が agent_test で検証 |
| 2 | `pnpm test` がすべて通過し、テスト数が前回のベースライン以上であること | qa レポート内のリグレッションチェック |
| 3 | `pnpm test:purity` が通過していること | qa レポート内の純度チェック |
| 4 | Architecture_Spec 内のすべての項目が実装されていること | architect のコードレビュー PASS |
| 5 | 五蘊への適合（新しいコンポーネントが正しく分類されていること） | architect のコードレビューで確認 |
| 6 | pushInput パターンの遵守 | architect のコードレビューで確認 |
| 7 | 既知のセキュリティ脆弱性がないこと | architect のコードレビューで確認 |
| 8 | 開発ログ (dev log) が記述されていること | Coordinator がファイルの存在を確認 |
| 9 | QA レポートおよびコードレビューの両方が PASS であること | Phase 4 で判定 |
| 10 | doc-keeper がプランを ✅ に更新し、反復ログを追記していること | Phase 4 の収束 |
| 11 | スナップショット (Snapshot) が作成されていること | `scripts/snapshot.sh` の完了 |
| 12 | 教訓 (Lessons Learned) が記録されていること | Phase 4 の振り返り |

### 2.2 各プラン固有の受入基準

#### Plan05.1: Session Isolation
- [ ] WebSocket 接続が session token / auth をサポートしていること。
- [ ] 複数のクライアント接続が互いに隔離されていること（セッションデータが交差しない）。
- [ ] 単一のエージェントインスタンスが複数のセッションに対応できること。
- [ ] セッションライフサイクル管理（作成、維持、破棄）。
- [ ] セッション隔離シナリオをカバーする新しいテスト。

#### Plan05.2: HTTP SSE
- [ ] HTTP Server-Sent Events トランスポートプラグインが利用可能であること。
- [ ] SSE 接続でリアルタイムのストリーミング応答を受信できること。
- [ ] 既存の HTTP webhook リスナーと共存し、衝突しないこと。
- [ ] SSE シナリオをカバーする新しいテスト。

#### Plan06: MCP Protocol (Model Context Protocol)
- [ ] MCP 仕様の tool, resource, prompt の3大機能を実装すること。
- [ ] MCP サーバーモード： OpenStarry が MCP サーバーとして外部クライアントから接続されること。
- [ ] MCP クライアントモード： OpenStarry が外部 MCP サーバーに接続してツールを取得すること。
- [ ] MCP 公式仕様に準拠していること（ researcher の調査により確認）。
- [ ] MCP シナリオをカバーする新しいテスト。

#### Plan07: Runtime Sandbox
- [ ] プラグイン実行環境の隔離 (vm または WASM)。
- [ ] リソース制限（ CPU 、メモリ、ファイルアクセス）。
- [ ] プラグイン署名検証メカニズム。
- [ ] サンドボックス脱出 (sandbox escape) シナリオをカバーする新しいテスト。

#### Plan08-09: TUI Dashboard
- [ ] ターミナル対話型ダッシュボード (Ink または Blessed)。
- [ ] エージェントのステータス、対話、ツール呼び出しをリアルタイムで表示すること。
- [ ] 複数セッションの切り替えをサポートすること。
- [ ] TUI シナリオをカバーする新しいテスト。

---

## 3. 反復スケジュールの提案

| 反復 | 対象プラン | ターゲットバージョン | 前提条件 |
|------|-----------|---------|---------|
| Cycle 1 | Plan05.1 + 05.2 + 05.5 | v0.2.1-beta | Plan05 ✅（満足済み） |
| Cycle 2 | Plan06 (MCP) | v0.3 | Plan05.1 ✅ |
| Cycle 3 | Plan07 (Sandbox) | v0.4 | Plan06 ✅ |
| Cycle 4 | Plan08-09 (TUI) | v0.5 | Plan07 ✅（または前倒し可能） |
| Extended | 新機能、安全監査 | v1.0 | すべてのプラン ✅ |

---

## 4. 使用方法

- **Coordinator** は、新しい反復を開始する前に本書を参照し、以下を確認します：
  1. 前提プランが完了しているか ✅
  2. そのプラン固有の受入基準
  3. 並行可能な作業があるか
- **architect** は、仕様書 (Spec) を設計する際、固有の受入基準を参照して網羅性を確保します。
- **qa** は、検証時に共通 DoD と固有基準を項目ごとにチェックします。
- **doc-keeper** は、プランを ✅ にする際、 DoD がすべて満たされていることを確認します。
