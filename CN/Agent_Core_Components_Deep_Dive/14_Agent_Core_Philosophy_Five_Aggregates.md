# 15. 哲学映射：五蕴与代理人架构 (The Five Aggregates Model)

本文档以东方哲学「五蕴 (The Five Aggregates)」重新诠释 OpenStarry 的架构。我们认为，一个具备完整生命特征的 Agent，必须具备这五个聚合体。

---

## 0. 核心本质：空 (Sunyata)

在五蕴聚合之前，**Agent Core** 本身是 **「空 (Sunyata)」** 的。它是一个纯粹的容器，没有人设，没有能力，没有感知。它等待着被五种插件填充。

---

## 1. 色蕴 (Rupa - Form)

*   **定义：** 物质的显相与接口。
*   **对应组件：** **UI Plugin**
*   **解释：** 这是 Agent 展示给用户的样子。无论是 CLI 的文字界面，还是 Web 的图形界面，都是 Agent 的「色身」。

## 2. 受蕴 (Vedana - Sensation)

*   **定义：** 接受刺激的感官通道。
*   **对应组件：** **Listener Plugin**
*   **解释：** Agent 的眼与耳。HTTP Server 接收请求、WebSocket 监听信息、Cron 监听时间流逝。这些都是输入的「感受」。

## 3. 想蕴 (Samjna - Perception)

*   **定义：** 认知、概念处理与推理。
*   **对应组件：** **Provider Plugin**
*   **解释：** Agent 的大脑皮层。LLM Provider 负责将输入的 Token 进行识别、联想与推理。它是思考的引擎。

## 4. 行蕴 (Samskara - Volition/Action)

*   **定义：** 意志驱动的造作与行动。
*   **对应组件：** **Tool Plugin**
*   **解释：** Agent 的手与脚。当想蕴产生了意图（Intent），行蕴负责将其转化为对物理世界的改变（如写文件、发送 API 请求）。

## 5. 识蕴 (Vijnana - Consciousness)

*   **定义：** 识别主体、自我意识与记忆的连续流。
*   **对应组件：** **Guide Plugin (Skill/Workflow)**
*   **解释：** 这是 Agent 的「灵魂」。
    *   **Core 是识蕴的载体**，但 **Guide 才是识蕴的内容**。
    *   是 Guide 告诉 Core：「你是一个资深工程师 (Identity)」，并注入了「先思考再行动 (Logic)」的行为准则。
    *   没有 Guide，Agent Core 就像一个植物人：有脑 (Provider)、有手 (Tool)、有耳 (Listener)，但没有自我意识。

---

## 总结：生命的构成

当这五个维度的插件被加载到 Core 容器中时，一个「数字物种」就诞生了。这就是 OpenStarry 的终极哲学。
