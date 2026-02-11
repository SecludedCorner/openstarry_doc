# 03. Agent Design & Template Service

This document details the "Design Subsystem" within the OpenStarry coordination layer. It is responsible for managing the "Genetic Blueprints" of Agents, namely the Templates.

## Core Positioning: Dynamic Design Workshop

In the OpenStarry architecture, **the Agent Core is the executor, the Orchestrator is the manager, and this service is the "Creator's" workbench.**

It is not just a static database of JSON files, but a **runtime service** that allows humans or high-level agents (such as the Master Agent) to dynamically create, modify, and instantiate new Agent roles via an API.

---

## Functional Responsibilities

### 1. Template Management
Responsible for storing and version-controlling all Agent Templates. A Template defines the immutable properties of an Agent:
*   **System Prompt:** Core personality and instructions.
*   **Plugin Manifest:** A list of plugins to be loaded (capability manifest).
*   **Default Configuration:** Default parameter configurations (e.g., LLM model, temperature).

### 2. Design Interface
Provides a set of standard APIs for external systems to invoke for design work:
*   `POST /templates`: Create a new template based on requirements.
*   `GET /templates/{id}`: Retrieve template details (used by the Daemon when starting an Agent).
*   `PUT /templates/{id}/plugins`: Add new capabilities to an existing template.

---

## Human-AI Collaborative Design Process

We promote an **"AI-Assisted AI Design"** model:

1.  **Human Intent:** An administrator tells the Master Agent: "I need an assistant specialized in log analysis."
2.  **Master Agent Derivation:** The Master Agent's LLM analyzes the requirement and derives that the assistant needs the `fs:read` tool and a `log-analysis-plugin`.
3.  **Invoke Design Service:** The Master Agent calls the `AgentDesignerTool` and sends an API request to this service.
4.  **Template Generation:** This service validates the request, generates a new `LogAnalyst_Template_v1`, and stores it.
5.  **Instantiation:** The Master Agent then calls the Daemon to spawn a working Agent instance based on the new template.

This design empowers OpenStarry with self-extension and evolutionary capabilities.