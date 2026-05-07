# humanize

> Repo: https://github.com/PolyArch/humanize  
> Category: `coding-agents`  
> Last checked: 2026-05-07

## 1. 一句话总结
Humanize 是一个 Claude Code 插件，用 Claude 负责实现、Codex 负责独立审查，把 AI coding 从一次性生成变成可追踪的迭代开发闭环。它的核心工作流叫 RLCR：Ralph-Loop with Codex Review。

## 2. 项目目的与核心问题
- 目标用户：重度使用 Claude Code、Codex CLI 做真实 repo 改动的开发者。
- 解决的问题：AI 一次性生成代码容易遗漏需求、偏离目标、缺少独立审查；Humanize 用 plan、round summary、goal tracker、Codex review 和 hook gate 把这个过程工程化。
- 为什么值得关注：它不是又一个代码生成 wrapper，而是把“实现者”和“审查者”拆成两个模型角色，并把每轮状态写入 `.humanize/`，方便复盘、监控和继续迭代。

## 3. 核心能力
- 从粗略想法生成 idea draft：`/humanize:gen-idea`。
- 从 draft 生成结构化 implementation plan：`/humanize:gen-plan`。
- 处理带 reviewer 注释的计划：`/humanize:refine-plan`。
- 启动 Claude 实现 + Codex 审查的 RLCR loop：`/humanize:start-rlcr-loop`。
- 单次咨询 Codex：`/humanize:ask-codex`。
- 可选调用 Gemini CLI 做深度 web research：`/humanize:ask-gemini`。
- 通过 `humanize monitor` 监控 RLCR、Codex、Gemini 和 skill 调用进度。

## 4. 实现思路 / 架构拆解
- 关键模块：
  - `.claude-plugin/`：Claude Code 插件元数据和 marketplace 信息。
  - `commands/`：Claude Code slash command 定义。
  - `scripts/`：核心 shell runtime，包括 loop setup、Codex/Gemini 调用、monitor helper。
  - `hooks/`：Claude Code hook gate，用于计划校验、读写限制、退出审查、状态保护。
  - `skills/`：Codex/Kimi 等 skill runtime 文档。
  - `prompt-template/`：Claude、Codex、plan、block 等阶段模板。
  - `tests/`：大量 shell 测试和 robustness 测试。
- 核心流程：
  1. 用户先准备或生成 plan。
  2. `start-rlcr-loop` 做 plan compliance pre-check 和 plan understanding quiz。
  3. Claude 根据 plan 执行任务，并维护 goal tracker。
  4. Claude 写 `round-N-summary.md` 后尝试退出。
  5. Codex 审查 summary；若未完成，反馈给 Claude 继续修。
  6. Codex 输出完成后进入 code review phase，用 `codex review --base <branch>` 检查代码质量。
  7. 无 `[P0-9]` 严重问题后进入 finalize phase。
- 依赖/生态：Claude Code plugin 系统、Codex CLI；可选 Gemini CLI；大量逻辑通过 Bash hooks 和 Markdown 状态文件协调。
- 设计亮点：把 human plan 理解作为前置步骤；把每轮状态持久化；把 implementation review 和 code review 分成两个阶段；用 Codex 作为独立 reviewer，而不是让同一个模型自我确认。

## 5. 安装与快速开始
```bash
# Add PolyArch marketplace
/plugin marketplace add PolyArch/humanize

# Optional: use development branch
/plugin marketplace add PolyArch/humanize#dev

# Install plugin
/plugin install humanize@PolyArch
```

要求本机可用 Codex CLI。典型使用：

```bash
/humanize:gen-idea "add undo/redo to the editor"
/humanize:gen-plan --input draft.md --output docs/plan.md
/humanize:refine-plan --input docs/plan.md
/humanize:start-rlcr-loop docs/plan.md
```

监控：

```bash
source <path-to-humanize>/scripts/humanize.sh
humanize monitor rlcr
humanize monitor skill
humanize monitor codex
humanize monitor gemini
```

## 6. 在 Claude Code 中的用法
- 适合让 Claude Code 做什么：负责读 plan、修改代码、运行测试、写 round summary、根据 Codex 反馈继续迭代。
- 推荐工作流：
  1. 先把需求写成 draft。
  2. 用 `/humanize:gen-plan` 生成计划。
  3. 人类 review plan，必要时用注释块标出问题。
  4. 用 `/humanize:refine-plan` 吸收注释。
  5. 用 `/humanize:start-rlcr-loop docs/plan.md` 开始实现。
- 注意事项：不要把不理解的 plan 直接丢给 loop 长时间跑；`--yolo` 只适合你已经充分 review 过 plan 的情况。

