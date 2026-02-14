# OpenStarry: The Agent Operating System

**OpenStarry** 是一个重新定义智能代理人 (AI Agent) 构建方式的核心架构。它参考了现代操作系统的设计哲学，融合东方“五蕴”思想，旨在打造一个高度模块化、安全、且具备拟人化生命特征的代理人协调层。

我们不只构建 Chatbot，我们构建的是**数字物种的操作系统**。

---

## 🏗️ 系统宏观架构 (Macro-System Architecture)

OpenStarry 采用三层递进的架构设计，模拟生物与其生存环境的共生关系：

### 1. Agent 协调管理层 (Management Zone)
**定位：系统的宿主环境 (Host) 与行政中枢。**
负责提供土壤与养分。这一层确保环境的稳定与安全，包含容器隔离 (Plumbing)、基于因果链的事件调度 (Orchestration)、安全戒律 (Policy) 以及硬件抽象层 (HAL)。它将物理世界的信号转换为 Agent 可理解的数据流。

### 2. Agent Core (Autonomous Life Zone)
**定位：纯粹的“五蕴”计算循环。**
它是“无头 (Headless)”且“无状态 (Stateless)”的生命内核。唯一的职责是维持“受、想、行、识”的计算循环。Core 本质上是空的，它在不同的插件加持下展现出不同的生命样态。

### 3. 能力插件层 (Capability Plugins)
**定位：赋予 Agent 个性、专业与灵魂的功能组件。**
插件决定了 Agent 的能力边界。包括通讯协议 (Protocol)、自我反思 (Reflection) 与状态记忆 (Memory) 插件。这让同一个 Core 可以随时从“代码专家”转化为“设备监控员”。

---

## 🔄 因果生命周期 (The Lifecycle)

在 OpenStarry 中，一个任务的执行被视为一次生命的起灭：
1. **缘起 (Origination)**：环境层侦测到需求。
2. **调度 (Scheduling)**：管理层根据需求匹配所需的插件。
3. **生起 (Arising)**：容器层加载核心并动态注入能力。
4. **运行 (Operation)**：核心处理“痛觉”，达成目标。
5. **寂灭 (Cessation)**：任务完成，经验存回记忆，实例随之销毁。

```mermaid
graph TD
    subgraph Host [🛡️ Management Zone (Host Environment)]
        direction TB
        Orchestrator[调度层] --> Container[容器层]
        Policy[安全策略层] -.-> Container
        HAL[硬件抽象层] --> InputFlow((感知流))
    end

    subgraph Runtime [⚡ Running Instance]
        direction LR
        InputFlow --> Core
        
        subgraph Core [🧠 Agent Core (Microkernel)]
            Loop[执行回路]
            State[状态机]
            Interceptor[异常拦截]
        end

        Core --> |1. Load| Plugins
        
        subgraph Plugins [🔌 Capability Plugins (The 5 Aggregates)]
            Guide[识：Guide]
            Tool[行：Tools]
            LLM[想：Provider]
            Mem[记忆：Memory]
            Pain[痛觉：Reflex]
        end
        
        Plugins --> |2. Inject| Core
        Interceptor -.-> |3. Pain Signal| Guide
        Guide -.-> |4. Correction| Loop
    end
```

---

## 💻 核心配置示例 (The Shape of an Agent)

OpenStarry 的强大在于其声明式的配置。以下是一个具备“痛觉”与“文件操作能力”的标准 Agent 定义：

```jsonc
// agent.json
{
  "identity": { "id": "dev-bot-01", "name": "Resilient Developer" },
  "plugins": [
    // [想] 大脑：注入认知引擎
    { "name": "@openstarry-plugin/provider-gemini" },
    
    // [行] 手脚：注入文件系统操作能力
    { "name": "@openstarry-plugin/standard-function-fs" },
    
    // [受] 感官：监听终端机输入
    { "name": "@openstarry-plugin/standard-function-stdio" },
    
    // [识] 灵魂：注入痛觉机制 (定义如何面对错误)
    { "name": "@openstarry-plugin/guide-pain-mechanism" }
  ],
  "policy": {
    // 管理层戒律：连续犯错 3 次即触发物理熔断
    "safety": { "max_consecutive_errors": 3 } 
  }
}
```

