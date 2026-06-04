---
name: dev-debugger
description: >
  Structured debugging workflow for development problems. Use when the user reports a bug, error,
  unexpected behavior, or says things like "帮我排查", "这个报错是怎么回事", "为什么XX不工作",
  "怎么修这个bug", "帮我看看这个问题", "调试一下". Also use when the user is stuck on a
  development issue and wants to systematically debug it. After resolution, auto-documents the
  fix to Feishu docx. Do NOT use for general coding tasks, feature requests, code review, or
  writing new features — only for debugging existing problems.
---

# Dev Debugger

Systematic debugging workflow: investigate → fix → confirm → document.

## Core Principles

- **Don't guess blindly.** Form explicit hypotheses and test them. Each step should narrow the problem space.
- **Only interrupt when you genuinely need the developer.** If you're confident about the next step, take it. Pause only at real forks: multiple plausible causes with similar probability, need domain knowledge you don't have, need the developer to run something on hardware you can't access.
- **Skip dead ends in the final write-up.** The documentation focuses on what actually mattered — the key contradiction and how it was resolved. Obvious failed attempts that any reasonable debugging would have tried are noise; omit them.

## Workflow

### Phase 1: Triage

1. **Capture the symptom.** What exactly happened? Get the error message, stack trace, log output, or unexpected behavior description.
2. **Reproduce if possible.** If the error is reproducible, understand the minimal steps.
3. **Locate the relevant code.** Read the files in the error stack, or search for the relevant module/function. Don't read entire files — target the specific area.

### Phase 2: Investigate

1. **Form a hypothesis.** Based on the symptom and code, state what you think is wrong.
2. **Test the hypothesis.** This could mean:
   - Adding targeted logging / print statements
   - Reading more code to trace data flow
   - Checking git blame for recent changes in the area
   - Running a specific test or reproducer
3. **If the hypothesis is wrong,** state why clearly. Form a new hypothesis. Repeat.
4. **When you hit a genuine fork** (multiple equally likely causes, or need the developer's domain knowledge), pause and ask:
   - What you've found so far
   - The plausible directions
   - Which direction the developer wants to pursue

Keep the developer informed at key moments: "Found X, investigating Y now."

### Phase 3: Fix

1. **Implement the minimal fix.** Don't refactor unrelated code. Don't add features. Fix the problem and only the problem.
2. **Verify.** Run the relevant tests. If the project has no tests for this path, explain how the developer can manually verify.
3. **If verification fails,** go back to Phase 2.

### Phase 4: Confirm

**CRITICAL — do NOT skip this step.** After the fix is verified:

1. Summarize concisely:
   - What the problem was (1 sentence)
   - What caused it (1 sentence)
   - What the fix was (1 sentence)
2. Ask explicitly: **"问题已解决了吗？"** or equivalent.
3. **Wait for the developer to say yes.** If they say no, go back to Phase 2. If they say yes, proceed to Phase 5.

### Phase 5: Document to Feishu

Only after the developer confirms resolution.

#### Step 1: Find the target folder

List folders in the developer's Feishu drive root:

```bash
lark-cli drive files list --params '{}' --format json
```

Filter for `"type": "folder"` entries. Present the folder names to the developer and ask which one to save into. If `dev_docs` exists, suggest it as the default.

#### Step 2: Write the document

Create a docx in the chosen folder. Use `lark-cli docs +create` with the DocxXML format. The document **title** should be a concise summary of the problem (e.g. "修复 XX 模型加载 OOM 问题").

The document body must follow this structure:

```
<title>文档标题</title>
<h1>问题现象</h1>
<p>一句话描述 + 关键错误信息/日志片段。</p>
<h1>根因分析</h1>
<p>导致问题的根本原因。描述关键矛盾，不要流水账式地记录排查过程。</p>
<h1>解决方案</h1>
<p>修复方案简述。附带关键代码变更（最多2-3个代码块）：</p>
<code language="python"># 变更1：xxx
...</code>
<code language="python"># 变更2：xxx
...</code>
<h1>涉及文件</h1>
<ul>
<li><code>path/to/file.py</code> — 改动说明</li>
</ul>
```

Rules for the document:
- **Keep it terse.** The developer can always expand later. Aim for one screenful.
- **Code blocks only for key changes.** Don't include boilerplate or trivial renames. 2-3 code blocks max.
- **Skip dead ends.** Don't document hypotheses that turned out to be wrong, unless the contrast is genuinely illuminating.
- **No "经验教训" section.** The user doesn't want it.

#### Step 3: Confirm document location

After creating, tell the developer the document URL and which folder it's in.
