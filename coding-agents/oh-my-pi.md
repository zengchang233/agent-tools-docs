# oh-my-pi

> Repo: https://github.com/can1357/oh-my-pi  
> Category: `coding-agents`  
> Last checked: 2026-05-27  
> Source snapshot: main branch shallow clone at `159f2d8fb1ac567e4db742171c7a126b07d5d5eb`; npm `@oh-my-pi/pi-coding-agent` latest dist-tag checked as `15.5.3`.

## 1. 一句话总结
oh-my-pi 是一个以 `omp` 命令提供的终端 AI coding agent：它把代码编辑、文件/URL 读取、LSP、DAP 调试器、子代理、浏览器、Web 搜索、MCP、记忆、SDK/RPC/ACP 集成到同一个 coding harness 里。它源自 Mario Zechner 的 Pi，但定位更像“带 IDE 能力的 Claude Code / Codex CLI 替代或补充”。

## 2. 项目目的与核心问题
- 目标用户：重度使用终端 AI coding agent 的开发者、希望把 agent 嵌入编辑器/自研平台的工具开发者、需要多模型/多 provider 路由的团队。
- 解决的问题：传统 coding agent 往往只有 shell + file edit，缺少 IDE 语义、真实调试器、稳定编辑格式、子代理隔离、Web/浏览器、长期记忆、插件和嵌入式协议；oh-my-pi 试图把这些能力做成开箱即用的一体化 harness。
- 为什么值得关注：它不是单一模型 wrapper，而是完整 agent runtime：有 TypeScript CLI/SDK、Rust native 加速层、模型/provider registry、工具发现、MCP、LSP/DAP、TUI、会话存储、扩展系统，以及用于编辑可靠性的 Hashline patch 格式。

## 3. 核心能力
- **终端 TUI coding agent**：默认运行 `omp` 进入交互式终端界面，工具调用以卡片形式展示，编辑可预览、可确认。
- **多入口运行**：交互式 TUI、一次性 `omp -p`、文本/JSON 输出、`omp --mode rpc` JSONL 协议、`omp --mode rpc-ui`、`omp acp` 编辑器协议。
- **强工具箱**：`read`、`write`、`edit`、`bash`、`search`、`find`、`ast_grep`、`ast_edit`、`eval`、`lsp`、`debug`、`task`、`irc`、`web_search`、`browser`、`github`、`recipe`、`todo_write`、`ssh`、`inspect_image`、`render_mermaid`、memory tools 等。
- **Hashline 编辑**：用内容 hash/锚点描述改动，减少传统 `str_replace` 或 patch 因上下文漂移导致的失败。
- **LSP 写入链路**：rename、references、diagnostics、code actions 等 IDE 级语义能力可被 agent 调用。
- **DAP 调试器**：通过 Debug Adapter Protocol 让 agent 驱动 lldb、dlv、debugpy 等调试器，而不是只靠打印日志。
- **子代理与并行任务**：`task` 可以拆分任务到隔离 worktree，`irc` 支持并行 agent 间短消息协调，子代理可以返回结构化结果。
- **Web 与浏览器**：`web_search` 集成多个搜索后端并把页面/PDF/GitHub/论坛/包注册表转为结构化 Markdown；`browser` 用 Puppeteer 驱动 Chromium/CDP。
- **模型/provider 路由**：支持大量 API provider、coding plan、OAuth/local provider、自定义 OpenAI/Anthropic/Gemini 兼容服务、fallback chains、路径级 model role、round-robin credentials。
- **配置继承与生态兼容**：能从 `.omp`、`.claude`、`.codex`、`.gemini` 等目录发现规则、skills、hooks、tools、extensions、MCP 等配置；README 也强调可读取 Cursor、Cline、Copilot 等已有规则格式。
- **SDK/RPC/ACP 嵌入**：Node/TypeScript 可直接 `createAgentSession()`，非 Node host 可用 JSONL RPC，编辑器可用 ACP。
- **扩展系统**：支持 slash commands、skills、hooks、custom tools、extensions、marketplace/plugins。

