# Findings — GitHub Repo Knowledge Base

## Workspace Purpose
这个 workspace 用于沉淀前沿 GitHub repo 的研究笔记，重点服务于：
- 理解 repo 的目的、实现和使用方式。
- 提高使用 agent tools、Claude Code、Codex 的生产效率。
- 按工具类型分类形成可检索知识库。

## Documentation Contract
每个 repo 使用一个独立 Markdown 文件：
- 文件名与 repo 名称一致：`<repo-name>.md`。
- 存放在按工具类型命名的文件夹中。
- 文档必须包含原始 repo 链接。
- 面向实践：除了介绍，也要说明如何在 Claude Code 和 Codex 中使用。

## Suggested Categories
| Folder | Use For |
|---|---|
| `coding-agents/` | 代码生成、代码审查、自动修复、软件工程 agent。 |
| `agent-frameworks/` | 多 agent 编排、agent runtime、agent SDK/framework。 |
| `mcp-tools/` | Model Context Protocol server/client/tooling。 |
| `llm-app-frameworks/` | RAG、LLM app、workflow、prompt orchestration 框架。 |
| `ai-gateways/` | AI API 网关、中转、账号池、额度分发、统一鉴权/计费/调度平台。 |
| `browser-automation/` | Browser agent、网页自动化、爬取与交互。 |
| `eval-observability/` | LLM eval、tracing、observability、benchmark。 |
| `devtools/` | CLI、IDE、工程效率、脚手架、开发辅助工具。 |
| `data-tools/` | 数据处理、向量库、检索、ETL 相关工具。 |
| `research-projects/` | 偏论文/实验性质、尚未产品化的前沿项目。 |
| `uncategorized/` | 暂时无法明确分类的 repo，后续再迁移。 |

## Future Research Notes
## Repo Notes

### PolyArch/humanize
- Repo: https://github.com/PolyArch/humanize
- Category chosen: `coding-agents`
- Rationale: Humanize directly orchestrates Claude Code implementation and Codex review for software engineering tasks. It is closer to a coding agent workflow/tool than a generic devtool or LLM app framework.
- Current README version checked: `1.16.0`
- Core concept: RLCR, or Ralph-Loop with Codex Review, creates a two-phase loop: Claude implementation reviewed by Codex summaries, then Codex code review against a base branch.
- Main commands: `/humanize:gen-idea`, `/humanize:gen-plan`, `/humanize:refine-plan`, `/humanize:start-rlcr-loop`, `/humanize:ask-codex`, `/humanize:ask-gemini`, `/humanize:cancel-rlcr-loop`.
- Important implementation areas: `.claude-plugin/`, `commands/`, `scripts/`, `hooks/`, `skills/`, `prompt-template/`, `tests/`.
- Practical caveat: best for planned medium/large code work; heavy for small one-off edits.

### OthmanAdi/planning-with-files
- Repo: https://github.com/OthmanAdi/planning-with-files
- Category chosen: `coding-agents`
- Rationale: Planning with Files is a cross-agent skill/plugin that changes how coding agents such as Claude Code and Codex plan, remember, recover context, and decide when work is complete. It is closer to an agent workflow tool than a generic devtool.
- Current version checked: `v2.37.0`, latest release published 2026-05-05.
- Core concept: use `task_plan.md`, `findings.md`, and `progress.md` as persistent filesystem memory for complex agent tasks.
- Main capabilities: three-file planning templates, lifecycle hooks, session catchup after `/clear`, parallel plan isolation under `.planning/`, active plan switching/resolution, Stop hook completion checks, and SHA-256 plan attestation via `/plan-attest`.
- Key implementation areas: `skills/planning-with-files/SKILL.md`, `templates/`, `scripts/`, `commands/`, `.claude-plugin/`, `.codex/`, platform-specific IDE folders, and `tests/`.
- Practical caveat: high leverage for multi-step or long-context work; overhead is unnecessary for simple one-off edits.