## 7. 在 Codex 中的用法
- 适合让 Codex 做什么：作为独立 reviewer，检查 summary 是否满足目标、代码 diff 是否有风险、是否存在 `[P0-9]` 严重问题。
- 推荐工作流：
  - 在 Humanize 内部由 hook 和 scripts 调用 Codex review。
  - 需要单次意见时使用 `/humanize:ask-codex "..."`。
  - 也可以用 `--skip-impl` 对已有改动直接进入 code review。
- 注意事项：默认 Codex 模型来自 config，README 当前默认配置为 `gpt-5.5:high`；生产环境应谨慎使用 `HUMANIZE_CODEX_BYPASS_SANDBOX`，因为它会绕过 sandbox/approval 保护。

## 8. 典型生产力场景
| 场景 | 怎么用 | 产出 |
|---|---|---|
| 中大型功能实现 | draft -> gen-plan -> refine-plan -> start-rlcr-loop | 可追踪的多轮实现、summary、review result |
| 已有 diff 审查 | `/humanize:start-rlcr-loop --skip-impl` | Codex 对当前改动的质量反馈 |
| 设计方案二次意见 | `/humanize:ask-codex "review this plan"` | 单次 Codex 分析结果 |
| 团队计划审阅 | 在 plan 中加入 `CMT:` 或 `<cmt>` 注释后运行 refine-plan | 清理后的 plan 和 QA ledger |
| 长任务监控 | `humanize monitor rlcr` | 实时 loop 状态和进度 |

## 9. 适合 / 不适合
### 适合
- 有明确目标、acceptance criteria 和可测试边界的 repo 改动。
- 需要独立 review 的复杂代码任务。
- 希望保留 AI coding 过程记录，方便复盘和继续接力。
- 使用 Claude Code + Codex CLI 的开发环境。

### 不适合
- 一两行修复或简单问答。
- 需求尚未想清楚、plan 明显不稳定的探索任务。
- 无法安装或稳定运行 Claude Code/Codex CLI 的环境。
- 不希望项目目录出现 `.humanize/` 状态文件的场景。

## 10. 同类工具定位
- 类似工具：Claude Code 原生 slash commands、OpenAI Codex CLI、GAAC、ralph-loop 类插件、各种 AI code review bot。
- 相比优势：把 Claude 实现和 Codex 审查明确拆分；有持久化状态、goal tracker、plan quiz、hook gate 和 monitor；更适合长任务闭环。
- 相比限制：依赖 Claude Code 插件生态和 Codex CLI；大量 shell/hook 状态机增加了使用复杂度；小任务会显得重。

## 11. 关键文件/目录导读
| Path | 作用 |
|---|---|
| `README.md` | 项目总览、安装、快速开始、文档入口。 |
| `docs/usage.md` | 命令参数、配置、监控、环境变量说明。 |
| `docs/install-for-claude.md` | Claude Code 插件安装说明。 |
| `docs/install-for-codex.md` | Codex skill runtime 安装说明。 |
| `.claude-plugin/plugin.json` | 插件名称、版本、描述、仓库和 license 元数据。 |
| `commands/start-rlcr-loop.md` | RLCR loop 的 command contract、pre-check、quiz 和阶段说明。 |
| `scripts/setup-rlcr-loop.sh` | 初始化 loop 状态、解析参数、设置默认配置。 |
| `hooks/` | Claude Code hooks 和状态保护逻辑。 |
| `prompt-template/codex/` | Codex review 相关 prompt 模板。 |
| `tests/` | 功能和鲁棒性测试。 |

## 12. 学习与掌控建议
1. 先不要直接 `--yolo`，用一个小型 repo 改动跑完整 RLCR，观察 `.humanize/rlcr/<timestamp>/` 下的状态文件。
2. 重点读 `commands/start-rlcr-loop.md`，它比 README 更能解释真实工作协议。
3. 把 plan 写得像工程任务单：目标、acceptance criteria、任务拆分、测试方式都明确，Humanize 的价值才会被放大。
4. 对已有改动可以先试 `--skip-impl`，把它当作 Codex code review wrapper。
5. 如果在 CI 或远程机器使用，先确认 sandbox 行为，不要轻易开启 `HUMANIZE_CODEX_BYPASS_SANDBOX`。

## 13. Sources
- https://github.com/PolyArch/humanize
- https://raw.githubusercontent.com/PolyArch/humanize/main/README.md
- https://raw.githubusercontent.com/PolyArch/humanize/main/docs/usage.md
- https://raw.githubusercontent.com/PolyArch/humanize/main/.claude-plugin/plugin.json
- https://raw.githubusercontent.com/PolyArch/humanize/main/commands/start-rlcr-loop.md
- https://raw.githubusercontent.com/PolyArch/humanize/main/scripts/setup-rlcr-loop.sh