## 4. 实现思路 / 架构拆解
- 关键模块：
  - `packages/coding-agent/`：`omp` CLI、SDK、TUI 模式、RPC/ACP、session、tools、MCP、LSP、DAP、extensions、slash commands。
  - `packages/agent/`：通用 agent loop、tool calling、上下文/compaction/telemetry。
  - `packages/ai/`：多 provider LLM client、streaming、model registry、auth、usage。
  - `packages/tui/`：终端 UI 渲染、输入、主题、组件。
  - `packages/natives/` + `crates/`：Rust/N-API native 层，覆盖 grep/search、embedded shell、AST、PTY、image/text/token 等高频能力。
  - `packages/hashline/`：Hashline patch 语言与应用器。
  - `docs/`：大量当前实现说明，包括 models、config、MCP、LSP、RPC、SDK、memory、approval、custom tools、hooks 等。
- 核心流程：
  1. 用户通过 `omp`、`omp -p`、RPC、ACP 或 SDK 创建 session。
  2. CLI/SDK 初始化 settings、auth storage、model registry、session manager、skills/rules/hooks/extensions/MCP/LSP/tools。
  3. `AgentSession` 维护会话 JSONL、消息、tool registry、模型选择、compaction、retry、todo、task/subagent 状态。
  4. 模型根据 system prompt 和 active tools 调用 `read`/`edit`/`bash`/`lsp`/`task` 等工具。
  5. 工具执行层按 approval policy 决策是否自动执行、询问用户或拒绝。
  6. 结果写回会话；交互式模式用 TUI 渲染，RPC/SDK 用事件流输出。
- 依赖/生态：Bun runtime、TypeScript、React/Solid/TUI 组件、Rust crates、N-API、Puppeteer、MCP、ACP、LSP、DAP、OpenAI/Anthropic/Gemini 等模型 provider。
- 设计亮点：
  - 把 IDE 语义、调试器、浏览器、Web 搜索、native search/shell 都统一成 agent 工具。
  - 用 `.omp` 原生配置，同时兼容 `.claude`、`.codex`、`.gemini` 等已有 agent 生态配置。
  - 暴露 SDK/RPC/ACP，既能作为个人 CLI，也能作为平台组件。
  - 会话和工具能力高度可观察、可扩展：session JSONL、events、extensions、custom tools、hooks、MCP、marketplace。

## 5. 安装与快速开始

### 5.1 安装
```bash
# macOS / Linux: 官方安装脚本
curl -fsSL https://omp.sh/install | sh

# Bun 全局安装，README 标注为推荐方式
bun install -g @oh-my-pi/pi-coding-agent

# Windows PowerShell
irm https://omp.sh/install.ps1 | iex

# mise pin 版本
mise use -g github:can1357/oh-my-pi
```

要求：Bun `>= 1.3.14`；当前 npm 包名是 `@oh-my-pi/pi-coding-agent`，bin 名称是 `omp`。

### 5.2 配置模型凭据
最简单方式是设置 provider 环境变量，例如：

```bash
# 任选你要用的 provider
export ANTHROPIC_API_KEY="..."
export OPENAI_API_KEY="..."
export GEMINI_API_KEY="..."
export OPENROUTER_API_KEY="..."
```

也可以把环境变量写入：

```text
<project>/.env
~/.omp/agent/.env
~/.omp/.env
~/.env
```

当前文档说明的加载优先级大致是：进程环境变量优先，其次项目 `.env`，再到 agent/config/home 级 `.env`。OAuth/订阅型 provider 可在交互式界面中使用 `/login`。

### 5.3 启动交互式 agent
```bash
# 在当前 repo 启动 TUI
omp

# 带初始 prompt 启动
omp "阅读这个 repo，给出架构摘要和改进建议"

# 指定模型，支持 fuzzy match
omp --model opus "review this branch"

# 限制本次可用工具
omp --tools read,edit,bash,lsp "修复 TypeScript 类型错误"

# 生产环境更保守：读操作自动，写/执行需要确认
omp --approval-mode always-ask
omp --approval-mode write

# 高信任本地实验：自动批准所有工具调用，谨慎使用
omp --yolo
```

### 5.4 一次性命令 / 脚本模式
```bash
# 非交互式，回答后退出
omp -p "列出 src 目录下最关键的 5 个文件"

# 读取文件作为初始上下文
omp @prompt.md @screenshot.png "根据这些上下文给出修复计划"

# JSON/text 模式
omp --mode json -p "输出 repo 结构摘要"

# 继续上一个 session
omp --continue "接着上次的计划继续"

# 选择或恢复指定 session
omp --resume
omp --resume <session-id-prefix-or-path>

# 不保存 session，适合 CI 或临时任务
omp --no-session -p "只做一次分析"
```

