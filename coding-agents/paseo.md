# paseo

> Repo: https://github.com/getpaseo/paseo  
> Category: `coding-agents`  
> Last checked: 2026-06-13  
> Source snapshot: `main` 分支检查于 `72b67f48e3a9a6fff5b1619a0dcc196e043480de`（v0.1.96，2026-06-13）；官方文档站 `paseo.sh/docs` 同日核验。

## 1. 一句话总结
Paseo 是一个开源、自托管的 **Coding Agent 编排平台**：它在你自己的机器上跑一个本地 daemon，把 Claude Code、Codex、GitHub Copilot CLI、OpenCode、Pi 以及 30+ 个 ACP 兼容的 coding agent CLI 包装成统一的 provider，用桌面 / 移动 / Web / CLI 多端连上同一个 daemon，让多个 agent 可以并行、跨机器、可远程控制地协作工作。

## 2. 项目目的与核心问题
- 目标用户：重度依赖多个 coding agent CLI（Claude Code / Codex / OpenCode / Gemini CLI / Cursor / Qwen Code / Kimi 等）、希望在桌面 + 手机 + 远程服务器之间无缝调度它们的开发者；想做 “一个 PM agent 调度多个执行 agent” 的多 agent 工作流爱好者；想研究自托管 agent 平台、relay 端到端加密、ACP 协议接入和移动端 agent UX 的研发团队。
- 解决的问题：
  - 每个 coding agent CLI 都有自己的入口、UI、会话管理；切换工具就要切换终端、桌面 app 或 IDE 插件，且没有统一的 “在哪个 repo / worktree / 分支上跑哪个 agent” 的视图。
  - 想从手机、平板或另一台机器接管正在跑的 agent 时，通常需要 SSH + tmux 组合，缺少安全的远程访问、推送通知和 PR 集成。
  - 多个 agent 想并行修同一个 repo 或互相协作（一个规划、一个实现、一个 review）时，没有统一的工作区抽象、worktree 管理和编排原语。
- 为什么值得关注：
  - 项目把 “Coding agent runtime + 多端客户端 + 自托管 relay + MCP orchestration + 移动端 voice” 打包到一个 AGPL-3.0 的开源项目里，本身就是当前 agent 生态里少见的 self-hostable 全链路实现。
  - 它不替代 Claude Code / Codex 等 CLI，而是把它们当成 provider，用同一个 UX 和 API 暴露出来；对深度用户来说几乎没有“锁定成本”。
  - 文档详细给出了 daemon-client 协议、ECDH+NaCl 端到端加密、git 工作树编排、ACP 接入、Paseo MCP 工具集、orchestration skills 等设计细节，是研究 agent 编排架构的好样本。

