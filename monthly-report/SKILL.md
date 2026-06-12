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

Interactive workflow: interview → draft → export. Thorough interview + careful draft review = one-pass export. Produces a concise, leadership-facing Feishu document.

## Core Principles

- **Leadership-facing.** Highlight results, business value, and system-level thinking, not implementation details.
- **Structured and scannable.** Tables for achievements, bullet lists for plans. One screenful per task.
- **All input from user.** Do NOT auto-extract from git log, CLAUDE.md, or other sources.

## Workflow

### Phase 1: Interview

Ask the user questions in this order. For each, show a brief example of a good answer.

**Step 1: Metadata**

```
月报覆盖哪个时间段？（如：2026年6月）
```

**Step 2: Task overview**

```
几个任务？每个给个简短标签 + 一句话状态（如"核心链路已跑通""初步打通""已结项"）。
```

**Step 3: Per task, collect these fields**

| # | Field | Notes |
|---|-------|-------|
| 1 | **任务标题** | e.g. "割草机移动抓取仿真平台搭建" |
| 2 | **背景与目标** | 1 paragraph. Why this task exists — team needs, existing demo it builds on, strategic context. |
| 3 | **代码仓库** | GitHub/GitLab URL, include branch if on feature branch |
| 4 | **开发记录** | Feishu docx URL of the detailed dev log |
| 5 | **核心产出** | 1 sentence: main modules/components delivered |
| 6 | **E2E 验证** *(optional)* | Performance metrics, test results |
| 7 | **集成状态** *(optional)* | Which sub-links are verified |
| 8 | **演示视频** | Local file paths (will be uploaded and embedded as cards) |
| 9 | **后续计划** | 2-4 bullets. System-level capability gaps, NOT implementation todos. Help the user reframe if they give code-level items — ask "这个要解决的系统级问题是什么？" |
| 10 | **任务状态** | Ongoing → `### 后续计划`. Concluded → `### 结论` with reason + what was gained. |

**Terminology defaults (apply proactively, don't ask):**

| User says | Use instead |
|-----------|-------------|
| "纯算法方案" | 板端轻量方案 |
| "待解决" / "计划" / "TODO" | 后续计划 |
| 代码变更量/commit数/LOC | Omit entirely, or weave into 核心产出 |

### Phase 2: Draft

Fill in this template with the collected info. Present for review. Do not export until confirmed.

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

<1 paragraph>

### 成果

| 维度 | 内容 |
|------|------|
| 代码仓库 | [<repo-name>](<URL>) |
| 核心产出 | <1 sentence> |
<!-- 可选行：E2E 验证、集成状态，有则加 -->
| 开发记录 | [<Doc Title>](<Feishu URL>) |
| 演示视频 | 见下方 |

<!-- 视频卡片在导出阶段用 docs +media-insert 嵌入，此处留文字占位即可 -->

### <结论 或 后续计划>

<!-- 已结项：结论段落；进行中：bullet list -->
- **<系统能力缺口>** — <为什么重要、对整体系统的意义>
- ...

---

## 任务二：...

```

---

**Rules embedded in the template:**
- 成果表格：`维度 | 内容` 两列。**不含"代码变更量"行。**
- 后续计划：bullet list，非表格。站在系统完整性和需求高度，每条 = **加粗关键词** + 破折号 + 一句话。
- **无"总结与展望"章节。** 每个任务已有自己的后续计划。
- 视频：草稿中写"见下方"占位，导出时用 `docs +media-insert --type file --file-view card` 嵌入。
- 语言精简，去掉开发过程、调试故事、迭代记录，只引开发文档链接。

### Phase 3: Export

Only after user confirms the draft.

**Prerequisites:** Read `../lark-shared/SKILL.md` and `lark-doc` create/update references first. lark-cli must be available.

**Step 1: Locate reports folder**
```bash
lark-cli drive +search --as user --query "reports" --doc-types folder --only-title
```

**Step 2: Create document** (draft file must be relative path; flag is `--parent-token`, NOT `--folder-token`)
```bash
lark-cli docs +create --api-version v2 --doc-format markdown \
  --parent-token <folder-token> --content @./monthly_report_draft.md
```

**Step 3: Upload and embed videos** (per video, in task order)
```bash
lark-cli drive +upload --file <relative-path> --parent-token <folder-token>
lark-cli docs +media-insert --type file --file-view card \
  --doc "<doc-url>" --file-token <token>
```

**Step 4: Verify and report** — fetch doc, confirm structure, tell user URL and folder.

---

## Edits After Export

For occasional post-export tweaks. Use `lark-cli docs +update --api-version v2`.

**Critical distinction:** XML-mode `str_replace` is block-scoped — can't match across `<tr>`, `<td>`, `<p>`. For table rows or multi-block content, always use `--doc-format markdown` and fetch in markdown first to see exact match text.

| Edit | Mode | Example |
|------|------|---------|
| Inline text | XML (default) | Fix a typo |
| Table row / multi-block | `--doc-format markdown` | Delete row, replace section |

**Delete a table row:**
```bash
lark-cli docs +update --api-version v2 --doc "<url>" --command str_replace \
  --doc-format markdown --pattern "| 维度 | 内容 |" --content ""
```
After deletion, check if a blank line split the table — if so, merge by replacing `|\n\n|` → `|\n|`.

**Replace a section:** Write old/new content to temp files, use `$(cat file)` to avoid shell escaping.

**Ambiguous matches:** Include surrounding context in the pattern to make it unique.

## Common Pitfalls

1. `--folder-token` for doc creation → use `--parent-token`.
2. Absolute paths with lark-cli → use relative paths (`@./file.md`).
3. XML mode for table edits → always use `--doc-format markdown`.
4. Text links for videos → always use `docs +media-insert --type file --file-view card`.
5. "代码变更量" row → never include; weave stats into 核心产出 if needed.
6. "待解决" / "计划" as heading → use `### 后续计划`.
7. "总结与展望" section → don't write one.
8. "纯算法方案" → 板端轻量方案.