---

## 🌟 十大核心宣言 (The Ten Tenets)

### 1. 代理人即操作系统进程 (Agent as OS Process)
Agent 不是一次性的脚本，而是具备持久生命周期、可被守护进程 (Daemon) 管理、监控、重启的数字实体。它有自己的 PID，有自己的状态，就像一个活着进程。

### 2. 一切皆插件 (Everything is a Plugin)
系统的每一个器官都是可替换的。工具是插件，监听器是插件，LLM 大脑是插件，甚至记忆策略和通讯协议也是插件。Core 只是一个空的插座板，所有能力都来自外部挂载。

### 3. 五蕴聚合架构 (Five Aggregates Architecture)
系统设计深度融合东方哲学。**Core 本质上是“空 (Sunyata)”的容器。** 它的生命特征完全由五种插件（五蕴）赋予：
*   **色 (UI)**、**受 (Listener)**、**想 (Provider)**、**行 (Tool)**。
*   **识 (Guide):** 这是最关键的组件。是 Guide Plugin 注入了记忆与人设，赋予了 Core “自我意识 (Vijnana)”。没有 Guide，Core 只是无意识的运算力。

### 4. 目录结构即协议 (Directory as Protocol)
无论是系统还是项目，无论是本地硬盘还是 USB 设备，只要目录结构符合 `plugins/`, `configs/` 的标准规范，系统即可自动识别并加载。物理结构直接映射了运行时逻辑。

### 5. 目录结构即权限 (Directory as Permission)
系统层与项目层采用同构设计，但权限严格隔离。插件的放置位置决定了其可见性范围；Agent 的运行位置决定了其权限边界。系统管理员无法直接染指业务插件，确保了安全隔离。

### 6. 拟人化的认知流与痛觉 (Anthropomorphic Cognitive Flow & Pain)
错误被转化为 Agent 的“痛觉 (Negative Feedback)”。系统内置反馈回路，将运行时错误注入 Context，迫使 Agent 在失败中自我反思与修正，模拟生物的试错学习过程。

### 7. 微内核与绝对纯净 (Microkernel & Absolute Purity)
Agent Core 采用严格的**微内核架构 (Microkernel Architecture)**。
*   **物理隔离:** 编译后的 Core 二进制档**严禁包含任何插件代码**。
*   **绝对纯净:** Core 只依赖于抽象接口 (SDK)，本身不具备任何具体能力。所有能力都必须在运行时通过外部插件动态注入。
*   **无头设计 (Headless):** 内核是去中心化的，不依赖任何特定的 UI 或 IO 设备。这保证了 Agent 的“灵魂”可以移植到任何“躯壳”中——从 CLI 到 Web，从 Docker 到 IoT 设备。
*   **意义:** 没有内置代码，就没有内置 Bug。

### 8. 控制理论闭环模型 (Control-Theoretic Loop Model)
不仅是执行回路，更是控制回路。系统将用户目标视为参考输入，将 Context 视为状态反馈，将 Tool Call 视为控制变量。Agent 的本质是一个不断最小化“目标与现状误差”的智能控制器。

### 9. 可插拔的记忆策略 (Pluggable Context Strategy)
记忆管理不再是硬编码的逻辑。开发者可以根据 Agent 的角色需求，动态更换记忆策略（滑动窗口、动态摘要、状态提取），灵活平衡成本与记忆深度。

### 10. 分形社会结构 (Fractal Social Structure)
系统具有自相似性。一个复杂的 Agent 可以由多个子 Agent 组成，对外暴露统一的 MCP 接口。这种分形设计允许我们构建无限层级的协作网络，实现“由一而生万物”的数字社会。

---

## 📚 文档导航地图 (Documentation Map)

