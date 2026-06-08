# CodeGuard — Agent Skill for Code Quality Scanning

[![Skill](https://img.shields.io/badge/Agent_Skill-codeguard-blue)](https://github.com/anthropics/skills)

CodeGuard is an **Agent Skill** that scans source code for 50 common defects across 5 dimensions, outputs a 0–100 quality score, and provides prioritized fix suggestions.

## Features

- **5 Scan Dimensions**: Error Handling, Security, Database Protection, Diagnostics, Resilience
- **50 Detection Rules**: Each with severity level, detection patterns, and fix examples
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
| Error Handling | 10 | Missing try-catch, empty catches, unhandled promises |
| Security | 10 | Hardcoded secrets, injection risks, auth gaps |
| Database Protection | 10 | Missing transactions, connection leaks, N+1 queries |
| Diagnostics | 10 | Missing logging, no error codes, no trace IDs |
| Resilience | 10 | No retries, no timeouts, no circuit breakers |

## Usage Examples

```
"Review the code quality of src/auth/login.py"
"Check my API routes for security issues"
"Is this code production-ready?"
"Scan the payment module for potential issues"
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
    └── resilience.md              # RES001–RES010
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
