<div align="center">

# Agent Tools Docs

**面向 AI Agent / Coding Agent / MCP / AI Gateway 等前沿开源工具的中文研究手册**

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

[English](README.md) · **简体中文**

把值得关注的开源 AI 工具，从“看过一个 README”整理成“能直接用于 Claude Code / Codex 日常工作流”的结构化笔记。

</div>

---

## 这个仓库是什么？

`agent-tools-docs` 是一个长期维护的开源工具研究库，重点记录前沿 GitHub repo 的：

- **项目定位**：它解决什么问题，适合谁用，不适合谁用；
- **核心实现**：关键模块、架构设计、运行链路和依赖生态；
- **上手方法**：安装、配置、常见命令和最小可用流程；
- **Agent 工作流**：如何和 Claude Code、Codex、MCP、CI、代码审查等流程结合；
- **生产力建议**：真实场景、风险边界、替代方案和长期维护注意事项。

它不是简单收藏夹，也不是 README 翻译；每篇文档都会尽量整理成可复用的使用手册和技术导读。

## 文档索引

### Coding Agents

| 文档 | 主题 | 适合阅读场景 |
|---|---|---|
| [Claude Code 使用方法与高效工作流](coding-agents/claude-code-usage.md) | Claude Code 实战工作流 | 想把 Claude Code 用成主力研发 agent |
| [Codex 使用方法与高效工作流](coding-agents/codex-usage.md) | OpenAI Codex CLI / Codex | 想系统理解 Codex 的执行、审查和并行能力 |
| [planning-with-files](coding-agents/planning-with-files.md) | 文件化计划与长期任务记忆 | 想解决长任务上下文丢失、目标漂移问题 |
| [humanize](coding-agents/humanize.md) | Claude 实现 + Codex 审查闭环 | 想把 AI coding 做成多轮可追踪开发流程 |
| [oh-my-pi](coding-agents/oh-my-pi.md) | 终端 AI coding agent / runtime | 想了解带 LSP、DAP、subagents、RPC/SDK 的完整 agent harness |
| [warp](coding-agents/warp.md) | Agentic development environment / agent 终端 | 想把 Warp Agent、Claude Code、Codex、MCP 和 Oz cloud agents 放进统一终端工作台 |

### AI Gateways

| 文档 | 主题 | 适合阅读场景 |
|---|---|---|
| [sub2api](ai-gateways/sub2api.md) | AI API 网关与账号池管理 | 想研究 Claude / OpenAI / Gemini 等上游额度分发、调度和计费平台 |

## 如何使用这个仓库？

### 如果你是读者

1. 先从上面的 **文档索引** 找到你关心的工具类型；
2. 阅读每篇文档开头的“一句话总结”和“适合 / 不适合”；
3. 如果准备上手，再看安装、配置、Claude Code / Codex 用法和风险说明；
4. 如果准备二次开发，重点看“实现思路 / 架构拆解”和“关键文件/目录导读”。

### 如果你想新增一个 repo

如果你使用 Claude Code、Codex 或其他 coding agent 参与贡献，请先阅读 [`AGENTS.md`](AGENTS.md)。里面记录了本仓库期望的研究流程、写作风格、分类规则，以及公开文件和本地工作记忆的边界。

只需要提供一个或多个 GitHub repo 链接。分析流程会按下面的方式进行：

1. 核验官方仓库、README、文档和版本信息；
2. 判断工具类型，并放入合适的分类目录；
3. 创建独立 Markdown 文档：`<category>/<repo-name>.md`；
4. 总结项目目的、核心实现、上手方式、Claude Code / Codex 工作流、适用场景和风险；
5. 中间研究笔记保留在本地，只把整理后的分类文档和 README 索引更新提交到公开仓库。

## 文档结构约定

单篇 repo 分析通常会包含：

- 一句话总结
- 项目目的与核心问题
- 核心能力
- 实现思路 / 架构拆解
- 安装与快速开始
- 在 Claude Code 中的用法
- 在 Codex 中的用法
- 典型生产力场景
- 适合 / 不适合
- 同类工具定位
- 关键文件 / 目录导读
- 学习与掌控建议
- Sources

通用模板位于：[`_templates/repo-analysis-template.md`](_templates/repo-analysis-template.md)。

## 分类约定

| 分类目录 | 说明 |
|---|---|
| `coding-agents/` | 终端/IDE/插件形态的 AI coding agent、工作流、审查工具 |
| `agent-frameworks/` | 构建 agent 应用、multi-agent、workflow runtime 的框架 |
| `mcp-tools/` | MCP server、MCP client、MCP 工具生态 |
| `llm-app-frameworks/` | LLM 应用开发框架、RAG、prompt/workflow 编排 |
| `ai-gateways/` | AI API 网关、中转、账号池、计费和模型路由平台 |
| `browser-automation/` | 浏览器控制、网页自动化、视觉/交互 agent 工具 |
| `eval-observability/` | 评测、观测、trace、回放、质量监控工具 |
| `devtools/` | 面向开发者效率的通用工具 |
| `data-tools/` | 数据处理、检索、索引、爬取和知识库工具 |
| `research-projects/` | 论文、实验性项目、研究原型 |
| `uncategorized/` | 暂未归类或需要进一步判断的项目 |

## 维护说明

- 面向读者的知识内容应沉淀在 `coding-agents/*.md`、`ai-gateways/*.md` 等分类文档里；
- 本地计划、草稿 findings、会话进度日志应保留在 `.planning/` 或其他 local-only 文件中，并保持 ignored；
- Badge 中的 Stars 由 GitHub / Shields 动态生成，Visitors 由第三方访问计数 badge 按页面加载累计，适合作为公开展示参考。

---

<div align="center">

如果这个知识库对你有帮助，欢迎 Star，也欢迎继续投喂值得研究的 AI 工具仓库。

</div>