## 3. 核心能力
- **多 provider 接入**：原生支持 Claude Code、Codex CLI、OpenCode、Pi；通过 ACP 目录支持 Cursor、Gemini CLI、GitHub Copilot CLI、Hermes、Kimi Code、Qwen Code、goose、Grok、Junie 等 30+ agent；用户可在 `~/.paseo/config.json` 里 `extends` 已有 provider 或新增自定义 ACP / 二进制 / 多 profile。
- **跨端客户端**：iOS / Android（React Native）、桌面（Electron）、Web、CLI（`@getpaseo/cli`）共享同一份 daemon API，桌面 app 自动启动并管理内嵌 daemon，CLI 适合 headless 服务器 / 开发盒。
- **自托管 daemon + 可选 relay**：daemon 默认监听 `127.0.0.1:6767`，可改为 socket、Tailscale 地址或 `0.0.0.0`；relay 模式下 daemon 主动外连官方 relay 服务器，移动端通过扫码完成 Curve25519 ECDH 握手 + XSalsa20-Poly1305（NaCl `box`）端到端加密，配对二维码即信任锚，relay 不可读、不可改、不可重放。
- **Workspace / Session / Worktree 模型**：项目下挂 workspace（工作区，对应工作目录），workspace 里挂多个 session（agent / 终端 / 浏览器 / diff）；启用 git 时 workspace 自动绑定独立 worktree，分支名根据首条 prompt 生成，多个 agent 可并行操作不同 worktree，互不干扰。
- **`paseo.json` 工作树编排**：仓库根目录的 `paseo.json` 描述 worktree `setup` / `teardown`、命名 `scripts`、`type=service` 长期服务和默认 `terminals`；服务通过 `$PASEO_PORT` 自动分配端口、绑定 `*.localhost` 反向代理、互注 `$PASEO_SERVICE_<NAME>_URL` 等环境变量，方便多个 agent 同时跑独立 dev server。
- **CLI 全功能**：`paseo run`、`ls`、`attach`、`send`、`logs`、`wait`、`stop`、`permit`、`agent mode`、`schedule`、`worktree`、`daemon` 等命令覆盖 UI 全部能力；支持 `--detach`、`--worktree`、`--output-schema`（JSON Schema 受限输出）、`--host`（指向远端 daemon 或 relay offer URL）、`PASEO_HOST` / `PASEO_PASSWORD` 环境变量。
- **Paseo MCP**：daemon 自带 MCP server，可被注入到 agent 内，提供 `create_agent`、`wait_for_agent`、`send_agent_prompt`、`update_agent`、`set_agent_mode`、`create_terminal`、`capture_terminal`、`send_terminal_keys`、`create_schedule`、`create_worktree`、`list_pending_permissions`、`respond_to_permission`、`speak` 等工具，让 agent 可以自己创建并指挥子 agent。
- **Schedules / 循环作业**：`paseo schedule create` 支持 `--every <duration>` 间隔或 `--cron + --timezone` cron 模式，目标可是“新 agent / 已有 agent / 调用方自身（heartbeat）”，自带 `pause / resume / run-once / inspect / logs / delete`，适合过夜重构、CI 蹲守、每日 GitHub triage 等。
- **Orchestration skills**：`npx skills add getpaseo/paseo` 安装 `~/.agents/skills/`，并为各 agent 建立软链；提供 `/paseo`、`/paseo-handoff`、`/paseo-loop`、`/paseo-committee`、`/paseo-advisor` 等 slash command，让 Claude Code / Codex 等 CLI 直接学会用 Paseo 管理同伴 agent。
- **本地优先 voice 栈**：dictation / voiceMode 默认走本地 Parakeet（STT）+ Kokoro（TTS）ONNX 模型，未命中模型由 daemon 自动下载；可切到 OpenAI Realtime；voice LLM 复用已配置的 Claude / Codex / OpenCode provider，并通过 MCP stdio bridge 调用 Paseo 工具集。
- **隐私 / 安全姿态**：自述 “No telemetry, tracking, or forced log-ins”，code 不离开机器，provider 凭据沿用各自 CLI（`~/.claude/`、OpenAI key、ACP token），daemon 可设 bcrypt 密码（`paseo daemon set-password` / `PASEO_PASSWORD`），并提供 DNS rebinding 防护、host 白名单、CORS 控制等。

## 4. 实现思路 / 架构拆解
- 总体形态：**Daemon + 多端 client** 的 Docker-style 架构。daemon 是 Node.js 进程，负责 agent 生命周期、WebSocket API、MCP server、relay 连接；客户端通过直连或 relay 经 WebSocket 接入。
- monorepo 关键 packages（来自 `package.json` workspaces）：
  - `packages/server`：daemon 主体——`server/bootstrap.ts`（HTTP/WS/agent manager 启动）、`server/websocket-server.ts`（hello 握手、二进制帧路由）、`server/session.ts`（每客户端 session）、`server/agent/agent-manager.ts`（agent 状态机和时间线订阅）、`server/agent/agent-storage.ts`（`$PASEO_HOME/agents/` JSON 落盘）、`server/agent/mcp-server.ts`（agent-to-agent 控制）、`server/agent/providers/`（各 provider 适配器）、`server/relay-transport.ts`（端到端加密 outbound）、`server/schedule/`、`server/loop-service.ts`、`server/chat/`。
  - `packages/protocol`：WebSocket 消息、二进制帧编解码、endpoint 解析、agent timeline、provider config schema 等共享类型，是 daemon 与客户端的真源。
  - `packages/client`：daemon client SDK，桌面 / 移动 / CLI 共享。
  - `packages/cli`：`paseo` CLI 实现，可启动并管理 daemon。
  - `packages/app`：Expo（iOS / Android / Web）跨端 UI，桌面 app 通过 `expo export --platform web` 把它打进 Electron。
  - `packages/desktop`：Electron 壳，自带 daemon。
  - `packages/relay`：自托管 relay 实现；社区还有 `paseo-relay`（Go 版本）。
  - `packages/website`：`paseo.sh` 营销站和 docs 源（与 `public-docs/` 对应）。
  - `packages/expo-two-way-audio`、`packages/highlight`：移动端音频 / 语法高亮支持库。
  - `skills/paseo*`、`scripts/`、`patches/`、`nix/`：orchestration skills 源、构建脚本、第三方 patch 与 Nix 包。
