---
name: multi-mode-review
description: Three-mode code review system that catches different bug categories by alternating review perspectives. Use when user says "review", "code review", "check my code", "find bugs", "look for issues", "review my changes", "what did I miss", "anything wrong with this", "sanity check", "spot check", "audit this code", "review PR", "check for bugs", "is this correct", or any request to evaluate code quality. Also use after completing a feature implementation, before merging, or when the user seems unsure about correctness. Trigger aggressively - if there is any hint the user wants code evaluated, use this skill.
---

# Multi-Mode Review

Three distinct review modes that activate different problem-finding behaviors. Each mode catches a different category of bugs. Using only one mode leaves entire bug categories undetected.

## Why Three Modes?

| Mode | What It Catches | Blind Spots |
|------|----------------|-------------|
| Fresh Eyes | Local logic errors, off-by-ones, missed edge cases, naming issues | Cannot see integration problems or systemic drift |
| Cross-Agent | Integration errors, boundary mismatches, coupling, missing error handling at interfaces | Cannot see local logic errors (different agent lacks original context) |
| Random Exploration | Architectural drift, dead code paths, inconsistent patterns, systemic issues | Cannot see local logic errors or specific integration bugs |

A single-mode review gives false confidence. The code "passes review" but only for one category of bugs.

## When to Use Which Mode

### Default: Run All Three

For any meaningful review request, run all three modes. They are complementary, not alternatives.

### If time-constrained, prioritize by situation:

- **Just finished writing code** -> Fresh Eyes first (catch logic errors before they propagate)
- **Integrating multiple components** -> Cross-Agent first (catch boundary mismatches early)
- **Codebase has grown organically / tech debt concerns** -> Random Exploration first (find systemic rot)
- **Pre-merge / PR review** -> All three, start with Cross-Agent (integration issues are the costliest to fix post-merge)
- **Bug hunt (something is wrong but unclear what)** -> Random Exploration first (trace actual execution to find where assumptions break)

## Mode 1: Fresh Eyes Review

### Why This Mode Exists

After writing code, you are blind to your own assumptions. You read what you intended, not what you wrote. Fresh Eyes forces a perspective shift: pretend you did not write this code. Look for what is wrong, not what is right.

This mode catches: off-by-one errors, unhandled edge cases, incorrect boolean logic, misleading variable names, missing null checks, wrong operator precedence, silent failures, resource leaks.

### Steps

1. **Identify the scope.** Determine which files were recently changed. Use `git diff` or `git diff --staged` to find them. If the user points to specific files, use those.

2. **Read each changed file completely.** Do not skim. Read every line as if seeing it for the first time.

3. **For each file, answer these questions:**
   - What happens when inputs are empty, null, undefined, or zero?
   - What happens when inputs are at boundary values (MAX_INT, empty string, single element array)?
   - Are all error paths handled? What happens when an async operation fails?
   - Are there implicit type coercions that could produce unexpected results?
   - Does the code handle the "unhappy path" as carefully as the "happy path"?
   - Are there race conditions in async code?
   - Are resources (file handles, connections, subscriptions) properly cleaned up?
   - Do variable names accurately describe what they hold?
   - Is there mutable state that could be corrupted by concurrent access?

