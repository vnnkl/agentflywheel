---
name: deep-planning
description: Enforce 85% planning before any implementation. Produce a comprehensive markdown plan covering architecture, data flow, error handling, edge cases, and security before writing code. Use when user wants to build a feature, implement something, start a new project, design architecture, scaffold an app, create a system, or begin any substantial coding work.
---

# Deep Planning

Stop. Before writing a single line of code, plan.

This skill enforces a counterintuitive discipline: spend 85% of effort on planning while the entire system still fits in your context window. Once code spreads across hundreds of files, holistic reasoning becomes impossible. A 3,000-line markdown plan is cognitively manageable. The codebase it describes is not.

## Why Planning Dominates

There are three reasoning spaces in software development. Each has a radically different cost when mistakes happen:

| Space | Artifact | Rework Cost |
|-------|----------|-------------|
| **Plan** | Markdown document | **1x** |
| **Task** | Bead / work unit | **5x** |
| **Code** | Source files | **25x** |

A missed edge case in a plan costs one paragraph to fix. That same edge case discovered during implementation costs refactoring across multiple files, updating tests, fixing cascading type errors, and debugging integration failures. Discovered in production, it costs all of that plus incident response.

Planning tokens are cheap. Implementation tokens are expensive. The rational strategy is to front-load reasoning into the cheapest space.

### What Planning Buys You

**Global reasoning.** During planning, the entire system fits in context. You can reason about how the auth layer interacts with the billing system interacts with the notification pipeline. Once those become 50 separate files, that reasoning is gone.

**Security emerges from completeness.** You do not need a separate "security phase." A plan detailed enough to describe every data flow, every trust boundary, every user interaction will naturally surface auth gaps, data exposure risks, and injection vectors. Security holes are planning holes.

**Mechanical translation.** A plan is "done" when converting it to code requires no creative problem-solving — only translation. If an implementer needs to make architectural decisions while coding, the plan failed.

## When This Skill Activates

Apply deep planning to any work that involves:
- Building a new feature with multiple components
- Starting a new project or application
- Designing system architecture
- Implementing workflows that cross multiple boundaries (API, database, UI, external services)
- Any task where "just start coding" would require backtracking

Do NOT apply to:
- Single-file bug fixes with obvious solutions
- Cosmetic changes (copy, styling)
- Adding a dependency or configuration value
- Tasks completable in under 20 lines of code

## The Planning Process

### Phase 1: Understand the Problem

Before planning the solution, make sure you understand the problem. Ask clarifying questions. Do not assume.

Key questions to resolve:
- What problem does this solve? For whom?
- What does success look like? How is it measured?
- What is explicitly out of scope?
- What existing systems does this interact with?
- What constraints exist (performance, cost, compliance, timeline)?

If working in an existing codebase, explore it first. Understand the patterns, conventions, and architectural decisions already in place. The plan must fit the codebase, not the other way around.

### Phase 2: Write the Plan

Produce a comprehensive markdown document. Save it to `./plans/<feature-name>.md` (create the directory if needed). The plan must cover every section below that is relevant to the feature.

#### Plan Structure

```markdown
# Plan: <Feature Name>

## Problem Statement
What problem this solves and why it matters. One paragraph.

## Goals
- Specific, measurable outcomes this feature achieves

## Non-Goals
- What this explicitly does NOT do (scope boundaries)

## Architecture

### System Overview
How the feature fits into the broader system. Describe the high-level
components and their responsibilities.

### Data Model
Entities, relationships, field types, constraints, indexes.
Be specific — "a users table" is not a data model.

### Data Flow
Trace every significant data path end-to-end:
- Where data enters the system
- How it transforms at each step
- Where it persists
- What triggers movement between components

### API Design
Endpoints, request/response shapes, status codes, pagination,
rate limiting. Include example payloads.

### State Management
What state exists, where it lives, how it synchronizes,
what happens when it conflicts.

## User Interactions
Every user-facing flow, step by step:
- What the user sees
- What they can do
- What happens when they do it
- What feedback they receive

## Edge Cases and Error Handling
For every component and interaction:
- What can go wrong?
- What does the system do when it goes wrong?
- What does the user see?
- How does the system recover?

Think about: network failures, partial writes, race conditions,
malformed input, authorization failures, timeouts, resource
exhaustion, concurrent modifications, empty states, boundary
values.

## Security Boundaries
- Authentication: who can access what
- Authorization: what can each role do
- Data exposure: what data is visible to whom
- Trust boundaries: where validated data crosses into
  unvalidated territory
- Input validation: what is validated, where, how
- Secrets management: what secrets exist, how they are stored

## Performance Considerations
- Expected load patterns
- Bottleneck predictions
- Caching strategy
- Query optimization notes

## Migration and Rollout
- Database migration steps
- Feature flag strategy
- Rollback plan
- Data backfill requirements

## Dependencies
- External services and their failure modes
- New libraries and why they were chosen
- Infrastructure requirements

## Open Questions
- Unresolved decisions (with options and tradeoffs)
- Unknowns that need research
- Decisions deferred to implementation
```

