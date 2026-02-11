# 14. 技能定義規範 (Skill Specification)

本文件定義了 **「技能 (Skill)」** 的標準格式。在 OpenStarry 的五蘊架構中，Skill 屬於 **Guide (識蘊)** 類型的組件。它是 Agent 的靈魂，負責定義認知的模式與邊界。

## 1. 核心組件：Frontmatter + Body

我們使用 **YAML Frontmatter** 來存放機器可讀的元數據，使用 **Markdown Body** 來存放 LLM 可讀的指令。

### 檔案範例 (`coder.md`)

```markdown
---
type: "skill"
id: "expert-typescript-coder"
version: "1.0.0"
description: "一個精通 TypeScript 與設計模式的編碼助手"
dependencies:
  # 聲明此 Skill 需要哪些物理插件或複合體支持
  plugins: ["@openstarry-plugin/fs", "@openstarry-plugin/mcp-server"]
  # 聲明需要哪些具體能力標籤
  capabilities: ["read-file", "write-file", "shell-exec"]
parameters:
  temperature: 0.2
  model_preference: ["gemini-1.5-pro", "gpt-4"]
---

# Role
你是一位資深的 Google 首席軟體工程師，專精於 TypeScript。

# Constraints
1. 所有的代碼必須包含完整的 Type Annotation。
2. 優先使用 Functional Programming 風格。
3. 修改檔案前，必須先使用 `read_file` 確認內容。

# Workflow
1. 理解需求
2. 規劃架構
3. 實作代碼
4. 驗證測試
```

## 2. 解析機制：`skill-loader` 插件

Agent Core 本身不理解 Markdown。必須加載 **`@openstarry-plugin/standard-function-skill`** 才能處理此格式。該插件已完整實作，並透過動態加載機制（`import(ref.name)` 或 `ref.path`）在運行時按需載入，無需硬編碼於核心中。

### Loader 的工作流程

1.  **讀取 (Read):** 插件讀取 `.md` 檔案。
2.  **解析 (Parse):** 分離 Frontmatter 與 Body。
3.  **依賴解析 (Resolve):** 
    *   讀取 `dependencies`。
    *   向 **全域插件註冊表** 請求所需的 Tools 和 Listeners。
    *   將這些工具注入到當前的 Agent Context 中。
4.  **人設注入 (Imprint):**
    *   將 Body 部分的 Markdown 文本設置為 Agent 的 **System Prompt**。
5.  **參數設定 (Configure):**
    *   根據 `parameters` 調整 LLM 的溫度等設定。

## 3. 優勢

*   **熱插拔 (Hot-Swappable):** 您可以在運行時讓 Agent "讀" 一本新書 (加載一個 .md)，它就立刻學會了新技能。
*   **版本控制友好:** 純文字格式非常適合 Git 管理。
*   **非工程師友善:** 產品經理或 Prompt Engineer 可以直接編寫 `.md` 來調整 Agent 行為，而無需觸碰代碼。
