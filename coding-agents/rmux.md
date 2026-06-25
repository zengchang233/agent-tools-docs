# rmux

> Repo: https://github.com/Helvesec/rmux
> Category: `coding-agents`
> Last checked: 2026-06-24
> Source snapshot: `main` 分支检查于 `0eae60d174dc63131db23cb209bf3aada7d24c54`（repo version `0.6.1`，2026-06-17）；官方站点 `rmux.io/docs`、crates.io、PyPI、npm registry 同日核验。

## 1. 一句话总结
RMUX 是一个用 Rust 写的现代终端 multiplexer：它像 tmux 一样管理本地 shell / pane / session，但额外提供 typed SDK、Ratatui widget 和端到端加密的 browser Web Share，适合把 Claude Code、Codex、长时间运行的 CLI/TUI、测试命令和远程协作会话放进一个可编程的本地终端运行时。

它不是 LLM coding agent 本身，更像 **agent-friendly terminal runtime / automation substrate**：让 agent 会话不丢、可脚本化、可快照、可等待输出、可只读或读写地分享给浏览器。

## 2. 项目目的与核心问题
- 目标用户：重度使用 terminal、tmux、Claude Code、Codex、OpenCode、Gemini CLI 等 CLI agent 的开发者；想从代码里驱动 shell/TUI 的工具作者；想把终端会话安全分享给同事、手机或浏览器的工程团队；想研究跨平台 PTY / IPC / terminal automation 的开发者。
- 解决的问题：
  - 传统 tmux 很适合人类长时间挂会话，但对程序化操作不友好：脚本通常要拼 tmux command、解析文本输出或依赖 control mode。
  - Coding agent 的一次任务可能跑很久，需要 detach / reattach、分屏观察、捕获输出、等待特定文本、在另一个 pane 里跑测试或 review。
  - 远程展示终端时，常见方案要么让 relay/server 看到明文，要么不是完整 multiplexer，要么缺少 operator / spectator 等细粒度角色。
- 为什么值得关注：
  - RMUX 把 tmux 风格 CLI、daemon-backed typed SDK、Ratatui 嵌入、浏览器共享和跨平台 Windows ConPTY 支持放在同一个开源 Rust workspace 中。
  - 它把“终端状态”变成可由 SDK 查询、等待、快照和分享的对象，而不只是线性滚屏文本。
  - 对 Claude Code / Codex 用户来说，它可以成为比裸终端更稳定的 agent 会话底座，但比 Paseo / Warp 这类完整 agent 平台更底层、更本地化。

## 3. 核心能力
- **tmux-like multiplexer**：提供 90+ tmux 风格命令，覆盖 `new-session`、`split-window`、`send-keys`、`attach-session`、`capture-pane`、`wait-for`、`list-commands` 等常见会话 / 窗口 / pane 操作。
- **本地 daemon runtime**：shell、PTY、session、window、pane、scrollback 和 process lifecycle 都由本地 daemon 管理；客户端通过本地 IPC 连接。
- **typed SDK**：Rust SDK `rmux-sdk` 可 `connect_or_start()` 本地 daemon，创建/复用 session，定位 window/pane，发送文本或按键，等待可见文本，捕获 `PaneSnapshot`，读取 output stream，并控制 Web Share。
- **Python / TypeScript SDK**：官方文档当前给出 `python -m pip install librmux` 与 `npm install @rmux/sdk`，适合在 pytest、Node test harness、自动化脚本里驱动终端。
- **Ratatui 集成**：`ratatui-rmux` 把 daemon-backed pane snapshot 渲染进 Rust TUI，不需要自己写 terminal emulator 或 VT parser。
- **Web Share**：`rmux web-share` 可把一个 pane/session 暴露给浏览器，终端执行仍留在本机；browser 收发加密 terminal frames，支持 operator（可输入）和 spectator（只读）角色、PIN、TTL、最大观众数、tunnel provider、自托管 static frontend。
- **跨平台 PTY / IPC**：Linux/macOS 使用 Unix PTY + Unix socket；Windows 使用 ConPTY + per-user named pipe，不要求 WSL。
- **配置兼容**：读取 `.rmux.conf`；如果没有 RMUX 配置，Unix/macOS/Windows 按平台约定查找配置，并可 best-effort fallback 到 tmux.conf；支持部分 tmux plugin 脚本通过私有 `tmux` shim 路由回 RMUX。
- **诊断与验证**：`rmux -V`、`rmux diagnose --human`、`rmux diagnose --json`、`rmux capabilities --json` 适合在脚本和 agent prompt 中先确认运行时能力。

