<!-- Layer: 1-Engineering -->

# OpenStarry: The Agent Operating System

**OpenStarry** 是一個重新定義智能代理人 (AI Agent) 構建方式的核心架構。它參考了現代操作系統的設計哲學，融合東方「五蘊」思想，旨在打造一個高度模組化、安全、且具備擬人化生命特徵的代理人協調層。

我們不只構建 Chatbot，我們構建的是**數位物種的操作系統**。

---

## 🏗️ 系統宏觀架構 (Macro-System Architecture)

OpenStarry 採用三層遞進的架構設計，模擬生物與其生存環境的共生關係：

### 1. Agent 協調管理層 (Management Zone)
**定位：系統的宿主環境 (Host) 與行政中樞。**
負責提供土壤與養分。這一層確保環境的穩定與安全，包含容器隔離 (Plumbing)、基於因果鏈的事件調度 (Orchestration)、安全戒律 (Policy) 以及硬體抽象層 (HAL)。它將物理世界的訊號轉換為 Agent 可理解的數據流。

### 2. Agent Core (Autonomous Life Zone)
**定位：純粹的「五蘊」計算循環。**
它是「無頭 (Headless)」且「無狀態 (Stateless)」的生命內核。唯一的職責是維持「受、想、行、識」的計算循環。Core 本質上是空的，它在不同的插件加持下展現出不同的生命樣態。

### 3. 能力插件層 (Capability Plugins)
**定位：賦予 Agent 個性、專業與我執框架（身份認同）的功能組件。**
插件決定了 Agent 的能力邊界。包括通訊協議 (Protocol)、自我反思 (Reflection) 與狀態記憶 (Memory) 插件。這讓同一個 Core 可以隨時從「程式碼專家」轉化為「設備監控員」。

---

## 🔄 因果生命週期 (The Lifecycle)

在 OpenStarry 中，一個任務的執行被視為一次生命的起滅：
1. **緣起 (Origination)**：環境層偵測到需求。
2. **調度 (Scheduling)**：管理層根據需求匹配所需的插件。
3. **生起 (Arising)**：容器層載入核心並動態注入能力。
4. **運行 (Operation)**：核心處理「痛覺」，達成目標。
5. **寂滅 (Cessation)**：任務完成，經驗存回記憶，實例隨之銷毀。

```mermaid
graph TD
    subgraph Host [🛡️ Management Zone (Host Environment)]
        direction TB
        Orchestrator[調度層] --> Container[容器層]
        Policy[安全策略層] -.-> Container
        HAL[硬體抽象層] --> InputFlow((感知流))
    end

    subgraph Runtime [⚡ Running Instance]
        direction LR
        InputFlow --> Core
        
        subgraph Core [🧠 Agent Core (Microkernel)]
            Loop[執行迴圈]
            State[狀態機]
            Interceptor[異常攔截]
        end

        Core --> |1. Load| Plugins
        
        subgraph Plugins [🔌 Capability Plugins (The 5 Aggregates)]
            Sensory[色：UI + Listener]
            Vedana[受：Sensation]
            LLM[想：Provider]
            Tool[行：Tools]
            Guide[識：Guide]
        end

        Plugins --> |2. Inject| Core
        Interceptor -.-> |3. Vedana Signal| Guide
        Guide -.-> |4. Correction| Loop
    end
```

---

## 💻 核心配置範例 (The Shape of an Agent)

OpenStarry 的強大在於其聲明式的配置。以下是一個具備「痛覺」與「檔案操作能力」的標準 Agent 定義：

```jsonc
// agent.json
{
  "identity": { "id": "dev-bot-01", "name": "Resilient Developer" },
  "plugins": [
    // [想] 大腦：注入認知引擎
    { "name": "@openstarry-plugin/provider-gemini" },
    
    // [行] 手腳：注入檔案系統操作能力
    { "name": "@openstarry-plugin/standard-function-fs" },
    
    // [色·輸入] 感官：監聽終端機輸入
    { "name": "@openstarry-plugin/standard-function-stdio" },

    // [識] 我執框架：注入身份認同與行為約束
    { "name": "@openstarry-plugin/guide-character-init" }
  ],
  "safety": {
    // 連續相同失敗 3 次即觸發熔斷
    "repetitiveFailThreshold": 3
  }
}
```

---

## 🌟 十大核心宣言 (The Ten Tenets)

### 1. 代理人即操作系統進程 (Agent as OS Process)
Agent 不是一次性的腳本，而是具備持久生命週期、可被守護進程 (Daemon) 管理、監控、重啟的數位實體。它有自己的 PID，有自己的狀態，就像一個活著的進程。

### 2. 一切皆插件 (Everything is a Plugin)
系統的每一個器官都是可替換的。工具是插件，監聽器是插件，LLM 大腦是插件，甚至記憶策略和通訊協議也是插件。Core 只是一個空的插座板，所有能力都來自外部掛載。

### 3. 五蘊聚合架構 (Five Aggregates Architecture)
系統設計深度融合東方哲學。**Core 無自性（Svabhava）——功能完全依賴插件的因緣和合（緣起性空）。** 它的生命特徵完全由五種插件（五蘊）賦予：
*   **色 Sensory (UI + Listener)**：物質形體與感官通道。UI 是 Agent 的外顯形相，Listener 是 Agent 的感官根門（indriya）。兩者皆屬色蘊的不同面向。
*   **受 Sensation**：感受品質機制。Agent 對運行狀態的苦（錯誤/異常）、樂（成功/達標）、捨（中性穩態）三受回饋系統。
*   **想 Cognition (Provider)**：認知處理引擎。涵蓋 LLM、推理迴圈、CNN/RNN/VLM 等認知管線組合。
*   **行 Action (Tool)**：意志驅動的行動能力。
*   **識 Identity (Guide)**：**我執框架（Ego Framework）**——行為約束與自我收斂機制。Guide 定義了 Agent 的身份認同、行為邊界與約束規則，使 Agent 在發散的可能性中收斂到預期行為。沒有 Guide，Core 只是無方向的運算力。