### 5.5 自定义模型 provider
创建或编辑：

```text
~/.omp/agent/models.yml
```

示例：

```yaml
providers:
  my-openai-compatible:
    baseUrl: https://api.example.com/v1
    apiKey: MY_PROVIDER_API_KEY
    api: openai-responses
    models:
      - id: my-coding-model
        name: My Coding Model
        reasoning: true
        input: [text]
        contextWindow: 128000
        maxTokens: 16384
```

常见 `api` 类型包括：

```text
openai-completions
openai-responses
openai-codex-responses
azure-openai-responses
anthropic-messages
google-generative-ai
google-vertex
```

### 5.6 项目级 `.omp` 配置建议
可以在 repo 根目录建立：

```text
.omp/
  AGENTS.md 或 SYSTEM.md
  settings.json
  mcp.json
  lsp.json
  commands/*.md
  rules/*.{md,mdc}
  skills/<name>/SKILL.md
  hooks/pre/*
  hooks/post/*
  tools/*.{ts,js,sh,py}
  extensions/*
```

建议从最小集合开始：

```text
.omp/
  AGENTS.md        # 项目工作约定
  mcp.json         # 项目需要的 MCP server
  lsp.json         # 仅当自动检测不够用时再加
  commands/        # 团队常用 slash command
  rules/           # 风格/安全规则
```

## 6. 在 Claude Code 中的用法
- 适合让 Claude Code 做什么：
  - 把 oh-my-pi 作为一个独立外部 CLI/harness 来安装、试跑、对比和集成。
  - 让 Claude Code 帮你编写 `.omp/AGENTS.md`、`.omp/rules/*.md`、`.omp/commands/*.md`、`.omp/mcp.json`、`.omp/lsp.json`。
  - 用 `omp -p` 对当前 repo 做第二意见分析，例如代码审查、架构摘要、测试计划。
  - 在平台/工具开发中，用 `omp --mode rpc` 或 SDK 做嵌入式 agent 原型。
- 推荐提示词/工作流：
  1. 让 Claude Code 先读取本项目已有 `AGENTS.md`、`.claude/`、`.codex/` 配置。
  2. 让 Claude Code 生成等价 `.omp/AGENTS.md` 或 `.omp/rules/`，不要直接覆盖原有配置。
  3. 用 `omp --approval-mode always-ask` 启动人工确认模式，跑一个小任务。
  4. 比较 Claude Code 与 `omp` 对同一任务的计划、diff、测试策略。
  5. 如果要自动化，再把稳定命令包装为 npm script、Makefile target 或 CI 中的 `omp -p` 检查。
- 示例：
```bash
# Claude Code 中可让它创建这些文件，然后手动试跑
mkdir -p .omp/commands .omp/rules

# 用 omp 作为第二意见 reviewer
omp -p "Review the current git diff. Return P0-P3 issues with confidence."

# 让 omp 只读分析，避免误改
omp --tools read,search,lsp -p "Explain the dependency graph of this package"
```
- 注意事项：
  - 不要让 Claude Code 和 `omp --yolo` 同时在同一个工作区自动改代码，容易产生并发冲突。
  - 若要双 agent 协作，建议一个负责实现，另一个只读审查；或使用不同 worktree。
  - `omp` 能读取 `.claude` 等配置，但不要假设两边对规则解释完全一致，关键安全/发布规则要显式写入 `.omp/AGENTS.md` 或 `.omp/rules/`。

## 7. 在 Codex 中的用法
- 适合让 Codex 做什么：
  - 把 `omp` 当成另一个 coding agent CLI 进行源码分析、使用手册生成、风险审查和集成建议。
  - 通过 shell 运行 `omp -p`，让 `omp` 对当前 diff 或大型 repo 给出独立分析。
  - 检查 oh-my-pi 的 TypeScript/Rust 源码、工具 registry、provider/model 配置、MCP/LSP/DAP 逻辑。
  - 编写面向 Codex/Claude/团队通用的 `.omp` 配置，让多 agent 工具共享规则。