## 4. 实现思路 / 架构拆解
- 总体形态：**CLI / SDK / TUI widget / Web Share frontend** 连接同一个本地 daemon。daemon 是终端状态和进程生命周期的权威；客户端只发送 typed request、读取响应或渲染输出。
- 核心流程：
  1. 用户执行 `rmux` 命令或 SDK 调用 `connect_or_start()`；如果本地 daemon 不存在，则启动或连接默认 endpoint。
  2. daemon 创建/复用 session、window、pane，并用平台 PTY 后端启动 shell 或子进程。
  3. CLI attach 模式渲染 pane；普通命令和 SDK 通过 IPC 操作 daemon 状态。
  4. SDK 可对 pane 执行 `send_text` / `send_key`，等待屏幕出现指定文本，捕获 snapshot 或订阅输出流。
  5. Web Share 场景下，daemon 为选定 pane/session 创建 WebSocket share；浏览器加载静态 frontend 后与本地 daemon 做密钥协商，tunnel 只转发密文。
- 关键 crates（来自 workspace 与 README 架构表）：
  - `rmux`：公开 CLI binary 与 hidden daemon entrypoint；`src/main.rs`、`src/daemon_main.rs` 是入口。
  - `rmux-core`：session、pane、layout、format、hook、buffer 等核心模型。
  - `rmux-pty`：PTY allocation、resize、child process control；隔离平台终端边界。
  - `rmux-ipc`：Unix socket / Windows named pipe 等本地 IPC endpoint 与 transport。
  - `rmux-server`：Tokio daemon、请求分发、状态管理。
  - `rmux-client`：本地客户端和 attach-mode plumbing。
  - `rmux-sdk`：公开 Rust SDK facade；不是 CLI parser，也不是 tmux control-mode wrapper。
  - `ratatui-rmux`：把 `PaneSnapshot` 渲染为 Ratatui widget。
  - `rmux-web-crypto`：Web Share 的 X25519 + ML-KEM-768 hybrid handshake、ChaCha20-Poly1305 record layer 与 WASM 边界。
  - `rmux-types`、`rmux-proto`、`rmux-os`：共享类型、IPC DTO/framing、OS helper。
  - `rmux-render-core`：浏览器和 native host 共用的纯数据渲染核心，当前 workspace-internal。
- 平台边界：
  - Linux/macOS：Unix PTY + Unix socket，默认 endpoint 类似 `/tmp/rmux-{uid}/default`。
  - Windows：ConPTY + per-user named pipe。
- 设计亮点：
  - **typed automation first**：把 pane/session 抽象成 SDK handle，避免从 tmux 文本输出里猜状态。
  - **local-first**：默认 shell 和进程都在本机；网络能力主要来自显式 `web-share`。
  - **web share 分层清楚**：terminal 执行、本地 daemon、静态 browser frontend、tunnel provider 是分开的，tunnel 不应看到 terminal 明文。
  - **兼容但不等同于 tmux**：命令和配置向 tmux 靠拢，但实现是 Rust 从头写的 runtime，适合新自动化接口和 Windows 原生支持。

## 5. 安装与快速开始

### 5.1 安装 RMUX binary
Linux / macOS 的便捷脚本：
```bash
curl -fsSL https://rmux.io/install.sh | sh
```

Cargo：
```bash
cargo install rmux --locked
```

macOS Homebrew：
```bash
brew install rmux
```

Windows：
```powershell
irm https://rmux.io/install.ps1 | iex
winget install rmux
# 或：scoop install rmux / choco install rmux
```

Linux 也提供 signed APT / DNF repository；release assets 同时发布 `.deb`、`.rpm`、tarball、Windows zip 与 SHA256 checksums。

### 5.2 最小 CLI 流程
```bash
# 创建一个后台 session
rmux new-session -d -s work

# 横向 split 一个 pane
rmux split-window -h -t work

# 给 session/pane 发送命令
rmux send-keys -t work 'echo "hello from rmux"' Enter

# 接回交互 UI
rmux attach-session -t work
```

常用自查命令：
```bash
rmux -V
rmux diagnose --human
rmux diagnose --json
rmux capabilities --json
rmux list-commands
rmux web-share --help
```

### 5.3 SDK 安装
Rust：
```bash
cargo add rmux-sdk
cargo add ratatui-rmux
```

Python：
```bash
python -m pip install librmux
```

TypeScript / Node：
```bash
npm install @rmux/sdk
```

