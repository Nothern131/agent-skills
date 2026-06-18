---
name: codeguard
description: "Automated code quality scanner that detects 80 common defects across 8 dimensions (error handling, security, database protection, diagnostics, resilience, memory safety, concurrency, resource management). Covers Python, JavaScript/TypeScript, C/C++, GDScript, and Go. Use this skill whenever the user asks for code review, code quality check, code audit, security scan, or wants to ensure their code is production-ready. Also use when the user mentions error handling, security vulnerabilities, database safety, observability, resilience, memory safety, concurrency, or resource leaks — even if they don't explicitly ask for a 'code review.'"
license: Apache-2.0
---

# CodeGuard — Automated Code Quality Scanner

CodeGuard scans source code for 80 common defects across 8 dimensions, outputs a 0–100 score, and provides prioritized fix suggestions. It ensures AI-generated code meets production-grade quality.

**Supported Languages:** Python, JavaScript/TypeScript, C/C++, GDScript, Go

## When to Trigger

- User asks for code review, code quality check, code audit, or security scan
- User mentions "code smell", "bad code", "code robustness", "production-ready"
- User wants to check error handling, security, database safety, observability, resilience, memory safety, concurrency, or resource management
- After generating significant code, proactively offer a quality scan
- User points to a specific file or module and asks for a health check

## Scan Dimensions

| # | Dimension | ID | Rules | Reference |
|---|-----------|----|-------|-----------|
| 1 | Error Handling | `error_handling` | 10 | [references/error_handling.md](references/error_handling.md) |
| 2 | Security | `permissions` | 10 | [references/permissions.md](references/permissions.md) |
| 3 | Database Protection | `database` | 10 | [references/database.md](references/database.md) |
| 4 | Diagnostics | `diagnosis` | 10 | [references/diagnosis.md](references/diagnosis.md) |
| 5 | Resilience | `resilience` | 10 | [references/resilience.md](references/resilience.md) |
| 6 | Memory Safety | `memory_safety` | 10 | [references/memory_safety.md](references/memory_safety.md) |
| 7 | Concurrency | `concurrency` | 10 | [references/concurrency.md](references/concurrency.md) |
| 8 | Resource Management | `resource_management` | 10 | [references/resource_management.md](references/resource_management.md) |

**Load the relevant reference file** when scanning a specific dimension. For full scans, load all eight. Each reference contains the rule table, detection patterns, and fix examples for all supported languages.

### Dimension Selection Guide

- **All projects**: Error Handling, Security, Diagnostics, Resource Management
- **Web/API projects**: + Database Protection, Resilience
- **C/C++ projects**: + Memory Safety, Concurrency
- **Game projects (GDScript)**: + Memory Safety (node lifecycle), Concurrency (deferred calls)
- **Concurrent systems**: + Concurrency, Resilience

## Workflow

### Step 1: Determine Scan Scope

Ask the user (or infer from context):
- **Full scan** — all 8 dimensions
- **Dimension scan** — only specified dimensions (e.g., "check security")
- **File scan** — scan one or more specific files
- **Language scan** — scan only dimensions relevant to the project's language

### Step 2: Read the Code

Read the source files to be scanned. For large codebases, focus on the files the user specified or the most recently modified files.

### Step 3: Execute Scan

For each dimension, load the corresponding reference file and check each rule against the code. Rules are independent — a violation in one rule does not affect others.

Detection approach:
1. Read the code and match against the detection patterns described in each reference
2. Use pattern matching (regex, code structure analysis) to identify violations
3. Record each violation with: rule ID, severity, file, line range, and description

### Step 4: Generate Report

Output a structured report:

