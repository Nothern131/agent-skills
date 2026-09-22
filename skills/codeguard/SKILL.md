---
name: codeguard
description: "Automated code quality scanner that detects 80 common defects across 8 dimensions (error handling, security, database protection, diagnostics, resilience, memory safety, concurrency, resource management). Covers Python, JavaScript/TypeScript, C/C++, GDScript, and Go. Use this skill whenever the user asks for code review, code quality check, code audit, security scan, or wants to ensure their code is production-ready. Also use when the user mentions error handling, security vulnerabilities, database safety, observability, resilience, memory safety, concurrency, or resource leaks — even if they don't explicitly ask for a 'code review.'"
license: Apache-2.0
---

# CodeGuard — Automated Code Quality Scanner

80 rules across 8 dimensions, 0–100 score, prioritized fix suggestions.

**Languages:** Python, JS/TS, C/C++, GDScript, Go

## When to Trigger

- Code review / quality check / audit / security scan
- "code smell", "bad code", "production-ready", "robustness"
- After generating significant code → proactively offer scan
- User points to specific file/module for health check

## 8 Dimensions

| Dimension | ID | Ref |
|-----------|----|-----|
| Error Handling | ERR | [references/error_handling.md](references/error_handling.md) |
| Security | PERM | [references/permissions.md](references/permissions.md) |
| Database Protection | DB | [references/database.md](references/database.md) |
| Diagnostics | DIAG | [references/diagnosis.md](references/diagnosis.md) |
| Resilience | RES | [references/resilience.md](references/resilience.md) |
| Memory Safety | MEM | [references/memory_safety.md](references/memory_safety.md) |
| Concurrency | CON | [references/concurrency.md](references/concurrency.md) |
| Resource Management | RES_M | [references/resource_management.md](references/resource_management.md) |

Load reference files on demand. Each contains rule table, detection patterns, fix examples.

### Dimension Selection

- **All projects**: ERR, PERM, DIAG, RES_M
- **Web/API**: + DB, RES
- **C/C++**: + MEM, CON
- **Game (GDScript)**: + MEM (node lifecycle), CON (deferred calls)
- Skip dimensions that don't apply (e.g. DB for no-database projects)

## Workflow

1. **Scope**: full / dimension / file / language scan
2. **Read**: source files (user-specified or recently modified)
3. **Scan**: load reference → match patterns → record violations (rule ID, severity, file, line, description)
4. **Report**: structured output (below)
5. **Fix**: before/after code per issue, prioritize Critical → High

## Scoring

```
score = 100 - (total_deduction / max_possible) × 100
```

| Severity | Deduction |
|----------|-----------|
| Critical | 40 |
| High | 25 |
| Medium | 15 |
| Low | 5 |

## Report Template

```markdown
# CodeGuard Scan Report

## Summary
- Score: X/100 | Files: N | Issues: M (Critical: a | High: b | Medium: c | Low: d)

## Dimension Scores
| Dimension | Score | Issues |
|-----------|-------|--------|
| ... | X/100 | N |

## Issues (by severity)
### CRITICAL
- **[RULE_ID]** file:line — description
  Fix: concrete suggestion

## Top Priority Fixes
1. [CRITICAL] RULE_ID — ...
```

## AI Security Blind Spots

AI code looks equally confident whether secure or not. Extra checks:

1. **Missing security annotation**: handles input/auth/payment but no `// SECURITY:` → flag as PERM010 variant
2. **Overconfident claims**: claims "secure" without evidence → flag as DIAG010 variant
3. **P0 code without review**: auth/payments/external API without `// SECURITY REVIEW:` → flag

### Scan Priority Tiers

| Tier | Code Type | Depth |
|------|-----------|-------|
| P0 | Auth, payments, input handling, external API | Full 8-dim + blind spot |
| P1 | DB ops, file ops, config | 8-dim |
| P2 | UI logic, utilities, tests | ERR + PERM + RES_M only |

**Density alert**: 3+ Critical/High in one scan → output warning, recommend stop generating until resolved.

## Guidelines

- Explain *why* a rule matters, not just *what* it checks
- Adapt severity by context (hardcoded secret in public repo = critical; local script = low)
- Don't scan code the user didn't ask about
- Group same-rule violations in report
- Match fix examples to project's primary language
- Inline scan is default — no MCP or external dependency required

## Examples

**Full scan**: "Review src/auth/login.py" → load all 8 refs, scan, report.

**Focused scan**: "Check my C code for memory bugs" → load MEM + RES_M refs only.