Rust SDK 示例：
```rust
use std::time::Duration;
use rmux_sdk::{EnsureSession, Rmux, SessionName};

#[tokio::main]
async fn main() -> rmux_sdk::Result<()> {
    let rmux = Rmux::builder().connect_or_start().await?;
    let session = rmux
        .ensure_session(
            EnsureSession::try_named(SessionName::new("ci")?)?
                .create_or_reuse()
                .detached(true),
        )
        .await?;

    let pane = session.pane(0, 0);
    pane.send_text("printf 'ready\\n'\n").await?;
    pane.expect_visible_text()
        .to_contain("ready")
        .timeout(Duration::from_secs(5))
        .await?;

    Ok(())
}
```

### 5.4 Web Share
```bash
# 分享当前 pane/session 到本机浏览器
rmux web-share

# 分享已存在的 session
rmux new-session -d -s demo
rmux web-share -t demo

# 使用内置 public tunnel preset
rmux web-share --tunnel-provider localhost-run

# 只允许只读观众
rmux web-share --spectator-only --ttl 3600

# 管理 share
rmux web-share list
rmux web-share lookup <share-id>
rmux web-share stop <share-id>
rmux web-share off
```

安全习惯：operator URL 等同于“可在你的本地 shell 输入命令”的凭据；公开分享优先用 spectator、PIN、TTL、最大观众数和边缘限流。

## 6. 在 Claude Code 中的用法
- 适合让 Claude Code 做什么：
  - 把 RMUX 当作 Claude Code 的持久终端底座：长任务、测试循环、日志观察、多个 pane 中的 planner / implementer / reviewer 都可以 detach 后继续运行。
  - 用 Claude Code 维护 `.rmux.conf`、团队 terminal workflow、agent 分屏模板和 Web Share 安全策略。
  - 让 Claude Code 分析 RMUX repo 本身的 Rust workspace：`rmux-core`、`rmux-server`、`rmux-sdk`、`rmux-web-crypto`、`src/cli_args*`。
  - 用 RMUX Web Share 给同事只读观察 Claude Code 正在做什么，避免直接共享 SSH 会话或完整桌面。
- 推荐工作流：
```bash
# 准备一个 agent workspace
rmux new-session -d -s agents
rmux split-window -h -t agents

# 左侧跑 Claude Code，右侧跑测试/日志/另一个 agent
rmux send-keys -t agents 'claude' Enter
rmux send-keys -t agents 'npm test -- --watch' Enter

# 需要离开时 detach；回来继续 attach
rmux attach-session -t agents
```

```text
请阅读当前项目的 README、AGENTS.md 和测试命令，帮我设计一份 .rmux.conf：
- prefix / split / resize 要适合 Claude Code + 测试 pane 的双栏工作流；
- 保留 tmux 用户容易理解的快捷键；
- 不要假设 RMUX 支持未验证的 tmux plugin；
- 输出配置和每条配置的理由。
```

```text
请只读分析 Helvesec/rmux 的架构：
1. 解释 daemon / IPC / PTY / SDK 的边界；
2. 列出 rmux-sdk 能否作为 Claude Code 自动化测试 harness；
3. 标出会执行 shell 或暴露浏览器访问的安全点。
不要修改文件。
```
- 注意事项：
  - RMUX 只是 shell runtime，不会改变 Claude Code 自身的权限、模型、approval 或 sandbox；Claude 能在 pane 里做什么，取决于你启动 Claude Code 时给它的权限。
  - `send-keys` / SDK `send_text` 是真实 shell 输入；让 Claude 自动生成这些命令时，要先审查 destructive command、凭据、生产环境连接和 long-running loop。
  - Web Share operator 链接不要发到公开频道；演示场景尽量使用 `--spectator-only`。

## 7. 在 Codex 中的用法
- 适合让 Codex 做什么：
  - 把 Codex CLI 放进 RMUX session 中，保持长时间实现 / review / 测试任务不断线。
  - 用 Codex 研究 RMUX 的 Rust 源码、写 SDK 示例、补充团队文档或给现有 tmux 工作流做迁移评估。
  - 让 Codex 写一个小型 automation harness：启动 RMUX session、发送命令、等待输出、捕获 pane snapshot，再把结果转成 CI artifact。
  - 在 RMUX 多 pane 中并行运行 Codex、Claude Code、测试 watcher、server logs，但用 git worktree 或明确角色避免多个 agent 同时写同一文件。
- 推荐工作流：
```bash
rmux new-session -d -s codex-review
rmux send-keys -t codex-review 'codex' Enter
rmux attach-session -t codex-review
```