Not every section applies to every feature. Skip sections that are genuinely irrelevant. But err on the side of including a section with a brief note rather than silently skipping it — the act of considering each section is the point.

### Phase 3: Review the Plan

Before declaring the plan complete, review it against the quality checklist below. Walk through the plan as if you were the implementer encountering it for the first time. Ask:

- Could I implement this without asking the planner any questions?
- Are there decisions I would need to make that the plan does not address?
- If I implemented this exactly as described, would it actually work?

If the answer to any of these is "no," the plan is not done.

### Phase 4: Present and Iterate

Present the plan to the user. Explicitly ask:
1. Does this match your understanding of the feature?
2. Are there flows or edge cases I missed?
3. Should any section be more or less detailed?
4. Are the open questions the right ones?

Iterate until the user confirms the plan is complete.

### Phase 5: Transition to Implementation

Only after the plan is reviewed and approved, begin implementation. During implementation:
- Reference the plan constantly
- If you discover the plan was wrong or incomplete, update the plan first, then update the code
- Do not make architectural decisions in code that contradict the plan — update the plan instead

## Plan Quality Checklist

Run through this before presenting the plan:

### Completeness
- [ ] Every user-facing flow is described step by step
- [ ] Every data entity is defined with fields and relationships
- [ ] Every API endpoint includes request/response shapes
- [ ] Error handling is specified for each component, not just the happy path
- [ ] Edge cases are enumerated, not hand-waved ("handle errors appropriately" is not a plan)

### Specificity
- [ ] Data model uses concrete field names and types, not abstractions
- [ ] API payloads include example JSON, not just descriptions
- [ ] State transitions are explicit (what triggers them, what changes)
- [ ] "It should handle X" is replaced with "When X occurs, the system does Y"

### Consistency
- [ ] Terminology is consistent throughout (same entity is not called "user," "account," and "profile" interchangeably)
- [ ] Data flows do not contradict each other
- [ ] Authorization rules are consistent across all endpoints

### Security
- [ ] Every endpoint has explicit auth requirements
- [ ] Every user input has explicit validation rules
- [ ] Data exposure is intentional (no accidental leaking of internal IDs, emails, etc.)
- [ ] Trust boundaries are identified (where does validated data meet unvalidated data?)

### Implementability
- [ ] An implementer could build this without making architectural decisions
- [ ] No section requires "figure this out later"
- [ ] Dependencies and integration points are concrete
- [ ] The plan accounts for the existing codebase patterns (if applicable)

### Honest Gaps
- [ ] Open questions are genuinely open (not things the planner was too lazy to decide)
- [ ] Risks are identified with mitigation strategies
- [ ] Assumptions are stated explicitly

## Anti-Patterns to Avoid

**The Vague Plan.** "The system should handle errors gracefully." This is not a plan. What errors? What does "gracefully" mean? What does the user see? What gets logged? What gets retried?

**The Happy Path Plan.** Describes what happens when everything works. Says nothing about what happens when things fail. Half of a system's complexity lives in error handling — a plan that ignores it is half a plan.

**The Copy-Paste Architecture.** "We will use a standard REST API with a database." This describes every web application ever built. The plan must describe THIS system's specific design decisions.

**The Premature Implementation.** Including file names, function signatures, or framework-specific code in the plan. Plans describe WHAT and WHY. Implementation describes HOW. Keep them separate — implementation details in plans become stale immediately.

**The Infinite Plan.** Planning is not a substitute for building. If the plan has been through three review cycles and you are still finding things to add, the remaining gaps are probably better discovered during implementation. Ship the plan at ~75% convergence.

## A Note on Judgment

This skill is a process, not a bureaucracy. The goal is not to produce a plan that checks every box. The goal is to do the hard thinking BEFORE the expensive phase begins.

A one-page plan for a well-understood feature in a familiar codebase can be more valuable than a fifty-page plan for a greenfield project — if that one page captures the genuinely hard decisions.

Use judgment. Plan deeply where uncertainty is high. Plan lightly where the path is clear. But always plan first.