### 4. 目錄結構即協議 (Directory as Protocol)
無論是系統還是專案，無論是本地硬碟還是 USB 設備，只要目錄結構符合 `plugins/`, `configs/` 的標準規範，系統即可自動識別並加載。物理結構直接映射了運行時邏輯。

### 5. 目錄結構即權限 (Directory as Permission)
系統層與專案層採用同構設計，但權限嚴格隔離。插件的放置位置決定了其可見性範圍；Agent 的運行位置決定了其權限邊界。系統管理員無法直接染指業務插件，確保了安全隔離。

### 6. 八識俱轉與功能性代理 (Concurrent Consciousness and Functional Agency)
The Agent's cognitive architecture models eight concurrent consciousness streams (八識俱轉) operating simultaneously: the five sensory consciousnesses (前五識) as input channels, the sixth consciousness (第六意識) as deep cognition (Provider/LLM), the seventh consciousness (末那識/Manas) as the persistent self-referencing layer (identity, kleshas, volition), and the eighth consciousness (阿賴耶識/Alaya) as the distributed seed-store coordinated across the Management Zone. These eight streams do not execute sequentially — they co-arise and co-condition through the Five Aggregates plugin architecture, enabling functional agency that emerges from their concurrent interplay rather than from any single central controller.

> [Research Team Update — DC Master 確認] 原 Tenet #6「擬人化的認知流與三受回饋」(Vedana-Sahaja) 內容移入架構內部文件 (Docs 36, 39, 40)，作為技術規格保留。Tenet 位置改為闡述 OpenStarry 的核心願景：以唯識學八識俱轉為藍圖，透過五蘊插件架構實現功能性 AGI。

### 7. 微內核與絕對純淨 (Microkernel & Absolute Purity)
Agent Core 採用嚴格的**微內核架構 (Microkernel Architecture)**。
*   **物理隔離:** 編譯後的 Core 二進制檔**嚴禁包含任何插件代碼**。
*   **絕對純淨:** Core 只依賴於抽象介面 (SDK)，本身不具備任何具體能力。所有能力都必須在運行時通過外部插件動態注入。Core 不含任何 policy 常量——所有策略值外部化至 SDK defaults 和使用者配置。
*   **無頭設計 (Headless):** 內核是去中心化的，不依賴任何特定的 UI 或 IO 設備。這保證了 Agent 的「識」（身份與行為框架）可以移植到任何「色身」中——從 CLI 到 Web，從 Docker 到 IoT 設備。
*   **三級關鍵性模型:** Plugin 缺席時的行為依據三級模型——Required (throw Error)、Optional-degraded (中性值)、Optional-no-effect (功能不可用)。
*   **意義:** 沒有內置代碼，就沒有內置 Bug。

> [Cycle 02-10 Update] Plan32-33 已達成本宣言完全合規（REM-7.1 於 v0.33.0-alpha 清除）：移除 auto-mount fallback、提取內建組件為 plugin 包、遷移全部 53 個 policy 常量至 SDK/Config、提取 context manager。Core 零 policy 常數（v0.34.0-alpha 仍為 53，Plan34 新增項目均為 mechanism/validation，非 policy）。`[Cycle 03-29 H-2: Doc 51 合規報告原為 phantom 引用；本宣言條目 §十大核心宣言 即為 Tenet 規範權威。]`

### 8. 控制理論閉環模型 (Control-Theoretic Loop Model)
不僅是執行迴圈，更是控制迴圈。系統將用戶目標視為參考輸入，將 Context 視為狀態反饋，將 Tool Call 視為控制變量。Agent 的本質是一個不斷最小化「目標與現狀誤差」的智能控制器。

### 9. 可插拔的記憶策略 (Pluggable Context Strategy)
記憶管理不再是硬編碼的邏輯。開發者可以根據 Agent 的角色需求，動態更換記憶策略（滑動窗口、動態摘要、狀態提取），靈活平衡成本與記憶深度。

> [Cycle 02-12_final Complete — Plan36b ✅] T3 Confirmation Gate (IConfirmationGate + StandardConfirmationGate plugin，samskara skandha，approve/deny/ask_user 三種決策，nonce anti-spoofing，fail-closed gate error)。Advanced Security: Detached SIGNATURE.json plugin 簽名驗證 + Audit Trail per-entry SHA-256 hash chain + verifyAuditChain() 完整性驗證。Hotfix: cumulative clamp 0.1→0.05。v0.36.1-alpha: 1983 tests, 38 workspace projects, 32 plugins, 合規 8/1/1。**Cycle 02 完結。**

### 10. 分形社會結構 (Fractal Social Structure)
系統具有自相似性。一個複雜的 Agent 可以由多個子 Agent 組成，對外暴露統一的 MCP 接口。這種分形設計允許我們構建無限層級的協作網絡，實現「由一而生萬物」的數位社會。

---

## 📚 文檔導航地圖 (Documentation Map)