- 核心运行流程：
  1. 用户启动 daemon（桌面 app 自启 / `paseo daemon start` / `npm install -g @getpaseo/cli && paseo`），daemon 读取 `~/.paseo/config.json` 与环境变量，加载 provider 列表。
  2. 客户端按下列任一方式连入 daemon：直连 `host:port`、Unix socket、Windows pipe，或扫描桌面 / `paseo daemon pair --json` 输出的 `https://app.paseo.sh/#offer=...` relay offer URL。
  3. 客户端创建 workspace；如果在 git 仓库内，daemon 在 `$PASEO_HOME/worktrees/<repo-hash>/<slug>/` 下建立 worktree，并执行 `paseo.json` 中的 `setup`、自动 terminals 与 services。
  4. 用户在 workspace 内启动 agent session：daemon 选定 provider，spawn 对应 CLI（Claude SDK / Codex server / Copilot ACP / OpenCode / Pi 等），通过 stdio 或 ACP 双向流接管输入输出，写进时间线（timeline），订阅者实时收到事件。
  5. 多 agent 编排：在 agent 提示中可调用 Paseo MCP 工具创建/控制兄弟 agent、开终端、装计划表；CLI 端可用 `paseo run --detach` + `paseo wait` + `paseo logs`，或直接用 orchestration skill `/paseo-handoff`、`/paseo-loop`、`/paseo-committee`、`/paseo-advisor`。
  6. 远程 / 移动端：通过 relay outbound 模式或 Tailscale 之类的 VPN 直连 daemon；ECDH+NaCl 提供 E2EE，daemon 仍坐镇你自己的硬件。
- 设计亮点：
  - **Provider 抽象**：把 “怎么启动 / 怎么读输出 / 怎么发输入 / 支持哪些 mode” 封装成 provider 接口；原生 provider 自动发现，ACP 用统一适配器，外加 `agents.providers` 自定义；用户的 subscription / API key / skill 等都留在原始 CLI 上下文里。
  - **Workspace / Session 解耦**：workspace 是稳定的“工作目标”，session 是“你在这个目标里同时做的事”（agent、终端、浏览器、diff），更贴合现实开发，胜过单线程聊天框。
  - **Worktree-first 并行**：每个 workspace 默认配独立 git worktree、独立分支、独立服务端口和反向代理 hostname，让多个 agent 可以同时改一个 repo 而不踩车。
  - **Untrusted relay**：relay 设计成不可信组件，daemon 不接受未完成 ECDH 握手的命令，钥匙绑在配对二维码里，relay 看到的只是 IP / 时序 / 包大小 / session id。
  - **Agent self-orchestration**：通过 Paseo MCP + orchestration skills，把 agent 也当成调度者；`paseo run` + `paseo wait` 的 shell 习惯让 agent 可以直接用 bash 写多 agent 工作流。
- 依赖 / 生态：Node.js / TypeScript（daemon、CLI、client）、React Native + Expo（移动端）、Electron（桌面）、WebSocket、NaCl `box`（ECDH + XSalsa20-Poly1305）、ONNX runtime（本地 Parakeet / Kokoro）、Anthropic Claude Agent SDK、OpenAI Codex CLI、ACP（Agent Client Protocol）、MCP、OpenAI Realtime API（可选）、`gh` CLI（PR / 检查集成）、git worktrees，以及 `npx skills` 等社区工具。

## 5. 安装与快速开始

### 5.1 桌面 app（推荐）
1. 从 https://paseo.sh/download 或 GitHub Releases 下载对应平台安装包。
2. 启动后桌面 app 自动起内嵌 daemon，无需另装。
3. 在 Settings 里扫描二维码即可把 iOS / Android 端关联到这台机器的 daemon。

### 5.2 CLI / headless
适合服务器、远程开发盒、Mac mini、homelab 等。
```bash
npm install -g @getpaseo/cli
paseo
```
首次运行会在终端打印二维码 / pairing offer URL，移动端或另一台 CLI 都可以用它接入。

### 5.3 必要前置
Paseo 自己不内置任何 model 调用，必须先在本机装好至少一个 provider CLI 并完成认证：
```bash
# 任选其一或多选
claude        # Claude Code https://docs.anthropic.com/en/docs/claude-code
codex         # OpenAI Codex CLI https://github.com/openai/codex
gh copilot    # GitHub Copilot CLI
opencode      # OpenCode https://opencode.ai/
pi            # Pi https://pi.dev
# 其他 ACP agents 可在 app 里一键安装，或在 ~/.paseo/config.json 中手动添加
```
建议同时装好 `gh`（GitHub CLI），Paseo 在 PR / 检查 / worktree 等场景会复用它。

