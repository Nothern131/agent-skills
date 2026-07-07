# Test Automation — Agent Skill for Automated Test Execution

[![Skill](https://img.shields.io/badge/Agent_Skill-test_automation-blue)](https://github.com/anthropics/skills)

Test Automation compensates for domestic AI models' lack of native test execution. It auto-detects the test framework, runs tests, parses results, and feeds back to the Closed Loop for autonomous quality verification.

## Features

- **Auto-detect framework**: pytest, unittest, Jest, Mocha, Go test, GUT, CTest, Catch2
- **Run tests**: Executes the appropriate test command for the detected framework
- **Parse results**: Extracts pass/fail/error counts and failure details
- **Closed Loop integration**: Feeds results back to determine if re-fix is needed
- **Graceful degradation**: Handles missing frameworks, timeouts, and no-test scenarios

## Supported Frameworks

| Language | Framework |
|----------|-----------|
| Python | pytest, unittest |
| JS/TS | Jest, Mocha |
| Go | go test |
| GDScript | GUT |
| C/C++ | CTest, Catch2 |

## Usage

This skill is auto-triggered by the model-upgrade rules (v3) when:
- After code generation (Closed Loop verification)
- After Closed Loop fix (re-verification)
- User says "test/验证/跑测试/检查"
- Before delivery (evidence of completion)

## Output

A structured test result summary:
- Framework detected
- Pass/fail/error counts with pass rate
- Failure details with error summaries
- Closed Loop feedback: whether re-fix is needed