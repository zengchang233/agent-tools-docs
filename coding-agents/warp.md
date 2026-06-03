# warp

> Repo: https://github.com/warpdotdev/warp  
> Category: `coding-agents`  
> Last checked: 2026-06-03  
> Source snapshot: `master` branch checked at `2249469e5d24e472cee6ce97d3d324293f67db71`; official docs pages checked on 2026-06-03.

## 1. 一句话总结
Warp 是一个从 terminal 演进出来的 **Agentic Development Environment**：它既是现代化终端，也是本地 / 云端 coding agents 的工作台。你可以直接用内置 Warp Agent / Oz，也可以在 Warp 里运行 Claude Code、OpenAI Codex、OpenCode、Gemini CLI 等第三方 CLI agent，并获得 Blocks、rich input、inline code review、notifications、remote control、MCP、Warp Drive 等增强能力。

## 2. 项目目的与核心问题
- 目标用户：重度使用终端、Claude Code / Codex / Gemini CLI 等 coding agents 的开发者；希望把 agent 执行、代码审查、上下文和团队工作流统一管理的工程团队；想研究现代 terminal + agent IDE 形态的工具开发者。
- 解决的问题：传统 terminal 对 agent 工作流不友好，命令输出是线性滚屏，输入框编辑能力弱，agent 对话、diff review、上下文附加、通知、远程任务和团队知识分散在多个工具里。Warp 试图把 terminal、code editor、local agents、cloud agents、MCP、shared knowledge 放在一个统一界面中。
- 为什么值得关注：`warpdotdev/warp` 已开源 client 代码；官方定位不再只是 “AI terminal”，而是围绕 Oz agents 的开发环境。对 Claude Code / Codex 用户来说，Warp 的价值不一定是替代这些 CLI，而是给它们套上一层更适合 agent 工作流的 terminal/IDE shell。

## 3. 核心能力
- **现代终端体验**：命令和输出按 **Blocks** 组织，支持按 Block 复制、搜索、过滤、导航；输入区支持多行编辑、点击定位光标、选择替换、autosuggestions、tab completions 和命令纠错。
- **Agent Mode**：从普通 terminal mode 切换到 Agent Mode，直接用自然语言让 Oz / Warp Agent 解释代码库、写代码、运行命令、调试错误或做重构。
- **Local agents**：本地 Warp Agent 嵌入在 terminal 中，可读取代码库、Warp Drive、连接工具和附加上下文，支持任务列表、web search、full terminal use、interactive code review、voice 等能力。
- **第三方 CLI agents 增强**：在 Warp 内运行 Claude Code、Codex、OpenCode、Gemini CLI、Cursor CLI 等受支持 agent 时，Warp 会自动检测并显示 agent toolbelt，提供 rich input、notifications、code review comments、attach code as context、vertical tabs、remote control 等功能。
- **Cloud agents / Oz**：Oz 平台用于触发、编排、执行和观察云端 agents，可处理 PR review、issue triage、dependency update、incident response、文档更新等后台或事件驱动任务。
- **Warp Drive / Rules / MCP**：Warp Drive 保存并同步 workflows、notebooks、prompts、environment variables；Rules 可用全局或项目级规则约束 agents；MCP servers 可扩展 local agents 的工具和数据源。
- **代码编辑与审查**：官方 docs 将 Warp 描述为包含 file tree、code editor、LSP support 和 interactive code review 的开发环境，而不是单纯 terminal emulator。
- **开源 client 与贡献流程**：repo 主要是 Rust Cargo workspace，`app/` 是主应用，`crates/warpui` / `crates/warpui_core` 是自研 UI 框架，`crates/warp_terminal`、`crates/editor`、`crates/graphql`、`crates/mcp` 等提供核心能力。