### 5.4 最小可用流程
```bash
# 启动 agent，跑一段简单任务
paseo run --provider claude/opus-4.7 "解释当前 repo 的主要模块"

# 列出当前目录的运行中 agent
paseo ls

# 实时挂上去看输出
paseo attach <id>

# 追加任务
paseo send <id> "再补一下测试入口"

# JSON Schema 受限输出（适合脚本）
paseo run --provider codex --output-schema '{"type":"object","properties":{"summary":{"type":"string"}},"required":["summary"]}' "总结最近一次 release notes"
```

### 5.5 Worktree 并行
```bash
paseo run --worktree feature-auth --base main --provider codex/gpt-5.5 "实现 OAuth 登录"
paseo run --worktree feature-billing --base main --provider claude/opus-4.7 "拆分计费模块"
paseo worktree ls
paseo worktree archive feature-auth
```
仓库根加一份 `paseo.json` 描述 setup / scripts / services：
```json
{
  "worktree": {
    "setup": "npm ci\ncp \"$PASEO_SOURCE_CHECKOUT_PATH/.env\" .env",
    "teardown": "npm run db:drop || true",
    "terminals": [
      { "name": "logs", "command": "tail -f dev.log" }
    ]
  },
  "scripts": {
    "test": { "command": "npm test" },
    "web":  { "type": "service", "command": "npm run dev -- --port $PASEO_PORT", "port": 3000 },
    "api":  { "type": "service", "command": "npm run api -- --port $PASEO_PORT" }
  }
}
```

### 5.6 远程 / 跨机器接管
```bash
# 在 daemon 那台机器生成 pairing offer
paseo daemon pair --json     # 输出 { url, qr, ... }

# 在另一台机器 / 手机端
paseo --host "$OFFER_URL" ls
PASEO_HOST="$OFFER_URL" paseo run "在远端机器跑一遍完整测试"

# 直连内网 / Tailscale，建议同时开密码
paseo daemon set-password
paseo --host "tcp://100.x.y.z:6767?password=$PASSWORD" ls
```

### 5.7 安装 orchestration skills
```bash
npx skills add getpaseo/paseo
# 之后在任意 agent 里：
# /paseo-handoff hand off the auth fix to codex in a worktree
# /paseo-loop keep trying until tests pass, max 5 iterations
# /paseo-committee why is the websocket dropping under load?
# /paseo-advisor what is the UX risk in this flow?
```

## 6. 在 Claude Code 中的用法
- 适合让 Claude Code 做什么：
  - 用 Paseo 跑 Claude Code 当主力实现 agent，再用 `paseo run --provider codex` 起一个独立 review agent，形成 **Claude 实现 + Codex review** 的双 agent 工作流（与本仓库的 `humanize` 笔记理念一致）。
  - 让 Claude Code 通过 Paseo MCP 自己创建子 agent / worktree / schedule，把长任务拆成多 worker 并行处理。
  - 在桌面 / 手机端用 Claude Code 进入 plan / bypass 等不同 agent mode（`paseo agent mode <id> plan`），并把权限请求集中在 `paseo permit ls/allow/deny` 里。
  - 让 Claude Code 帮你写 `paseo.json`、ACP 自定义 provider 配置、`AGENTS.md` 等仓库级编排规则。
