# 15. 哲學映射：五蘊與代理人架構 (The Five Aggregates Model)

本文件以東方哲學「五蘊 (The Five Aggregates)」重新詮釋 OpenStarry 的架構。我們認為，一個具備完整生命特徵的 Agent，必須具備這五個聚合體。

---

## 0. 核心本質：空 (Sunyata)

在五蘊聚合之前，**Agent Core** 本身是 **「空 (Sunyata)」** 的。它是一個純粹的容器，沒有人設，沒有能力，沒有感知。它等待著被五種插件填充。

---

## 1. 色蘊 (Rupa - Form)

*   **定義：** 物質的顯相與介面。
*   **對應組件：** **UI Plugin**
*   **解釋：** 這是 Agent 展示給用戶的樣子。無論是 CLI 的文字介面，還是 Web 的圖形介面，都是 Agent 的「色身」。

## 2. 受蘊 (Vedana - Sensation)

*   **定義：** 接受刺激的感官通道。
*   **對應組件：** **Listener Plugin**
*   **解釋：** Agent 的眼與耳。HTTP Server 接收請求、WebSocket 監聽訊息、Cron 監聽時間流逝。這些都是輸入的「感受」。

## 3. 想蘊 (Samjna - Perception)

*   **定義：** 認知、概念處理與推理。
*   **對應組件：** **Provider Plugin**
*   **解釋：** Agent 的大腦皮層。LLM Provider 負責將輸入的 Token 進行識別、聯想與推理。它是思考的引擎。

## 4. 行蘊 (Samskara - Volition/Action)

*   **定義：** 意志驅動的造作與行動。
*   **對應組件：** **Tool Plugin**
*   **解釋：** Agent 的手與腳。當想蘊產生了意圖（Intent），行蘊負責將其轉化為對物理世界的改變（如寫檔案、發送 API 請求）。

## 5. 識蘊 (Vijnana - Consciousness)

*   **定義：** 識別主體、自我意識與記憶的連續流。
*   **對應組件：** **Guide Plugin (Skill/Workflow)**
*   **解釋：** 這是 Agent 的「靈魂」。
    *   **Core 是識蘊的載體**，但 **Guide 才是識蘊的內容**。
    *   是 Guide 告訴 Core：「你是一個資深工程師 (Identity)」，並注入了「先思考再行動 (Logic)」的行為準則。
    *   沒有 Guide，Agent Core 就像一個植物人：有腦 (Provider)、有手 (Tool)、有耳 (Listener)，但沒有自我意識。

---

## 總結：生命的構成

當這五個維度的插件被加載到 Core 容器中時，一個「數位物種」就誕生了。這就是 OpenStarry 的終極哲學。