## 4. 实现思路 / 架构拆解
- 关键模块：
  - `app/`：主应用，包括 terminal、AI integration / Agent Mode、Drive、auth、settings、workspace/session 等。
  - `crates/warpui/`、`crates/warpui_core/`、`crates/warpui_extras/`：WarpUI 自研 UI 框架，使用 Entity-Component-Handle 风格组织 view/model。
  - `crates/warp_terminal/`：terminal emulation 相关能力。
  - `crates/editor/`：输入编辑器与文本编辑能力。
  - `crates/ai/`：AI / agent 相关逻辑。
  - `crates/mcp/`：MCP 相关实现。
  - `crates/graphql/`、`crates/warp_graphql_schema/`：GraphQL client/schema 交互。
  - `crates/lsp/`、`crates/warp_completer/`、`crates/input_classifier/`、`crates/natural_language_detection/`：代码/命令补全、语言服务和自然语言识别等能力。
  - `script/`：bootstrap、run、format、presubmit、platform-specific setup、bundle、release 和技能安装脚本。
- 核心流程：
  1. 用户在 Warp terminal 中运行普通 shell 命令，Warp 将命令与输出分块保存为 Blocks。
  2. 当用户进入 Agent Mode 或输入自然语言 prompt，Warp 将当前会话、代码库、选择文本、文件、URL、图片、Blocks、Rules、MCP servers 等作为可用上下文。
  3. Warp Agent / Oz 生成计划、调用 terminal/code/MCP/web 等能力，并把 diff、task list、命令输出反馈到 UI。
  4. 如果用户运行第三方 CLI agent，Warp 通过 agent detection 激活 toolbelt，把 rich input、code review、notifications、remote control 等能力叠加到该 CLI 会话上。
  5. Cloud agents 场景下，外部 trigger / schedule / integration / API 触发 Oz orchestration，agent 在远程 host/environment 执行，生成可审计的 run transcript、logs 和 artifacts。
- 依赖/生态：Rust、Cargo workspace、platform-specific native code、GraphQL、MCP、LSP、custom UI framework；产品层连接 Claude Code、Codex、OpenCode、Gemini CLI 等 agent 生态，以及 Slack/GitHub/CI/webhook/API 等云端触发源。
- 设计亮点：
  - 把 terminal output 结构化为 Blocks，给 agent、复制、搜索和审查提供更稳定的单元。
  - 不是只内置一个 agent，而是同时支持自家 Oz / Warp Agent 与第三方 CLI agents。
  - 通过 Warp Drive、Rules、MCP 把个人和团队上下文抽象成可复用对象。
  - 本地交互和云端后台 agents 使用相近的工作流概念，适合从个人终端逐步扩展到团队自动化。

## 5. 安装与快速开始

### 5.1 安装官方构建
```bash
# macOS
brew install --cask warp
```

```powershell
# Windows
winget install Warp.Warp
```

```bash
# Debian / Ubuntu，下载 .deb 后安装
sudo apt install ./warp-terminal.deb
```

Linux 还提供 `.deb`、`.rpm`、pacman package 和 AppImage 等方式。包管理器安装后，Linux 可用 `warp-terminal` 启动。首次启动可以注册 / 登录账号，也可以跳过；官方 docs 说明首次打开需要联网，之后 terminal 可离线运行，但 AI、实时协作等功能需要网络。

### 5.2 终端基础用法
```bash
# 任意普通 shell 命令都会形成 Block
ls -la

# 多行输入：Shift+Enter 插入换行，Enter 一次性执行
echo "step 1"
echo "step 2"
echo "step 3"
```

常用体验：
- `⌘↑` / `⌘↓`（macOS）或 `Ctrl+↑` / `Ctrl+↓`（Windows/Linux）在 Blocks 之间跳转。
- 在单个 Block 中搜索输出，而不是搜索整个滚屏历史。
- 右方向键接受 shell history autosuggestion。
- `Tab` 查看命令、flag、文件路径等增强补全。

### 5.3 启动 Agent Mode
```text
# macOS: ⌘↩
# Windows / Linux: Ctrl+Shift+Enter

Explain the architecture of this project
```

适合先从只读问题开始：
```text
阅读当前 repo，列出入口模块、核心数据流和测试命令。不要修改文件。
```

再逐步进入执行型任务：
```text
运行测试，定位失败原因，先给出计划和预计改动文件，等我确认后再修改。
```

