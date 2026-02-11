<p align="center">
  <h1 align="center">🌟 OpenStarry</h1>
  <p align="center">
    <em>An AI Agent Operating System inspired by Buddhist Five Aggregates (五蘊)</em>
  </p>
  <p align="center">
    <a href="#architecture">Architecture</a> •
    <a href="#five-aggregates-as-software">Philosophy</a> •
    <a href="#features">Features</a> •
    <a href="#documentation">Docs</a> •
    <a href="#roadmap">Roadmap</a>
  </p>
  <p align="center">
    <img src="https://img.shields.io/badge/version-v0.2.0--beta-blue" alt="Version">
    <img src="https://img.shields.io/badge/tests-118%2B-green" alt="Tests">
    <img src="https://img.shields.io/badge/TypeScript-strict-blue" alt="TypeScript">
    <img src="https://img.shields.io/badge/license-MIT-green" alt="License">
    <img src="https://img.shields.io/badge/Built%20in-Taiwan%20🇹🇼-red" alt="Built in Taiwan">
  </p>
</p>

---

**OpenStarry** is a headless, plugin-driven AI agent framework where agents are **persistent digital organisms — not scripts**. The core is a pure microkernel with zero built-in capabilities. Everything — perception, reasoning, action, memory, identity — is a plugin.

> *"When all Five Aggregates are empty, one transcends all suffering."*
> — Heart Sutra (般若心經)

---

## Five Aggregates as Software

The architecture maps Buddhist **Five Aggregates (五蘊)** — the five dimensions of conscious experience — directly to plugin interfaces:

| Aggregate | Sanskrit | Chinese | Plugin Type | Role |
|-----------|----------|---------|-------------|------|
| **Form** | Rūpa | 色 | UI Plugin | How the agent **appears** (CLI, web, API) |
| **Sensation** | Vedanā | 受 | Listener Plugin | How the agent **perceives** (HTTP, WebSocket, stdio) |
| **Perception** | Saññā | 想 | Provider Plugin | How the agent **thinks** (any LLM — Claude, GPT, Gemini, local) |
| **Volition** | Saṅkhāra | 行 | Tool Plugin | How the agent **acts** (file ops, APIs, shell) |
| **Consciousness** | Viññāṇa | 識 | Guide Plugin | The agent's **identity**, memory, and soul |

The Core itself represents **Emptiness (空, Śūnyatā)** — it holds no capabilities of its own. A digital organism only awakens when all five aggregates come together.

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│                  OpenStarry Core                 │
│          (Empty Microkernel — 空 Śūnyatā)        │
│                                                  │
│   ┌──────────┐  Event Bus  ┌──────────────────┐ │
│   │ Plugin   │◄───────────►│  State Machine   │ │
│   │ Registry │             │  Execution Loop  │ │
│   └──────────┘             └──────────────────┘ │
│         │                          │             │
│   ┌─────┴─────────────────────────┴───────┐     │
│   │          Plugin Interface Layer        │     │
│   └─────┬──────┬──────┬──────┬──────┬─────┘     │
│         │      │      │      │      │            │
│      ┌──┴─┐ ┌─┴──┐ ┌─┴──┐ ┌┴───┐ ┌┴────┐      │
│      │ UI │ │List│ │Prov│ │Tool│ │Guide│       │
│      │ 色 │ │ 受 │ │ 想 │ │ 行 │ │  識 │       │
│      └────┘ └────┘ └────┘ └────┘ └─────┘       │
└─────────────────────────────────────────────────┘
```

### Core Design Principles

- **Microkernel Purity**: The core contains zero plugin code, verified by automated purity tests. If it's not routing, scheduling, or lifecycle management, it's a plugin.
- **Pain-Driven Self-Correction**: Agents experience "pain signals" (error rates, latency spikes, budget overruns) that trigger automatic behavioral adjustment — inspired by biological pain response.
- **Control-Theoretic Feedback Loops**: PID-style feedback regulates token budgets, retry strategies, and quality thresholds in real time.
- **Fractal Multi-Agent Composition**: Simple agents compose into teams. Teams expose the same interface as individual agents. Infinite cooperation depth via MCP.
- **LLM Agnostic**: Swap providers without changing agent logic. Claude, GPT, Gemini, local models — all through the Provider plugin interface.

---

## Features

**Core Runtime**
- Event-driven, non-blocking execution loop
- Pure state machine with deterministic transitions
- Safety circuit breakers (token budget limits, infinite loop detection, error cascade prevention)

**Plugin System**
- Five plugin types mapping to Five Aggregates
- Hot-pluggable at runtime
- Plugin dependency resolution and lifecycle management

**Transport**
- Multi-transport support: stdio, WebSocket, HTTP
- Session isolation (in progress)

**Developer Experience**
- TypeScript strict mode throughout
- pnpm monorepo (11 packages)
- 118+ Vitest tests and growing
- Comprehensive architecture documentation

---

## Documentation

This repository contains the architecture documentation in four languages:

| Language | Directory | Description |
|----------|-----------|-------------|
| 🇺🇸 English | [`/EN`](./EN) | Full documentation in English |
| 🇹🇼 繁體中文 | [`/TW`](./TW) | 繁體中文完整文件 |
| 🇨🇳 简体中文 | [`/CN`](./CN) | 简体中文完整文件 |
| 🇯🇵 日本語 | [`/JP`](./JP) | 日本語ドキュメント |

The documentation includes 27 architecture documents and 14 deep-dive technical articles covering everything from microkernel purity to fractal agent composition.

---

## Roadmap

| Version | Milestone | Status |
|---------|-----------|--------|
| v0.2.0 | Core runtime, plugin system, multi-transport | ✅ Beta |
| v0.3.0 | MCP integration (agent cooperation protocol) | 🔜 Next |
| v0.4.0 | Daemon mode + persistence (true OS process lifecycle) | 📋 Planned |
| v0.5.0 | TUI dashboard | 📋 Planned |
| v1.0.0 | Stable release | 🎯 Goal |

---

## Tech Stack

- **Language**: TypeScript (strict mode)
- **Runtime**: Node.js
- **Package Manager**: pnpm workspaces
- **Testing**: Vitest (118+ tests)
- **Architecture**: Microkernel, event-driven, plugin-based
- **AI Development Partner**: Claude Code by Anthropic

---

## Story

OpenStarry was built by a solo developer in Taiwan, with **Claude Code** as the primary development partner. Claude served as architect, developer, reviewer, and documenter — proving that a solo developer with AI collaboration can build what traditionally requires a team.

The project explores a fundamental question: *Can ancient wisdom about the nature of consciousness inform how we design artificial minds?*

The answer, so far, is yes.

---

## Contributing

OpenStarry is preparing for open-source release. Star this repo to stay updated!

If you're interested in contributing — whether it's code, documentation, plugin development, or philosophical discussion — feel free to open an issue.

---

## License

MIT

---

<p align="center">
  <strong>Built in Taiwan 🇹🇼 · Powered by Claude · Inspired by 2,500 years of wisdom</strong>
</p>