### 1. 系统架构文档 (Architecture Documentation)
*定义系统的愿景、角色与宏观启动流程。*
* [00_设计哲学 (OpenStarry Design Philosophy)](./Architecture_Documentation/00_OpenStarry_Design_Philosophy.md)
* [01_架构概览 (Architecture Overview)](./Architecture_Documentation/01_Architecture_Overview.md)
* [02_无头代理核心 (Headless Agent Core)](./Architecture_Documentation/02_Headless_Agent_Core.md)
* [03_代理设计与模板服务 (Agent Design & Template Service)](./Architecture_Documentation/03_Agent_Design_and_Template_Service.md)
* [04_插件基础设施 (Plugin Infrastructure)](./Architecture_Documentation/04_Plugin_Infrastructure.md)
* [05_Linux 设计原则启发 (Linux Design Principles Inspiration)](./Architecture_Documentation/05_Linux_Design_Principles_Inspiration.md)
* [06_插件接口示例 (Plugin Interface Examples)](./Architecture_Documentation/06_Plugin_Interface_Examples.md)
* [07_支持引擎生态系 (Supporting Engines Ecosystem)](./Architecture_Documentation/07_Supporting_Engines_Ecosystem.md)
* [08_命令与工具设计 (Command & Tool Design)](./Architecture_Documentation/08_Command_And_Tool_Design.md)
* [09_通讯协议策略 (Communication Protocol Strategy)](./Architecture_Documentation/09_Communication_Protocol_Strategy.md)
* [10_引导与插件加载 (Bootstrapping & Plugin Loading)](./Architecture_Documentation/10_Bootstrapping_And_Plugin_Loading.md)
* [11_代理管理工具设计 (Agent Manager Tool Design)](./Architecture_Documentation/11_Agent_Manager_Tool_Design.md)
* [12_工作流引擎工具设计 (Workflow Engine Tool Design)](./Architecture_Documentation/12_Workflow_Engine_Tool_Design.md)
* [13_编排守护进程设计 (Orchestrator Daemon Design)](./Architecture_Documentation/13_Orchestrator_Daemon_Design.md)
* [14_系统启动序列 (System Boot Sequence)](./Architecture_Documentation/14_System_Boot_Sequence.md)
* [15_启动与任务流 (System Startup & Task Flow)](./Architecture_Documentation/15_System_Startup_and_Task_Flow.md)
* [16_插件类型哲学映射 (Plugin Types Philosophical Mapping)](./Architecture_Documentation/16_Plugin_Types_Philosophical_Mapping.md)
* [17_宿主引导模式 (Host Bootstrapping Pattern)](./Architecture_Documentation/17_Host_Bootstrapping_Pattern.md)
* [18_插件加载协议 (Plugin Loading Protocol)](./Architecture_Documentation/18_Plugin_Loading_Protocol.md)
* [19_代理协调层 (Agent Coordination Layer)](./Architecture_Documentation/19_Agent_Coordination_Layer.md)
* [20_依赖编织与控制回路 (Dependency Wiring & Control Loop)](./Architecture_Documentation/20_Dependency_Injection_and_Control_Loop.md)
* [21_插件接口深度解析 (Plugin Interface Deep Dive)](./Architecture_Documentation/21_Plugin_Interface_Deep_Dive.md)
* [22_代理人协调层：归一化与适配 (Agent Coordination Layer: Normalization)](./Architecture_Documentation/22_Agent_Coordination_Layer_Normalization.md)
* [23_动态插件加载与命名 (Dynamic Plugin Loading & Naming)](./Architecture_Documentation/23_Dynamic_Plugin_Loading_and_Naming.md)
* [24_Runner 架构 (Runner Architecture)](./Architecture_Documentation/24_Runner_Architecture.md)
* [25_PushInput 事件架构 (PushInput Event Architecture)](./Architecture_Documentation/25_PushInput_Event_Architecture.md)
* [26_插件服务与生命周期管理 (Plugin Service & Lifecycle Management)](./Architecture_Documentation/26_Plugin_Service_And_Lifecycle_Management.md)
* [27_系统拓扑与管理层架构 (System Topology & Management Zone)](./Architecture_Documentation/27_System_Topology_and_Management_Zone.md)

