# 實作範例：Tool - 讀取文件

本文件提供一個簡單的 Tool 插件如何實現其接口的具體程式碼範例。

---

```javascript
// 示例：一個用於讀取文件的 Tool 插件
class ReadFile_Tool {
  // name: 工具的唯一標識符，供 LLM 調用
  name = "file_system:read_file";

  // description: 工具功能的詳細描述，供 LLM 理解其用途
  description = "Reads the entire content of a specified file and returns it as a string.";

  // args: 描述工具需要的參數，供 LLM 構造請求和核心進行驗證
  // 這裡可以使用 JSON Schema 等格式來進行更嚴格的定義
  args = {
    path: {
      type: "string",
      description: "The absolute or relative path to the file."
    }
  };

  /**
   * 執行工具的核心邏輯
   * @param {object} args - 經過驗證的參數對象，例如 { path: "/path/to/file.txt" }
   * @returns {Promise<string>} - 工具執行的結果
   */
  async execute(args) {
    const fs = require('fs/promises');
    try {
      const content = await fs.readFile(args.path, 'utf-8');
      return content;
    } catch (error) {
      // 最好返回一個結構化的錯誤信息
      return `Error reading file: ${error.message}`;
    }
  }
}

// 插件入口點，導出工具實例或類
module.exports = new ReadFile_Tool();
```
