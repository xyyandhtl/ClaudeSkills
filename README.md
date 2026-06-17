# ClaudeSkills

Personal Claude Code skills collection. Each subdirectory is a standalone skill that can be installed individually.

## Available Skills

| Skill | Description |
|-------|-------------|
| [dev-debugger](./dev-debugger/) | Structured debugging workflow: investigate → fix → confirm → document to Feishu |
| [integrate-algorithm](./integrate-algorithm/) | Integrate external algorithms/libraries/models into a framework project: study → map → fix imports → test → debug → tune → document |
| [monthly-report](./monthly-report/) | Interactive workflow for writing structured monthly/periodic work reports and exporting to Feishu |
| [session-to-doc](./session-to-doc/) | Export a coding session's work to a structured Feishu document. Reusable by other skills as a final documentation step. |
| [closed-loop-dev-migration](./closed-loop-dev-migration/) | Migrate the closed-loop development methodology to a new project: analysis → adaptation → pilot → validation |

## Install a Skill

```bash
# Install a single skill into Claude Code
claude skills add <skill-name> --source /path/to/ClaudeSkills/<skill-name>

# Or symlink directly
ln -s /path/to/ClaudeSkills/<skill-name> ~/.claude/skills/<skill-name>
```

## Adding a New Skill

Each skill lives in its own directory with this minimal structure:

```
my-new-skill/
├── SKILL.md              # Required: YAML frontmatter (name, description) + instructions
├── scripts/              # Optional: executable scripts for repetitive tasks
├── references/           # Optional: reference docs loaded on demand
└── assets/               # Optional: templates, images, etc.
```

1. Create a new directory under the repo root
2. Write `SKILL.md` with proper frontmatter
3. Commit and push

See [Claude Code skill documentation](https://docs.anthropic.com/en/docs/claude-code/skills) for the full SKILL.md specification.