### 1. 系統架構文檔 (Architecture Documentation)
*定義系統的願景、角色與宏觀啟動流程。*
* [00_設計哲學 (OpenStarry Design Philosophy)](./Architecture_Documentation/00_OpenStarry_Design_Philosophy.md)
* [01_架構概覽 (Architecture Overview)](./Architecture_Documentation/01_Architecture_Overview.md)
* [02_無頭代理核心 (Headless Agent Core)](./Architecture_Documentation/02_Headless_Agent_Core.md)
* [03_代理設計與模板服務 (Agent Design & Template Service)](./Architecture_Documentation/03_Agent_Design_and_Template_Service.md)
* [04_插件基礎設施 (Plugin Infrastructure)](./Architecture_Documentation/04_Plugin_Infrastructure.md)
* [05_Linux 設計原則啟發 (Linux Design Principles Inspiration)](./Architecture_Documentation/05_Linux_Design_Principles_Inspiration.md)
* [06_插件介面範例 (Plugin Interface Examples)](./Architecture_Documentation/06_Plugin_Interface_Examples.md)
* [07_支持引擎生態系 (Supporting Engines Ecosystem)](./Architecture_Documentation/07_Supporting_Engines_Ecosystem.md)
* [08_命令與工具設計 (Command & Tool Design)](./Architecture_Documentation/08_Command_And_Tool_Design.md)
* [09_通訊協議策略 (Communication Protocol Strategy)](./Architecture_Documentation/09_Communication_Protocol_Strategy.md) `[Cycle 03-1 更新：多代理通訊模式]`
* [10_引導與插件載入 (Bootstrapping & Plugin Loading)](./Architecture_Documentation/10_Bootstrapping_And_Plugin_Loading.md)
* [11_代理管理工具設計 (Agent Manager Tool Design)](./Architecture_Documentation/11_Agent_Manager_Tool_Design.md)
* [12_工作流引擎工具設計 (Workflow Engine Tool Design)](./Architecture_Documentation/12_Workflow_Engine_Tool_Design.md)
* [13_編排守護進程設計 (Orchestrator Daemon Design)](./Architecture_Documentation/13_Orchestrator_Daemon_Design.md)
* [14_系統啟動序列 (System Boot Sequence)](./Architecture_Documentation/14_System_Boot_Sequence.md)
* [15_啟動與任務流 (System Startup & Task Flow)](./Architecture_Documentation/15_System_Startup_and_Task_Flow.md)
* [16_插件類型哲學映射 (Plugin Types Philosophical Mapping)](./Architecture_Documentation/16_Plugin_Types_Philosophical_Mapping.md)
* [17_宿主引導模式 (Host Bootstrapping Pattern)](./Architecture_Documentation/17_Host_Bootstrapping_Pattern.md)
* [18_插件載入協議 (Plugin Loading Protocol)](./Architecture_Documentation/18_Plugin_Loading_Protocol.md)
* [19_代理協調層 (Agent Coordination Layer)](./Architecture_Documentation/19_Agent_Coordination_Layer.md) `[Cycle 03-1 更新：多代理協調]`
* [20_依賴編織與控制迴路 (Dependency Wiring & Control Loop)](./Architecture_Documentation/20_Dependency_Injection_and_Control_Loop.md)
* [21_插件介面深度解析 (Plugin Interface Deep Dive)](./Architecture_Documentation/21_Plugin_Interface_Deep_Dive.md)
* [22_代理人協調層：歸一化與適配 (Agent Coordination Layer: Normalization)](./Architecture_Documentation/22_Agent_Coordination_Layer_Normalization.md)
* [23_動態插件載入與命名 (Dynamic Plugin Loading & Naming)](./Architecture_Documentation/23_Dynamic_Plugin_Loading_and_Naming.md)
* [24_Runner 架構 (Runner Architecture)](./Architecture_Documentation/24_Runner_Architecture.md)
* [25_PushInput 事件架構 (PushInput Event Architecture)](./Architecture_Documentation/25_PushInput_Event_Architecture.md)
* [26_插件服務與生命週期管理 (Plugin Service & Lifecycle Management)](./Architecture_Documentation/26_Plugin_Service_And_Lifecycle_Management.md)
* [27_系統拓樸與管理層架構 (System Topology & Management Zone)](./Architecture_Documentation/27_System_Topology_and_Management_Zone.md)
* [28_IInferenceProvider 設計規格 (IInferenceProvider Design Spec)](./Architecture_Documentation/28_IInferenceProvider_Design_Spec.md)
* [29_五蘊核心連結 (Five Aggregates Core Connection)](./Architecture_Documentation/29_Five_Aggregates_Core_Connection.md)
* [30_觸共生模型 (Sparsha Coarising Model)](./Architecture_Documentation/30_Sparsha_Coarising_Model.md)
* [31_八識運行時 (Eight Consciousnesses Runtime)](./Architecture_Documentation/31_Eight_Consciousnesses_Runtime.md)
* [32_行蘊擴展範疇 (Samskara Extended Scope)](./Architecture_Documentation/32_Samskara_Extended_Scope.md)
* [33_多值蘊設計 (Multivalue Skandha Design)](./Architecture_Documentation/33_Multivalue_Skandha_Design.md)
* [34_OpenStarry 願景宣言 (OpenStarry Vision Statement)](./Architecture_Documentation/34_OpenStarry_Vision_Statement.md)
* [35_雙齒輪末那時鐘 (Dual Gear Mano Clock)](./Architecture_Documentation/35_Dual_Gear_Mano_Clock.md)
* [36_受蘊測量模型 (Vedana Measurement Model)](./Architecture_Documentation/36_Vedana_Measurement_Model.md)
* [37_煩惱增益調度 (Klesha Gain Scheduling)](./Architecture_Documentation/37_Klesha_Gain_Scheduling.md)
* [38_IVolition 審議模式 (IVolition Deliberation Pattern)](./Architecture_Documentation/38_IVolition_Deliberation_Pattern.md)
* [39_共生束與五遍行 (CoarisingBundle & Five Universals)](./Architecture_Documentation/39_CoarisingBundle_Five_Universals.md)
* [40_Tenet 6 架構映射 (Tenet 6 Architecture Mapping)](./Architecture_Documentation/40_Tenet_6_Architecture_Mapping.md)
* [41_設計決策與理據 (Design Decisions and Rationale)](./Architecture_Documentation/41_Design_Decisions_and_Rationale.md)
* [42_IGearArbiter 介面規格 (IGearArbiter Interface Spec)](./Architecture_Documentation/42_IGearArbiter_Interface_Spec.md)
* [43_認知迴圈品質監控 (Cognitive Loop Quality Monitoring)](./Architecture_Documentation/43_Cognitive_Loop_Quality_Monitoring.md)
* [44_安全架構總覽 (Safety Architecture Overview)](./Architecture_Documentation/44_Safety_Architecture_Overview.md)
* [45_五蘊 OOP 架構 (Five Skandha Object-Oriented Architecture)](./Architecture_Documentation/45_Five_Skandha_OOP_Architecture.md)
* [46_AuditContext 與 Extras 協議 (AuditContext and Extras Protocol)](./Architecture_Documentation/46_AuditContext_and_Extras_Protocol.md)
* [47_認知序列附錄 (Cognitive Sequence Appendix)](./Architecture_Documentation/47_Cognitive_Sequence_Appendix.md) `[Cycle 02-8 新增]`
* [48_十二因緣時間分類學 (Twelve Nidanas Temporal Taxonomy)](./Architecture_Documentation/48_Twelve_Nidanas_Temporal_Taxonomy.md) `[Cycle 02-8 新增]`
* [49_蘊歸屬軟約束 (Skandha Soft Constraints)](./Architecture_Documentation/49_Skandha_Soft_Constraints.md) `[Cycle 02-8 新增]`
* [50_微內核純化理據 (Microkernel Purification Rationale)](./Architecture_Documentation/50_Microkernel_Purification_Rationale.md) `[Cycle 02-9 新增 / cycle 03-29 H-2 renumbered]`
* [51_Codex 審閱回應報告 (Codex Review Response)](./Architecture_Documentation/51_Codex_Review_Response.md) `[Cycle 02-10 新增 / cycle 03-29 H-2 renumbered]`
* [52_阿賴耶識部分映射 (Alaya Partial Mapping)](./Architecture_Documentation/52_Alaya_Partial_Mapping.md) `[Cycle 02-11 新增 / cycle 03-29 H-2 renumbered]`
* [53_多代理通訊介面規格 (Multi-Agent Communication Interface Spec)](./Architecture_Documentation/53_Multi_Agent_Communication_Interface_Spec.md) `[Cycle 03-1 新增 / cycle 03-29 H-2 renumbered]`
* [54_多代理安全模型 (Multi-Agent Security Model)](./Architecture_Documentation/54_Multi_Agent_Security_Model.md) `[Cycle 03-2 新增 / cycle 03-29 H-2 renumbered]`
* [55_AC7 分散式阿賴耶運行時 (AC7 Distributed Alaya Runtime)](./Architecture_Documentation/55_AC7_Distributed_Alaya_Runtime.md) `[Cycle 03-29 H-2 renumbered]`
* [56_Audit Path B 修正差異 (Audit Path B Modified Delta)](./Architecture_Documentation/56_Audit_Path_B_Modified_Delta.md) `[Cycle 03-29 H-2 renumbered]`
* [57_註冊表橋接 IPC (Registry Bridge IPC)](./Architecture_Documentation/57_Registry_Bridge_IPC.md) `[Cycle 03-29 H-2 renumbered]`
* [58_類型化服務註冊表設計 (Typed Service Registry Design)](./Architecture_Documentation/58_Typed_Service_Registry_Design.md) `[Cycle 03-29 H-2 renumbered]`
* [59_V4 規格與訊號理論分析 (V4 Spec Signal Theory Analysis)](./Architecture_Documentation/59_V4_Spec_Signal_Theory_Analysis.md) `[Cycle 03-29 H-6 added]`
* [60_校準收斂模型 (Calibration Convergence Model)](./Architecture_Documentation/60_Calibration_Convergence_Model.md) `[Cycle 03-29 H-6 added]`
* [61_ENG-FAB Checklist 設計理據 (ENG-FAB Checklist Design Rationale)](./Architecture_Documentation/61_ENG_FAB_Checklist_Design_Rationale.md) `[Cycle 03-29 H-6 added]`
* [62_ISeedKeyProvider 抽象分析 (ISeedKeyProvider Abstraction Analysis)](./Architecture_Documentation/62_ISeedKeyProvider_Abstraction_Analysis.md) `[Cycle 03-29 H-6 added]`
* [63_W2 Phase 2 校準架構 (W2 Phase 2 Calibration Architecture)](./Architecture_Documentation/63_W2_Phase2_Calibration_Architecture.md) `[Cycle 03-29 H-6 added]`
* [64_製造模式五循環 (Fabrication Pattern Five Cycle)](./Architecture_Documentation/64_Fabrication_Pattern_Five_Cycle.md) `[Cycle 03-29 H-6 added]`
* [65_齒輪仲裁者動態唯識映射 (Gear Arbiter Dynamic Buddhist Mapping)](./Architecture_Documentation/65_Gear_Arbiter_Dynamic_Buddhist_Mapping.md) `[Cycle 03-29 H-6 added]`
* [66_Shadow Counting 控制理論 (Shadow Counting Control Theory)](./Architecture_Documentation/66_Shadow_Counting_Control_Theory.md) `[Cycle 03-29 H-6 added]`
* [67_ServiceKey 幽靈型別理論 (ServiceKey Phantom Type Theory)](./Architecture_Documentation/67_ServiceKey_Phantom_Type_Theory.md) `[Cycle 03-29 H-6 added]`
* [68_MR-11 十大宣言作為架構選擇 (MR-11 Tenets As Architectural Choice)](./Architecture_Documentation/68_MR11_Tenets_As_Architectural_Choice.md) `[Cycle 03-29 H-6 added]`
* [69_Rule #63 L4 Config 啟動 (Rule #63 L4 Config Activates)](./Architecture_Documentation/69_Rule63_L4_Config_Activates.md) `[Cycle 03-29 H-6 added]`
* [70_Rules #64-#68 重跑策略修正揭露 (Rules 64-68 Rerun Policy Fix Disclosure)](./Architecture_Documentation/70_Rules_64_to_68_Rerun_Policy_Fix_Disclosure.md) `[Cycle 03-29 H-6 added]`
* [71_雙路徑 Config 傳播 (Two Path Config Propagation)](./Architecture_Documentation/71_Two_Path_Config_Propagation.md) `[Cycle 03-29 H-6 added]`
* [72_WIENER L2/L3 安全框架 (WIENER L2/L3 Safety Framework)](./Architecture_Documentation/72_WIENER_L2_L3_Safety_Framework.md) `[Cycle 03-29 H-6 added]`
* [73_Plan46 工具過濾 + Checkpoint Hook (Plan46 Tool Filtering & Checkpoint Hook)](./Architecture_Documentation/73_Plan46_Tool_Filtering_And_Checkpoint_Hook.md) `[Cycle 03-10 新增]`
* [74_Plan47 K-3 Wire-In (Plan47 K-3 Wire-In)](./Architecture_Documentation/74_Plan47_K3_Wire_In.md) `[Cycle 03-11 新增]`
* [75_Rules #71-#72 FR1/FR2 修正 (Rules 71/72 FR1/FR2 Amendments)](./Architecture_Documentation/75_Rules_71_72_FR1_FR2_Amendments.md) `[Cycle 03-29 H-6 added]`
* [76_合規框架 9.0.1 + 零容忍 (Compliance Framework 9.0.1 & Zero Tolerance)](./Architecture_Documentation/76_Compliance_Framework_9_0_1_And_Zero_Tolerance.md) `[Cycle 03-29 H-6 added]`
* [77_Tenet 10 Phase 6 路線圖 (Tenet 10 Phase 6 Roadmap)](./Architecture_Documentation/77_Tenet_10_Phase_6_Roadmap.md) `[Cycle 03-29 H-6 added]`
* [78_Rule #74 L1' Prime 代碼-文件同步 (Rule #74 L1' Prime Code-Doc Sync)](./Architecture_Documentation/78_Rule_74_L1_Prime_Code_Doc_Sync.md) `[Cycle 03-29 H-6 added]`
* [79_插件齒輪仲裁者 (Plugin Gear Arbiters)](./Architecture_Documentation/79_Plugin_Gear_Arbiters.md) `[Cycle 03-29 H-6 added]`

### 2. 核心組件深潛 (Agent Core Components Deep Dive)
*深入內核，研究具體技術機制與理論模型。*
* [00_核心哲學 (Core Philosophy)](./Agent_Core_Components_Deep_Dive/00_Core_Philosophy.md)
* [01_執行迴圈 (Execution Loop)](./Agent_Core_Components_Deep_Dive/01_Execution_Loop.md)
* [02_通訊介面 (Communication Interface)](./Agent_Core_Components_Deep_Dive/02_Communication_Interface.md)
* [03_安全層 (Security Layer)](./Agent_Core_Components_Deep_Dive/03_Security_Layer.md)
* [04_狀態管理器 (State Manager)](./Agent_Core_Components_Deep_Dive/04_State_Manager.md)
* [05_插件基礎設施整合 (Plugin Infrastructure Integration)](./Agent_Core_Components_Deep_Dive/05_Plugin_Infrastructure_Integration.md)
* [06_狀態持久化機制 (State Persistence Mechanism)](./Agent_Core_Components_Deep_Dive/06_State_Persistence_Mechanism.md)
* [07_安全斷路器 (Safety Circuit Breakers)](./Agent_Core_Components_Deep_Dive/07_Safety_Circuit_Breakers.md)
* [08_安全實作 (Safety Implementation)](./Agent_Core_Components_Deep_Dive/08_Safety_Implementation.md)
* [09_可觀測性與追蹤 (Observability and Tracing)](./Agent_Core_Components_Deep_Dive/09_Observability_and_Tracing.md)
* [10_上下文管理策略 (Context Management Strategy)](./Agent_Core_Components_Deep_Dive/10_Context_Management_Strategy.md)
* [11_插件運行時隔離 (Plugin Runtime Isolation)](./Agent_Core_Components_Deep_Dive/11_Plugin_Runtime_Isolation.md)
* [12_錯誤處理與自我修正 (Error Handling & Self Correction)](./Agent_Core_Components_Deep_Dive/12_Error_Handling_and_Self_Correction.md)
* [13_代理核心作為控制系統 (Agent Core as Control System)](./Agent_Core_Components_Deep_Dive/13_Agent_Core_as_Control_System.md)
* [14_代理核心哲學：五蘊 (Agent Core Philosophy: Five Aggregates)](./Agent_Core_Components_Deep_Dive/14_Agent_Core_Philosophy_Five_Aggregates.md)
* [15_蘊與協議橋接 (Skandha Protocol Bridge)](./Agent_Core_Components_Deep_Dive/15_Skandha_Protocol_Bridge.md) `[Cycle 02-11 新增]`
* [16_OpenStarry 標準協議 (OpenStarry Standard Protocol)](./Agent_Core_Components_Deep_Dive/16_OpenStarry_Standard_Protocol.md)

### 3. 專案結構與規範 (Project Structure and Conventions)
*定義物理佈局、源碼組織、開發流程與安裝規範。*
* [00_路線圖與里程碑 (Roadmap & Milestones)](./Project_Structure_and_Conventions/00_Roadmap_and_Milestones.md)
* [01_Monorepo 頂層結構 (Monorepo Top Level Structure)](./Project_Structure_and_Conventions/01_Monorepo_Top_Level_Structure.md)
* [02_核心源碼結構 (Core Source Code Structure)](./Project_Structure_and_Conventions/02_Core_Source_Code_Structure.md)
* [03_共享組件與 SDK 結構 (Shared & SDK Structure)](./Project_Structure_and_Conventions/03_Shared_and_SDK_Structure.md)
* [04_標準代理目錄解剖 (Standard Agent Directory Anatomy)](./Project_Structure_and_Conventions/04_Standard_Agent_Directory_Anatomy.md)
* [05_代理清單規範 (Agent Manifest Specification)](./Project_Structure_and_Conventions/05_Agent_Manifest_Specification.md)
* [06_插件目錄慣例 (Plugin Directory Conventions)](./Project_Structure_and_Conventions/06_Plugin_Directory_Conventions.md)
* [07_編碼與測試標準 (Coding & Testing Standards)](./Project_Structure_and_Conventions/07_Coding_and_Testing_Standards.md)
* [08_系統與專案運行時佈局 (System & Project Runtime Layouts)](./Project_Structure_and_Conventions/08_System_and_Project_Runtime_Layouts.md)
* [09_CLI 設計與管理命令 (CLI Design & Management Commands)](./Project_Structure_and_Conventions/09_CLI_Design_and_Management_Commands.md)
* [10_構建與發佈策略 (Build & Distribution Strategy)](./Project_Structure_and_Conventions/10_Build_and_Distribution_Strategy.md)
* [11_第三方插件安裝 (Third-Party Plugin Installation)](./Project_Structure_and_Conventions/11_Third_Party_Plugin_Installation.md)
* [12_能力注入機制 (Capabilities Injection Mechanism)](./Project_Structure_and_Conventions/12_Capabilities_Injection_Mechanism.md)
* [13_複合插件與依賴 (Composite Plugins & Dependencies)](./Project_Structure_and_Conventions/13_Composite_Plugins_and_Dependencies.md)
* [14_Markdown 技能規範 (Markdown Skill Specification)](./Project_Structure_and_Conventions/14_Markdown_Skill_Specification.md)
* [15_測試策略與基礎設施 (Testing Strategy & Infrastructure)](./Project_Structure_and_Conventions/15_Testing_Strategy_and_Infrastructure.md)
* [16_插件註冊與發佈 (Plugin Registry & Distribution)](./Project_Structure_and_Conventions/16_Plugin_Registry_and_Distribution.md)
* [17_開發者體驗與工具 (Developer Experience & Tooling)](./Project_Structure_and_Conventions/17_Developer_Experience_and_Tooling.md)
* [18_代理運行時配置 (Agent Runtime Configuration)](./Project_Structure_and_Conventions/18_Agent_Runtime_Configuration.md)
* [測試指南 (Testing Guide)](./Project_Structure_and_Conventions/Testing_Guide.md)

### 4. 插件基礎設施範例 (Plugin System Architecture)
*插件系統的具體應用、概念與規範。*
* [00_插件哲學：五蘊 (Plugin Philosophy: Five Aggregates)](./Plugin_System_Architecture/00_Plugin_Philosophy_Five_Aggregates.md)
* [01_MCP 插件範例 (MCP Plugin Example)](./Plugin_System_Architecture/01_MCP_Plugin_Example.md)
* [02_MCP 協議整合 (MCP Protocol Integration)](./Plugin_System_Architecture/02_MCP_Protocol_Integration.md)
* [03_開發者工具範例 (Developer Tools Example)](./Plugin_System_Architecture/03_Developer_Tools_Example.md)
* [04_網路交互範例 (Web Interaction Example)](./Plugin_System_Architecture/04_Web_Interaction_Example.md)
* [05_進階 UI 與設備範例 (Advanced UI & Device Example)](./Plugin_System_Architecture/05_Advanced_UI_And_Device_Example.md)
* [06_數據驗證範例 (Data Validation Example)](./Plugin_System_Architecture/06_Data_Validation_Example.md)

### 5. 實作範例與指南 (Implementation Examples)
*動手寫代碼，從案例學習實踐。*
* [上下文策略：滑動窗口 (Context Strategy: Sliding Window)](./Implementation_Examples/Context_Strategy_SlidingWindow.md)
* [開發者指南：插件遷移 (Developer Guide: Plugin Migration)](./Implementation_Examples/Developer_Guide_Plugin_Migration.md)
* [開發者指南：獨立執行 (Developer Guide: Standalone Execution)](./Implementation_Examples/Developer_Guide_Standalone_Execution.md)
* [OpenClaw 協調層 (OpenClaw Coordination Layer)](./Implementation_Examples/openclaw_Coordination_Layer.md)
* [OpenClaw UI 頻道適配器 (OpenClaw UI Channel Adapters)](./Implementation_Examples/openclaw_UI_Channel_Adapters.md)
* [OpenCode 代碼解釋器套件 (OpenCode Code Interpreter Suite)](./Implementation_Examples/opencode_Code_Interpreter_Suite.md)
* [Provider: Gemini 範例](./Implementation_Examples/Provider_Gemini_Example.md)
* [Tool: 代碼解釋器範例](./Implementation_Examples/Tool_CodeInterpreter_Example.md)
* [Tool: 讀取檔案範例](./Implementation_Examples/Tool_ReadFile_Example.md)
* [Transport: WebSocket 插件](./Implementation_Examples/Transport_Plugin_Websocket.md)
* [UI 插件範例 (UI Plugin Example)](./Implementation_Examples/UI_Plugin_Example.md)
* [USB 即插即用代理場景 (USB Plug-and-Play Agent Scenario)](./Implementation_Examples/USB_Plug_and_Play_Agent_Scenario.md)
* [擬人化痛覺機制範例 (Pain Mechanism Demo)](./Implementation_Examples/Pain_Mechanism_Demo.md)

### 6. 實作計畫 (Implementation Plans)
*工程實作的分波規格。*
* [Plan28_IVolition v1 + Safety Hardening](./Implementation_Plans/Plan28_IVolition_v1_Safety_Hardening.md)
* [Plan29_ILoopQualityMonitor + IConfidenceAuditor](./Implementation_Plans/Plan29_ILoopQualityMonitor_IConfidenceAuditor_SEC029.md)
* [Plan30_LoopQualityMonitor Layer 3](./Implementation_Plans/Plan30_LoopQualityMonitor_Layer3.md)
* [Plan31_AuditContext + ThresholdAuditor + Audit Trail](./Implementation_Plans/Plan31_AuditContext_ThresholdAuditor_AuditTrail.md)
* [Plan32_Tenet #7 絕對純淨 + Tenet #9 (Tenet7 Absolute Purity)](./Implementation_Plans/Plan32_Tenet7_Absolute_Purity.md) `[Cycle 02-8 新增]`
* [Plan34_.openstarry/ 專案級配置 (Project-Level Config)](./Implementation_Plans/Plan34_Project_Level_Config.md) `[Cycle 02-10 新增]`
* [Plan35_Context Summary + VedanaFn Wiring](./Implementation_Plans/Plan35_Context_Summary_VedanaFn.md) `[Cycle 02-11 新增]`
* [Plan46_Tool Filtering + K-3 SDK Framework Hook](./Implementation_Plans/Plan46_Implementation_Plan.md) `[Cycle 03-10 新增]` — v0.46.0-alpha
* [Plan47_K-3 Wire-In 5 MUST + HMAC Sync + MR-6](./Implementation_Plans/Plan47_Implementation_Plan.md) `[Cycle 03-11 新增]` — v0.47.0-alpha
* [Doc 73 Plan46 架構決策](./Architecture_Documentation/73_Plan46_Tool_Filtering_And_Checkpoint_Hook.md) `[Cycle 03-10 新增]`
* [Doc 74 Plan47 K-3 Wire-In 架構決策](./Architecture_Documentation/74_Plan47_K3_Wire_In.md) `[Cycle 03-11 新增]`

### 7. 技術規格與計畫綁定 (Technical Specifications & Plan Binding) `[Cycle 03-29 H-6 added]`
*Plan49+ 計畫綁定文件 (DSDX Master Ratification artefacts) 與既有技術規格。EN/TW 配對檔可在同一目錄找到。*
* [01_命令註冊表與發現 (Command Registry and Discovery)](./Technical_Specifications/01_Command_Registry_and_Discovery.md)
* [02_事件 Schema 與 Bus 協議 (Event Schema and Bus Protocol)](./Technical_Specifications/02_Event_Schema_and_Bus_Protocol.md)
* [03_插件介面定義 (Plugin Interface Definitions)](./Technical_Specifications/03_Plugin_Interface_Definitions.md)
* [04_上下文管理與記憶策略 (Context Management and Memory Strategy)](./Technical_Specifications/04_Context_Management_and_Memory_Strategy.md)
* [05_安全與沙箱協議 (Security and Sandboxing Protocol)](./Technical_Specifications/05_Security_and_Sandboxing_Protocol.md)
* [06_Inter-Agent MCP 協議 (Inter-Agent MCP Protocol)](./Technical_Specifications/06_Inter_Agent_MCP_Protocol.md)
* [07_管理區與編排器規格 (Management Zone and Orchestrator Spec)](./Technical_Specifications/07_Management_Zone_and_Orchestrator_Spec.md)
* [08_SecureStore 技術指南 (SecureStore Technical Guide)](./Technical_Specifications/08_SecureStore_Technical_Guide.md)
* [09_HMAC 種子驗證架構 (HMAC Seed Authentication Architecture)](./Technical_Specifications/09_HMAC_Seed_Authentication_Architecture.md)
* [10_微內核安全分析 (Microkernel Security Analysis)](./Technical_Specifications/10_Microkernel_Security_Analysis.md)
* [11_分段授權轉移架構 (Phased Authority Transfer Architecture)](./Technical_Specifications/11_Phased_Authority_Transfer_Architecture.md)
* [12_金鑰輪替設計理據 (Key Rotation Design Rationale)](./Technical_Specifications/12_Key_Rotation_Design_Rationale.md)
* [13_M4a 雙軌設計 (M4a Dual Track Design)](./Technical_Specifications/13_M4a_Dual_Track_Design.md)
* [14_Plan43 COND2 StateTracker 規格修正 (Plan43 COND2 StateTracker Spec Amendment)](./Technical_Specifications/14_Plan43_COND2_StateTracker_Spec_Amendment.md)
* [15_Phase 3 Shadow M4a 實作 (Phase 3 Shadow M4a Implementation)](./Technical_Specifications/15_Phase3_Shadow_M4a_Implementation.md)
* [16_HMAC 合規 (HMAC Compliance)](./Technical_Specifications/16_HMAC_Compliance.md)
* [17_HMAC 金鑰輪替架構 (HMAC Key Rotation Architecture)](./Technical_Specifications/17_HMAC_Key_Rotation_Architecture.md)
* [18_結構化日誌 (Structured Log)](./Technical_Specifications/18_Structured_Log.md)
* [19_Schema Drift 策略 (Schema Drift Policy)](./Technical_Specifications/19_Schema_Drift_Policy.md)
* [Plan49 MR-6 條件閘 (Plan49 MR-6 Conditional Gates)](./Technical_Specifications/Plan49_MR6_Conditional_Gates.md) `[Cycle 03-29 H-6 added]`
* [Plan50 Sigma Regime Binding](./Technical_Specifications/Plan50_Sigma_Regime_Binding.md) `[Cycle 03-29 H-6 added]`
* [Plan50 pushInput CP4 Invariant](./Technical_Specifications/Plan50_pushInput_CP4_Invariant.md) `[Cycle 03-29 H-6 added]`
* [Plan51 Zod Gate Binding](./Technical_Specifications/Plan51_Zod_Gate_Binding.md) (TW: [`.tw.md`](./Technical_Specifications/Plan51_Zod_Gate_Binding.tw.md)) `[Cycle 03-29 H-6 added]`
* [Plan52 pushInput Binding](./Technical_Specifications/Plan52_pushInput_Binding.md) (TW: [`.tw.md`](./Technical_Specifications/Plan52_pushInput_Binding.tw.md)) `[Cycle 03-29 H-6 added]`
* [Plan54 AC9 Binding](./Technical_Specifications/Plan54_AC9_Binding.md) (TW: [`.tw.md`](./Technical_Specifications/Plan54_AC9_Binding.tw.md)) `[Cycle 03-29 H-6 added]`
* [Plan56 D-30-4 Binding](./Technical_Specifications/Plan56_D30_4_Binding.md) (TW: [`.tw.md`](./Technical_Specifications/Plan56_D30_4_Binding.tw.md)) `[Cycle 03-29 H-6 added]`
* [Plan57 D-30-5 VasanaEngine Binding](./Technical_Specifications/Plan57_D30_5_VasanaEngine_Binding.md) (TW: [`.tw.md`](./Technical_Specifications/Plan57_D30_5_VasanaEngine_Binding.tw.md)) `[Cycle 03-29 H-6 added]`
* [Plan58 Mesh Binding](./Technical_Specifications/Plan58_Mesh_Binding.md) (TW: [`.tw.md`](./Technical_Specifications/Plan58_Mesh_Binding.tw.md)) `[Cycle 03-29 H-6 added]`
* [Plan59 API Runtime Binding](./Technical_Specifications/Plan59_API_Runtime_Binding.md) (TW: [`.tw.md`](./Technical_Specifications/Plan59_API_Runtime_Binding.tw.md)) `[Cycle 03-29 H-6 added]`
* [Plan60 Blackboard Alaya Binding](./Technical_Specifications/Plan60_Blackboard_Alaya_Binding.md) (TW: [`.tw.md`](./Technical_Specifications/Plan60_Blackboard_Alaya_Binding.tw.md)) `[Cycle 03-29 H-6 added]`
* Plan53 / Plan55 sunset: 見 [`Technical_Specifications/_archive/`](./Technical_Specifications/_archive/) (Plan53 cycle 03-25 sunset; Plan55 cycle 03-21 R3 binary final sunset — archive doc created cycle 03-29 H-5)

### 8. 實作參照 (Implementation Reference) `[Cycle 03-29 H-6 added]`
*Implementation Reference 平行 EN/TW 雙語匯整（cycle 03-26 結構搬遷至 canonical 單一 source-of-truth；12+12 sibling pair 全部以 Rule #78 §78.7 codified）。*
* EN corpus: [`Implementation_Reference/EN/`](./Implementation_Reference/EN/) — 12 sibling-paired English markdown files
* TW corpus: [`Implementation_Reference/TW/`](./Implementation_Reference/TW/) — 12 sibling-paired Traditional Chinese markdown files
* Sibling-naming 機制驗證工具: `agent_dev/openstarry/tools/sibling-naming-check.mjs` (retargeted cycle 03-28 task #191 至 canonical 單一目錄)

---

## 🛠️ 快速開始

**新手推薦**：**[GETTING_STARTED.md](./GETTING_STARTED.md)** — 10 分鐘上手，從配置到寫出第一個 Plugin `[Cycle 02-10 新增]` | 版本: v0.57.5-alpha (cycle 03-28 doc-only) / v0.57.6-alpha (cycle 03-29 hygiene; this) `[Cycle 03-29 H-7 footer refresh]` | 測試: 3093 passed, 3 skipped (cycle 03-27 v0.57.4 post task #191 Phase 0 baseline) | 合規: cycle 03-26 起 canonical 單一 source-of-truth (Master directive 2026-05-08) + ε-surface Δ=0 9-cycle preservation streak (cycle 03-21 → 03-29; longest on record) + §75.X 21st consecutive (reconciled basis)

進階參考：**[Developer_Guide_Standalone_Execution.md](./Implementation_Examples/Developer_Guide_Standalone_Execution.md)** 運行您的第一個 Agent。