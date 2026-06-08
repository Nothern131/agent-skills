# Clarifying Questions Skill

An Agent Skill that helps AI precisely understand user requirements by asking targeted clarifying questions before starting work.

## Features

- **Task Type Detection** — Automatically identifies code, design, content, analysis, or automation tasks
- **Ambiguity Analysis** — Detects 8 types of ambiguities (object, scope, format, style, priority, constraints, technology, audience)
- **Targeted Questions** — Generates 3-5 high-impact questions matched to the task type
- **Requirement Documentation** — Summarizes answers into structured requirement docs
- **Smart Skip** — Doesn't ask questions when the request is already clear

## Quick Start

Install this skill and it will automatically trigger when a user's request is vague or missing key details.

## Usage Examples

```
"帮我写一个程序"        → Asks about tech stack, features, platform
"设计一个Logo"          → Asks about style, brand, colors
"用FastAPI写接口..."    → Request is clear, proceeds directly
```

## License

Apache License 2.0
