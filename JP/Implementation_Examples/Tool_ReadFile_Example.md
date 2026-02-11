# 実装例：ツール (Tool) - ファイルの読み取り

このドキュメントでは、シンプルな Tool プラグインがそのインターフェースをどのように実装するかについて、具体的なコード例を提供します。

---

```javascript
// 例：ファイルを読み取るための Tool プラグイン
class ReadFile_Tool {
  // name: ツールの唯一の識別子。 LLM が呼び出す際に使用される
  name = "file_system:read_file";

  // description: ツールの機能の詳細な説明。 LLM がその用途を理解するために使用される
  description = "Reads the entire content of a specified file and returns it as a string.";

  // args: ツールが必要とする引数の説明。 LLM がリクエストを構築し、コアが検証を行うために使用される
  // ここでは、より厳密な定義のために JSON Schema などの形式を使用できる
  args = {
    path: {
      type: "string",
      description: "The absolute or relative path to the file."
    }
  };

  /**
   * ツールの実行ロジックの核となる部分
   * @param {object} args - 検証済みの引数オブジェクト。例： { path: "/path/to/file.txt" }
   * @returns {Promise<string>} - ツール実行の結果
   */
  async execute(args) {
    const fs = require('fs/promises');
    try {
      const content = await fs.readFile(args.path, 'utf-8');
      return content;
    } catch (error) {
      // 構造化されたエラーメッセージを返すのが望ましい
      return `Error reading file: ${error.message}`;
    }
  }
}

// プラグインのエントリポイント。ツールインスタンスまたはクラスをエクスポートする
module.exports = new ReadFile_Tool();
```