4. **Check for logic errors specifically:**
   - Verify loop bounds (off-by-one is the most common bug)
   - Verify conditional logic (De Morgan's law violations, inverted conditions)
   - Verify comparison operators (< vs <=, == vs ===)
   - Verify short-circuit evaluation side effects
   - Verify that early returns do not skip necessary cleanup

5. **Document each finding** with:
   - File path and line number
   - The specific issue
   - Why it is a problem (not just "looks wrong" - explain the failure scenario)
   - Suggested fix

### Output Format

```
## Fresh Eyes Review

### Critical
- **[file:line]** Description of issue
  - Failure scenario: What happens when...
  - Fix: ...

### Important
- **[file:line]** Description of issue
  - Failure scenario: ...
  - Fix: ...

### Minor
- **[file:line]** Description of issue
  - Fix: ...
```

## Mode 2: Cross-Agent Review

### Why This Mode Exists

Integration bugs happen at boundaries between components. The original author understands both sides of an interface, so boundary mismatches are invisible to them. A separate agent reviewing the code has no shared context - it must reconstruct assumptions from the code alone. This is exactly the perspective needed to catch integration errors.

This mode catches: mismatched function signatures between caller and callee, inconsistent error handling across module boundaries, type mismatches at API boundaries, missing validation at trust boundaries, assumptions in one module that do not hold in another, event/callback contract violations, shared state mutations across modules.

### Steps

Spawn a subagent using the Agent tool with these instructions:

```
You are a cross-agent code reviewer. You did NOT write this code. Your job is to find integration issues - bugs that live at the boundaries between components.

Review the following files: [list changed files]

Focus exclusively on:

1. **Interface contracts**: For every function/method call that crosses a module boundary, verify:
   - Caller passes arguments in the correct order and type
   - Caller handles all possible return values (including null, undefined, error cases)
   - Caller and callee agree on error signaling (exceptions vs error codes vs Result types)
   - Optional parameters have sensible defaults on both sides

2. **Data flow across boundaries**: Trace data as it moves between modules:
   - Is data validated at trust boundaries (user input, API responses, file reads)?
   - Are data transformations between modules consistent (date formats, string encoding, numeric precision)?
   - Are there assumptions about data shape that could break?

3. **State management across components**:
   - Is shared state accessed consistently (same locks, same ordering)?
   - Do components agree on lifecycle (initialization order, cleanup order)?
   - Are there hidden temporal dependencies (A must run before B)?

4. **Error propagation**:
   - When module A calls module B which calls module C, does an error in C propagate correctly to A?
   - Are errors translated appropriately at each boundary?
   - Are there swallowed errors that leave the system in an inconsistent state?

5. **Configuration and environment assumptions**:
   - Do modules agree on environment variable names?
   - Do modules agree on file paths, URL formats, encoding?
   - Are there hardcoded values in one module that should match a constant in another?

For each finding, specify BOTH sides of the boundary (which module assumes what, and which module violates that assumption).
```

The subagent's independence is the value here. Do not prime it with your own understanding of the code - let it discover issues from scratch.

### Output Format

```
## Cross-Agent Review

### Integration Issues
- **[moduleA:line <-> moduleB:line]** Description
  - Module A assumes: ...
  - Module B does: ...
  - Failure scenario: ...
  - Fix: ...

### Boundary Concerns
- **[boundary description]** Description
  - Risk: ...
  - Recommendation: ...
```

## Mode 3: Random Code Exploration

### Why This Mode Exists

Local reviews (Fresh Eyes) and interface reviews (Cross-Agent) both examine code in isolation. Some bugs are only visible when you trace a complete execution flow through the system. Architectural drift, dead code, inconsistent patterns, and missing error handling in rare paths only emerge when you follow a user action from entry point to final side effect.

This mode catches: dead code paths, inconsistent error handling patterns, architectural drift (new code ignoring established conventions), missing logging/monitoring in critical paths, performance issues from unnecessary work, security gaps in rarely-exercised paths.

### Steps

1. **Pick 2-3 user actions to trace.** Choose actions that exercise critical or recently-changed code paths. Good choices:
   - The primary user action the code supports
   - An error/failure scenario
   - An edge case the user mentioned or that is implied by the domain

2. **For each action, trace the complete execution flow:**
   - Start at the entry point (HTTP handler, CLI command, event listener, UI callback)
   - Follow every function call, tracking what data is passed and transformed
   - Note every branch point (if/else, switch, try/catch)
   - Follow the flow through all layers (handler -> service -> repository -> database)
   - Track side effects (writes, external calls, events emitted)
   - End at the final response or state change

3. **While tracing, look for:**
   - **Dead code**: Branches that can never execute given current callers
   - **Inconsistent patterns**: One endpoint validates input, another does not. One service logs errors, another swallows them.
   - **Missing error handling**: Happy path works, but what if the database is down? What if the external API times out?
   - **Architectural drift**: New code using a different pattern than established code (e.g., direct database calls when the codebase has a repository layer)
   - **Performance concerns**: N+1 queries, unnecessary serialization/deserialization, loading full objects when only IDs are needed
   - **Security gaps**: Missing auth checks, unvalidated redirects, SQL injection in dynamically constructed queries

4. **Compare patterns across flows.** If tracing 3 different actions, do they all handle errors the same way? Do they all validate input? Do they all log the same way? Inconsistencies are bugs waiting to happen.

### Output Format

```
## Random Code Exploration

### Flow: [User Action Description]
Entry: [file:function]
Path: file:function -> file:function -> ... -> [final side effect]

Findings:
- **[file:line]** Description
  - Context: Found while tracing [action], at [point in flow]
  - Impact: ...
  - Fix: ...

### Cross-Flow Inconsistencies
- **Pattern**: [e.g., "error handling"]
  - Flow A does: ...
  - Flow B does: ...
  - Recommendation: ...
```

## Running the Full Review

When the user asks for a review, execute all three modes. For efficiency, run Fresh Eyes yourself and spawn the Cross-Agent review as a subagent in parallel. Then run Random Exploration after both complete (its findings build on context from the first two modes).

### Sequence

1. **In parallel:**
   - Perform Fresh Eyes review on changed files
   - Spawn Cross-Agent subagent to review the same files
2. **After both complete:**
   - Perform Random Exploration, informed by findings from modes 1 and 2
3. **Synthesize:**
   - Combine findings from all three modes
   - Deduplicate (if the same issue appears in multiple modes, consolidate)
   - Prioritize by severity: Critical > Important > Minor
   - Present the combined report to the user

### Combined Report Format

```
# Code Review Report

## Summary
- X critical issues, Y important issues, Z minor issues
- Modes used: Fresh Eyes, Cross-Agent, Random Exploration

## Critical Issues
[issues from any mode, consolidated]

## Important Issues
[issues from any mode, consolidated]

## Minor Issues
[issues from any mode, consolidated]

## Architectural Observations
[patterns, drift, inconsistencies found during exploration]
```

## Tips for Effective Reviews

- **Do not soften findings.** If something is broken, say it is broken. "This might potentially be a concern" is not useful. "This will throw a NullPointerException when user.address is undefined" is useful.
- **Every finding needs a failure scenario.** If you cannot describe when the bug triggers, it might not be a real bug.
- **Suggest fixes, do not just identify problems.** The reviewer's job is not done until the path forward is clear.
- **Separate real bugs from style preferences.** Only flag style issues under "Minor" and only if they genuinely reduce readability or maintainability.
