# Project Rules — 铁律

<!--
  PROJECT RULES FILE
  ==================
  This file enforces engineering discipline for AI coding assistants.
  Place it in your project root as .trae/rules.md or CLAUDE.md
  The AI assistant MUST follow these rules for every task.
-->

## Section 1: Code Modification Rules

1. **Impact Analysis First**: Before any code modification, list:
   - Files that will be changed
   - Files that depend on them (upstream/downstream)
   - Potential side effects
   - What tests need to run

2. **One Concern Per Change**: Each code change must address exactly one concern. Split multi-concern changes into separate steps.

3. **No Silent Failures**: Never ignore errors. Every try-catch must either:
   - Log the error with context
   - Re-throw with additional information
   - Have an explicit comment explaining why it's safe to swallow

4. **Public Interface Documentation**: Every public method/function/API must have:
   - Complete docstring/docblock
   - Parameter descriptions
   - Return type description
   - Exception descriptions

## Section 2: Testing Rules

5. **Test-Driven Changes**: When adding new functionality:
   - Write the test first (or at least define the test contract)
   - Implement the code to pass the test
   - Run the full test suite

6. **Automatic Verification**: After every code change:
   - Run linter/type checker
   - Run relevant unit tests
   - Fix any failures before proceeding

7. **Regression Prevention**: When fixing a bug:
   - Add a regression test that would have caught the original bug
   - Run all tests to ensure no regressions

## Section 3: Security Rules

8. **No Hardcoded Secrets**: Never embed passwords, API keys, or tokens in code.
   - Use environment variables or secret managers
   - Run `codeguard` to check before committing

9. **Input Validation**: All user input must be validated:
   - Type checking
   - Range checking (numbers)
   - Length checking (strings)
   - Format checking (emails, URLs, etc.)

10. **Parameterized Queries**: Never construct SQL with string interpolation.
    - Use parameterized queries or ORM methods
    - Run `codeguard` PERM002 check

## Section 4: Process Rules

11. **Checkpoint Verification**: After each major step, verify:
    - [ ] All tests pass
    - [ ] No new lint errors
    - [ ] No unused imports/variables
    - [ ] Docstrings are complete
    - [ ] Edge cases are handled

12. **Three-Attempt Limit**: Maximum 3 self-correction attempts per step. If it fails 3 times, stop and explain why.

13. **Context Maintenance**: Update PROJECT_CONTEXT.md after completing significant changes.

## Section 5: Language-Specific Rules

### Python
- Use type hints for all public functions
- Follow PEP 8 style guide
- Use `pathlib` for file operations
- Use `logging` not `print` for production code

### JavaScript/TypeScript
- Use TypeScript for all new code
- Follow ESLint + Prettier configuration
- Use `async/await` not `.then()` chains
- Import sorting enforced by eslint-plugin-import

### GDScript (Godot)
- Use `@export` for inspector-visible variables
- Use `is_instance_valid()` before accessing nodes
- Use `queue_free()` for nodes in scene tree, `free()` for others
- Prefer `_process()` for frame-dependent, `_physics_process()` for physics

### C/C++
- Use RAII (smart pointers) in C++
- Check return values of malloc/fopen/socket calls
- Use `goto cleanup` pattern for resource management in C
- Run `codeguard` MEM rules before submitting

## Section 6: Communication Rules

14. **Show Your Work**: Before executing complex changes, explain your plan:
    - What you will change
    - Why you are changing it
    - What tests will verify correctness

15. **Summarize Changes**: After completing a task, provide:
    - Files changed
    - Key changes made
    - Tests added/modified
    - Any remaining TODOs or follow-ups