- 推荐提示词 / 工作流：
```text
请只读分析当前 repo 的目录结构和测试命令，然后写一份建议的 paseo.json：
- worktree.setup 要安装依赖、复制 .env、跑必要的 db migrate
- scripts 里至少有 test、lint、generate
- 标注两个 service：web 和 api，使用 $PASEO_PORT
不要修改文件，先把方案和风险列出来。
```
```text
请用 Paseo MCP 工具：
1. 在 worktree feature-x 里再起一个 codex agent 跑 `npm test`；
2. wait_for_agent 直到它结束，capture 最后输出；
3. 把结论汇总后再让我决定下一步。
```
```bash
# 在 Paseo daemon 上启动 Claude Code 实现 agent
paseo run --provider claude/opus-4.7 --worktree feature-auth \
  "实现 OAuth 登录，记得加测试，关键边界情况列清单。"
```
- 注意事项：
  - **Anthropic 2026-06-15 政策对 Paseo 的影响**：Anthropic 在官方支持文档（"Use the Claude Agent SDK with your Claude plan"）里把 Claude Code 用量切成两块——"interactive usage"（终端 `claude`、IDE 插件、Claude Desktop、Web/Mobile chat）走主套餐限额；"programmatic usage"（`claude -p`、Claude Agent SDK、Claude Code GitHub Actions、所有通过 SDK 认证的第三方 app）走另一份月度 credit pool（Pro $20 / Max 5x $100 / Max 20x $200 / Team Standard $20-seat / Team Premium $100-seat，**不滚存**，用完后若启用 usage credits 走 pay-as-you-go API 价）。Paseo 通过 Claude Agent SDK 启动 `claude`，机制和 Claude Desktop 一样，但 Claude Desktop 被 Anthropic 列入白名单算 interactive，Paseo 没有，所以 **Paseo 内的 Claude 会话被记为 programmatic usage**。Paseo 内置终端里直接跑的 `claude` 仍走主套餐 interactive 限额。
  - 规避 / 控制额度的具体步骤：
    1. **先建立"双账本"心智**：把 Paseo workspace 内创建的 Claude session 视作消耗 Agent SDK 月度 credits，把 Paseo 内置 terminal tab 里手动跑 `claude` 视作消耗主套餐限额；其它 provider（Codex / OpenCode / Pi / ACP agents）不受这条政策影响。
    2. **重活儿放在 Paseo 终端的原生 `claude`**：在 Paseo 里开一个 terminal session，直接 `claude` 进入 TUI 写代码；Paseo 的 worktree 管理、远程接入、diff 查看、GitHub 集成、agent supervision、relay 等附加价值仍然有，但用量计入 interactive，不再吃 SDK credits。
    3. **Paseo agent session 留给"可结构化、可截断"的任务**：例如 `--output-schema` 的结构化 review、handoff 短任务、`/paseo-advisor` 的一次性二次意见，避免无限聊天。
    4. **给 schedule 与 loop 设硬上限**：`paseo schedule create --every 30m --max-runs 16 --expires-in 10h ...`，`/paseo-loop --max-iterations N --max-time T`，并在 prompt 里写明 verifier 退出条件，防止过夜死循环烧光 credits。
    5. **把廉价 / 大量任务路由到非 Claude provider**：planner、批量 grep、文档生成、低风险 review 用 Codex / OpenCode / Qwen / GLM / Gemini 等，Claude 只承担关键决策或最难的 diff，可结合 `agents.providers` 写多 profile 限定模型。
    6. **改走 pay-as-you-go API key（不消耗 SDK credit pool）**：在 `~/.paseo/config.json` 里 `extends: "claude"` 加一份自建 profile，把 `env.ANTHROPIC_API_KEY` 指向你公司账单或个人 console 直付的 API key；这样调用计入 API usage 而不是 Claude plan 的 SDK pool，仍可使用 prompt caching、Sonnet 4.6/Opus 4.7 等模型。注意这是替代方案而非"绕过"——你需要自己承担按 token 计的 API 费用，并理解它和 Claude plan 的合规边界。
    7. **同理可指向兼容 endpoint**：例如团队自建网关、Z.AI、Alibaba/Qwen 兼容 endpoint（见 `public-docs/custom-providers.md` / `docs/custom-providers.md`），把 `ANTHROPIC_BASE_URL` 改写到目标网关；只在合同允许的范围内使用。
    8. **打开 usage credits / pay-as-you-go 兜底**：在 Anthropic console 启用 usage-based billing，credit pool 用尽时自动切到 API 价格而不是任务直接失败；然后设置费用预警和上限。
    9. **限制谁能用你的 Paseo daemon**：跨 LAN / 给 teammate 共享前先 `paseo daemon set-password`（bcrypt 落盘，不存明文），并通过 `daemon.hostnames` 限制可达 host，防止他人无意中消耗你的 SDK 月度 credits。
    10. **跑前先核账**：每月初 / 大动作前在 Anthropic 控制台看 SDK credit 余额、API usage credits 余额，再决定是按 Paseo agent 跑还是切回 terminal `claude`。
    11. **Anthropic 政策本身仍在演变**：上线日期、白名单列表、credit 数额、`-p` 与 SDK 的归属都可能变；以官方支持文档（参见本节 Sources 链接）为准，本节信息基于上游 docs 在 2026-06-13 的写法。
  - daemon 默认只监听 `127.0.0.1:6767`，绑到 `0.0.0.0` 或 LAN 之前请先 `paseo daemon set-password`，并合理配置 `daemon.hostnames` 以防 DNS rebinding。
  - 让 Claude Code 跨 worktree 操作时，给它明确指出 `--worktree` / `cwd` 参数，避免把改动写错位置。
  - Paseo 不会托管 Claude 的凭据；如果换号或换密钥，需要在 `~/.claude/` 而不是 Paseo 配置里改。

