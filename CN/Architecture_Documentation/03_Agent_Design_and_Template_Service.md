# 03. 代理人设计与模板服务 (Agent Design & Template Service)

本文档详细描述了 OpenStarry 协调层中的「设计子系统」。它负责管理代理人的「基因蓝图」，即模板 (Templates)。

## 核心定位：动态设计工作坊

在 OpenStarry 架构中，**Agent Core 是执行者，Orchestrator 是管理者，而本服务则是「造物主」的工具台。**

它不仅是一个静态的 JSON 文件数据库，而是一个**运行时服务**，允许人类或高级代理人（如 Master Agent）通过 API 动态地创建、修改和实例化新的代理人角色。

---

## 功能职责

### 1. 模板管理 (Template Management)
负责存储和版本控制所有的 Agent Template。一个 Template 定义了 Agent 的不可变属性：
*   **System Prompt:** 核心人格与指令。
*   **Plugin Manifest:** 需要加载的插件列表（能力清单）。
*   **Default Configuration:** 默认的参数配置（如 LLM 型号、温度）。

### 2. 设计接口 (Design Interface)
提供一组标准 API，供外部系统调用以进行设计工作：
*   `POST /templates`: 根据需求创建新模板。
*   `GET /templates/{id}`: 获取模板详情（供 Daemon 启动 Agent 时使用）。
*   `PUT /templates/{id}/plugins`: 为现有模板增加新能力。

---

## 人机协同设计流程

我们提倡一种 **「由 AI 辅助的 AI 设计 (AI-Assisted AI Design)」** 模式：

1.  **人类意图:** 管理员告诉主代理人：「我需要一个专门分析日志的助手。」
2.  **主代理人推导:** Master Agent 的 LLM 分析需求，推导出该助手需要 `fs:read` 工具和 `log-analysis-plugin`。
3.  **调用设计服务:** Master Agent 调用 `AgentDesignerTool`，向本服务发送 API 请求。
4.  **模板生成:** 本服务验证请求，生成一个新的 `LogAnalyst_Template_v1` 并存储。
5.  **实例化:** Master Agent 随后调用 Daemon，基于该新模板启动一个工作代理人实例。

这种设计使得 OpenStarry 具备了自我扩展和进化的能力。