# Claude Code 使用方法与高效工作流

> Tool: Claude Code  
> Category: `coding-agents`  
> Last checked: 2026-05-07

## 1. 一句话总结
Claude Code 最适合当“主开发者 + 任务调度器”：主会话负责目标、取舍和最终整合，subagent 负责探索、审查、测试、并行验证等副任务。高效使用的关键不是把问题一次性丢给 Claude，而是把任务切成可验证的工作包，并把重复流程沉淀成 skills、commands、hooks 和项目记忆。

## 2. 基础使用方式
- `claude`：进入当前项目的交互式开发会话。
- `/init`：为项目生成或更新 `CLAUDE.md`，记录项目结构、常用命令、约束和风格。
- `/memory`：查看、编辑当前会话加载的 `CLAUDE.md`、本地记忆和 auto memory。
- `/compact [instructions]`：上下文过长时压缩，但要带明确保留重点，例如“保留当前 bug 复现、已改文件、失败测试和下一步”。
- `/review`：让 Claude 对当前 diff 或 PR 做代码审查。
- `/agents`：创建、编辑、查看 custom subagents。
- `/mcp`：管理外部工具连接，例如 GitHub、Linear、数据库、浏览器、文档检索等。
- `/permissions`：管理工具权限，避免长期使用过宽权限。

日常节奏建议：

1. 新项目先跑 `/init`，把 build/test/lint 命令、目录结构、禁止事项写进 `CLAUDE.md`。
2. 开始任务时给 Claude 明确目标、验收标准、允许改动范围和验证方式。
3. 让 Claude 先探索和计划，再实现；复杂任务先要求“列出风险和需要确认的点”。
4. 改完后要求它运行相关测试、解释 diff、再触发 review 或独立 subagent 审查。

## 3. 提示词写法
低效提示：

```text
帮我优化这个项目。
```

高效提示：

```text
目标：把用户设置页的保存逻辑从同步请求改成异步队列。
范围：只改 packages/web/settings 和 shared/api 相关文件。
验收标准：
1. 保存按钮点击后立即显示 pending 状态；
2. 请求失败时保留用户输入并显示错误；
3. 现有 settings 单元测试通过，必要时补测试。
工作方式：先让 Explore subagent 找相关调用链，再给我计划；我确认后再改。
```

更适合 agent 的提示通常包含：

- `目标`：最终要达到什么状态。
- `范围`：哪些目录可以改，哪些不能动。
- `验收标准`：用行为描述，不只说“修好”。
- `验证方式`：具体测试、lint、手工页面、截图、日志。
- `协作方式`：是否先计划、是否允许并行 subagent、是否需要 review。

## 4. Subagents 怎么用
Claude Code 的 subagent 是在同一个 Claude Code session 里由主会话调度的专门 worker。每个 subagent 有自己的上下文、系统提示和工具权限，完成后只把结果返回主会话。适合用在“过程信息很多，但最终只需要结论”的任务。

### 4.1 内置 subagents
常见内置模式：

| Agent | 适合 | 不适合 |
|---|---|---|
| Explore | 读代码、找调用链、查日志、定位文件 | 直接改代码 |
| Plan | 在 plan mode 中收集上下文并形成计划 | 已经明确的一两行改动 |
| General-purpose | 多步骤、需要读写和推理的副任务 | 和主任务强耦合、必须实时决策的核心路径 |

你可以显式要求：

```text
先用 Explore subagent 找出认证流程的所有入口和中间件，不要改文件。返回：关键文件、调用链、风险点。
```

```text
让一个 subagent 独立审查当前 diff，只关注并发、错误处理和数据一致性。主会话继续补测试。
```

### 4.2 自定义 subagents
当你反复需要同一种 worker，就创建 custom subagent。项目级位置：

```text
.claude/agents/<agent-name>.md
```

用户级位置：

```text
~/.claude/agents/<agent-name>.md
```

示例：`.claude/agents/regression-reviewer.md`

```markdown
---
name: regression-reviewer
description: Use proactively after code changes to find behavioral regressions, missing tests, and risky edge cases. Read-only review only.
tools: Read, Grep, Glob, Bash
---

You are a regression-focused reviewer. Inspect the current diff and nearby code.
Prioritize concrete bugs over style comments.
Return findings ordered by severity with file paths, line numbers, and reproduction or test suggestions.
Do not edit files.
```

关键点：

- `description` 写清楚触发时机；想让 Claude 主动用，就写 “Use proactively”。
- 对 review/search 类 agent 限制成只读工具。
- 对 implementation 类 agent 只给它负责的目录或文件范围，避免多人改同一片代码。
- 返回格式要固定，否则主会话难整合。

### 4.3 Forked subagents
Forked subagents 适合“需要完整上下文，但又想并行试思路”的场景，例如：

```text
/fork 基于当前方案尝试另一种更小改动的实现路径，只给出 diff 方案和风险，不要提交。
```

适用场景：

- 当前对话已经包含大量背景，普通 subagent 重新解释成本高。
- 想并行比较两个实现方案。
- 想在主会话继续做实现时，让 fork 起草测试、迁移方案或 review 清单。

注意：fork 是实验能力，使用前确认本机 Claude Code 版本和环境变量支持情况。

## 5. Multiagent / Agent Teams
Subagent 是“主会话调度，worker 回报结果”。Agent teams 是多个 Claude Code session 组成团队，有共享任务和 agent 间通信，适合更大任务。

适合 agent teams：

- 前端、后端、测试可以分开做的大功能。
- 多个假设并行排查的复杂 bug。
- 研究和审查任务，需要多个 agent 互相挑战结论。
- 跨层改造，例如 API schema、UI、测试、文档同时改。

