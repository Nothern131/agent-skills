# Agent Skills

A collection of Agent Skills for AI coding assistants that support the [Agent Skills specification](https://agentskills.io/specification).

## Skills

| Skill | Description |
|-------|-------------|
| [codeguard](./skills/codeguard/) | Automated code quality scanner — 80 rules across 8 dimensions (error handling, security, database, diagnostics, resilience, memory safety, concurrency, resource management) with 0–100 scoring. Covers Python, JS/TS, C/C++, GDScript, Go |
| [clarifying-questions](./skills/clarifying-questions/) | Helps AI precisely understand user requirements by asking targeted clarifying questions before starting work |
| [model-upgrade](./skills/model-upgrade/) | Engineering framework that enables any AI coding assistant (domestic or international) to achieve GPT/Claude parity through structured rules, test-feedback loops, task decomposition, and project context awareness |

## Installation

### Claude Code

```bash
/plugin marketplace add Nothern131/agent-skills
```

Or manually add to your `.claude/skills/` directory.

### Trae

Copy the skill folder into your Trae skills directory:
- **macOS**: `~/.trae/skills/`
- **Windows**: `%USERPROFILE%\.trae\skills\`

### Cursor

1. Open Cursor Settings → Features → Rules
2. Add the skill content as a Project Rule or User Rule
3. Or place `SKILL.md` content in `.cursor/rules/` directory

### Windsurf (Codeium)

1. Open Windsurf Settings → AI → Rules
2. Add the skill as a custom rule
3. Or place in `.windsurfrules` file

### Cline (VS Code Extension)

1. Open Cline settings → Custom Instructions
2. Paste the skill instructions from `SKILL.md`
3. Or add to `.clinerules` file in project root

### Aider

```bash
# Add as custom instruction file
aider --message-file skills/codeguard/SKILL.md
# Or add to .aider.conf.yml
echo "read: skills/codeguard/SKILL.md" >> .aider.conf.yml
```

### Continue.dev

1. Open Continue config (`~/.continue/config.json`)
2. Add the skill as a custom context provider or in `customCommands`
3. Reference the `SKILL.md` file path

### GitHub Copilot

1. Create `.github/copilot-instructions.md` in your repo
2. Paste the skill content from `SKILL.md`
3. Copilot will automatically follow these instructions

### Augment

1. Open Augment settings → Custom Instructions
2. Add the skill instructions from `SKILL.md`

### Zed

1. Open Zed settings → AI → Context Servers
2. Add the skill files as context

### Generic / Any AI Assistant

Most AI coding tools support custom instructions or rules files. The universal approach:

1. Copy `SKILL.md` content into your tool's custom instructions / system prompt / rules file
2. Copy the `references/` folder alongside it for on-demand rule loading
3. The YAML frontmatter (`name`, `description`) helps the AI decide when to activate the skill

Common config file names:
| Tool | Config File |
|------|------------|
| Claude Code | `.claude/skills/` |
| Trae | `.trae/skills/` |
| Cursor | `.cursor/rules/` |
| Windsurf | `.windsurfrules` |
| Cline | `.clinerules` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Aider | `.aider.conf.yml` |
| Continue | `~/.continue/config.json` |

## Skill Structure

Each skill follows the [Agent Skills specification](https://agentskills.io/specification):

```
skill-name/
├── SKILL.md           # Required — metadata + instructions
├── LICENSE.txt        # License
├── README.md          # Skill documentation
└── references/        # Optional — detailed reference docs (loaded on demand)
```

## Creating a New Skill

Use the [template](./template/) as a starting point:

```markdown
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---

# My Skill Name

[Add your instructions here]
```

The frontmatter requires only two fields:
- `name` — A unique identifier (lowercase, hyphens for spaces)
- `description` — What the skill does and when to trigger it

## License

Each skill has its own license. See the `LICENSE.txt` file in each skill folder.
