# Agent Skills

A collection of Agent Skills for Claude Code, Trae, and other AI coding assistants that support the [Agent Skills specification](https://agentskills.io/specification).

## Skills

| Skill | Description |
|-------|-------------|
| [codeguard](./skills/codeguard/) | Automated code quality scanner — 80 rules across 8 dimensions (error handling, security, database, diagnostics, resilience, memory safety, concurrency, resource management) with 0–100 scoring. Covers Python, JS/TS, C/C++, GDScript, Go |
| [clarifying-questions](./skills/clarifying-questions/) | Helps AI precisely understand user requirements by asking targeted clarifying questions before starting work |

## Quick Start

### Install in Claude Code

```bash
/plugin marketplace add Nothern131/agent-skills
```

### Install in Trae

Copy the skill folder into your Trae skills directory.

### Manual Installation

Copy the desired skill folder (e.g., `skills/codeguard/`) into your local skills directory.

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