### 2. 核心组件深潜 (Agent Core Components Deep Dive)
*深入内核，研究具体技术机制与理论模型。*
* [00_核心哲学 (Core Philosophy)](./Agent_Core_Components_Deep_Dive/00_Core_Philosophy.md)
* [01_执行回路 (Execution Loop)](./Agent_Core_Components_Deep_Dive/01_Execution_Loop.md)
* [02_通讯接口 (Communication Interface)](./Agent_Core_Components_Deep_Dive/02_Communication_Interface.md)
* [03_安全层 (Security Layer)](./Agent_Core_Components_Deep_Dive/03_Security_Layer.md)
* [04_状态管理器 (State Manager)](./Agent_Core_Components_Deep_Dive/04_State_Manager.md)
* [05_插件基础设施整合 (Plugin Infrastructure Integration)](./Agent_Core_Components_Deep_Dive/05_Plugin_Infrastructure_Integration.md)
* [06_状态持久化机制 (State Persistence Mechanism)](./Agent_Core_Components_Deep_Dive/06_State_Persistence_Mechanism.md)
* [07_安全断路器 (Safety Circuit Breakers)](./Agent_Core_Components_Deep_Dive/07_Safety_Circuit_Breakers.md)
* [08_安全实现 (Safety Implementation)](./Agent_Core_Components_Deep_Dive/08_Safety_Implementation.md)
* [09_可观测性与追踪 (Observability and Tracing)](./Agent_Core_Components_Deep_Dive/09_Observability_and_Tracing.md)
* [10_上下文管理策略 (Context Management Strategy)](./Agent_Core_Components_Deep_Dive/10_Context_Management_Strategy.md)
* [11_插件运行时隔离 (Plugin Runtime Isolation)](./Agent_Core_Components_Deep_Dive/11_Plugin_Runtime_Isolation.md)
* [12_错误处理与自我修正 (Error Handling & Self Correction)](./Agent_Core_Components_Deep_Dive/12_Error_Handling_and_Self_Correction.md)
* [13_代理核心作为控制系统 (Agent Core as Control System)](./Agent_Core_Components_Deep_Dive/13_Agent_Core_as_Control_System.md)
* [14_代理核心哲学：五蕴 (Agent Core Philosophy: Five Aggregates)](./Agent_Core_Components_Deep_Dive/14_Agent_Core_Philosophy_Five_Aggregates.md)
* [16_OpenStarry 标准协议 (OpenStarry Standard Protocol)](./Agent_Core_Components_Deep_Dive/16_OpenStarry_Standard_Protocol.md)

### 3. 项目结构与规范 (Project Structure and Conventions)
*定义物理布局、源码组织、开发流程与安装规范。*
* [00_路线图与里程碑 (Roadmap & Milestones)](./Project_Structure_and_Conventions/00_Roadmap_and_Milestones.md)
* [01_Monorepo 顶层结构 (Monorepo Top Level Structure)](./Project_Structure_and_Conventions/01_Monorepo_Top_Level_Structure.md)
* [02_核心源码结构 (Core Source Code Structure)](./Project_Structure_and_Conventions/02_Core_Source_Code_Structure.md)
* [03_共享组件与 SDK 结构 (Shared & SDK Structure)](./Project_Structure_and_Conventions/03_Shared_and_SDK_Structure.md)
* [04_标准代理目录解剖 (Standard Agent Directory Anatomy)](./Project_Structure_and_Conventions/04_Standard_Agent_Directory_Anatomy.md)
* [05_代理清单规范 (Agent Manifest Specification)](./Project_Structure_and_Conventions/05_Agent_Manifest_Specification.md)
* [06_插件目录惯例 (Plugin Directory Conventions)](./Project_Structure_and_Conventions/06_Plugin_Directory_Conventions.md)
* [07_编码与测试标准 (Coding & Testing Standards)](./Project_Structure_and_Conventions/07_Coding_and_Testing_Standards.md)
* [08_系统与项目运行时布局 (System & Project Runtime Layouts)](./Project_Structure_and_Conventions/08_System_and_Project_Runtime_Layouts.md)
* [09_CLI 设计与管理命令 (CLI Design & Management Commands)](./Project_Structure_and_Conventions/09_CLI_Design_and_Management_Commands.md)
* [10_构建与发布策略 (Build & Distribution Strategy)](./Project_Structure_and_Conventions/10_Build_and_Distribution_Strategy.md)
* [11_第三方插件安装 (Third-Party Plugin Installation)](./Project_Structure_and_Conventions/11_Third_Party_Plugin_Installation.md)
* [12_能力注入机制 (Capabilities Injection Mechanism)](./Project_Structure_and_Conventions/12_Capabilities_Injection_Mechanism.md)
* [13_复合插件与依赖 (Composite Plugins & Dependencies)](./Project_Structure_and_Conventions/13_Composite_Plugins_and_Dependencies.md)
* [14_Markdown 技能规范 (Markdown Skill Specification)](./Project_Structure_and_Conventions/14_Markdown_Skill_Specification.md)

