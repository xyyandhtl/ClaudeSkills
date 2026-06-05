---
name: integrate-algorithm
description: >
  Structured workflow for integrating an external algorithm, library, or model into a
  framework project. Use when the user says "集成XX算法到项目里", "把XX模型集成进来",
  "integrate this open-source library into the project", "import external algorithm into
  the codebase", or is bringing outside code that needs adaptation to fit the project's
  architecture, module system, and conventions. Also use when the user asks to "集成" or
  "接入" any external repository, model checkpoint, or standalone algorithm. Do NOT use
  for adding simple dependencies (pip install / npm install), regular feature development,
  or bug fixes — only when significant adaptation and integration work is needed.
---

# Integrate Algorithm

Systematic workflow for integrating an external algorithm, library, or model into a target framework project.

## Core Principles

- **Understand before mapping.** Read the external code and the target framework's architecture docs (CLAUDE.md, CONTRIBUTING.md, etc.) before writing any code. A wrong mapping creates more rework than a slow start.
- **Adapt to the framework, not the other way around.** The external algorithm should conform to the project's module/plugin/service patterns, not introduce its own conventions.
- **Fix incrementally.** One module at a time. Run tests after each change. Don't batch unrelated fixes.
- **No network at runtime.** Models, weights, and configs should load from local paths. Add `local_files_only=True` or equivalent for all remote fetchers.
- **Document at the end, not throughout.** Write the integration summary after the work stabilizes. Use the `session-to-doc` skill for the final export.

## Workflow

### Phase 1: Study the external algorithm

Before touching any project code, understand what you're integrating:

1. **Read the external codebase structure.** Identify:
   - Entry points (main scripts, inference APIs)
   - Model loading and checkpoint paths
   - Key dependencies (PyTorch, TensorFlow, ONNX, etc.)
   - Configuration system
   - Any `sys.path` hacks or vendored dependencies

2. **Run a minimal inference test if possible.** Verify the external code actually works before integrating it. This catches broken checkpoints, missing dependencies, and version mismatches early.

3. **Note the input/output contract.** What goes in (images, text, sensor data)? What comes out (tensors, masks, positions)?

### Phase 2: Study the target framework

Read the project's architecture documentation:

1. **Read CLAUDE.md or equivalent** — understand the module system, plugin patterns, configuration, testing conventions.
2. **Find a similar existing module** as a reference. For example, if integrating a navigation algorithm, study an existing navigation module to understand the stream/skill/blueprint patterns.
3. **Identify the integration points:**
   - Where should the new module live in the directory tree?
   - What streams/topics does it consume and produce?
   - What spec/protocol does it expose to other modules?
   - How is it composed into a runnable blueprint?

### Phase 3: Map and scaffold

Design the mapping from external algorithm to target framework:

1. **Module mapping:** External model class → framework Module subclass
2. **API mapping:** External function calls → framework RPC/skill methods
3. **Data mapping:** External I/O formats → framework stream types
4. **Config mapping:** External hardcoded params → framework config system

Create the minimal file skeleton first — `__init__.py`, the main module file, a test file — before filling in implementation.

### Phase 4: Fix imports and dependencies

This is usually the most mechanical but error-prone phase:

1. **Remove vendoring hacks.** Delete `_vendor/` directories. Replace `sys.path.insert()` with proper relative imports.
2. **Fix relative imports.** External code often uses absolute imports assuming a specific `PYTHONPATH`. Convert to package-relative imports (`from .submodule import X`).
3. **Local model loading.** For ML models:
   - Find where checkpoints are downloaded/cached
   - Add `local_files_only=True` for HuggingFace or equivalent for other frameworks
   - Fix cache directory paths
   - Confirm models load without network access
4. **Compatibility shims.** If the external code uses a different version of a shared dependency, add minimal compatibility fixes (e.g., adding a missing parameter with a default value). Document why each shim is needed.

**Test after each import fix.** Don't fix all imports at once — fix one, test, repeat.

### Phase 5: Write tests

Write tests early, before deep debugging:

1. **Model loading tests** — does the model instantiate and load weights correctly?
2. **Inference smoke tests** — given a known input, does forward() return the expected shapes and dtypes?
3. **Logic/unit tests** — coordinate transforms, math utilities, configuration parsing
4. **Integration/skill tests** — does the skill method accept input and return the expected string?

Use the project's existing test framework. Match the project's test style (fixtures, mocking patterns, markers).

If model tests require GPU and large checkpoints, mark them appropriately so they can be skipped in CI.

### Phase 6: Debug and iterate

This phase dominates the timeline. Use a tight loop:

1. **Run the module end-to-end.** If the framework has a run command (like `dimos run`), use it.
2. **When something breaks:**
   - Read the error carefully — don't guess
   - Form a hypothesis about the cause
   - Test with minimal change
   - If stuck, consult the `dev-debugger` skill for systematic debugging
3. **Common integration bugs:**
   - Stream/topic name mismatches
   - Serialization format differences
   - Worker process vs main process state (fork issues)
   - Missing configuration entries
   - Visualization/dashboard entity paths

4. **Fix one thing at a time.** After each fix, re-run to confirm. Stacking fixes makes it impossible to know which one worked.

### Phase 7: Tune runtime behavior

Once the module runs end-to-end, tune for production:

1. **Frame rate / throughput.** If the algorithm can't run in real time, add configurable throttling (e.g., `--frame-hz 5.0`).
2. **Termination / edge cases.** What happens when:
   - Input is empty or malformed?
   - The model produces no output?
   - A timeout is hit?
   - A dependency service (VLM, database) is unavailable?
3. **Health monitoring.** Add per-call health checks for external services (API keys, network connections). Log errors explicitly rather than silently swallowing them.
4. **Configuration surface.** Move hardcoded tuning parameters to the module's config or constructor. Document each parameter.

### Phase 8: Document

After the integration stabilizes:

1. **Call the `session-to-doc` skill** to generate the integration summary and export it to Feishu. If `session-to-doc` is not available, follow its workflow manually:
   - Write `INTEGRATION_SUMMARY.md` in the module directory
   - Use the template from `session-to-doc` for structure
   - Export to Feishu with `lark-cli docs +create`

2. **Update the project's CLAUDE.md** if the integration adds new patterns, conventions, or gotchas that future developers should know.

## Common Patterns

### For ML Model Integration

```
External repo structure:
  model/
    backbone.py      → stays as-is after import fixes
    adaptor.py       → stays as-is
    inference.py     → maps to framework Module

Target framework structure:
  project/
    domain/model_name/
      __init__.py
      inference.py   ← wraps external inference
      tests/
        __init__.py
        test_inference.py
    agents/skills/
      skill_for_model.py  ← uses the inference module
    robot/<platform>/blueprints/
      blueprint_with_model.py  ← composes everything
```

### For Import Fixes (Python)

Before:
```python
import sys
sys.path.insert(0, os.path.join(os.path.dirname(__file__), '_vendor'))
from external_lib import Something
```

After:
```python
from project.domain.model_name.external_lib import Something
```

### For Local Model Loading (Python / HuggingFace)

Before:
```python
model = AutoModel.from_pretrained("org/model-name")
```

After:
```python
model = AutoModel.from_pretrained(
    "org/model-name",
    cache_dir=Path(ckpt_dir) / "hf_cache",
    local_files_only=True,
)
```

## References

- For systematic debugging during Phase 6, call the `dev-debugger` skill.
- For the final documentation step, call the `session-to-doc` skill.
- Read the project's CLAUDE.md before starting — it contains framework conventions, testing instructions, and code style rules.