不适合 agent teams：

- 顺序依赖强的任务。
- 多个 worker 会改同一个文件。
- 需求还没定义清楚。
- 小修小补，协调成本超过收益。

拆分方式示例：

```text
启用 agent team。Team lead 负责整合和最终 diff。
Agent A：只负责 backend API 和数据迁移。
Agent B：只负责 frontend 表单和状态。
Agent C：只负责测试计划、补测试和回归风险。
要求每个 agent 先提交计划和预计改动文件，避免文件冲突。
```

## 6. Skills / Commands：把重复流程产品化
如果你经常复制同一套提示词，就应该做成 skill 或 command。Claude Code 现在推荐用 skill：一个目录加 `SKILL.md`，可放模板、脚本、示例和参考资料。

适合沉淀为 skill 的流程：

- “总结当前 diff 并给 commit message”
- “按团队规范做 PR review”
- “前端改动后启动 dev server、截图、检查布局”
- “排查线上错误日志并输出 root cause”
- “写迁移计划并列出 rollback”

示例：

```text
.claude/skills/review-diff/SKILL.md
```

```markdown
---
description: Review current git diff for bugs, regressions, missing tests, and risky assumptions.
---

## Current diff

!`git diff HEAD`

## Instructions

Review the diff as a senior engineer. Findings first, ordered by severity.
Ignore pure style unless it creates maintainability risk.
For every finding, include file path, line, why it matters, and a concrete fix or test.
```

## 7. MCP：给 agent 接工具
MCP 的价值是让 Claude Code 能接真实系统，而不是只靠本地文件：

- GitHub：读 issue、PR、review comment、CI 状态。
- Linear/Jira：读需求、建任务、同步状态。
- Docs/search：查内部文档、API 文档。
- DB/observability：查只读数据、日志、trace。
- Browser/Figma：做前端验证和设计还原。

使用建议：

- 写操作默认谨慎，优先只读 MCP。
- 把高风险 MCP 工具权限拆细，不要一次批准全部。
- 对 MCP 写操作加 hooks 或人工确认。
- 让 Claude 在计划里说明“准备调用哪个外部系统、为什么、会不会写入”。

## 8. Hooks：把质量门槛自动化
Hooks 适合自动触发格式化、测试、权限检查、敏感文件保护和通知。常见用法：

- `PreToolUse`：阻止改 `.env`、密钥、生产配置、generated files。
- `PostToolUse`：写完代码后自动运行 formatter。
- `Stop`：会话准备结束前检查是否有未运行测试或未总结 diff。
- MCP 写操作前检查目标环境，避免写到生产。

不要把复杂业务逻辑塞进 hooks。Hooks 会自动执行 shell 命令，必须保持短、小、可审计。

## 9. 高效工作流模板
### 9.1 新功能
```text
先不要写代码。
目标：<功能目标>
范围：<目录/模块>
验收标准：<行为列表>
请做：
1. 用 Explore subagent 找相关代码路径；
2. 给出实现计划、风险和测试策略；
3. 标出哪些文件会改；
4. 等我确认后再实现。
```

### 9.2 复杂 bug
```text
这是 bug：<现象>
已知信息：<日志/复现/版本>
请并行处理：
1. Explore subagent 查调用链和最近相关改动；
2. 另一个 subagent 提出 3 个根因假设和验证方法；
3. 主会话整理最小复现和修复计划。
先不要改代码，除非能给出可验证根因。
```

### 9.3 大重构
```text
目标：<重构目标>
约束：不能改变外部行为，必须小步提交。
请把任务拆成阶段，每阶段包含：
- 改动范围
- 行为不变性的验证方式
- 可回滚点
- 是否适合交给 subagent 并行
```

### 9.4 独立审查
```text
让 regression-reviewer subagent 审查当前 diff。
只关注：
1. 行为回归；
2. 并发/缓存/状态一致性；
3. 错误处理；
4. 缺失测试。
不要给风格建议。
```

## 10. 常见低效用法与改法
| 低效用法 | 问题 | 改法 |
|---|---|---|
| 一次性说“实现整个功能” | agent 会猜范围和验收标准 | 先要求计划、文件清单、测试策略 |
| 所有探索都塞主会话 | 上下文被日志和搜索结果污染 | 用 Explore subagent 返回摘要 |
| 每次复制同一段 review 提示 | 不稳定且浪费 | 做成 skill 或 custom subagent |
| 多 agent 同时改同一文件 | 冲突和返工 | 明确 owner 和写入范围 |
| 不运行验证就结束 | 产出不可交付 | 把测试命令写进 `CLAUDE.md` 和 Stop checklist |
| 长会话从不 compact | 后期行为漂移 | 阶段性 `/compact`，保留目标、diff、失败测试 |

## 11. 我的推荐配置方向
优先做这 5 件事：

1. 每个常用项目都有清晰 `CLAUDE.md`：架构、命令、风格、禁止事项。
2. 建 3 个用户级 subagents：`codebase-explorer`、`regression-reviewer`、`test-runner`。
3. 建 3 个 skills：`review-diff`、`write-pr-description`、`debug-failing-test`。
4. 给高风险项目加 hooks：保护 `.env`、生产配置、迁移文件和 generated files。
5. 对大任务显式要求 Claude 使用 subagents，并规定每个 subagent 的产出格式。

## 12. Sources
- https://code.claude.com/docs/en/sub-agents
- https://code.claude.com/docs/en/agent-teams
- https://code.claude.com/docs/en/slash-commands
- https://code.claude.com/docs/en/memory
- https://docs.anthropic.com/en/docs/claude-code/mcp
- https://docs.anthropic.com/en/docs/claude-code/hooks