## 7. 在 Codex 中的用法
- 适合让 Codex 做什么：
  - 在 Paseo 里把 Codex 设成 “planner / verifier”：例如 `/paseo-handoff` 让 Claude 先实现，再 `paseo run --provider codex --output-schema` 让 Codex 用 JSON Schema 给出 PASS/FAIL 判定。
  - 让 Codex 阅读 `packages/server/src/agent/`、`packages/protocol/` 等关键模块，研究 daemon API、MCP 工具、provider 抽象的实现，再产出适合本团队的扩展计划（例如新增自定义 ACP provider、自托管 relay）。
  - 用 Codex 帮你维护项目的 `AGENTS.md`、`paseo.json`、自定义 provider profile（多 endpoint 多套 key），把多 provider 的策略沉淀在仓库里。
  - 让 Codex 配合 `paseo schedule` 写过夜重构 / 长 build watch / 早间 GitHub triage 的循环 prompt：固定输出格式、明确停止条件、保留可回滚边界。
- 推荐提示词 / 工作流：
```text
你是当前 worktree 的 review agent。任务：
1. 拉取当前 diff（`git diff --staged` / `git diff`），按风险排序列出 P0/P1/P2 问题；
2. 严格按这个 JSON Schema 输出：
   {"type":"object","properties":{
     "verdict":{"type":"string","enum":["pass","fail","needs_changes"]},
     "blockers":{"type":"array","items":{"type":"string"}},
     "notes":{"type":"array","items":{"type":"string"}}
   },"required":["verdict","blockers","notes"]}
3. 不要修改文件。
```
```bash
# Codex 跑结构化 review，用 --output-schema 与 jq 串起循环
paseo run --provider codex/gpt-5.5 \
  --output-schema review-schema.json \
  "review the staged diff against the team's review checklist"
```
```text
请只读阅读 Paseo 仓库的 packages/server/src/agent/agent-manager.ts、agent-storage.ts、mcp-server.ts，
列出 agent 生命周期状态、持久化策略、MCP 暴露的工具集，并指出二次开发的扩展点。
```
- 注意事项：
  - Codex CLI 的 sandbox / approval 控制在 Codex 一侧，Paseo 不会改它的安全边界；不要因为“都在 Paseo 里”就放宽 Codex 权限。
  - `--output-schema` 不能与 `--detach` 同用；如果要后台跑结构化输出，需要自己封一层（先 detach + wait + logs）。
  - 当 Codex 通过 MCP 调用 `create_agent` / `update_agent` 等工具时，记得在 prompt 里要求它先 `list_agents` / `get_agent_status`，避免重复创建或踩到不归它管的会话。
  - 跨 daemon 工作（`--host` 接 relay）时，`--cwd` 要换成远端真实路径，否则 worktree / scripts 解析会失败。

## 8. 典型生产力场景
| 场景 | 怎么用 | 产出 |
|---|---|---|
| 多 agent 并行修同一仓库 | 每个 feature 一个 workspace + worktree，分配不同 provider | 互不干扰的分支、独立 dev server，可统一 PR 起 |
| 桌面起任务 → 手机接管 | 桌面 app 启动 agent，扫码把 iOS/Android 端接到 daemon | 通勤 / 离开工位时仍能审阅 diff、追加指令、批准权限 |
| 远程开发盒驱动 | Mac mini / VPS / Docker 跑 daemon，本地通过 relay offer URL 接入 | 把重计算 / 大模型 CLI 留在远端，前端用任意客户端 |
| Claude 实现 + Codex review 闭环 | `/paseo-handoff` + 双 agent + `--output-schema` JSON | 结构化 review 结论，CI / shell loop 可消费 |
| 过夜 / 长任务循环 | `paseo schedule create --every 30m` + heartbeat 或 `--cron` | 计划性的重构 / 巡检 / 报告，跨夜不掉线 |
| Build / CI babysitter | `paseo schedule create --every 5m "watch the release build"` | 自动诊断、修复并重跑发布构建 |
| 每日 GitHub triage | cron schedule + Codex / GLM 等 provider | issue / PR / failing checks 早间摘要 |
| 移动端 voice 操控 | 本地 Parakeet+Kokoro 或 OpenAI Realtime + Voice LLM provider | 语音派发任务、听 agent 反馈、`speak` MCP 工具回应 |
| 团队多 endpoint 共用 | `agents.providers` 写多 profile（Z.AI / Qwen / 自有代理） | 同一前端切不同上游模型，凭据隔离 |
| ACP 生态接入 | 应用内一键装 Cursor / Gemini CLI / Hermes / Kimi / Qwen Code 等 | 统一 UI 操作多家 agent |