```markdown
# CodeGuard Scan Report

## Summary
- **Overall Score**: X/100
- **Files Scanned**: N
- **Languages Detected**: Python, C++
- **Total Issues**: M (Critical: a | High: b | Medium: c | Low: d)

## Dimension Scores
| Dimension | Score | Issues |
|-----------|-------|--------|
| Error Handling | X/100 | N |
| Security | X/100 | N |
| Database Protection | X/100 | N |
| Diagnostics | X/100 | N |
| Resilience | X/100 | N |
| Memory Safety | X/100 | N |
| Concurrency | X/100 | N |
| Resource Management | X/100 | N |

## Issues (sorted by severity)

### CRITICAL
- **[MEM001]** src/buffer.c:42 — Buffer overflow: strcpy without bounds check
  Fix: Use strncpy with size validation

### HIGH
- **[ERR002]** src/handler.js:15 — Empty catch block swallows error
  Fix: Log or rethrow the error

...

## Top Priority Fixes
1. [CRITICAL] MEM001 — ...
2. [CRITICAL] PERM001 — ...
3. [HIGH] ERR002 — ...
```

### Step 5: Provide Fix Suggestions

For each issue found, provide:
- The rule ID and what it means
- A concrete code fix (before/after) in the project's language
- Prioritize critical and high severity issues

## Scoring Algorithm

```
overall_score = 100 - (total_deduction / max_possible_deduction) × 100
```

Deduction per violation:
| Severity | Deduction |
|----------|-----------|
| Critical | 40 |
| High | 25 |
| Medium | 15 |
| Low | 5 |

Dimension score uses the same formula, scoped to that dimension's rules only.

## Inline Scan Mode

When no external tools are available, scan code manually by reading source files and applying the rules from the reference documents. This is the default mode — no MCP server or external dependency is required.

## Examples

**Example 1: Full scan**
> "Review the code quality of src/auth/login.py"

Load all 8 reference files, read the file, execute scan, output report.

**Example 2: Security-focused scan**
> "Check my API routes for security issues"

Load only `references/permissions.md`, read the route files, scan for PERM rules.

**Example 3: C/C++ memory safety scan**
> "Check this C code for memory bugs"

Load `references/memory_safety.md` and `references/resource_management.md`, scan for MEM and RES_M rules.

**Example 4: GDScript game project scan**
> "Review my Godot plugin code"

Load relevant references, focus on ERR, RES_M (node lifecycle), and CON (deferred calls) rules.

**Example 5: Quick health check**
> "Is this code production-ready?"

Load all references, scan, output a concise summary with top 3 issues to fix.

## Guidelines

- Always explain *why* a rule matters, not just *what* it checks
- Prioritize actionable fixes over theoretical concerns
- For code that doesn't use databases, skip the database dimension
- For non-C/C++ code, skip the memory safety dimension (unless GDScript node lifecycle applies)
- Adapt severity based on context — a hardcoded secret in a public repo is critical; in a local script it's low
- Don't flag issues in code the user didn't ask to scan
- When multiple violations of the same rule exist, group them in the report
- Match fix examples to the project's primary language

## AI Security Blind Spot Detection

> AI-generated code has a unique vulnerability: the code looks just as confident whether it's secure or not.
> CodeGuard adds security-aware scanning on top of the 8 dimensions.

### Security Confidence Annotation

When scanning AI-generated code, CodeGuard should also check for:

1. **Missing security annotations**: If the code handles user input, auth, payments, or sensitive data but has no `// SECURITY:` comment, flag it as a `PERM010` variant (missing security awareness).

2. **Overconfident security claims**: If the code claims something is "secure" or "safe" without evidence (no standard library usage, no test, no audit), flag it as `DIAG010` variant (security claim without audit trail).

3. **Security review gaps**: For P0 code (auth, payments, user input, external APIs), if no `// SECURITY REVIEW:` annotation exists, flag it as `DIAG010` variant.

### Scan Priority Tiers

When scanning AI-generated code, apply tiered depth:

| Tier | Code Type | Scan Depth |
|------|-----------|------------|
| P0 | Auth/authorization, payments, user input handling, external API calls | Full 8-dimension scan + security blind spot check |
| P1 | Database operations, file operations, config management | 8-dimension scan |
| P2 | Pure UI logic, internal utilities, test code | Error handling + security + resource management only |

### Vulnerability Density Alert

If a single scan finds 3+ Critical/High issues:
- Output a **VULNERABILITY DENSITY WARNING** at the top of the report
- Recommend stopping new code generation until issues are resolved
- Suggest the generation pattern may need adjustment
