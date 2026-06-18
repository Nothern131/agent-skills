---
name: model-upgrade
description: "A comprehensive engineering framework that enables any AI coding assistant (domestic or international) to achieve GPT/Claude parity through five layers of defense: structured rules, context injection, task decomposition, test-feedback loops, and last-mile delivery guarantee. Use this skill as the FIRST thing activated in any coding session. It enforces engineering discipline, provides prompt templates, and manages the full development lifecycle — making even weaker models produce production-grade output through process, not just raw model capability."
license: Apache-2.0
---

# Model Upgrade Framework — 模型升级计划

> **Premise:** Domestic AI models (Qwen, GLM, Yi, etc.) are very close to GPT/Claude in single-turn code generation. The gap widens only in: complex multi-step reasoning, instruction following, self-correction loops, and long-context maintenance. This framework bridges that gap through engineering discipline.

## Core Philosophy

```
Quality = Model_Raw_Capability × Engineering_Multiplier

Engineering_Multiplier = Rules_Following × Iterative_Verification × Task_Decomposition × Context_Awareness
```

If the model is 80% of GPT's raw capability but you multiply it by 1.5x through process, you get 120% of GPT's effective output.

## When to Trigger

This is your **default mode** — always activate at the start of any coding session:
- Any significant coding task (feature, refactor, debug)
- Multi-file changes or cross-module work
- Tasks that require testing, deployment, or architecture decisions
- When the model shows signs of losing track or skipping steps

## Five-Layer Defense System

### Layer 1: Project Constitution (Rules)
### Layer 2: Context Injection (PROJECT_CONTEXT.md)
### Layer 3: Task Decomposition (Break big → small)
### Layer 4: Test-Feedback Loop (Verify and self-correct)
### Layer 5: Last-Mile Defense (90%→100% delivery guarantee)

---

## Layer 1: Project Constitution — 铁律

Before any work begins, enforce these non-negotiable rules:

### Rule 1: Impact Analysis First
**Never** start writing code without first listing the impact scope.

```
Before any code modification, output:
1. Files that will be changed
2. Files that depend on them (upstream/downstream)
3. Potential side effects
4. What tests need to run
```

### Rule 2: One Concern Per Change
Each code change must address exactly one concern. If multiple things need changing, split into separate steps.

### Rule 3: Write Tests or Update Tests with Every Change
No code modification without:
- A new test for new functionality, OR
- An updated test for existing functionality

### Rule 4: Automatic Lint/Check After Each Change
After every file modification, automatically check:
- Syntax (linter, type checker)
- If syntax errors found, self-correct immediately (max 3 attempts)
- Only proceed to next step when current step passes checks

### Rule 5: Document Public Interfaces
Every public method, function, or API must have:
- Complete docstring/docblock
- Parameter descriptions
- Return type description
- Exception descriptions

---

## Layer 2: Context Injection — 全项目感知

### Step 1: Generate PROJECT_CONTEXT.md at Session Start

When a task requires understanding the codebase, generate a project context file:

```markdown
# PROJECT_CONTEXT.md

## Directory Structure (relevant parts)
```
src/
├── api/
│   ├── user.py          # User API endpoints
│   ├── auth.py          # Authentication middleware
├── models/
│   ├── user.py          # User model (SQLAlchemy)
├── services/
│   ├── user_service.py  # Business logic for users
└── tests/
    ├── test_user.py     # User-related tests
```

## Key Entity Relationships
- User (models/user.py) ←→ Order (models/order.py): one-to-many
- Authentication (api/auth.py) protects routes in api/user.py

## Critical Configuration
- Database: PostgreSQL, connection via SQLAlchemy ORM
- Auth: JWT tokens, 15min expiry

## Recent Changes
- [2024-01-15] Added payment integration to user service

## Dependencies & Contracts
- user_service.py expects: auth token in header
- user_service.py returns: User DTO with id, name, email, created_at
```

### Step 2: Inject Context Into Every Task
Before processing any request, load the relevant parts of PROJECT_CONTEXT.md and include them in the system prompt.

### Step 3: Update Context After Changes
After completing a task, update PROJECT_CONTEXT.md to reflect the new state.

---

## Layer 3: Task Decomposition — 任务拆解

### Pattern A: Big Task → Small Steps

**Bad (one-shot generation):**
> "Write a user management module with CRUD operations, authentication, and tests"

**Good (decomposed):**
```
Step 1: Define User model and database schema
Step 2: Generate unit tests for User model
Step 3: Implement CRUD service layer (no tests yet)
Step 4: Run tests, fix any failures
Step 5: Create API endpoints
Step 6: Add input validation
Step 7: Add authentication to endpoints
Step 8: Run all tests, fix any failures
Step 9: Update documentation
```

