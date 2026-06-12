---
name: monthly-report
description: >
  Interactive workflow for writing structured monthly/periodic work reports and exporting
  them to Feishu (飞书) documents. Use when the user says "写月报", "工作月报", "月度汇报",
  "周报", "季度汇报", "写个汇报", "导出月报到飞书", or similar. Also use when the user
  wants to summarize multiple development tasks into a leadership-facing report with
  structured formatting (tables, video cards, bullet plans). Do NOT use for general
  note-taking, single-task summaries, or creating README files.
---

# Monthly Report (工作月报)

Interview → research → draft → export. The model reads the user's code repos and dev docs to understand each task, then fills in the template. Produces a concise, leadership-facing Feishu document.

## Core Principles

- **Leadership-facing.** Highlight results, business value, and system-level thinking, not implementation details.
- **Structured and scannable.** Tables for achievements, bullet lists for plans. One screenful per task.
- **Model does the research.** User provides pointers (repo, doc, video); model reads and extracts what it needs.

## Workflow

### Phase 1: Gather Pointers

Ask the user for the reporting period and, for each task, the key resources to research.

**Step 1: Period**

```
月报覆盖哪个时间段？（如：2026年6月）
```

**Step 2: Per task, ask for these three things**

逐个任务询问：

```
任务 N：
- 代码仓库地址（GitHub/GitLab URL，如有特定分支请注明）
- 开发记录文档（飞书 docx URL）
- 演示/调试视频（本地文件路径，可多个）
```

Ask the user for any additional context they want to add (task framing, strategic background, special status like "已结项"). But the model should extract the detailed content by reading the provided resources.

### Phase 2: Research & Draft

For each task, read the provided resources to understand what was done:

1. **Code repo**: Read CLAUDE.md, README, and key source files to understand the module structure, architecture, and capabilities.
2. **Dev doc**: Fetch the Feishu docx (`lark-cli docs +fetch --api-version v2 --doc-format markdown`) to understand the development process, design decisions, and current status.
3. **Videos**: Note file paths for later upload; optionally skim to understand the demo content.

From this research, extract: task title, background & goals, core deliverables, E2E results, integration status, and system-level plans. Fill in the template below.

---

```markdown
# 工作月报（YYYY年M月）

## 概览

本月围绕N个方向开展工作：

- **<任务标签>** — <一句话进展>
- ...

---

## 任务一：<任务标题>

### 背景与目标

<1 paragraph — business context, team needs, strategic motivation>

### 成果

| 维度 | 内容 |
|------|------|
| 代码仓库 | [<repo-name>](<URL>) |
| 核心产出 | <1 sentence — main modules/components delivered> |
| 开发记录 | [<Doc Title>](<Feishu URL>) |
| 演示视频 | 见下方 |

<!-- 成果表格可额外补充：E2E 验证、集成状态等行 -->

### <结论 或 后续计划>

<!-- 已结项任务用"结论"，进行中用"后续计划" -->
<!-- 后续计划用 bullet list，每条 = **加粗关键词** + 破折号 + 一句话，站在系统完整性和需求高度 -->

- **<系统能力缺口>** — <为什么重要、对整体系统的意义>

---

## 任务二：...

```

---

**Template encodes these rules — no additional notes needed:**
- 成果表格两列 `维度 | 内容`。不含"代码变更量"行，不纯算法→板端轻量。
- 后续计划是 bullet list 不是表。无"总结与展望"章节。
- 视频在草稿中写"见下方"占位，导出时用卡片嵌入。
- 语言精简，去掉开发过程和调试故事，只引开发文档链接。

Present the draft for review. Do not export until confirmed.

### Phase 3: Export

Only after user confirms.

**Prerequisites:** Read `../lark-shared/SKILL.md` and `lark-doc` create/update references. lark-cli must be available.

```bash
# 1. Find reports folder
lark-cli drive +search --as user --query "reports" --doc-types folder --only-title

# 2. Create document (--parent-token, NOT --folder-token; draft must be relative path)
lark-cli docs +create --api-version v2 --doc-format markdown \
  --parent-token <folder-token> --content @./monthly_report_draft.md

# 3. Upload and embed each video (in task order)
lark-cli drive +upload --file <relative-path> --parent-token <folder-token>
lark-cli docs +media-insert --type file --file-view card \
  --doc "<doc-url>" --file-token <token>

# 4. Verify
lark-cli docs +fetch --api-version v2 --doc "<doc-url>" --doc-format markdown
```