### 4. 插件基础设施示例 (Plugin System Architecture)
*插件系统的具体应用、概念与规范。*
* [00_插件哲学：五蕴 (Plugin Philosophy: Five Aggregates)](./Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md)
* [01_MCP 插件示例 (MCP Plugin Example)](./Plugin_System_Architecture/01_MCP_Plugin_Example.md)
* [02_MCP 协议整合 (MCP Protocol Integration)](./Plugin_System_Architecture/02_MCP_Protocol_Integration.md)
* [03_开发者工具示例 (Developer Tools Example)](./Plugin_System_Architecture/03_Developer_Tools_Example.md)
* [04_网络交互示例 (Web Interaction Example)](./Plugin_System_Architecture/04_Web_Interaction_Example.md)
* [05_进阶 UI 与设备示例 (Advanced UI & Device Example)](./Plugin_System_Architecture/05_Advanced_UI_And_Device_Example.md)
* [06_数据验证示例 (Data Validation Example)](./Plugin_System_Architecture/06_Data_Validation_Example.md)

### 5. 实施示例与指南 (Implementation Examples)
*动手写代码，从案例学习实践。*
* [上下文策略：滑动窗口 (Context Strategy: Sliding Window)](./Implementation_Examples/Context_Strategy_SlidingWindow.md)
* [开发者指南：独立执行 (Developer Guide: Standalone Execution)](./Implementation_Examples/Developer_Guide_Standalone_Execution.md)
* [OpenClaw 协调层 (OpenClaw Coordination Layer)](./Implementation_Examples/openclaw_Coordination_Layer.md)
* [OpenClaw UI 频道适配器 (OpenClaw UI Channel Adapters)](./Implementation_Examples/openclaw_UI_Channel_Adapters.md)
* [OpenCode 代码解释器套件 (OpenCode Code Interpreter Suite)](./Implementation_Examples/opencode_Code_Interpreter_Suite.md)
* [Provider: Gemini 示例](./Implementation_Examples/Provider_Gemini_Example.md)
* [Tool: 代码解释器示例](./Implementation_Examples/Tool_CodeInterpreter_Example.md)
* [Tool: 读取文件示例](./Implementation_Examples/Tool_ReadFile_Example.md)
* [Transport: WebSocket 插件](./Implementation_Examples/Transport_Plugin_Websocket.md)
* [UI 插件示例 (UI Plugin Example)](./Implementation_Examples/UI_Plugin_Example.md)
* [USB 即插即用代理场景 (USB Plug-and-Play Agent Scenario)](./Implementation_Examples/USB_Plug_and_Play_Agent_Scenario.md)
* [拟人化痛觉机制示例 (Pain Mechanism Demo)](./Implementation_Examples/Pain_Mechanism_Demo.md)


---

## 🛠️ 快速开始

准备好开始了吗？请参阅 **[Developer_Guide_Standalone_Execution.md](./Implementation_Examples/Developer_Guide_Standalone_Execution.md)** 运行您的第一个 Agent。