```text
目标：评估当前 repo 是否适合用 RMUX 承载 Codex 长任务。
请只读检查：
- 是否有稳定的测试命令；
- 是否需要多个服务/日志 pane；
- 是否有禁止 Web Share 或远程 tunnel 的安全要求；
然后输出一份 RMUX session 布局建议和风险清单。
```

```text
请阅读 rmux-sdk 的 examples 和 tests，写一个 Rust 示例：
- connect_or_start 本地 daemon；
- ensure_session("ci")；
- 发送 `cargo test`；
- wait_for_text 或 collect output until exit；
- 失败时打印 pane snapshot。
先给方案，不要直接改文件。
```
- 注意事项：
  - Codex 的 sandbox / approval 仍在 Codex 一侧；RMUX 只是让 Codex 会话持久化和可视化，不应绕过审批。
  - 如果 Codex 在 RMUX pane 里再启动另一个 agent，要把“谁能写文件、谁只读 review、谁跑测试”写清楚，否则很容易产生并发 diff 冲突。
  - 让 Codex 生成 Web Share/tunnel 命令时，要求它默认 spectator-only，除非你明确需要 operator 链接。

## 8. 典型生产力场景
| 场景 | 怎么用 | 产出 |
|---|---|---|
| 长时间 coding agent 会话 | `rmux new-session -d -s agents` 后在 pane 中运行 `claude` / `codex` | 可 detach/reattach 的 agent 任务，不因终端关闭而丢失 |
| 实现 + 测试双 pane | 一个 pane 跑 agent，另一个 pane 跑 `npm test --watch` / server logs | agent 与验证输出同屏，可随时捕获 |
| 多 agent 对照 | 左侧 Claude Code，右侧 Codex review，第三个 pane 跑测试 | 低成本 planner/worker/reviewer 工作流 |
| TUI/CLI 自动化测试 | SDK 发送输入、等待可见文本、捕获 snapshot | 比 expect/截图更结构化的终端测试 harness |
| 团队远程排障 | `rmux web-share --spectator-only -t incident` | 同事或手机浏览器只读观察终端状态 |
| Rust TUI 嵌入真实 pane | `ratatui-rmux` 渲染 daemon snapshot | 在自研 TUI 中嵌入运行中的 shell/pane |
| tmux 迁移试点 | 用现有 `.tmux.conf` best-effort fallback + `rmux list-commands` 核对 | 了解哪些命令/配置可迁移，哪些需要改写 |

## 9. 适合 / 不适合
### 适合
- 你已经习惯 tmux/zellij，但想要更强的 SDK 自动化能力。
- 你经常跑 Claude Code / Codex / OpenCode 这类 CLI agent，需要持久 session、分屏和可捕获输出。
- 你想从 Rust/Python/TypeScript 测试代码中驱动真实 terminal，而不是 mock CLI。
- 你需要临时把本地终端分享给浏览器，并希望 relay/tunnel 只看到密文。
- 你需要 Windows 原生 ConPTY 支持，而不是只在 Unix/tmux 环境里工作。

### 不适合
- 你只需要成熟稳定、插件生态完整的 tmux，并依赖大量 tmux plugin 或复杂 status line。
- 你需要一个完整 agent 平台（账号、provider 管理、移动端任务编排、worktree orchestration）；这类需求更接近 Paseo / Warp / IDE agent。
- 公司策略禁止任何 browser share、public tunnel、远程 WebSocket，哪怕 terminal payload 端到端加密。
- 你不希望引入本地 daemon 或后台进程，只想运行一次性 shell 命令。
- 你期望 RMUX 自己调用 LLM；它管理终端，不提供模型或 agent provider。

## 10. 同类工具定位
- **tmux**：生态最成熟、服务器上最常见；RMUX 的优势是 typed SDK、Windows 原生、Web Share E2EE 和 Rust workspace，可视为 agent automation 友好的新实现。限制是 tmux 兼容还应按实际命令/配置验证。
- **zellij**：更现代的 terminal workspace / layout / plugin 体验；RMUX 更强调 tmux-like CLI 和 SDK automation。
- **Paseo**：完整 coding agent 编排平台，管理 Claude/Codex/provider/worktree/移动端；RMUX 更底层，只管终端 multiplexer 和 terminal automation。
- **Warp**：带 AI/agent/workspace 的终端产品；RMUX 更接近开源本地 runtime，可嵌入、自托管、脚本化。
- **tmate / sshx / ttyd / GoTTY / Upterm**：偏终端分享或远程接入；RMUX 的 Web Share 是 multiplexer 的一部分，并强调 browser-to-daemon 加密、operator/spectator 和静态 frontend 分离。
- **expect / pexpect / Playwright terminal demos**：能做终端自动化，但通常缺少持久 multiplexer、typed pane/session model 和统一的 attach/share 体验。