### 5.4 在 Warp 中运行第三方 CLI agent
```bash
# 已安装 Claude Code 时
claude

# 已安装 OpenAI Codex CLI 时
codex

# 其他受支持 agents 也可直接在 Warp 里启动
opencode
gemini
```

Warp 会自动检测受支持 CLI agent，并显示 agent toolbelt。当前官方 docs 列出的支持对象包括 Claude Code、OpenAI Codex、OpenCode、Amp、Auggie、Copilot CLI、Cursor CLI、Gemini CLI、Droid、Pi。不同 agent 的增强能力有差异；例如 notifications 需要一次性设置，且并非所有 agent 都支持。

### 5.5 从源码构建 / 贡献 Warp
```bash
git clone https://github.com/warpdotdev/warp.git
cd warp
./script/bootstrap
cargo run
```

repo README 给出的贡献者流程：
```bash
./script/bootstrap   # platform-specific setup
./script/run         # build and run Warp
./script/presubmit   # fmt, clippy, and tests
```

开发常用命令：
```bash
cargo run
cargo bundle --bin warp
cargo nextest run --no-fail-fast --workspace --exclude command-signatures-v2
cargo test --doc
./script/format
./script/presubmit
```

官方 docs 建议日常使用官方构建；自构建 `warp-oss` 使用独立配置/数据目录，不自动更新，也没有生产代码签名身份，更适合开发、审计和贡献。

## 6. 在 Claude Code 中的用法
- 适合让 Claude Code 做什么：
  - 把 Warp 当作 Claude Code 的增强 terminal：在 Warp 中启动 `claude`，使用 Warp 的 Blocks、rich input、toolbelt、notifications、tabs 和 code review 体验。
  - 让 Claude Code 为当前 repo 编写/维护 `AGENTS.md` 项目规则，使 Warp Agent、Codex、Claude Code 等工具都能读取统一约定。
  - 分析 `warpdotdev/warp` 源码结构，定位 terminal、AI、MCP、Drive、settings 等模块。
  - 为团队整理 Warp Drive workflows、Rules、MCP server 配置草案。
- 推荐提示词/工作流：
```text
请先读取 AGENTS.md / README / package or Cargo metadata，给出这个 repo 在 Warp 中使用 Claude Code 的建议。不要修改文件。
```

```text
把当前项目的开发约定整理成 AGENTS.md，要求同时适用于 Claude Code、Codex 和 Warp Agent。保留现有规则，不要引入未验证命令。
```

```bash
# 在 Warp 中运行 Claude Code，利用 Warp UI 管理会话
claude
```
- 注意事项：
  - Warp 增强的是 CLI agent 的外壳体验，不会改变 Claude Code 自身的权限、安全边界和模型行为。
  - 如果 Claude Code 和 Warp Agent 同时改同一个工作区，容易出现并发 diff 冲突；建议一个 agent 实现，另一个只读 review，或用独立 worktree。
  - 需要谨慎配置 notifications、remote control、MCP 和 shell 权限，尤其是团队共享机器或生产 repo。

## 7. 在 Codex 中的用法
- 适合让 Codex 做什么：
  - 在 Warp 中启动 `codex`，把 Codex CLI 的执行/审查能力和 Warp 的 agent toolbelt 结合起来。
  - 让 Codex 研究 Warp 源码：`app/`、`crates/ai/`、`crates/mcp/`、`crates/warpui*`、`crates/warp_terminal/`、`script/`。
  - 用 Codex 生成 Warp adoption guide：安装、快捷键、Agent Mode、Claude Code/Codex toolbelt、MCP、风险边界。
  - 让 Codex 为本仓库或团队项目维护 `AGENTS.md`，把 “repo 链接 -> 正式文档 -> README index -> commit/push” 流程固定下来。
- 推荐提示词/工作流：
```bash
# 在 Warp 中直接启动 Codex
codex
```

```text
目标：评估当前 repo 是否适合迁移到 Warp + Codex 工作流。
请检查 README、AGENTS.md、常用脚本和测试命令；输出安装步骤、推荐设置、风险和不建议自动化的环节。先只读分析。
```