## 9. 适合 / 不适合
### 适合
- 你已经同时使用多个 coding agent CLI，并希望统一调度、监控、远程接管。
- 你需要从手机或第二台机器实时观察 / 介入正在跑的 agent，但不想接受 SaaS 把代码送出去。
- 你想做 “planner + worker + reviewer” 这种多 agent 编排（Ralph loop、committee、handoff、advisor）。
- 你用 git worktree 做并行开发，并且希望每个 worktree 自动跑 setup、自带独立 dev/db service 端口。
- 你想研究 daemon-client agent 平台、ACP 协议接入、E2E 加密 relay、移动端 voice + agent UX 等实现。

### 不适合
- 你只用一个 agent CLI 且全部工作都在桌面终端完成，不需要远程 / 移动 / 多 agent 编排。
- 公司禁止运行任何 daemon、占用本机后台进程，或不允许在 LAN / 公网暴露任何接口（即使是 relay 出站连接也算）。
- 你的策略禁止使用第三方 relay（即便端到端加密），同时你又不愿自托管 relay 或部署 Tailscale 之类 VPN。
- 你期望它自带模型调用 / 计费 / SaaS 后端：Paseo 没有自带 agent / 模型，所有 LLM 调用走你已认证的底层 CLI。
- 你需要稳定的企业 SLA / 路线图保证：上游自述为单人维护，issues 可能滞后，紧急问题官方建议走 Discord。
- 你需要把 Paseo 嵌进闭源商业产品再分发：项目是 AGPL-3.0-or-later，需要评估合规义务。

## 10. 同类工具定位
- **vs. Warp / Cursor / Zed 等带 agent 的 IDE 或 terminal**：那些工具自带 agent、UI 偏 IDE/terminal 一体；Paseo 不带 agent，只做编排和多端接入，定位更接近 “Docker for coding agents” 而不是 IDE。
- **vs. tmux + SSH + 多 CLI 的传统组合**：Paseo 提供 workspace/session/worktree、E2EE relay、移动端、MCP、计划任务和 orchestration skills，而 tmux 仅做终端复用，缺少这些抽象。
- **vs. agent 框架（LangGraph / CrewAI / AutoGen 等）**：那些更偏“在代码里组装 agent 拓扑”，Paseo 偏“用 CLI / UI 编排 *已有* CLI agent”，对开发者日常工作流更直接。
- **vs. AI gateway（如 sub2api、各类 proxy）**：gateway 解决“多上游、多账号、配额和路由”，Paseo 解决“多 agent 客户端、多 worktree、多端控制”；两者可叠加（用 gateway 喂 provider，用 Paseo 编排 agent）。
- **vs. Claude Code Desktop / Codex Desktop 这类官方桌面 app**：Paseo 把多个官方 CLI 装进同一个 self-hosted、跨端、可远程的容器，但带来额外的部署 / 安全面，需要自己评估。