### Pattern B: Fill-in-the-Blank for Complex Tasks

For each step, provide a template the model must fill:

```
Task: Implement user registration

Context:
- API endpoint: POST /api/users/register
- Request body: {"username": str, "email": str, "password": str}
- Response: 201 with user DTO
- Must validate: email format, password strength, duplicate username
- Must use: User model from models/user.py, validate from utils/validator.py

Template:
```python
from services.user_service import UserService
from models.user import User
from utils.validator import validate_email, validate_password_strength
from exceptions import ApiException

def register_user(request):
    # 1. Validate input
    validate_email(request.json["email"])
    if not validate_password_strength(request.json["password"]):
        raise ApiException(400, "Password too weak")
    
    # 2. Check duplicate username
    if User.exists(username=request.json["username"]):
        raise ApiException(409, "Username already exists")
    
    # 3. Create user
    user = User.create(
        username=request.json["username"],
        email=request.json["email"],
        password_hash=hash_password(request.json["password"])
    )
    
    # 4. Return DTO
    return UserDTO.from_model(user), 201
```
```

### Pattern C: Checkpoint-Based Execution

For each decomposed step:
1. Model completes the step
2. **Self-check:** Does it pass syntax check? Do related tests pass?
3. **Checkpoint pass:** Show summary of what was done and ask for confirmation
4. If checkpoint fails → self-correct before proceeding

---

## Layer 4: Test-Feedback Loop — 测试修正循环

### Loop Architecture

```
Generate/Modify Code
    ↓
Run Linter/Type Checker
    ↓ (fail) ←────────────┐
Self-Correct (attempt 1) ──┘
    ↓ (fail)
Self-Correct (attempt 2)
    ↓ (fail)
Self-Correct (attempt 3)
    ↓ (still fail)
Report error with details, stop automatic retry
    ↓ (pass)
Run Unit Tests
    ↓ (fail) ←────────────┐
Read error message ────────┘
Feed error to model → self-correct
(Max 3 iterations)
    ↓ (pass)
Mark step complete
    ↓
Next step or final verification
```

### Implementation Rules

1. **Syntax Check First:** Always run linter before running tests
2. **Error Message Is Your Friend:** Never silently skip test failures. Read the error message carefully.
3. **Root Cause, Not Band-Aid:** When fixing errors, fix the root cause, not the symptom
4. **Three-Attempt Limit:** Maximum 3 self-correction attempts per step. If it fails 3 times, stop and explain why.
5. **Regression Prevention:** When fixing a bug, add a regression test

### Error Analysis Protocol

When a test fails:
```
1. Read the error message carefully
2. Identify: which file, which line, what type of error
3. Trace: where the error originates, what code path leads to it
4. Fix: apply the minimal change needed
5. Verify: re-run the specific failing test, not just all tests
6. Document: add a comment explaining why this edge case needed handling
```

---

## Prompt Templates — 提示词模板

Use these templates to get the best output from any model:

### Template 1: Complex Refactoring
```
I need to refactor [module_name]. Please follow these steps:

1. UNDERSTAND: Read [files] and summarize the current architecture.
2. PLAN: Propose a refactoring strategy. List affected files and the approach for each.
3. EXECUTE: Apply changes one file at a time.
4. VERIFY: Run tests after each file change.
5. DOCUMENT: Update any affected documentation.

Constraints:
- Do not change external API contracts
- Maintain backward compatibility
- Preserve all existing test coverage
```

### Template 2: Bug Fix
```
I'm encountering this bug: [error message / symptoms]

Steps to fix:
1. REPRODUCE: Write or run a test that demonstrates the bug
2. TRACE: Read the relevant code and explain the root cause
3. FIX: Apply the minimal fix
4. VERIFY: Run the reproduction test and all related tests
5. REGRESSION: Add a regression test to prevent future recurrence

Here is the error message:
[insert error]
```

### Template 3: Feature Implementation
```
Implement [feature_name]. Follow this checklist:

1. Define data models / types
2. Write the API contract (request/response)
3. Generate tests for the contract
4. Implement service layer
5. Run tests and fix failures
6. Create API endpoint
7. Add input validation
8. Add error handling
9. Run full test suite
10. Update API documentation

Constraints:
- Use [framework/stack]
- Follow project coding standards in [rules_file]
- All public methods must have docstrings
```

### Template 4: Code Review
```
Review the following code for:
1. Security: SQL injection, XSS, auth bypass, secret exposure
2. Error handling: Missing try-catch, unhandled exceptions
3. Performance: N+1 queries, unnecessary loops, memory leaks
4. Readability: Naming, complexity, dead code
5. Best practices: DRY, SOLID, framework conventions

Provide a scored report with specific line references.
```