```text
阅读 warpdotdev/warp 的 README、WARP.md 和 Cargo workspace，解释 terminal / AI / UI framework / MCP 相关模块。不要 clone 全量历史，只看当前主分支。
```
- 注意事项：
  - Codex 的 sandbox/approval 和 Warp Agent/CLI toolbelt 权限是两层控制，不能因为在 Warp 中运行就放宽 Codex 权限。
  - 如果使用 `codex review` 或 `codex exec`，建议让 Warp 负责会话可视化，Codex 负责明确边界的实现或审查。
  - 对需要最新文档的 Warp 功能（Cloud agents、CLI agent support、MCP、pricing/credits），让 Codex 开启联网核验并记录 check date。

## 8. 典型生产力场景
| 场景 | 怎么用 | 产出 |
|---|---|---|
| 日常 terminal 工作 | 用 Warp 替代传统 terminal，利用 Blocks、multi-line input、autosuggestion、completion | 更容易复制、搜索、复盘的命令历史 |
| 本地代码库理解 | Agent Mode 输入 “Explain the architecture of this project” 或更具体的问题 | 架构摘要、关键文件、后续阅读路径 |
| Claude Code / Codex 增强外壳 | 在 Warp 中运行 `claude` / `codex`，使用 toolbelt、rich input、code review comments | 更接近 IDE 的 CLI agent 体验 |
| 当前 diff 审查 | 让 Warp Agent 或第三方 CLI agent review current diff，并用 inline comments 反馈 | P0-P3 问题、修复建议、可迭代 review |
| 团队知识复用 | 把 prompts、workflows、env vars、notebooks 放到 Warp Drive；项目规则写入 `AGENTS.md` | 跨成员一致的 agent 上下文 |
| MCP 工具接入 | 在 Settings > Agents > MCP servers 或 Warp Drive 中配置 MCP server | 让 local agents 调用内部工具/数据源 |
| 后台自动化 | 用 Cloud agents / Oz 响应 GitHub、Slack、CI、schedule、API trigger | PR review、issue triage、依赖更新、事故响应等可审计 runs |
| 贡献 Warp 本身 | clone repo，运行 `./script/bootstrap`、`cargo run`、`./script/presubmit` | 本地 `warp-oss` 构建、可提交 PR 的代码改动 |

## 9. 适合 / 不适合
### 适合
- 你已经大量使用 terminal，并希望终端本身更适合 agent 工作流。
- 你用 Claude Code、Codex、OpenCode、Gemini CLI 等 CLI agent，但想要更好的输入、上下文、通知、diff review 和会话管理。
- 团队希望把 repo rules、prompts、MCP、workflows 和 cloud agent runs 统一管理。
- 你愿意接受一个带账号、云端同步、AI provider 调用和可选 cloud agents 的开发环境。
- 你想研究 Rust terminal emulator、自研 UI framework、agent integration、MCP、GraphQL client 等实现。

### 不适合
- 你只需要极轻量、完全本地、无账号、无 AI、无云同步的传统 terminal。
- 你的工作环境禁止把 prompt、代码上下文或命令输出发送到外部 LLM provider。
- 你不能接受 agent 运行 shell、读写文件、联网、连接 MCP server 或远程 cloud environment 的安全面。
- 你希望所有功能都离线可用；Warp terminal 可离线运行，但 AI、实时协作、cloud agents 等能力依赖网络。
- 你要对生产环境自动执行命令；这种场景需要严格的 approvals、least privilege、审计和隔离环境，不适合直接放开 agent。

