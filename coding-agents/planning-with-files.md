# planning-with-files

> Repo: https://github.com/OthmanAdi/planning-with-files  
> Category: `coding-agents`  
> Last checked: 2026-05-08

## 1. 一句话总结
`planning-with-files` 是一个面向 Claude Code、Codex、Cursor、Gemini CLI 等 coding agents 的持久化规划技能/插件。它把 `task_plan.md`、`findings.md`、`progress.md` 作为 agent 的“磁盘工作记忆”，用 hooks 在关键时刻自动重读、提醒更新和校验完成度，减少长任务中的目标漂移、上下文丢失和重复犯错。

## 2. 项目目的与核心问题
- 目标用户：长期使用 Claude Code、Codex、Cursor、Gemini CLI、Copilot 等 agent 做多步骤研发、研究、排障、文档工作的开发者和团队。
- 解决的问题：agent 的上下文窗口是易失的，`TodoWrite` 或内存中的计划会在 `/clear`、自动压缩、长会话后丢失；错误和决策如果不落盘，agent 容易重复失败或忘记目标。
- 为什么值得关注：它把“上下文工程”变成了一个可安装的工作流，提供模板、脚本、slash commands、多 IDE hooks、session recovery、parallel plan isolation 和 plan attestation，而不只是建议用户手动写笔记。

## 3. 核心能力
- 三文件工作记忆：`task_plan.md` 记录阶段、状态、决策和错误；`findings.md` 存研究发现；`progress.md` 存会话日志和测试结果。
- 生命周期自动化：通过 hook 在用户提交、工具使用前后、会话开始和停止时自动注入计划上下文、提醒更新进度、检查是否全部完成。
- 会话恢复：`session-catchup.py` 会从 Claude Code / Codex 等会话存储中查找规划文件最后更新时间之后的对话，帮助 `/clear` 后补回丢失上下文。
- 多任务隔离：`init-session.sh "Task Name"` 可创建 `.planning/YYYY-MM-DD-slug/` 独立计划目录，`set-active-plan.sh` 和 `resolve-plan-dir.sh` 管理当前活动计划。
- 安全边界：v2.36.1 起用 `BEGIN/END PLAN DATA` 包裹注入内容；v2.37.0 新增 `/plan-attest` / `attest-plan.sh`，用 SHA-256 锁定已批准的 `task_plan.md`，检测到篡改时阻止注入。
- 多平台支持：README 标称支持 Claude Code、Cursor、GitHub Copilot、Mastra Code、Gemini CLI、Kiro、Codex、Hermes、CodeBuddy、FactoryAI Droid、OpenCode、Continue、Pi Agent、OpenClaw、Antigravity、Kilocode、AdaL CLI 等。

## 4. 实现思路 / 架构拆解
- 关键模块：
  - `skills/planning-with-files/SKILL.md`：核心技能说明、触发条件、三文件规则、hook 配置和安全边界。
  - `templates/`：默认 `task_plan.md`、`findings.md`、`progress.md` 模板，以及 analytics 模板。
  - `scripts/`：初始化、完成度检查、session catchup、多计划解析/切换、plan attestation、版本同步等脚本。
  - `.claude-plugin/`：Claude Code plugin metadata。
  - `commands/`：Claude Code slash commands，例如 `/planning-with-files:plan`、`/planning-with-files:status`、`/plan-attest`。
  - `.codex/`：Codex skill、`hooks.json` 和 hook adapter。
  - `.cursor/`、`.gemini/`、`.github/`、`.mastracode/` 等：不同 IDE/agent 的适配层。
- 核心流程：
  1. 安装 skill/plugin。
  2. agent 遇到复杂任务时创建三份规划文件。
  3. hook 在关键节点重读计划前几十行，把任务目标拉回注意力窗口。
  4. 文件写入后提醒更新 `progress.md`，阶段完成后更新 `task_plan.md`。
  5. Stop hook 检查所有 `### Phase` 是否 `**Status:** complete`，未完成时提醒或阻止提前结束。
  6. 如果会话被清空，session catchup 从历史对话中提取未同步的后续上下文。
