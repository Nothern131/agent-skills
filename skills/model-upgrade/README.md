# Model Upgrade Framework

> **English name:** Model Upgrade Framework  
> **Chinese name:** 模型升级计划  
> **Tagline:** Engineering discipline that makes any AI coding assistant produce GPT/Claude-level output.

## What Is This?

A comprehensive framework that bridges the gap between domestic AI models (Qwen, GLM, Yi, DeepSeek, etc.) and international leaders (GPT-4, Claude, Gemini) through **structured engineering practices**, not raw model capability.

## Why Does This Exist?

Domestic models are very close to GPT/Claude in single-turn code generation. The gap appears in:
- **Complex multi-step reasoning** — domestic models lose track over long tasks
- **Instruction following** — domestic models occasionally skip secondary constraints
- **Self-correction** — domestic models struggle with autonomous error recovery loops
- **Long-context maintenance** — domestic models may miss details in large codebases

This framework compensates for those gaps through **process**, not model capability.

## How Does It Work?

### Four-Layer Defense System

```
Quality = Model_Raw_Capability × Engineering_Multiplier

Engineering_Multiplier = Rules × Context × Decomposition × Verification
```

| Layer | What It Does | File |
|-------|-------------|------|
| **Iron Rules** | Non-negotiable engineering discipline | `templates/rules.md` |
| **Project Context** | Full codebase awareness | `templates/PROJECT_CONTEXT.md` |
| **Task Decomposition** | Big tasks → small verifiable steps | Built into SKILL.md |
| **Test-Feedback Loop** | Automatic verify → fix → verify cycle | Built into SKILL.md |

### Key Features

- **15+ Engineering Rules** — Impact analysis, one-concern-per-change, three-attempt limit, checkpoint verification
- **4 Prompt Templates** — Ready-to-use templates for refactoring, bug fixing, feature implementation, code review
- **Self-Enhancement Strategies** — Few-shot examples, step-by-step verification, context window optimization, MCP tool augmentation
- **Language-Specific Adjustments** — Tailored guidance for Python, JS/TS, GDScript, C/C++
- **Complete Session Workflow** — From session start to end, every step is defined

## Installation

### Option 1: As an Agent Skill

Copy the `model-upgrade/` folder into your AI coding assistant's skills directory:

| Tool | Path |
|------|------|
| Claude Code | `.claude/skills/model-upgrade/` |
| Trae | `.trae/skills/model-upgrade/` |
| Cursor | `.cursor/rules/model-upgrade/` |
| Windsurf | `.windsurfrules` (merge content) |

### Option 2: As Project Rules

1. Copy `templates/rules.md` to your project root as `.trae/rules.md` or `CLAUDE.md`
2. Copy `templates/PROJECT_CONTEXT.md` to your project root
3. The AI assistant will automatically follow these rules

### Option 3: Manual Integration

Paste the content of `SKILL.md` into your AI assistant's custom instructions or system prompt.

## Quick Start

### Step 1: Initialize Your Project

```bash
# Copy templates to your project
cp skills/model-upgrade/templates/rules.md .trae/rules.md
cp skills/model-upgrade/templates/PROJECT_CONTEXT.md ./PROJECT_CONTEXT.md
```

### Step 2: Generate Initial Context

Start a session with your AI assistant and say:
> "Initialize the project context. Generate PROJECT_CONTEXT.md for this codebase."

### Step 3: Use the Framework

For any coding task, the AI assistant will:
1. Load PROJECT_CONTEXT.md
2. Analyze impact
3. Break task into steps
4. Execute with verification
5. Update context

## Examples

### Example 1: Refactoring

```
"Refactor the user authentication module following the Model Upgrade Framework."

The AI will:
1. List all files affected
2. Propose a step-by-step plan
3. Execute one file at a time
4. Run tests after each file
5. Update PROJECT_CONTEXT.md
```

### Example 2: Bug Fix

```
"Fix the database connection pool leak using the Model Upgrade Framework."

The AI will:
1. Generate a reproduction test
2. Trace the root cause
3. Apply minimal fix
4. Run full test suite
5. Add regression test
```

### Example 3: New Feature

```
"Implement OAuth2 login following the Model Upgrade Framework."

The AI will:
1. Define data models
2. Write API contract tests
3. Implement service layer
4. Create API endpoints
5. Add validation and error handling
6. Run full verification
```

## Comparison: Without vs With Framework

| Scenario | Without Framework | With Framework |
|----------|------------------|----------------|
| Multi-file refactor | Model loses track, misses dependencies | Impact analysis first, step-by-step execution |
| Bug fix | Model guesses, makes it worse | Reproduce → Trace → Fix → Verify |
| New feature | Model generates incomplete code | Contract-first, test-driven, verified |
| Long session | Model forgets earlier decisions | PROJECT_CONTEXT.md always refreshed |
| Complex task | Model tries to do everything at once | Task decomposition, checkpoint verification |

## Supported Models

This framework works with **any** AI coding assistant:
- Domestic: Qwen, GLM, DeepSeek, Yi, Baichuan, Doubao
- International: GPT-4, Claude, Gemini
- Open-source: Llama, Mistral, InternLM

The framework compensates for weaknesses in weaker models and maximizes strengths in stronger models.

## License

Apache License 2.0 — see [LICENSE.txt](LICENSE.txt) for details.