### Wei-Shaw/sub2api
- Repo: https://github.com/Wei-Shaw/sub2api/tree/main
- Category chosen: `ai-gateways`
- Rationale: Sub2API is a deployable AI API gateway/SaaS platform that manages upstream AI subscriptions, quota distribution, authentication, billing, scheduling, request forwarding, and a Vue admin console. It is broader than a small devtool and is not a coding agent itself.
- Current main commit checked from shallow clone: `f5bd25bea045e728846b38bf18080ffa48d133c6`.
- Visible stack: Go backend with Gin, Ent, Wire, PostgreSQL, Redis, h2c support, payment providers, model pricing resources, plus Vue 3/Vite/Tailwind/Pinia frontend.
- Initial deployment modes: one-click binary install, Docker Compose with auto setup, and source build with embedded frontend.
- First implementation note: `backend/cmd/server/main.go` handles version/setup flags, web setup wizard or Docker auto setup, then initializes the main server through Wire.
- Path correction: guessed `backend/internal/server/routes/routes.go` does not exist; route files need locating under the actual `backend/internal/server/routes/` directory.
- Gateway routes live in `backend/internal/server/routes/gateway.go`: Anthropic-compatible `/v1/messages`, `/v1/responses`, OpenAI-compatible `/v1/chat/completions`, `/v1beta` Gemini, Codex aliases, and Antigravity-only routes.
- Gateway scheduling centers on `GatewayService`: parse request once, derive sticky session hash, resolve group/platform, apply model-routing rules, filter schedulable accounts, check quota/window cost/RPM, then sort by priority/load/last used and acquire Redis-backed concurrency slots.
- Supported platforms/account types are defined in `backend/internal/domain/constants.go`: platforms include Anthropic, OpenAI, Gemini, Antigravity; account types include OAuth, setup-token, API key, upstream passthrough, Bedrock, and Google service account.
- Config surface includes h2c, CORS/CSP/security URL allowlist, response header filtering, billing circuit breaker, image concurrency, scheduler waiting/loads/outbox, TLS fingerprinting, Gemini OAuth/quota simulation, and Docker auto-setup secrets.
- Created `ai-gateways/sub2api.md` as the final learning document.

## Future Research Notes
第一个 repo 已归档。后续继续按 repo 链接创建分类文档。

### can1357/oh-my-pi
- Repo: https://github.com/can1357/oh-my-pi
- Category candidate: `coding-agents`
- Rationale: oh-my-pi / `omp` is positioned as a terminal-first AI coding agent with built-in code editing, LSP, debugger, subagents, browser/web search, model-provider routing, and SDK/RPC/ACP entry points.
- Official README describes installation via `curl -fsSL https://omp.sh/install | sh`, Bun global install `bun install -g @oh-my-pi/pi-coding-agent`, Windows PowerShell installer, and mise-pinned versions.
- Initial feature set observed from README: hash-anchored edits, summarized read/search, LSP, DAP debugger, subagents, web_search, browser, Hindsight memory, ACP/editor integration, Node SDK, RPC mode, and multi-provider routing.
- Source-level clone checked at commit `159f2d8fb1ac567e4db742171c7a126b07d5d5eb`; npm `@oh-my-pi/pi-coding-agent` latest dist-tag resolved to `15.5.3` during this session.
- Monorepo package map: `pi-coding-agent` (CLI/SDK), `pi-agent-core` (runtime), `pi-ai` (multi-provider client), `pi-tui` (terminal UI), `pi-natives` (Rust/N-API search/shell/AST/PTY/etc.), `hashline` (content-hash patch language), `omp-stats`, and `swarm-extension`.
- Main runtime modes: interactive TUI, one-shot print (`omp -p`), JSON/text output modes, JSONL RPC (`omp --mode rpc`), RPC UI, and ACP editor mode (`omp acp`).
- Core config roots: native `.omp` has highest provider priority, and OMP can inherit or interoperate with `.claude`, `.codex`, `.gemini`, Cursor/Windsurf/Cline/Copilot/VScode-style rule/config formats.
- Built-in tools enumerated in source include `read`, `bash`, `edit`, `write`, `search`, `find`, `ast_grep`, `ast_edit`, `eval`, `lsp`, `debug`, `task`, `irc`, `web_search`, `browser`, `github`, `recipe`, `todo_write`, memory tools, image/mermaid/calculator helpers, and hidden `resolve`/`yield`/review helpers.
- Safety note: tool approval modes are `always-ask`, `write`, and `yolo`; README/CLI expose `--auto-approve`/`--yolo`, but production use should prefer prompting for exec/destructive tools.

- Created `coding-agents/oh-my-pi.md` as the final usage manual.
- Final positioning: keep `can1357/oh-my-pi` under `coding-agents`; it is an end-user coding agent plus embeddable runtime, not just a devtool. It can also touch `agent-frameworks` because of SDK/RPC/ACP/subagents, but its primary user-facing artifact is the `omp` coding agent CLI.
- Practical usage guidance: use `omp` as an independent terminal agent, `omp -p` as a one-shot reviewer/researcher alongside Claude Code/Codex, `--mode rpc`/SDK/ACP for embedding, and conservative `--approval-mode always-ask` or `write` for real repositories.