- 依赖/生态：主要是 shell、PowerShell、Python 脚本和各平台的 Agent Skills / Hooks / Plugin 机制；不依赖重型 runtime。
- 设计亮点：把“agent 应该记住什么”从模型上下文迁移到项目文件系统，并用 lifecycle hooks 强制刷新注意力。

## 5. 安装与快速开始
通用 Agent Skills 安装：

```bash
npx skills add OthmanAdi/planning-with-files --skill planning-with-files -g
```

Claude Code plugin 安装：

```text
/plugin marketplace add OthmanAdi/planning-with-files
/plugin install planning-with-files@planning-with-files
```

Claude Code 常用命令：

```text
/planning-with-files:plan
/planning-with-files:status
/planning-with-files:start
/plan-attest
```

Codex 工作区安装：

```bash
git clone https://github.com/OthmanAdi/planning-with-files.git /tmp/planning-with-files
cp -r /tmp/planning-with-files/.codex .
git add .codex/
git commit -m "Add planning-with-files skill for Codex"
```

Codex 还需要在 `~/.codex/config.toml` 中启用 hooks：

```toml
[features]
codex_hooks = true
```

## 6. 在 Claude Code 中的用法
- 适合让 Claude Code 做什么：中大型功能实现、复杂 bugfix、repo 研究、跨文件重构、长期文档整理、需要多轮验证的任务。
- 推荐提示词/工作流：
  - “使用 planning-with-files，为这个任务创建计划并持续更新。”
  - 先执行 `/planning-with-files:plan`，让 Claude 生成三文件。
  - 每完成一个阶段，要求 Claude 更新 `progress.md` 和 `task_plan.md`。
  - 计划稳定后执行 `/plan-attest`，降低 `task_plan.md` 被外部内容污染后反复注入的风险。
- 注意事项：
  - 不要把网页、issue、README 等外部原文写进 `task_plan.md`；把研究材料放到 `findings.md`。
  - 小改动和单文件任务不必启用，维护三文件会有额外成本。
  - 如果并行处理多个任务，优先用 `.planning/<plan-id>/` 隔离计划，而不是共用根目录三文件。

## 7. 在 Codex 中的用法
- 适合让 Codex 做什么：长会话编码、PR 级别改动、复杂排障、需要跨上下文恢复的 repo 分析。
- 推荐提示词/工作流：
  - 在 repo 中提交 `.codex/` 目录，团队共享同一套 skill 和 hooks。
  - 开启 `codex_hooks = true` 后，Codex 会在 `SessionStart`、`UserPromptSubmit`、`PreToolUse`、`PostToolUse`、`Stop` 节点自动执行相关逻辑。
  - 使用 `task_plan.md` 作为任务状态源，要求 Codex 在最终回答前确认阶段完成度。
- 注意事项：
  - 官方 Codex 文档和该 repo 的 `docs/codex.md` 均要求 hooks 功能可用；Windows 上 Codex hooks 当前有限制。
  - 如果已经有 `~/.codex/hooks.json`，需要合并而不是覆盖。
  - 避免同时安装全局和工作区 hooks，否则可能重复输出提醒。

## 8. 典型生产力场景
| 场景 | 怎么用 | 产出 |
|---|---|---|
| 大型功能开发 | 让 agent 先拆阶段，再逐阶段实现和验证 | 可恢复的计划、进度日志、错误记录、代码改动 |
| Repo 研究归档 | 每读几个文件/网页就把结论写入 `findings.md` | 结构化研究笔记和最终文档 |
| 复杂 CI/bug 排障 | 把每次失败、假设和修复写进 plan/progress | 避免重复同一失败路径 |
| 多任务并行 | 用 `init-session.sh "Task Name"` 创建独立 `.planning/` 目录 | 多个互不污染的任务上下文 |
| 团队统一 agent 工作流 | 提交 `.codex/` 或 Claude plugin 配置 | 一致的规划模板和 hook 行为 |

## 9. 适合 / 不适合
### 适合
- 3 步以上、多工具调用、多文件修改的任务。
- 需要长期记忆、研究沉淀、错误追踪、阶段验收的 agent workflow。
- 团队想把 agent 工作习惯标准化到 repo 中。
- 经常遇到 `/clear`、上下文压缩、长会话漂移的用户。

