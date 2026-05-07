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

## Future Research Notes
第一个 repo 已归档。后续继续按 repo 链接创建分类文档。
