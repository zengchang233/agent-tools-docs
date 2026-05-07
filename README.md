# Open Source Agent Tools Docs

这个 workspace 用来归档和理解前沿 GitHub repo，重点关注：

- repo 的目的、实现方式和使用方法；
- 如何把 repo 用到 Claude Code / Codex 的日常工作流里；
- 如何通过分类文档形成可复用的 agent tools 知识库。

## 使用方式

你之后只需要发一个或多个 GitHub repo 链接。我会：

1. 读取/核验 repo 的官方信息；
2. 判断工具类型并创建分类文件夹；
3. 按 repo 名称创建独立 Markdown 文档：`<category>/<repo-name>.md`；
4. 在文档中总结：目的、核心实现、用法、Claude Code/Codex 工作流、适用场景和生产力建议；
5. 把研究过程写入 `findings.md` 和 `progress.md`。

## 分类约定

常见分类包括但不限于：

- `coding-agents/`
- `agent-frameworks/`
- `mcp-tools/`
- `llm-app-frameworks/`
- `browser-automation/`
- `eval-observability/`
- `devtools/`
- `data-tools/`
- `research-projects/`
- `uncategorized/`

## 模板

通用 repo 分析模板在：

- `_templates/repo-analysis-template.md`