### 不适合
- 简单问答、一次性命令、单文件小修改。
- 不想让项目根目录或 `.planning/` 下出现规划文件的场景。
- 对 hook 注入极度敏感、但又不愿启用 attestation 或严格区分可信/不可信内容的流程。
- 需要完整项目管理系统、看板、多人审批流的组织；它更像 agent 本地工作记忆，而不是 Jira/Linear 替代品。

## 10. 同类工具定位
- 类似工具：Claude Code 原生 TodoWrite、手写 `PLAN.md` / `NOTES.md`、Cursor rules、repo 内 `AGENTS.md`、更重的项目管理工具。
- 相比优势：
  - 文件持久化，不依赖当前上下文窗口。
  - hooks 自动重读和校验，比纯手写文档更能约束 agent 行为。
  - 多平台适配广，特别覆盖 Claude Code 与 Codex。
  - v2.37.0 的 plan attestation 对“计划文件被污染后自动注入”这个风险有明确防线。
- 相比限制：
  - 需要额外维护 planning files；任务越小，开销越明显。
  - hooks 行为依赖各 IDE/agent 的实现和版本。
  - 本质是流程约束，不会替代测试、代码 review 或真实项目管理。

## 11. 关键文件/目录导读
| Path | 作用 |
|---|---|
| `README.md` | 项目定位、安装方式、平台支持、版本变更概览。 |
| `skills/planning-with-files/SKILL.md` | 核心技能说明、触发条件、操作规则、hook 与安全边界。 |
| `templates/task_plan.md` | 任务阶段、状态、决策、错误记录模板。 |
| `templates/findings.md` | 研究发现、技术决策、资源链接模板。 |
| `templates/progress.md` | 会话日志、测试结果、错误日志、恢复检查模板。 |
| `scripts/init-session.sh` | 创建根目录三文件；带任务名时创建 `.planning/YYYY-MM-DD-slug/` 隔离计划。 |
| `scripts/session-catchup.py` | 从会话记录中恢复规划文件最后更新后的对话上下文。 |
| `scripts/check-complete.sh` | Stop hook 使用的阶段完成度检查器。 |
| `scripts/resolve-plan-dir.sh` | 解析当前活动计划目录，支持 `$PLAN_ID`、`.planning/.active_plan` 和 newest fallback。 |
| `scripts/set-active-plan.sh` | 切换或查看活动计划。 |
| `scripts/attest-plan.sh` | 对当前计划文件做 SHA-256 attestation，防止篡改后继续注入。 |
| `.codex/hooks.json` | Codex 生命周期 hook 配置。 |
| `.codex/hooks/` | Codex hook adapter 和 shell hook 实现。 |
| `.claude-plugin/plugin.json` | Claude Code plugin 元数据。 |
| `commands/` | Claude Code slash commands。 |
| `docs/codex.md` | Codex 安装、hook 启用和排障说明。 |
| `tests/` | session recovery、Codex hooks、parallel plan、attestation 等回归测试。 |

## 12. 学习与掌控建议
1. 先把它当成“三文件协议”使用：任何复杂任务都先创建计划，再把发现和进度分开写。
2. 只在长任务上启用完整流程；小任务保持轻量，避免计划文件变成负担。
3. 团队使用 Codex 时优先工作区安装 `.codex/`，让 hook 行为进入版本控制。
4. 把外部网页/API/issue 的内容写入 `findings.md`，不要写进会被频繁自动注入的 `task_plan.md`。
5. 在计划稳定后运行 `/plan-attest`，尤其是 repo 中存在不可信输入、自动生成内容或多人编辑计划文件时。

## 13. Sources
- https://github.com/OthmanAdi/planning-with-files
- https://github.com/OthmanAdi/planning-with-files/blob/master/README.md
- https://github.com/OthmanAdi/planning-with-files/releases/tag/v2.37.0
- https://github.com/OthmanAdi/planning-with-files/blob/master/skills/planning-with-files/SKILL.md
- https://github.com/OthmanAdi/planning-with-files/blob/master/docs/codex.md
- https://github.com/OthmanAdi/planning-with-files/blob/master/docs/installation.md
- https://github.com/OthmanAdi/planning-with-files/blob/master/.codex/hooks.json
- https://github.com/OthmanAdi/planning-with-files/tree/master/scripts
