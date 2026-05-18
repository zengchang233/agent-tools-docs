# Task Plan — GitHub Repo 理解与归档 Workspace

## Goal
把 `/data/projects/opensource/docs` 作为长期知识库：用户提供前沿 GitHub repo 链接后，分析该 repo 的目的、核心实现、适用场景，以及在 Claude Code / Codex 中的用法，并按工具类型分类保存为独立 Markdown 文档。

## Current Phase
Phase 5 complete: `Wei-Shaw/sub2api` learning document is archived.

## Operating Rules
1. 每次收到 repo 链接，先确认 repo 名称、官方链接、当前 README/文档信息；需要最新信息时必须联网核验。
2. 输出文档命名与 repo 一致：`<repo-name>.md`。
3. 按工具所属类型创建文件夹，例如：`agent-frameworks/`、`coding-agents/`、`mcp-tools/`、`llm-app-frameworks/`、`eval-observability/`、`browser-automation/`、`devtools/` 等。
4. 文档必须包含 repo 链接。
5. 文档必须覆盖：目的、解决的问题、核心实现/架构、安装/快速开始、在 Claude Code 中的用法、在 Codex 中的用法、适合/不适合场景、同类工具对比或定位、生产力建议。
6. 若分类不确定，先使用最贴近的类型文件夹，并在文档中说明分类依据。
7. 研究过程中遵守 planning-with-files：关键发现写入 `findings.md`，过程写入 `progress.md`。

## Phases

### Phase 1: Initialize workspace memory
- Create planning files and persistent workflow notes.
- **Status:** complete

### Phase 2: Create reusable documentation template
- Add a repo analysis template for future repo summaries.
- **Status:** complete

### Phase 3: Process first repo link
- User provided `PolyArch/humanize`; create categorized repo document.
- **Status:** complete

### Phase 4: Process second repo link
- User provided `OthmanAdi/planning-with-files`; create categorized repo document.
- **Status:** complete

### Phase 5: Process third repo link
- User provided `Wei-Shaw/sub2api`; create categorized repo document.
- **Status:** complete

## Default Repo Analysis Workflow
1. Parse repo URL and infer repo slug.
2. Browse official GitHub repo and docs; prefer primary sources.
3. Identify category and create category folder if absent.
4. Draft `<category>/<repo-name>.md` using `_templates/repo-analysis-template.md`.
5. Include repo URL, dates checked, and citations/source links when appropriate.
6. Update `findings.md` with classification/important notes.
7. Update `progress.md` with files created/changed.

## Errors Encountered
| Error | Attempt | Resolution |
|---|---:|---|
| `planning-with-files` completion check reported `0/0 phases complete` | 1 | Converted phase tracking from a Markdown table to the skill-recognized `### Phase` plus `**Status:** complete` format. |
| `git diff --stat` failed because `/data/projects/opensource/docs` is not a git repository | 1 | Treated as expected workspace condition; verification used file reads and planning check instead. |
