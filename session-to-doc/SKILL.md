---
name: session-to-doc
description: >
  Export the current or a specified coding session's work into a structured Feishu document.
  Use when the user says "总结本次session", "写个开发总结", "导出到飞书", "生成总结文档",
  "记录一下这次改动", "写个集成总结", or similar. Also used as the final step of other
  development workflow skills (debugging, feature implementation, algorithm integration) to
  auto-document results. Do NOT use for general note-taking, writing README files, or
  creating project documentation unrelated to a specific coding session.
---

# Session to Document

Compile a coding session's work into a structured markdown summary, then export it to a Feishu docx in the user's preferred folder.

## Core Principles

- **Summarize the what and why, not the play-by-play.** Don't narrate every step. Focus on design decisions, architecture, key changes, and current state.
- **Keep it terse.** One screenful per section. The developer can expand later.
- **Be honest about status.** Clearly flag known issues, unfinished work, and areas that need attention.
- **Organize for a cold reader.** Someone who joins the project later should understand the integration at a glance.

## Workflow

### Phase 1: Generate the summary file

Write a markdown summary to the appropriate location in the project. The file should be named `INTEGRATION_SUMMARY.md` (for integration work) or `<TOPIC>_SUMMARY.md` (for other work), placed in the module/feature directory that was the main focus of the session.

Use this template structure:

```markdown
# <Title>

> 状态: **<current status>** | 日期: <YYYY-MM-DD>

---

## 一、概述

Briefly describe what was done and why.

### 核心模块

| 模块 | 路径 | 职责 |
|------|------|------|
| ... | ... | ... |

### 依赖

List key dependencies (models, libraries, external services).

---

## 二、开发过程

### 阶段 N: <phase name>

**问题**: What was the challenge or starting state.

**解决/改动**:
- Key change 1
- Key change 2

---

## 三、设计说明

### 数据流 / 架构

Describe the data flow or architecture with a diagram if helpful.

### 关键配置

| 参数 | 默认值 | 说明 |
|------|--------|------|
| ... | ... | ... |

---

## 四、当前状态

### 当前进展

A brief paragraph summarizing what's done and what stage the work is in.

### 已知情况

- Item 1 (use soft language — "可能", "可考虑", not rigid checkboxes)
- Item 2

### 运行方式

```bash
# commands to run / test
```

---

## 五、相关文件清单

| 文件 | 变更类型 |
|------|----------|
| ... | ... |
```

Rules for the summary:
- **Don't list every commit.** Group related changes into phases.
- **Status should be vague for in-progress work.** Use phrases like "系统功能联调阶段" rather than detailed checklists.
- **Known issues use soft language.** "可能不一致", "可考虑结合", not rigid "[ ] TODO" items.
- **Include run/test commands** so the next developer can pick up quickly.

### Phase 2: Export to Feishu

After the markdown file is written and the user confirms:

#### Step 1: Find the target folder

Search for the user's project folder in Feishu:

```bash
lark-cli drive +search --as user --query "<folder-name>" --doc-types folder --only-title
```

The default folder name to try is the project's parent directory name (e.g., for `~/Projects/topsun_dimos`, try `topsun_dimos`). If not found, list the drive root:

```bash
lark-cli drive files list --params '{}' --format json
```

Present matching folders to the user and let them choose.

#### Step 2: Create the document

Use the relative path from the project root:

```bash
lark-cli docs +create --api-version v2 --doc-format markdown \
  --parent-token <folder-token> \
  --content @<relative-path-to-summary.md>
```

**IMPORTANT**: `--content @file` requires a **relative path** from the current working directory. Do NOT use absolute paths.

#### Step 3: Report

Tell the user the document URL and confirm it's in the right folder:

```
文档已导出:
- 标题: <title>
- 位置: <folder-name>
- 链接: <url>
```