- 推荐提示词/工作流：
```bash
# 让 omp 做只读代码审查
omp --tools read,search,lsp -p "Review this repository architecture. Focus on hidden coupling and test gaps."

# 让 omp 做当前 diff 审查
omp --approval-mode always-ask -p "Review git diff against main. Do not modify files."

# 让 Codex 管理工作区，omp 只作为一次性外部分析器
omp --no-session --tools read,search -p "Summarize the public API surface of packages/coding-agent/src/sdk.ts"
```
- 注意事项：
  - Codex 运行外部 CLI 时要关注 sandbox/approval；`omp` 自己也有 approval mode，两个权限层要分别控制。
  - 如果 Codex 正在改文件，尽量让 `omp` 只读，或让 `omp` 在单独 git worktree 中执行写入。
  - `omp` 支持 `AGENTS.md`/`.codex` 等配置发现，但 Codex 的系统指令优先级与 `omp` 的配置发现机制不同，不能把一个工具的安全边界直接等同于另一个工具。

## 8. 典型生产力场景
| 场景 | 怎么用 | 产出 |
|---|---|---|
| 日常代码实现 | `omp` 交互式启动，开启 `read,edit,bash,lsp`，用 `--approval-mode write` 控制执行 | 带 LSP 语义和测试反馈的代码 diff |
| 当前 diff 审查 | `omp -p "Review git diff..."`，或用 `/review` 类内置流程 | P0-P3 风险、信心分、发布建议 |
| 大型 repo 快速理解 | `omp --tools read,search,lsp -p "map architecture"` | 关键目录、调用链、风险模块 |
| 重构/rename | 启用 `lsp`，让 agent 使用 LSP references/rename | 跨文件一致修改，减少漏改 re-export/import |
| Debug 难题 | 启用 `debug`，配置 DAP 后让 agent attach/step/evaluate | 调试器证据、根因定位、修复方案 |
| 并行探索 | 用 `task` 子代理拆分模块，必要时 `irc` 协调 | 多 worker findings，父 agent 汇总 |
| Web/论文/issue research | 配置 `web_search` provider，读取 URL/PDF/GitHub PR | 带来源结构化摘要和引用线索 |
| 平台嵌入 | Node SDK `createAgentSession()` 或 `omp --mode rpc` | 自研 IDE/后台服务中的 agent session |
| 编辑器集成 | `omp acp` 通过 Agent Client Protocol 接入 Zed/ACP host | 编辑器文件系统/terminal/permission 路由 |
| 团队规则复用 | `.omp/AGENTS.md`、rules、commands、skills、MCP | 可版本化的项目 agent 约定 |
| 本地模型实验 | `models.yml` 接入 Ollama/LM Studio/llama.cpp/vLLM | 本地/私有模型 coding agent 测试 |
| MCP 工具接入 | `.omp/mcp.json` 配置 stdio/http/sse MCP server | 外部系统工具变成 agent tool |

## 9. 适合 / 不适合
### 适合
- 想要一个“功能全、工具多、可扩展”的终端 coding agent，而不仅是简单 chat + patch。
- 需要多 provider、多模型角色、fallback、local model、OAuth/coding plan 的用户。
- 希望 agent 能调用 LSP、调试器、浏览器、Web 搜索、MCP、子代理和自定义工具。
- 想把 coding agent 嵌入自研产品、编辑器或自动化系统，需要 SDK/RPC/ACP 的团队。
- 需要复用 `.claude`、`.codex`、`.gemini`、Cursor/Cline/Copilot 规则的多 agent 工作区。

### 不适合
- 只需要稳定、简单、低学习成本的官方 Claude Code 或 Codex CLI 流程。
- 对安全边界要求极高，但无法逐项审查工具权限、MCP server、浏览器/SSH/exec 行为的环境。
- 不愿安装/维护 Bun、Rust native addon、复杂 provider 配置和频繁变化的前沿项目。
- 简单一次性问答或一两行补丁；`omp` 的完整 harness 会显得重。
- 团队已经有严格规范的 IDE/CI/code-review bot，且不需要本地 agent 持久会话和工具扩展。