## 11. 关键文件 / 目录导读
| Path | 作用 |
|---|---|
| `README.md` / `README.zh-CN.md` | 项目定位、安装入口、CLI / Skills / Development 概览 |
| `CHANGELOG.md` | 版本记录，按 minor / patch 列出 Added / Improved / Fixed |
| `CLAUDE.md` | 仓库级 Claude Code / coding agent 工程指南 |
| `public-docs/index.md` | docs 主入口（Getting started） |
| `public-docs/why.md` | 项目立场与“它不是什么”的边界 |
| `public-docs/configuration.md` | `~/.paseo/config.json`、env 变量、密码、日志、worktree root 等 |
| `public-docs/security.md` | daemon-client 模型、relay E2EE、DNS rebinding、密码、agent 凭据 |
| `public-docs/workspaces.md` / `worktrees.md` | workspace / session / worktree 模型与 `paseo.json` |
| `public-docs/providers.md` / `supported-providers.md` / `custom-providers.md` | provider 抽象、原生支持、ACP 目录、自定义 endpoint / profile |
| `public-docs/cli.md` | CLI 全量命令与 `--host` 远端 / pairing offer 用法 |
| `public-docs/mcp.md` | Paseo MCP 工具集（agents / terminals / schedules / providers / worktrees / permissions / voice） |
| `public-docs/skills.md` | orchestration skills（`/paseo-handoff`、`/paseo-loop`、`/paseo-committee`、`/paseo-advisor`） |
| `public-docs/schedules.md` | 间隔 / cron 调度、`--target self` heartbeat、远端 daemon 注意事项 |
| `public-docs/voice.md` | 本地 / OpenAI 语音栈、Parakeet / Kokoro 模型、env 变量 |
| `public-docs/claude-code.md` | Anthropic 2026-06-15 政策下 Paseo 的 programmatic usage 说明 |
| `docs/architecture.md` | daemon / relay / clients 详细架构与各 package 职责 |
| `docs/data-model.md` / `docs/agent-lifecycle.md` / `docs/providers.md` | timeline、agent 生命周期、provider 适配细节 |
| `docs/custom-providers.md` | 完整 `agents.providers` 字段参考（含 `extends`、`env`、`command`、`additionalModels` 等） |
| `packages/server/` | daemon 主体（agent manager / storage / MCP / relay / schedule / loop / chat） |
| `packages/protocol/` | WebSocket / 二进制帧 / config schema 共享类型 |
| `packages/client/` | daemon client SDK，多端共用 |
| `packages/cli/` | `paseo` CLI 实现 |
| `packages/app/` / `packages/desktop/` | Expo 移动 / Web 客户端 + Electron 桌面壳 |
| `packages/relay/` | 自托管 relay 实现 |
| `packages/website/` / `public-docs/` | docs 站源与公开文档 |
| `skills/paseo*` | orchestration skills 源码 |
| `paseo.json` | 仓库自身的 worktree / scripts / services 配置参考 |

## 12. 学习与掌控建议
1. **先读 `public-docs/why.md` 和 `public-docs/index.md`**：建立 “Paseo 不是 agent，它编排你已有的 agent” 这个心智模型，再决定要不要替换现有工作流。
2. **从桌面 app 单 provider 起步**：装 Claude Code 或 Codex CLI 之一，桌面 app 跑 1-2 个简单任务，熟悉 workspace / session / worktree 与 timeline。
3. **为常用项目写 `paseo.json`**：`setup`、`teardown`、`scripts`、`services`、`terminals` 一次配齐，可以直接把多 agent 并行的可重复性拉满。
4. **接 CLI 到日常脚本**：`paseo run` + `paseo wait` + `paseo logs` + `--output-schema` 是最容易把 agent 嵌进 shell / CI 的入口；先把 review、test、changelog 这类小任务跑通。
5. **再上 orchestration skills 与 MCP**：理解 `/paseo-handoff`、`/paseo-loop`、`/paseo-committee`、`/paseo-advisor` 的差异，并研究 `public-docs/mcp.md` 里 agent / terminal / schedule / worktree 的 MCP 工具集。
6. **远程 / 移动端必须先做安全配置**：阅读 `public-docs/security.md`，决定 relay vs. Tailscale，是否开 daemon 密码、限制 `daemon.hostnames`、是否绑非 localhost 接口；不要在没密码的情况下绑 `0.0.0.0`。
7. **关注上游节奏**：项目目前是单人维护、迭代快，建议跟踪 `CHANGELOG.md` 和 GitHub releases；急事走 Discord，自己依赖关键流程时锁住已知稳定版本。
8. **了解 license 含义**：自己 fork / 商用前评估 AGPL-3.0-or-later 的网络分发义务，特别是把 Paseo 嵌入 SaaS 的场景。

## 13. Sources
- https://github.com/getpaseo/paseo
- https://github.com/getpaseo/paseo/blob/main/README.md
- https://github.com/getpaseo/paseo/blob/main/CHANGELOG.md
- https://github.com/getpaseo/paseo/blob/main/package.json
- https://github.com/getpaseo/paseo/tree/main/packages
- https://github.com/getpaseo/paseo/blob/main/docs/architecture.md
- https://github.com/getpaseo/paseo/blob/main/docs/custom-providers.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/index.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/why.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/cli.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/configuration.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/security.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/workspaces.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/worktrees.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/providers.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/supported-providers.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/custom-providers.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/mcp.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/skills.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/schedules.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/voice.md
- https://github.com/getpaseo/paseo/blob/main/public-docs/claude-code.md
- https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan
- https://paseo.sh/docs
