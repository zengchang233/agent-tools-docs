# Codex 使用方法与高效工作流

> Tool: OpenAI Codex CLI / Codex  
> Category: `coding-agents`  
> Last checked: 2026-05-07

## 1. 一句话总结
Codex 最适合当“代码库里的执行型工程师和独立 reviewer”：它能在本地读写文件、运行命令、做代码审查、接 MCP 和 plugins，也适合把复杂任务拆给 subagents 并行推进。高效使用 Codex 的关键是给它清晰的任务边界、验证标准和并行授权，而不是只把它当聊天机器人。

## 2. 常用入口
本机 `codex --help` 显示的核心命令：

| 命令 | 用途 |
|---|---|
| `codex` | 进入交互式 TUI，会把当前目录当 workspace |
| `codex exec` / `codex e` | 非交互执行一次任务，适合脚本、CI、批处理 |
| `codex review` | 非交互代码审查 |
| `codex resume` | 恢复之前的会话 |
| `codex fork` | 从历史会话 fork 新分支上下文 |
| `codex apply` | 应用上次 Codex 产出的 diff |
| `codex mcp` | 管理 MCP servers |
| `codex plugin` | 管理插件 |
| `codex cloud` | 浏览 Codex Cloud 任务并应用到本地 |

常用启动参数：

```bash
codex -C /path/to/repo
codex --search
codex -m gpt-5.5
codex -s workspace-write
codex -a on-request
codex exec "summarize this repository and suggest test commands"
codex review --base main
```

参数使用建议：

- `-C`：显式指定工作目录，避免在错误目录启动。
- `--search`：需要最新资料、文档、价格、版本、外部事实时开启。
- `-m`：复杂实现、架构和 review 用强模型；机械任务可用更快模型。
- `-s workspace-write`：日常开发默认推荐；只有可信隔离环境才用 `danger-full-access`。
- `-a on-request`：交互开发时让 Codex 在高风险命令前请求确认。
- `-a never`：只适合已隔离、可重复、非交互任务。

## 3. 交互式 Codex 的正确节奏
不要直接说“帮我改好”。更好的节奏：

1. 让 Codex 先读项目结构、测试命令、相关文件。
2. 要求它给计划、风险、预计改动文件。
3. 确认后实现。
4. 实现期间允许它运行测试和格式化。
5. 完成后要求总结 diff、验证结果、残留风险。
6. 再跑 `codex review` 或让独立 subagent 审查。

示例：

```text
目标：修复 checkout 页面优惠券重复应用的问题。
范围：只改 packages/web/checkout 和 packages/shared/pricing。
验收标准：
1. 同一个 couponId 只能应用一次；
2. 刷新页面后状态一致；
3. 相关单元测试通过，必要时补测试。
工作方式：
先探索相关调用链并列计划。不要先改代码。
计划里写出预计改动文件、测试命令和风险。
```

## 4. `codex exec`：把 Codex 当脚本化 worker
`codex exec` 适合一次性、可描述清楚、最好能自动验证的任务：

```bash
codex exec -C . "Find dead code in src/auth. Do not edit files. Return file paths and reasons."
```

```bash
codex exec -C . "Update README installation section based on package.json scripts. Edit README only."
```

```bash
codex exec -C . --search "Check the latest official migration notes for this library, then update docs/migration.md with a concise summary and sources."
```

适合：

- 批量文档整理。
- 单文件或小范围改动。
- 代码库问答。
- 生成报告。
- CI 前的自动 review。

不适合：

- 需要多次人类决策的需求澄清。
- 高风险迁移。
- 需要大量外部登录交互的任务。
- 复杂 MCP 工具链还不稳定时的全自动任务。

## 5. `codex review`：把 Codex 固定成独立 reviewer
常用方式：

```bash
codex review
codex review --base main
codex review --base origin/main
```

适合在这些节点使用：

- 自己或 Claude Code 改完后。
- PR 前。
- 大重构每个阶段结束后。
- CI 失败修复后。