## 11. 关键文件/目录导读
| Path | 作用 |
|---|---|
| `README.md` | 项目定位、安装方式、CLI/SDK quickstart、架构图、workspace crate 表、平台支持、配置与验证命令。 |
| `Cargo.toml` | Cargo workspace、版本 `0.6.1`、root binary、crate 成员与 release profile。 |
| `src/main.rs` | `rmux` CLI binary 入口。 |
| `src/daemon_main.rs` | `rmux-daemon` / hidden daemon entrypoint。 |
| `src/cli_args.rs`, `src/cli_args/` | tmux-like CLI command surface、参数解析、completion、队列化命令。 |
| `crates/rmux-core/` | session/window/pane/layout/format/hook/buffer 等核心模型。 |
| `crates/rmux-server/` | Tokio daemon、请求处理和 runtime 状态。 |
| `crates/rmux-client/` | 本地 client 与 attach mode plumbing。 |
| `crates/rmux-pty/` | PTY / ConPTY、resize、child process 控制。 |
| `crates/rmux-ipc/` | Unix socket / Windows named pipe 等本地 IPC。 |
| `crates/rmux-sdk/` | 公开 Rust SDK；看 `examples/` 和 `tests/` 理解 terminal automation API。 |
| `crates/ratatui-rmux/` | Ratatui widget integration。 |
| `crates/rmux-web-crypto/` | Web Share crypto core 与 WASM 边界。 |
| `docs/ARCHITECTURE.md` | 本地 runtime、Web Share runtime、trust boundary 和 release output 的架构说明。 |
| `docs/scripting-sdk.md` | Rust SDK 定位、安装、示例、capabilities/diagnose 建议。 |
| `docs/web-share.md` | Web Share 架构、crypto、角色、tunnel、CLI 管理和安全边界。 |
| `docs/human-friendly-config.md` | 面向人类交互的 starter config。 |
| `spec/feature-inventory-v1.yaml` | SDK/API 功能、测试覆盖和平台状态的 inventory。 |
| `tests/`, `crates/rmux-sdk/tests/` | CLI surface、IPC、tmux compatibility、SDK smoke、Windows/Unix 行为测试。 |
| `scripts/` | package/release/check/smoke 脚本，包括 no-network、unsafe、platform-neutrality 等检查。 |

## 12. 学习与掌控建议
1. 先当 tmux 替代品试用：`new-session`、`split-window`、`send-keys`、`attach-session`、`capture-pane`、`wait-for`，并用 `rmux list-commands` 核对你依赖的命令是否支持。
2. 再引入 agent workflow：为 Claude Code / Codex 建一个固定 `agents` session，约定 pane 角色（实现、测试、日志、review），避免多个 agent 同时写同一工作树。
3. Web Share 先只读：默认用 `--spectator-only --ttl ...` 给同事或手机看，确认 PIN、TTL、tunnel 和 self-hosted frontend 策略后再开放 operator。
4. 如果要自动化 CLI/TUI，优先看 `docs/scripting-sdk.md` 和 `crates/rmux-sdk/examples/`，不要从 `capture-pane` 文本硬解析状态；能用 typed handle / wait / snapshot 就用 SDK。
5. 从 tmux 迁移时保持保守：复杂 plugin、status line、shell hook 和 terminal feature 逐条验证；把不支持的配置记录在团队文档或 `.rmux.conf` 注释里。
6. 生产环境或客户数据场景下，把 RMUX 当成“可执行本地 shell 的控制面”：审查谁能拿到 operator link、谁能写 SDK 脚本、哪些命令可以被 agent 自动发送。

## 13. Sources
- https://github.com/Helvesec/rmux
- https://rmux.io/docs/get-started/
- https://rmux.io/docs/cli/
- https://rmux.io/docs/api/
- https://rmux.io/docs/web-share/
- https://github.com/Helvesec/rmux/blob/main/docs/ARCHITECTURE.md
- https://github.com/Helvesec/rmux/blob/main/docs/scripting-sdk.md
- https://github.com/Helvesec/rmux/blob/main/docs/web-share.md
- https://crates.io/crates/rmux
- https://crates.io/crates/rmux-sdk
- https://pypi.org/project/librmux/
- https://www.npmjs.com/package/@rmux/sdk