## 10. 同类工具定位
- 类似工具：Claude Code、OpenAI Codex CLI、Aider、OpenCode、Gemini CLI、Continue、Zed Agent/ACP 生态、Cursor/Cline/Windsurf 等 IDE agent。
- 相比优势：
  - 终端优先但能力接近 IDE：LSP、DAP、browser、MCP、subagents、memory、tools discovery 都在同一工具面里。
  - 多入口：CLI/TUI、one-shot、SDK、RPC、ACP，适合个人使用也适合平台集成。
  - 多 provider 路由和配置能力非常强，适合对比模型、组合低价/高性能模型、接入本地模型。
  - Native Rust 层减少对外部 `rg`/`grep`/shell 工具的依赖，跨平台一致性更好。
  - 能导入/发现其他 agent 生态的规则/skills/MCP 配置，迁移成本低。
- 相比限制：
  - 功能面很大，配置和权限模型复杂，新用户需要时间建立安全工作流。
  - 官方模型/官方 agent CLI 的产品稳定性、权限沙箱、企业支持可能更直接。
  - 如果只在 Claude Code 或 Codex 中工作，增加 `omp` 可能带来双 agent 状态、会话和配置管理成本。
  - 项目仍在快速演进，文档与实现细节需要经常核验。

## 11. 关键文件/目录导读
| Path | 作用 |
|---|---|
| `README.md` | 项目总览、安装方式、特性展示、工具列表、provider/routing、SDK/RPC/ACP 入口、包结构。 |
| `packages/coding-agent/package.json` | npm 包 `@oh-my-pi/pi-coding-agent`、bin `omp`、版本、依赖、exports。 |
| `packages/coding-agent/src/cli.ts` | CLI 入口，命令注册与 Bun runtime guard。 |
| `packages/coding-agent/src/cli/args.ts` | CLI 参数解析：`--model`、`--tools`、`--mode`、`--approval-mode`、`--continue`、`--resume` 等。 |
| `packages/coding-agent/src/main.ts` | CLI orchestration：settings、auth、model registry、session、mode dispatch。 |
| `packages/coding-agent/src/sdk.ts` | Node/TypeScript SDK：`createAgentSession`、session/model/auth/tool discovery。 |
| `packages/coding-agent/src/modes/` | TUI、print、RPC、ACP 等运行模式。 |
| `packages/coding-agent/src/tools/index.ts` | 内置工具 registry，判断哪些工具启用、隐藏、可发现。 |
| `packages/coding-agent/src/tools/` | 每个工具实现：read/write/edit/bash/lsp/debug/task/web_search/browser 等。 |
| `packages/coding-agent/src/session/` | 会话 JSONL、历史、分支、blob、resume/fork/list 等。 |
| `packages/coding-agent/src/config/` | settings、models、keybindings、MCP schema、config file loader。 |
| `packages/coding-agent/src/lsp/` | LSP client、defaults、配置和启动逻辑。 |
| `packages/coding-agent/src/dap/` | Debug Adapter Protocol client/config/session。 |
| `packages/coding-agent/src/mcp/` | MCP transport、loader、manager、tool bridge、OAuth。 |
| `packages/coding-agent/src/extensibility/` | skills、hooks、extensions、custom tools、custom commands、plugins/marketplace。 |
| `packages/coding-agent/src/task/` | 子代理/task 编排、隔离、输出管理。 |
| `packages/coding-agent/src/hindsight/` / `src/memories/` | 长期记忆 / Hindsight / memory pipeline。 |
| `packages/agent/` | 通用 agent runtime、agent loop、compaction、telemetry。 |
| `packages/ai/` | 多 provider LLM client、models、auth、streaming、usage。 |
| `packages/tui/` | 终端 UI 渲染库。 |
| `packages/natives/` | N-API native package。 |
| `crates/pi-natives` | Rust native addon 聚合层。 |
| `crates/pi-shell` | embedded shell / PTY / process management。 |
| `crates/pi-ast` | tree-sitter / ast-grep 相关 AST 能力。 |
| `crates/pi-iso` | task workspace isolation backend。 |
| `packages/hashline/` | Hashline patch/anchor 编辑格式。 |
| `.omp/` | 项目自身 dogfood 配置：commands、rules、skills。 |
| `docs/models.md` | `models.yml` 自定义 provider/model、model equivalence、routing 说明。 |
| `docs/config-usage.md` | `.omp`/`.claude`/`.codex`/`.gemini` config discovery 与优先级。 |
| `docs/mcp-config.md` | MCP server 配置格式、stdio/http/sse、auth、schema。 |
| `docs/lsp-config.md` | LSP 自动检测与项目/用户配置。 |
| `docs/sdk.md` | SDK 安装、`createAgentSession()`、事件、session manager。 |
| `docs/rpc.md` | `omp --mode rpc` JSONL 协议、commands/responses/events。 |
| `docs/approval-mode.md` | 工具 approval tier、`always-ask`/`write`/`yolo` 行为。 |
| `docs/custom-tools.md` | 自定义工具模块 contract、Zod schema、execute/onUpdate/render。 |
| `docs/hooks.md` | hook 运行、pre/post、event 修改和安全注意。 |
| `docs/environment-variables.md` | provider/search/runtime 环境变量与 `.env` 加载优先级。 |
| `docs/keybindings.md` | 交互式 TUI 快捷键和用户 remap。 |
| `docs/tools/*.md` | 每个 built-in tool 的详细参考。 |