高质量 review 请求应该限定关注点：

```text
Review current diff against main.
Focus on behavioral regressions, race conditions, error handling, and missing tests.
Ignore formatting and naming unless they hide a bug.
Return findings first, ordered by severity, with file paths and concrete fixes.
```

## 6. Codex 里的 subagents / multiagent
Codex 当前最有价值的 agent 能力是“主 agent 继续推进关键路径，同时把独立副任务交给 subagents”。这类能力在复杂代码任务里很重要，因为搜索、测试、审查和方案比较会消耗大量上下文。

适合交给 subagent 的任务：

- 只读探索：查调用链、找相关文件、总结架构。
- 独立 review：审查当前 diff、找回归风险。
- 测试 worker：补测试、跑测试、定位失败原因。
- 方案比较：针对同一目标提出替代实现和风险。
- 文档 worker：根据已实现 diff 更新文档或 changelog。

不适合交给 subagent：

- 下一步马上依赖结果的阻塞任务。
- 需要频繁和主 agent 共同修改同一文件的任务。
- 需要主 agent 持续判断产品取舍的任务。
- 范围模糊、没有明确产出的探索。

高效写法：

```text
可以使用 subagents。
主 agent 负责实现核心修复。
并行安排：
1. explorer：只读，找 checkout coupon 状态流和历史测试；
2. reviewer：只读，审查当前方案的并发和状态一致性风险；
3. test worker：只负责 packages/web/checkout 的测试，不改生产代码。
每个 subagent 返回：发现、建议、涉及文件、是否阻塞。
不要让多个 agent 改同一个文件。
```

更保守的写法：

```text
先不要并行写代码。
可以派 subagent 做只读探索和 review。
主 agent 等探索结果后再决定实现方案。
```

## 7. 多 agent 拆分原则
并行不是越多越好。拆分要满足三个条件：

- 任务之间依赖少。
- 写入文件集不重叠。
- 每个 worker 有清晰产出。

推荐拆分方式：

| 任务类型 | 主 agent | subagent A | subagent B | subagent C |
|---|---|---|---|---|
| Bug 修复 | 定根因、改核心代码 | 查调用链 | 找历史相关提交/测试 | 独立 review |
| 新功能 | 设计接口、整合 diff | 前端实现 | 后端实现 | 测试和文档 |
| 重构 | 控制阶段和行为不变 | 模块 A | 模块 B | 回归测试 |
| 文档研究 | 定结构和结论 | 官方资料 | 社区实践 | 示例验证 |

冲突控制：

- 明确每个 worker 的“文件所有权”。
- 让 worker 先报计划和文件清单。
- 主 agent 负责最后整合，不让 worker 互相覆盖。
- 同一文件只能有一个 writer，其他 agent 只读 review。

## 8. Skills：把你的工作习惯交给 Codex
Codex 支持 skills，这适合沉淀重复流程。你可以把一套长期使用的工作方法写成 skill，例如：

- OSS repo 分析模板。
- PR review 标准。
- 前端视觉验证流程。
- CI failure triage。
- “先计划、再实现、再 review”的固定闭环。

适合 skill 的内容：

- 清晰步骤。
- 输出格式。
- 常用命令。
- 文件模板。
- 判断标准。

不适合 skill 的内容：

- 只适用于一次任务的背景。
- 大量会过期的外部事实。
- 需要每次都人工判断的模糊偏好。

示例 skill 思路：

```markdown
---
name: pr-review
description: Review a diff for production bugs, regressions, missing tests, and risky assumptions.
---

Run git diff against the requested base.
Findings first. No praise. No style-only comments.
For each finding include severity, file path, line, why it matters, and suggested fix.
If no issues, say so and list remaining test gaps.
```

## 9. MCP 和 plugins
MCP 给 Codex 接外部工具，plugins 则把 skills、MCP、工作流打包。高效用法是把工具接到 Codex 能真正行动的地方：