---

## Self-Enhancement — 自我增强策略

### Strategy 1: Few-Shot Examples
When the model produces output that's close but not quite right, feed it a corrected example and ask it to follow the pattern.

```
Here is how this should look:
[insert correct example]

Please apply this same pattern to the current task.
```

### Strategy 2: Step-by-Step Verification
After each step, explicitly ask the model to self-verify:

```
Before proceeding to the next step, verify:
- [ ] All tests pass
- [ ] No new lint errors
- [ ] No unused imports/variables
- [ ] Docstrings are complete
- [ ] Edge cases are handled

If any item fails, fix it before marking complete.
```

### Strategy 3: Context Window Optimization
Domestic models have large context windows (128K+). Use them wisely:

```
Priority order for context injection:
1. Current task requirements (always first)
2. Related source files (files being modified and their imports)
3. Test files (to understand expected behavior)
4. Configuration files (if relevant to the change)
5. Related documentation or spec files
```

### Strategy 4: Tool Augmentation via MCP
When the model has access to tools, always prefer tool execution over model reasoning:

```
Tool priority:
1. Database schema query → Never guess table structures
2. API documentation (OpenAPI/Swagger) → Never guess API contracts
3. Git diff/status → Never guess what changed
4. Linter/type checker → Never guess syntax errors
5. Test runner → Never guess test results
```

---

## Language-Specific Adjustments

### For Chinese-Output Scenarios
Domestic models often produce better Chinese explanations. Use this advantage:
- Let the model explain architecture decisions in Chinese
- Keep code comments in English (universal standard)
- Use Chinese for user-facing documentation

### For English-Output Scenarios (Code)
When generating code:
- Specify output language explicitly: "Generate Python code with English comments"
- Use English for variable names, function names, docstrings
- This ensures consistency even if the model mixes languages

### For GDScript / Game Development
When working with Godot:
- Always check `is_instance_valid(node)` before accessing nodes
- Use `queue_free()` or `free()` correctly (not interchangeably)
- Be careful with `_process` vs `_physics_process`
- Memory management: nodes added to tree are owned by scene, others must be freed

### For C/C++
- Always check return values of malloc/fopen/socket calls
- Use RAII (smart pointers) in C++ instead of raw new/delete
- Handle goto-cleanup pattern for resource management in C
- Memory safety rules from codeguard apply

---

## Layer 5: Last-Mile Defense — 最后一公里防御

> Agent能干90%，最后10%永远搞不定。根因：隐性需求未显性化、缺乏闭环验证、纠错指数级增长。
> This layer solves "the code is correct but not what the user wanted."

### Rule 1: Hidden Requirement Mining (Ask More Than Once)

Hidden requirements don't surface in the first question. They emerge during execution.

**Trigger points:**
- After completing each sub-step, check for newly exposed hidden requirements
- When the user says "almost but not quite", ask "what's missing?"
- When there are multiple implementation choices, list options for the user to choose

**Hidden requirement checklist (self-ask after each step):**
- [ ] Is what the user said the same as what I understood?
- [ ] Are there assumptions I made that the user didn't state?
- [ ] Does the user really want this implementation, or is it just what I think is reasonable?
- [ ] Are there edge cases the user might care about that I didn't ask?
- [ ] Is current progress aligned with user expectations?

**Hidden requirement template:**
```
I completed [step], current state is [description].
Before continuing, I want to confirm:
1. [Most likely hidden requirement] — Do you expect [A] or [B]?
2. [Second likely spot] — Does this behavior match your expectation?
3. [Potential assumption] — I assumed [X], is this correct?
```

### Rule 2: User Intent Verification (Close the Loop Beyond Code)

Code passing tests ≠ satisfying user intent. Verify "what I built is what you want" at every key node.

**Dual-loop verification:**
```
Inner loop (code correctness):
  Syntax check → Tests pass → Lint pass → Codeguard scan pass

Outer loop (user intent match):
  Complete sub-step → Show result/effect → User confirms "this is what I want" → Continue
                                         ↓ User says "not right"
                                       Ask what's different → Fix → Re-show
```

**Intent verification triggers:**
- After completing the first runnable version of core functionality (not after everything is done)
- After making key architecture/design decisions
- When the user might have different expectations for the result
- Before switching from one sub-step to the next

**Prohibited behaviors:**
- Never complete 3+ sub-steps without intent verification
- Never assume "code runs = user is satisfied"
- Never continue when the user says "almost" without asking what's missing

### Rule 3: Error Cascade Prevention (Fixes Don't Create New Errors)

