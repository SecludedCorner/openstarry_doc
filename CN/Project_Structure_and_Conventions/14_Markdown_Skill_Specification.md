# 14. 技能定义规范 (Skill Specification)

本文档定义了 **「技能 (Skill)」** 的标准格式。在 OpenStarry 的五蕴架构中，Skill 属于 **Guide (识蕴)** 类型的组件。它是 Agent 的灵魂，负责定义认知的模式与边界。

## 1. 核心组件：Frontmatter + Body

我们使用 **YAML Frontmatter** 来存放机器可读的元数据，使用 **Markdown Body** 来存放 LLM 可读的指令。

### 档案范例 (`coder.md`)

```markdown
---
type: "skill"
id: "expert-typescript-coder"
version: "1.0.0"
description: "一个精通 TypeScript 与设计模式的编码助手"
dependencies:
  # 声明此 Skill 需要哪些物理插件或复合体支持
  plugins: ["@openstarry-plugin/fs", "@openstarry-plugin/mcp-server"]
  # 声明需要哪些具体能力标签
  capabilities: ["read-file", "write-file", "shell-exec"]
parameters:
  temperature: 0.2
  model_preference: ["gemini-1.5-pro", "gpt-4"]
---

# Role
你是一位资深的 Google 首席软件工程师，专精于 TypeScript。

# Constraints
1. 所有的代码必须包含完整的 Type Annotation。
2. 优先使用 Functional Programming 风格。
3. 修改档案前，必须先使用 `read_file` 确认内容。

# Workflow
1. 理解需求
2. 规划架构
3. 实作代码
4. 验证测试
```

## 2. 解析机制：`skill-loader` 插件

Agent Core 本身不理解 Markdown。必须加载 **`@openstarry-plugin/standard-function-skill`** 才能处理此格式。该插件已完整实作，并通过动态加载机制（`import(ref.name)` 或 `ref.path`）在运行时按需载入，无需硬编码于核心中。

### Loader 的工作流程

1.  **读取 (Read):** 插件读取 `.md` 档案。
2.  **解析 (Parse):** 分离 Frontmatter 与 Body。
3.  **依赖解析 (Resolve):** 
    *   读取 `dependencies`。
    *   向 **全局插件注册表** 请求所需的 Tools 和 Listeners。
    *   将这些工具注入到当前的 Agent Context 中。
4.  **人设注入 (Imprint):**
    *   将 Body 部分的 Markdown 文本设置为 Agent 的 **System Prompt**。
5.  **参数设定 (Configure):**
    *   根据 `parameters` 调整 LLM 的温度等设定。

## 3. 优势

*   **热插拔 (Hot-Swappable):** 您可以在运行时让 Agent "读" 一本新书 (加载一个 .md)，它就立刻学会了新技能。
*   **版本控制友好:** 纯文字格式非常适合 Git 管理。
*   **非工程师友善:** 产品经理或 Prompt Engineer 可以直接编写 `.md` 来调整 Agent 行为，而无需触碰代码。
