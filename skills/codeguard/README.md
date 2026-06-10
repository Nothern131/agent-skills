# CodeGuard — Agent Skill for Code Quality Scanning

[![Skill](https://img.shields.io/badge/Agent_Skill-codeguard-blue)](https://github.com/anthropics/skills)

CodeGuard is an **Agent Skill** that scans source code for 80 common defects across 8 dimensions, outputs a 0–100 quality score, and provides prioritized fix suggestions.

## Features

- **8 Scan Dimensions**: Error Handling, Security, Database Protection, Diagnostics, Resilience, Memory Safety, Concurrency, Resource Management
- **80 Detection Rules**: Each with severity level, detection patterns, and fix examples
- **Multi-Language**: Python, JavaScript/TypeScript, C/C++, GDScript, Go
- **Scoring System**: 0–100 overall and per-dimension scores
- **Prioritized Fixes**: Critical → High → Medium → Low
- **No External Dependencies**: Pure inline scan mode — works by reading code and applying rules

## Quick Start

### Install in Claude Code

```bash
# Add this repo as a skill source
# Then install the codeguard skill
```

### Manual Installation

Copy the `codeguard/` folder into your skills directory. The skill will activate when you mention code review, code quality, security scan, or similar phrases.

## Scan Dimensions

| Dimension | Rules | Focus |
|-----------|-------|-------|
| Error Handling | 10 | Missing try-catch, empty catches, unhandled promises, C error codes |
| Security | 10 | Hardcoded secrets, injection risks, auth gaps, command injection |
| Database Protection | 10 | Missing transactions, connection leaks, N+1 queries |
| Diagnostics | 10 | Missing logging, no error codes, no trace IDs |
| Resilience | 10 | No retries, no timeouts, no circuit breakers |
| Memory Safety | 10 | Buffer overflow, use-after-free, null deref, RAII (C/C++) |
| Concurrency | 10 | Race conditions, deadlocks, stale references, GDScript deferred calls |
| Resource Management | 10 | File/connection leaks, unbounded cache, GDScript node lifecycle |

## Usage Examples

```
"Review the code quality of src/auth/login.py"
"Check my API routes for security issues"
"Is this code production-ready?"
"Scan the payment module for potential issues"
"Check this C code for memory bugs"
"Review my Godot plugin code"
```

## Project Structure

```
codeguard/
├── SKILL.md                        # Main skill definition (metadata + instructions)
├── LICENSE.txt                     # Apache 2.0 license
├── README.md                       # This file
└── references/                    # Detailed rule references (loaded on demand)
    ├── error_handling.md           # ERR001–ERR010
    ├── permissions.md             # PERM001–PERM010
    ├── database.md                # DB001–DB010
    ├── diagnosis.md               # DIAG001–DIAG010
    ├── resilience.md              # RES001–RES010
    ├── memory_safety.md           # MEM001–MEM010
    ├── concurrency.md             # CON001–CON010
    └── resource_management.md     # RES_M001–RES_M010
```

## Scoring Algorithm

```
score = 100 - (total_deduction / max_possible_deduction) × 100
```

| Severity | Deduction |
|----------|-----------|
| Critical | 40 |
| High | 25 |
| Medium | 15 |
| Low | 5 |

## License

Apache License 2.0 — see [LICENSE.txt](LICENSE.txt) for details.