- GitHub：查 PR、issue、review comment、CI logs，修完后推 PR。
- Docs：查官方文档，避免凭记忆写过时 API。
- Browser/Playwright：前端截图、交互验证。
- 数据库/日志：只读排查线上问题。
- 内部 CLI：把复杂系统操作封成安全命令，让 Codex 调。

建议：

- 能用只读就先只读。
- 写操作前让 Codex 说明意图和影响。
- 高频工作做成 plugin 或 skill，而不是每次手打长提示。
- 给 Codex 提供稳定 CLI，比让它猜内部系统 API 更可靠。

## 10. Claude Code + Codex 组合
你平时同时用 Claude Code 和 Codex，可以把它们分工：

| 场景 | Claude Code | Codex |
|---|---|---|
| 快速实现 | 主实现者 | 独立 review |
| 大功能 | 计划、拆任务、调 Claude subagents | 审查方案和 diff |
| 复杂 bug | 交互式排查 | 并行根因分析或 review |
| 代码库学习 | 生成学习路径和问题清单 | 深挖具体调用链 |
| PR 前 | 整理说明、补文档 | `codex review --base main` |

推荐闭环：

```text
Claude Code 负责实现 -> Codex review -> Claude Code 修复 review findings -> Codex 再审 -> 人类最终确认
```

如果在 Claude Code 里安装 Codex plugin，可以直接用：

```text
/codex:review --background
/codex:status
/codex:result
/codex:adversarial-review --base main <focus>
```

这样 Claude Code 主会话继续工作，Codex 在后台做独立审查。

## 11. 高效工作流模板
### 11.1 代码库理解
```text
不要改文件。
请理解这个 repo 的请求入口、核心模块、测试方式和部署方式。
可以使用 subagents 并行探索不同目录。
输出：
1. 架构图式文字说明；
2. 关键文件；
3. 常用开发命令；
4. 新人最该读的 5 个文件；
5. 潜在风险和技术债。
```

### 11.2 实现任务
```text
目标：<目标>
范围：<可改目录>
禁止：<不能改的文件/行为>
验收：<可测试行为>
允许使用 subagents，但写文件必须分配 owner。
先给计划和文件清单；确认后实现。
完成后运行测试并总结 diff。
```

### 11.3 独立审查
```text
Review current diff against origin/main.
Focus on bugs, data loss, security, concurrency, and missing tests.
Do not suggest broad refactors.
Findings first with severity and file paths.
```

### 11.4 CI 修复
```text
查看最近失败的 CI 日志。
先总结失败原因和最小修复范围。
只修改导致 CI 失败的相关文件。
修复后运行对应本地测试。
不要顺手重构。
```

## 12. 常见低效用法与改法
| 低效用法 | 问题 | 改法 |
|---|---|---|
| 把 Codex 当问答工具 | 没利用文件读写和命令验证 | 让它读代码、改文件、跑测试 |
| 不给范围 | 容易扩大改动 | 指定目录、禁止事项、验收标准 |
| 不允许 subagents | 主上下文被探索和日志塞满 | 明确允许只读探索和独立 review |
| 所有 agent 都能写 | 冲突多 | 每个 worker 固定写入范围 |
| 不用 `codex review` | 缺少独立审查 | 每个可提交 diff 后跑 review |
| 重复手写长提示 | 不稳定 | 做成 skill/plugin |

## 13. 我的推荐配置方向
优先做这几件事：

1. 给常用 repo 写 `AGENTS.md` 或等价项目说明：命令、结构、风格、测试、危险区域。
2. 把常用审查标准做成 Codex skill。
3. 大任务默认允许 subagents，但要求文件 ownership。
4. PR 前固定跑 `codex review --base main`。
5. 给内部系统封装 CLI/MCP，让 Codex 调稳定接口。
6. Claude Code 负责交互式实现时，用 Codex 做后台独立 reviewer。

## 14. Sources
- https://github.com/openai/codex
- https://developers.openai.com/codex/use-cases
- https://github.com/openai/codex-plugin-cc
- `codex --help` from local CLI, checked 2026-05-07
