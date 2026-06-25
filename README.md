<div align="center">

# Agent Tools Docs

**A curated research handbook for AI agents, coding agents, MCP tools, AI gateways, and other frontier open-source AI tooling.**

<p>
  <a href="https://github.com/zengchang233/agent-tools-docs/stargazers">
    <img alt="GitHub stars" src="https://img.shields.io/github/stars/zengchang233/agent-tools-docs?style=for-the-badge&logo=github&label=Stars&color=ffd700">
  </a>
  <img alt="Visitors" src="https://api.visitorbadge.io/api/visitors?path=zengchang233%2Fagent-tools-docs&label=Visitors&countColor=%23263759&style=for-the-badge">
  <a href="https://github.com/zengchang233/agent-tools-docs/commits/main">
    <img alt="Last commit" src="https://img.shields.io/github/last-commit/zengchang233/agent-tools-docs?style=for-the-badge&logo=git&label=Last%20Commit">
  </a>
  <img alt="Markdown docs" src="https://img.shields.io/badge/Docs-Markdown-2ea44f?style=for-the-badge&logo=markdown">
</p>

**English** · [简体中文](README.zh-CN.md)

Turn interesting AI-tool repositories from “I skimmed the README” into structured notes that can be used directly in Claude Code, Codex, and real engineering workflows.

</div>

---

## What is this repository?

`agent-tools-docs` is a long-term knowledge base for studying frontier open-source AI tooling. Each note focuses on:

- **Positioning**: what problem the project solves, who it is for, and when not to use it;
- **Implementation**: core modules, architecture, execution flow, and ecosystem dependencies;
- **Getting started**: installation, configuration, common commands, and a minimal usable workflow;
- **Agent workflows**: how to combine the tool with Claude Code, Codex, MCP, CI, and code review;
- **Productivity guidance**: realistic use cases, risk boundaries, alternatives, and maintenance notes.

This is not just a bookmark list or a README translation collection. The goal is to turn each repository into a reusable technical guide and usage manual.

> Note: most long-form project notes are currently written in Chinese, with this English README serving as the default entry point.

## Documentation Index

### Coding Agents

| Document | Topic | Best for |
|---|---|---|
| [Claude Code usage and efficient workflows](coding-agents/claude-code-usage.md) | Claude Code practical workflow | Using Claude Code as a primary development agent |
| [Codex usage and efficient workflows](coding-agents/codex-usage.md) | OpenAI Codex CLI / Codex | Understanding Codex execution, review, and parallel-agent workflows |
| [planning-with-files](coding-agents/planning-with-files.md) | File-based planning and persistent task memory | Preventing context loss and goal drift during long-running agent tasks |
| [humanize](coding-agents/humanize.md) | Claude implementation + Codex review loop | Building traceable multi-round AI coding workflows |
| [oh-my-pi](coding-agents/oh-my-pi.md) | Terminal AI coding agent / runtime | Exploring a full agent harness with LSP, DAP, subagents, RPC, and SDK support |
| [Zed, OMP, agent harnesses, and LLMs](coding-agents/zed-omp-agent-harness-llm.md) | Zed External Agents / OMP / ACP model routing | Understanding how Zed, OMP profiles, agent harnesses, model roles, and LLM providers fit together |
| [rmux](coding-agents/rmux.md) | Rust terminal multiplexer and typed SDK for agent workflows | Keeping Claude Code, Codex, long-running shells, and browser-shared terminal sessions alive and scriptable |
| [warp](coding-agents/warp.md) | Agentic development environment / terminal for local and cloud agents | Using Warp as a terminal-first workspace for Warp Agent, Claude Code, Codex, MCP, and Oz cloud agents |
| [paseo](coding-agents/paseo.md) | Self-hosted coding agent orchestrator with mobile, desktop, web, and CLI clients | Running Claude Code, Codex, Copilot, OpenCode, Pi, and 30+ ACP agents on your own machines with remote, multi-agent, and worktree workflows |

### AI Gateways

| Document | Topic | Best for |
|---|---|---|
| [sub2api](ai-gateways/sub2api.md) | AI API gateway and account-pool management | Studying quota distribution, routing, billing, and gateway design for Claude / OpenAI / Gemini-style upstreams |

## How to use this repository

### If you are a reader

1. Start from the **Documentation Index** and choose the tool category you care about;
2. Read the opening summary and the “suitable / not suitable” section of each note;
3. If you plan to try the tool, continue with installation, configuration, Claude Code / Codex workflows, and risk notes;
4. If you plan to build on top of it, focus on architecture, implementation breakdown, and key files/directories.

### If you want to add a repository

If you contribute with Claude Code, Codex, or another coding agent, read [`AGENTS.md`](AGENTS.md) first. It captures the expected research process, writing style, category rules, and public-vs-local file boundaries for this repository.

Provide one or more GitHub repository links. The analysis workflow is:

1. Verify the official repository, README, docs, and version information;
2. Classify the project and choose the right category directory;
3. Create a standalone Markdown document at `<category>/<repo-name>.md`;
4. Summarize the project purpose, implementation, setup, Claude Code / Codex workflows, use cases, and risks;
5. Keep intermediate research notes local, and publish only the polished category document plus README index updates.

## Document format

A typical repository analysis includes:

- One-sentence summary
- Project purpose and core problem
- Core capabilities
- Implementation / architecture breakdown
- Installation and quick start
- How to use it with Claude Code
- How to use it with Codex
- Typical productivity scenarios
- Suitable / not suitable
- Positioning among similar tools
- Key files and directories
- Learning and adoption advice
- Sources

The reusable template lives at [`_templates/repo-analysis-template.md`](_templates/repo-analysis-template.md).

## Categories

| Category | Description |
|---|---|
| `coding-agents/` | Terminal, IDE, or plugin-based AI coding agents, workflows, and review tools |
| `agent-frameworks/` | Frameworks for building agent apps, multi-agent systems, and workflow runtimes |
| `mcp-tools/` | MCP servers, MCP clients, and MCP ecosystem tools |
| `llm-app-frameworks/` | LLM application frameworks, RAG, prompt tooling, and workflow orchestration |
| `ai-gateways/` | AI API gateways, relays, account pools, billing, and model routing platforms |
| `browser-automation/` | Browser control, web automation, visual agents, and interaction tooling |
| `eval-observability/` | Evaluation, observability, tracing, replay, and quality monitoring tools |
| `devtools/` | General developer productivity tools |
| `data-tools/` | Data processing, retrieval, indexing, crawling, and knowledge-base tools |
| `research-projects/` | Papers, experimental projects, and research prototypes |
| `uncategorized/` | Projects that are not classified yet or need further investigation |

## Maintenance notes

- Public-facing knowledge should live in category documents such as `coding-agents/*.md` and `ai-gateways/*.md`;
- Local planning memory, scratch findings, and session progress logs should stay ignored in `.planning/` or other local-only files;
- Stars are generated dynamically through GitHub / Shields, and the visitor counter is provided by a third-party badge service. They are intended as public display indicators.

---

<div align="center">

If this knowledge base is useful to you, consider giving it a star or sending more AI-tool repositories worth studying.

</div>