The three-attempt limit stops bleeding, but doesn't prevent cascading: fix A → breaks B → fix B → breaks C.

**Cascade prevention rules:**
1. **Assess impact before fixing**: List code paths affected by the fix. If 3+ modules affected, confirm strategy with user first.
2. **Immediate regression after fix**: Verify not just the fixed bug, but that previously passing features still work.
3. **Fix complexity budget**: If a bug fix fails 2+ attempts, stop and report to user: current problem, what was tried, why it can't be fixed, suggested alternatives (workaround, degradation, redesign).
4. **No chain fixes**: Never fix 3+ interrelated bugs in one response. Fix one at a time, verify after each.

**Cascade detection signals:**
- Fix A breaks B → Fix B breaks C → This is a cascade, STOP immediately
- Same file modified 3+ times consecutively → Direction may be wrong, reassess
- Tests go from passing to failing → Roll back to last passing state, don't fix on top of failures

### Rule 4: Delivery Pace Control (Progressive Delivery)

Not all steps are equally important. Deliver core first, verify direction, then add details.

**Delivery pace principles:**
1. **MVP first**: Deliver minimum viable version, let user confirm direction, then add details
   - V1: Core functionality works + basic error handling
   - V2: Edge cases + performance optimization
   - V3: Polish details + complete documentation

2. **User confirmation checkpoints** (mandatory, not optional):
   - After first version of core functionality → MUST pause, wait for user confirmation
   - After architecture decisions → MUST pause, wait for user confirmation
   - After 5+ sub-steps without confirmation → MUST pause, show progress

3. **No "finish everything at once"**:
   - Never generate complete implementations of 3+ files at once
   - Never advance to the next phase without user confirmation
   - Never assume the user will be satisfied with the final result without intermediate checks

**Delivery pace template:**
```
Phase [N] complete. Current deliverables:
- [Completed feature 1]
- [Completed feature 2]
- [Not yet completed feature 3] (planned for next phase)

Please confirm:
1. Is the current direction correct?
2. Do priorities need adjustment?
3. Can I proceed to the next phase?
```

### Rule 5: Expectation Alignment (Prevent "Done But Wrong")

The most expensive error is not a code bug — it's completing an entire feature that the user didn't want.

**Expectation alignment check (must complete before implementation):**
1. **Restate requirements**: Rephrase the user's request in your own words, confirm understanding matches
2. **Confirm boundaries**: Explicitly state what's "in scope" and "out of scope"
3. **Confirm priorities**: If time is limited, what matters most
4. **Confirm completion criteria**: How do we know when it's "done"

**Expectation alignment template:**
```
I understand you need:
- Goal: [rephrase in your own words]
- Scope: includes [A, B, C], excludes [X, Y]
- Priority: [A] > [B] > [C]
- Completion criteria: [specific verifiable criteria]

Is this understanding correct? Anything to add or correct?
```

**Expectation alignment triggers:**
- When receiving a new task (first alignment)
- When task understanding has drifted (re-align)
- When user says "not right" / "that's not what I meant" (immediate alignment)
- After completing core functionality, before entering detail phase (second alignment)

---

## Session Workflow — 标准会话流程

Every coding session should follow this flow:

```
1. [Session Start]
   → Load PROJECT_CONTEXT.md (or generate if missing)
   → Load .trae/rules.md or CLAUDE.md if present
   → Confirm understanding of the task

2. [Task Analysis]
   → Break task into steps
   → Identify affected files
   → Identify tests to run
   → List constraints

3. [Execution Loop] (per step)
   → Read relevant code
   → Generate/modify code
   → Run syntax check
   → Fix syntax errors (max 3)
   → Run relevant tests
   → Fix test failures (max 3)
   → Self-verify against checklist
   → Mark step complete

4. [Final Verification]
   → Run full test suite
   → Run linter on all changed files
   → Generate change summary
   → Update PROJECT_CONTEXT.md

5. [Session End]
   → Commit message (if git available)
   → Document any remaining issues
   → Note: What still needs work
```

---

## Guidelines

- **Process over raw intelligence:** Good process makes average models perform like great models
- **Verify at every step:** Don't trust the model's self-assessment alone — always have automated checks
- **Small steps win:** The smaller each step, the more reliable the output
- **Feedback is gold:** Every error message is a learning opportunity — feed it back
- **Document everything:** PROJECT_CONTEXT.md is your single source of truth
- **Adapt to the model:** Different models have different strengths — adjust your templates accordingly
- **When in doubt, slow down:** If the model starts producing low-quality output, stop and re-inject context
- **Human-in-the-loop for critical changes:** Architecture decisions, database migrations, and security-related changes should always get human review