## 10. 同类工具定位
- 类似工具：iTerm2 / Ghostty / macOS Terminal 等传统或现代 terminal；Cursor / VS Code + agent 插件等 IDE；Claude Code / Codex / OpenCode / Gemini CLI 等 CLI coding agents；GitHub Actions / CI bots / 自研 agent platform 等后台自动化工具。
- 相比传统 terminal：Warp 的优势是 Blocks、现代输入编辑、agent mode、MCP、Drive、CLI agent toolbelt 和 cloud agents；限制是更重、更产品化，且部分能力依赖账号和网络。
- 相比 Claude Code / Codex CLI：Warp 不是同一层级的替代品，更像 agent terminal/IDE shell；它可以承载这些 CLI，并增强 UI、上下文和协作体验。
- 相比 Cursor：Cursor 更像代码编辑器优先，Warp 更像 terminal/agent workspace 优先；如果团队工作大量发生在 shell、CI、远程环境和 CLI agents 中，Warp 更贴近使用路径。
- 相比自研 agent 平台：Oz / Cloud agents 提供现成 trigger、orchestration、observability 和 team session；限制是要接受 Warp/Oz 平台边界、credits/billing、账号权限和数据流。

## 11. 关键文件/目录导读
| Path | 作用 |
|---|---|
| `README.md` | 项目定位、安装入口、Open Source / contributing 概览、license 信息 |
| `WARP.md` | 面向 coding agents / contributors 的工程指南：开发命令、架构、测试、PR workflow、代码风格 |
| `CONTRIBUTING.md` | 贡献流程、issue/spec/PR 规则、本地开发与安全报告说明 |
| `Cargo.toml` | Rust workspace 配置，`app` 和 `crates/*` 是主要 members |
| `app/` | 主 Warp app，包括 terminal、AI integration、Drive、auth、settings、workspace/session 等 |
| `crates/ai/` | AI / agent 相关 crate |
| `crates/mcp/` | MCP 能力实现 |
| `crates/warp_terminal/` | terminal emulation 相关实现 |
| `crates/editor/` | 输入/文本编辑相关实现 |
| `crates/warpui/`、`crates/warpui_core/` | WarpUI 自研 UI framework |
| `crates/graphql/`、`crates/warp_graphql_schema/` | GraphQL client/schema 相关代码 |
| `crates/warp_cli/` | Warp CLI 相关代码 |
| `script/bootstrap` | 平台依赖初始化，并处理 common skills 安装 |
| `script/run` | 本地运行开发构建 |
| `script/presubmit` | 格式化、clippy、测试等提交前检查 |
| `skills-lock.json`、`.agents/` | Warp repo 自身使用的 agent skills / project rules 相关文件 |

## 12. 学习与掌控建议
1. **先把 Warp 当终端用**：熟悉 Blocks、多行输入、补全、历史建议和快捷键，不要一开始就把所有 AI 功能打开。
2. **再尝试 Agent Mode 的只读问题**：例如解释架构、找测试命令、总结 diff；确认上下文和权限边界后再让 agent 修改文件。
3. **把 Claude Code / Codex 放进 Warp 跑**：如果你已经习惯这些 CLI，先观察 toolbelt、rich input、notifications、inline code review 是否提高效率。
4. **为项目写好 `AGENTS.md`**：把构建、测试、权限、安全、文档风格写成项目规则，让 Warp Agent、Codex、Claude Code 等工具共享同一套约定。
5. **谨慎使用 Cloud agents**：先在低风险 repo 或 sandbox 中试用；明确 credentials、credits、网络访问、PR 权限和审计责任。
6. **研究源码时从 `WARP.md` 开始**：它比 README 更适合 contributor / coding agent，包含开发命令、架构和 PR 检查要求。

## 13. Sources
- https://github.com/warpdotdev/warp
- https://github.com/warpdotdev/warp/blob/master/README.md
- https://github.com/warpdotdev/warp/blob/master/WARP.md
- https://github.com/warpdotdev/warp/blob/master/CONTRIBUTING.md
- https://docs.warp.dev/
- https://docs.warp.dev/getting-started/quickstart
- https://docs.warp.dev/getting-started/quickstart/installation-and-setup
- https://docs.warp.dev/agent-platform/local-agents/overview
- https://docs.warp.dev/agent-platform/cli-agents/overview
- https://docs.warp.dev/agent-platform/cloud-agents/overview
- https://docs.warp.dev/agent-platform/cloud-agents/platform
- https://docs.warp.dev/knowledge-and-collaboration/warp-drive
- https://docs.warp.dev/knowledge-and-collaboration/mcp
- https://docs.warp.dev/agent-platform/capabilities/rules