## 12. 学习与掌控建议
1. **先以只读模式学习**：第一次在真实 repo 用 `omp --tools read,search,lsp -p "summarize architecture"`，确认它如何读项目、如何输出。
2. **谨慎设置 approval**：默认/高信任模式可能自动执行写入或命令；生产 repo 建议先用 `--approval-mode always-ask` 或 `write`。
3. **从模型凭据开始最小配置**：先只配置一个可靠 provider；跑通后再加 fallback、roles、round-robin credentials、自定义 providers。
4. **用 `.omp/AGENTS.md` 固化项目约定**：把测试命令、代码风格、禁止事项、安全规则写进项目级配置，避免每次口头提醒。
5. **LSP/DAP 分阶段接入**：先确认 LSP 自动检测是否工作，再为特殊语言写 `.omp/lsp.json`；调试器能力需要本机 DAP adapter 可用。
6. **子代理用隔离 worktree**：多 agent 并行很强，但也容易冲突；复杂任务建议让 `task` 使用隔离目录，并要求结构化 findings。
7. **把 `omp -p` 当成 reviewer**：即使你主力使用 Claude Code/Codex，也可以让 `omp` 做一次性只读审查，尤其适合大 diff、架构理解和 Web research。
8. **想嵌入时先读 SDK/RPC**：Node 项目优先 SDK；跨语言或强隔离场景优先 `omp --mode rpc`；编辑器集成看 ACP。
9. **定期复核 docs 与版本**：项目迭代很快，涉及安全、权限、provider、MCP、memory 的配置不要只凭旧笔记，使用前重新看 README/docs。
10. **避免双 agent 同写同目录**：Claude Code、Codex、omp 同时写一个工作区时，最好分配清晰角色或使用独立 worktree。

## 13. Sources
- https://github.com/can1357/oh-my-pi
- https://github.com/can1357/oh-my-pi/blob/main/README.md
- https://www.npmjs.com/package/@oh-my-pi/pi-coding-agent
- https://github.com/can1357/oh-my-pi/blob/main/packages/coding-agent/package.json
- https://github.com/can1357/oh-my-pi/blob/main/packages/coding-agent/DEVELOPMENT.md
- https://github.com/can1357/oh-my-pi/blob/main/docs/models.md
- https://github.com/can1357/oh-my-pi/blob/main/docs/config-usage.md
- https://github.com/can1357/oh-my-pi/blob/main/docs/sdk.md
- https://github.com/can1357/oh-my-pi/blob/main/docs/rpc.md
- https://github.com/can1357/oh-my-pi/blob/main/docs/mcp-config.md
- https://github.com/can1357/oh-my-pi/blob/main/docs/lsp-config.md
- https://github.com/can1357/oh-my-pi/blob/main/docs/approval-mode.md
- https://github.com/can1357/oh-my-pi/blob/main/docs/custom-tools.md
- https://github.com/can1357/oh-my-pi/blob/main/docs/environment-variables.md
- https://github.com/can1357/oh-my-pi/blob/main/docs/memory.md
- https://github.com/can1357/oh-my-pi/tree/main/docs/tools
