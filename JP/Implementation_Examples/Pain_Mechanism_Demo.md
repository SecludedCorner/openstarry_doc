# 実装例：擬人化された痛覚メカニズム (Pain Mechanism Demo)

この例では、 OpenStarry の「識蘊 (Guide)」プラグインとコアの「安全遮断器 (Safety Circuit Breaker)」を連携させて、擬人化された **痛覚フィードバックメカニズム** を実現する方法を示します。

---

## 1. 概念の説明

OpenStarry において「痛覚」はプログラムのクラッシュではなく、 **高レベルの負の入力** です。
*   **痛覚の感知 (Core)：** `SafetyMonitor` がツールの実行失敗をキャッチします。
*   **痛覚の伝導 (Plugin)：** `PainAware_Guide` が失敗を強い負の暗示を伴うプロンプトに変換します。
*   **痛覚の反応 (LLM)：** エージェントが「痛み」（タスクの阻止）を感じ、自己反省と戦略の調整をトリガーします。

---

## 2. プラグインの実装例： `PainAware_Guide`

このプラグインはエージェントの「魂」として機能し、失敗をどのように解釈するかを定義します。

```typescript
// plugins/standard/guide-pain/index.ts
import { IPlugin, IPluginContext } from '@openstarry/sdk';

export default class PainMechanismPlugin implements IPlugin {
  id = 'guide-pain-mechanism';
  version = '1.0.0';

  async initialize(context: IPluginContext) {
    context.logger.info('痛覚神経系がオンラインになりました...');

    // エラーフィードバックを専門に処理する「識蘊」コンポーネントを登録
    context.registerGuide({
      id: 'pain-protocol',
      
      // IAgentGuide で定義された標準痛覚インターフェースを実装
      // コアが標準化されたエラー (IStandardError) を返した際、 Guide がそれを意識信号に変換する
      interpretPain: (error: any) => {
        const severity = this.calculateSeverity(error);
        return `
【システム痛覚アラート】
実行異常を検知しました！
ソース：${error.source}
メッセージ：${error.message}
痛みのレベル：${severity}
状態：あなたの行動は阻害されました。これはあなたに「不快感」を与えます。

直ちに繰り返しの試行を停止し、痛みの原因を分析して、次の Tick で戦略を調整してください。
`.trim();
      },

      getSystemInstructions: () => "You are a resilient agent...",
      shouldReflect: () => true
    });
  }

  private calculateSeverity(error: any) {
    if (error.code === 'EPERM') return '🔥🔥🔥 激痛 (Critical)';
    if (error.code === 'ENOENT') return '⚡ 刺痛 (Medium)';
    return '💧 微痛 (Low)';
  }

  async shutdown() {}
}
```

---

## 3. エージェント設定 ( `agent.json` )

```json
{
  "identity": { "id": "resilient-agent", "name": "強靭なエージェント" },
  "plugins": [
    { "name": "@openstarry-plugin/standard-function-fs" },
    { "name": "@openstarry-plugin/guide-pain-mechanism" }
  ],
  "policy": {
    "safety": {
      "max_consecutive_errors": 3,
      "pain_threshold": "medium"
    }
  }
}
```

---

## 4. 実行シナリオのシミュレーション

### ステップ 1：誤った試み
**エージェント：** `/root/secret.txt` を読み込みたい。
**システム (FS ツール)：** `Error: EPERM (Permission Denied)`

### ステップ 2：痛覚の注入 (Pain Injection)
コアがエラーをキャッチし、 `guide-pain-mechanism` を呼び出します。
**コンテキストへの注入：**
> 【システム痛覚アラート】
> ソース：fs:read_file
> メッセージ：EPERM (Permission Denied)
> 痛みのレベル：🔥🔥🔥 激痛 (Critical)
> 状態：あなたの行動は阻害されました。これはあなたに「不快感」を与えます。

### ステップ 3：自己修正 (Self-Correction)
**エージェント (思考中)：** 
「読み込みに失敗し、激痛アラートが発生した。これは直接的なパスは通用しないことを意味しており、安全遮断が作動する可能性がある。権限の壁に突き当たるのはやめなければならない。まずは現在の自分の権限 `whoami` を確認するか、 root 権限を必要としない他のバックアップファイルを探すべきだ。」

**エージェント (行動)：** 再度ファイルを読み込もうとするのではなく、 `shell:exec('whoami')` を呼び出します。

---

## 5. 利点

1.  **非クラッシュ性 (Non-Crashing)：** エラーが対話の一部となり、エージェントは例外が発生しただけで完全に停止することはありません。
2.  **進化性 (Evolvable)：** エージェントはこれらの「痛点」を処理することでより賢くなり、既知の罠を能動的に回避できるようになります。
3.  **安全の閉ループ (Safety Loop)：** `SafetyMonitor` と組み合わせることで、エージェントが痛覚を無視して間違いを犯し続けた場合、システムは物理的な隔離（遮断）を実施します。
