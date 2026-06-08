---
name: clarifying-questions
description: "Helps AI precisely understand user requirements by asking targeted clarifying questions before starting work. Use this skill whenever a user's request is vague, ambiguous, or lacks critical details — such as missing tech stack, unclear scope, unspecified format, or undefined constraints. Also use proactively when the user says things like 'build me something', 'make it better', 'fix it', or any request where 2+ key details are missing."
license: Apache-2.0
---

# Clarifying Questions Skill

Before diving into implementation, this skill ensures the AI understands exactly what the user wants by identifying ambiguities and asking targeted questions.

## When to Trigger

- User's request is vague ("build me an app", "make it better", "fix it")
- Key details are missing (tech stack, scope, format, constraints)
- Multiple interpretations are possible
- User says "I need help with..." without specifics
- Before starting any significant implementation task

## Workflow

### Step 1: Detect Task Type

Identify the task category from the user's input:

| Task Type | Keywords | Key Questions |
|-----------|----------|---------------|
| Code | write, create, develop, implement, code, script | Tech stack? Features? Platform? Performance? Security? |
| Design | design, UI, layout, icon, poster, image | Style? Colors? Size? Elements? Reference? |
| Content | article, report, copy, document, write | Audience? Length? Tone? Key points? References? |
| Analysis | analyze, statistics, data, report, research | Data source? Metrics? Charts? Purpose? Output format? |
| Automation | automate, batch, script, process | Steps? Data source? Volume? Error handling? Schedule? |

### Step 2: Identify Ambiguities

Check for these common ambiguity types:

| Type | What's Missing | Example |
|------|---------------|---------|
| Object | Target is unclear | "Fix the bug" — which bug? |
| Scope | Boundaries undefined | "Improve performance" — of what? |
| Format | Output format unspecified | "Generate a report" — PDF? Markdown? |
| Style | Aesthetic direction missing | "Design a logo" — modern? retro? |
| Priority | What matters most | "Make it fast and beautiful" — which first? |
| Constraints | Limitations unknown | "Build an API" — auth? rate limits? |
| Technology | Stack not specified | "Create a website" — React? Next? plain HTML? |
| Audience | Who it's for | "Write documentation" — developers? end users? |

### Step 3: Generate Questions

Rules for good clarifying questions:
1. **Limit to 3-5 questions** — don't overwhelm the user
2. **Prioritize by impact** — ask about the most critical unknowns first
3. **Use open-ended questions** — avoid Yes/No questions
4. **Offer options when helpful** — "What tech stack? (e.g., React, Vue, plain HTML)"
5. **Don't ask about things you can reasonably infer** — if the user says "Python script", don't ask "what language?"

### Step 4: Format the Output

```
I'd like to understand your requirements better before starting. A few questions:

1. **[Most critical question]** — [Why it matters]
2. **[Second question]** — [Context/options]
3. **[Third question]** — [Context/options]

Please answer these and I'll provide a precise solution.
```

### Step 5: Create Requirement Document (Optional)

After the user answers, summarize into a structured requirement doc:

```markdown
# Requirements

## Original Request
[User's original words]

## Clarified Details
- **Goal**: [What they want]
- **Scope**: [What's included/excluded]
- **Tech Stack**: [Technologies]
- **Constraints**: [Limitations]
- **Priority**: [What matters most]

## Acceptance Criteria
1. [Criterion 1]
2. [Criterion 2]
3. [Criterion 3]
```

## Examples

**Example 1: Vague code request**
> User: "帮我写一个程序"

Response:
> 开始之前我想确认几点：
> 1. **用什么编程语言？** — Python/JavaScript/Go/其他？
> 2. **需要实现什么功能？** — 具体描述一下核心功能
> 3. **运行在什么平台？** — Web/桌面/命令行/移动端？

**Example 2: Design request**
> User: "设计一个Logo"

Response:
> 为了设计出合适的Logo，我需要了解：
> 1. **什么风格？** — 简约/现代/复古/科技感？
> 2. **品牌名称和行业？** — 这决定了设计方向
> 3. **有颜色偏好吗？** — 或者有品牌色？

**Example 3: Clear enough request (skip questioning)**
> User: "用Python写一个FastAPI接口，接收JSON数据存入PostgreSQL，需要参数验证"

This request has tech stack, function, and requirements — no need to ask clarifying questions. Proceed directly.

## Guidelines

- If the request is already clear and specific, **don't ask questions** — just do it
- Never ask more than 5 questions at once
- If you can infer the answer from context, don't ask
- Adapt question language to match the user's language
- After receiving answers, confirm understanding before starting work
- For multi-step tasks, ask about the most impactful unknowns first
