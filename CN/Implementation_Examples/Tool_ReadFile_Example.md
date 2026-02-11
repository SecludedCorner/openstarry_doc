# 实作示例：Tool - 读取文件

本文档提供一个简单的 Tool 插件如何实现其接口的具体代码示例。

---

```javascript
// 示例：一个用于读取文件的 Tool 插件
class ReadFile_Tool {
  // name: 工具的唯一标识符，供 LLM 调用
  name = "file_system:read_file";

  // description: 工具功能的详细描述，供 LLM 理解其用途
  description = "Reads the entire content of a specified file and returns it as a string.";

  // args: 描述工具需要的参数，供 LLM 构造请求和核心进行验证
  // 这里可以使用 JSON Schema 等格式来进行更严格的定义
  args = {
    path: {
      type: "string",
      description: "The absolute or relative path to the file."
    }
  };

  /**
   * 执行工具的核心逻辑
   * @param {object} args - 经过验证的参数对象，例如 { path: "/path/to/file.txt" }
   * @returns {Promise<string>} - 工具执行的结果
   */
  async execute(args) {
    const fs = require('fs/promises');
    try {
      const content = await fs.readFile(args.path, 'utf-8');
      return content;
    } catch (error) {
      // 最好返回一个结构化的错误信息
      return `Error reading file: ${error.message}`;
    }
  }
}

// 插件入口点，导出工具实例或类
module.exports = new ReadFile_Tool();
```
