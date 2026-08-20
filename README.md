# Agent Skills

A collection of Agent Skills for AI coding assistants that support the [Agent Skills specification](https://agentskills.io/specification).

> **v5** — 15-rule framework, 8 skills, 30 auto-trigger rules, Autonomous Loop, 7-step YAGNI ladder, Meta-skill, Slice-check batching. **Mandatory enforcement** — rules apply to all tasks, no exceptions. 56% shorter than v4.5 (214 vs 484 lines).

## Skills

### Project Understanding
| Skill | Description |
|-------|-------------|
| [codebase-indexer](./skills/codebase-indexer/) | Generates a codebase map (directory tree, module descriptions, entry points, dependency graph) for domestic AI models lacking native codebase indexing |

### Quality Assurance
| Skill | Description |
|-------|-------------|
| [codeguard](./skills/codeguard/) | 80 rules across 8 dimensions (error handling, security, database, diagnostics, resilience, memory safety, concurrency, resource management) with 0–100 scoring. Covers Python, JS/TS, C/C++, GDScript, Go |
| [test-automation](./skills/test-automation/) | Auto-detects test framework, runs tests, parses results, and feeds back to Closed Loop. Supports pytest, unittest, Jest, Mocha, Go test, GDScript GUT, CTest, Catch2 |

### Engineering Efficiency
| Skill | Description |
|-------|-------------|
| [ponytail-ladder](./skills/ponytail-ladder/) | 7-step decision ladder that prevents over-engineering. **-54% code, -22% tokens, 100% security.** Based on [Ponytail](https://github.com/DietrichGebert/ponytail) (62K stars, MIT). 3 intensity levels (lite/full/ultra). |
| [skill-creator](./skills/skill-creator/) | **Meta-skill**: teaches AI agents how to create new skills autonomously. Includes YAML format, 5-step creation flow, 10-item validation checklist, 7 anti-patterns, and auto-registration to trigger table. |
| [slice-check](./skills/slice-check/) | **Slice-check batching**: splits large tasks into functional slices (≈ one day of a top engineer's work), enforcing a quality gate (run-verify → codeguard → adversarial review → delivery template) after each slice, with per-slice user confirmation. Prevents writing thousands of lines before reviewing. |

### AI Enhancement
| Skill | Description |
|-------|-------------|
| [model-upgrade](./skills/model-upgrade/) | Engineering framework enabling GPT/Claude parity through structured rules, test-feedback loops, task decomposition, and project context — now with 15-rule constitution, Auto Loop, and 7-step ladder |
| [clarifying-questions](./skills/clarifying-questions/) | Structured questioning system that helps AI precisely understand user requirements before starting work |

## Key Features (v4.5)

### 15-Rule Constitution
1-4: Pre-coding (ask first, explore first, impact analysis, simplicity first)
5-8: During coding (single focus, match style, no silent failures, security labels)
9-10: Post-coding (goal-driven verification, evidence completion)
11-13: Model behavior (anti-hallucination, Chinese comments, no fluff)
14: Skill-first — must call Skill tool when matched
15: **7-step decision ladder** — before writing any code, climb the ladder: need it? → exists? → stdlib? → native? → installed dep? → one line? → minimum implementation

### Autonomous Loop (v4)
- **Closed Loop**: auto-verify after changes, auto-diagnose on failure, auto-fix → re-verify
- **3-round stop-loss**: auto-fix attempts capped at 3, then requests human intervention
- **Cascade check**: Grep all callers after each fix to prevent cascading failures
- **Log-driven diagnosis**: read logs before fixing, never guess blind
- **Intent re-alignment**: verify direction still matches user's core intent each loop

### 30 Auto-Trigger Rules
30 trigger conditions in the 2.11 table automatically load the right skill — no waiting for user to ask. Covers: codebase exploration, testing, code review, frontend design, game development, data analysis, web scraping, MCP building, and more.

### Slice-Check Batching (v5.1)
Batches large tasks into functional slices (≈ one day of a top engineer's work) with a mandatory quality gate after each slice: run-verify → codeguard scan → adversarial review → delivery template. Per-slice user confirmation prevents finishing thousands of lines before review. Triggered for ≥3-file / multi-module / new-feature tasks.

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
