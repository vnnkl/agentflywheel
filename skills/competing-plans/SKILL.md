---
name: competing-plans
description: "Generate multiple independent implementation plans in parallel, then synthesize a best-of-all-worlds plan combining the strongest ideas from each. Use when planning a feature, designing architecture, creating an implementation strategy, or when the user says competing plans, parallel plans, plan shootout, multi-plan, best-of-all-worlds, or plan synthesis. This skill produces dramatically better plans than single-shot planning because different agent perspectives emphasize different aspects — one catches edge cases another misses, one proposes cleaner architecture while another has better error handling."
---

# Competing Plans

Generate 2-4 independent plans in parallel, then synthesize one superior plan by combining the best elements from each.

## Why This Works

A single planning pass has blind spots. The agent that writes it tends to commit early to one architectural direction, one error handling strategy, one set of tradeoffs. Competing plans break this by producing genuinely independent perspectives:

- Plan A might have the cleanest architecture but weak error handling
- Plan B might nail edge cases but over-engineer the data layer
- Plan C might propose the simplest solution but miss security concerns

The synthesis step combines Plan A's architecture, Plan B's edge case coverage, and the simplicity of Plan C. The result is better than any individual plan.

This is the "Competing Plans" concept from Jeffrey Emanuel's Agentic Coding Flywheel methodology.

## When to Use

- Feature implementation planning (the primary use case)
- System architecture decisions
- Refactoring strategies
- Migration approaches
- Any decision where exploring multiple directions before committing pays off

Do NOT use for trivial tasks (single-file changes, simple bug fixes). The overhead of parallel agents is wasted on problems with obvious solutions.

## The Workflow

```
[Brief] <- User describes what they want
   |
[1. CLARIFY] <- Refine requirements with the user
   |
[2. BRIEF] <- Write a tight planning brief
   |
[3. COMPETE] <- Spawn 3 parallel agents, each producing a full plan
   |
[4. EVALUATE] <- Score each plan on key dimensions
   |
[5. SYNTHESIZE] <- Merge the strongest elements into one plan
   |
[6. PRESENT] <- Show the synthesized plan with rationale
```

---

## Phase 1: Clarify

Understand the problem well enough to write a brief that different agents will interpret consistently. Ask the user 2-4 focused questions.

Key things to nail down:
- **Scope boundaries** -- what is explicitly out of scope?
- **Constraints** -- performance targets, tech stack requirements, timeline
- **Quality expectations** -- testing requirements, quality gates
- **Context** -- existing codebase patterns, related systems

Use the codebase to fill gaps. Read CLAUDE.md, check existing patterns, look at the tech stack. The less ambiguity in the brief, the more useful the competing plans.

---

## Phase 2: Write the Planning Brief

Create a crisp brief that all competing agents receive identically. The brief is the control variable -- same input, different reasoning produces the value.

### Brief Template

```markdown
## Planning Brief

### Objective
[1-2 sentences: what are we building/changing and why]

### Requirements
- [Functional requirement 1]
- [Functional requirement 2]
- ...

### Constraints
- Tech stack: [languages, frameworks, databases]
- Must integrate with: [existing systems/APIs]
- Performance: [targets if any]
- Timeline: [if relevant]

### Out of Scope
- [Explicitly excluded item 1]
- [Explicitly excluded item 2]

### Existing Context
- [Relevant codebase patterns found during clarification]
- [Key files or modules this touches]
- [Testing patterns in use]

### Quality Gates
- [e.g., pnpm typecheck && pnpm lint]
- [e.g., 80% test coverage]
```

Keep it factual and constraint-oriented. Do not suggest solutions -- that biases the competing plans toward the same approach.

---

## Phase 3: Compete

Spawn exactly 3 agents in parallel using the Agent tool. Each agent receives the same brief but a different planning persona. The personas push agents to emphasize different concerns, producing genuinely diverse plans.

Launch all three simultaneously:

```
Agent plan-alpha: "You are a pragmatic engineer who values simplicity and shipping speed.
Read the codebase context, then create a COMPLETE implementation plan for this brief.

PLANNING BRIEF:
[paste the brief from Phase 2]

Your plan must include:
1. Architecture overview (components, data flow, key abstractions)
2. Implementation phases with specific files to create/modify
3. Data model / schema changes (if any)
4. API design (if any)
5. Error handling strategy
6. Testing approach with specific test cases
7. Edge cases and how to handle them
8. Risks and mitigations
9. Estimated complexity (number of files, rough LOC)

Prioritize: getting it working with minimal moving parts. Avoid premature abstraction.
Return the full plan as structured markdown."

Task plan-bravo: "You are a defensive engineer who prioritizes robustness, security, and handling failure gracefully.
Read the codebase context, then create a COMPLETE implementation plan for this brief.

PLANNING BRIEF:
[paste the brief from Phase 2]

Your plan must include:
1. Architecture overview (components, data flow, key abstractions)
2. Implementation phases with specific files to create/modify
3. Data model / schema changes (if any)
4. API design (if any)
5. Error handling strategy
6. Testing approach with specific test cases
7. Edge cases and how to handle them
8. Risks and mitigations
9. Estimated complexity (number of files, rough LOC)

Prioritize: comprehensive error handling, input validation, security hardening, graceful degradation.
Return the full plan as structured markdown."

Task plan-charlie: "You are an architect who values clean abstractions, extensibility, and long-term maintainability.
Read the codebase context, then create a COMPLETE implementation plan for this brief.

PLANNING BRIEF:
[paste the brief from Phase 2]

Your plan must include:
1. Architecture overview (components, data flow, key abstractions)
2. Implementation phases with specific files to create/modify
3. Data model / schema changes (if any)
4. API design (if any)
5. Error handling strategy
6. Testing approach with specific test cases
7. Edge cases and how to handle them
8. Risks and mitigations
9. Estimated complexity (number of files, rough LOC)

Prioritize: separation of concerns, clean interfaces, patterns that scale to future requirements.
Return the full plan as structured markdown."
```

### Why These Three Personas

| Persona | Strength | Typical blind spot |
|---------|----------|-------------------|
| Alpha (Pragmatist) | Minimal complexity, ships fast | May skip validation, under-engineer |
| Bravo (Defender) | Error handling, security, edge cases | May over-engineer the happy path |
| Charlie (Architect) | Clean abstractions, extensibility | May add layers not needed yet |

The personas are complementary by design. Alpha keeps things simple. Bravo catches what Alpha skips. Charlie provides structure that scales. The synthesis combines all three perspectives.

### Adjusting Agent Count

- **2 agents** -- Use for smaller features where two perspectives suffice. Drop Charlie (architect) since YAGNI matters less for small scope.
- **4 agents** -- Add a fourth persona for domain-heavy work. Example: a "domain expert" persona that focuses on business rules, data integrity, and domain modeling.
- **Default to 3** -- This hits the sweet spot for most features. More agents means more synthesis work with diminishing returns.

---

## Phase 4: Evaluate

After all agents return their plans, score each on these dimensions:

### Scoring Matrix

Create a comparison table:

```markdown
| Dimension           | Alpha | Bravo | Charlie | Notes |
|---------------------|-------|-------|---------|-------|
| Architecture        | 3     | 2     | 4       | Charlie's module separation is cleanest |
| Simplicity          | 4     | 2     | 3       | Alpha avoids unnecessary layers |
| Error handling      | 2     | 4     | 3       | Bravo covers retry logic others miss |
| Edge cases          | 2     | 4     | 3       | Bravo identified 3 edge cases others missed |
| Security            | 2     | 4     | 3       | Bravo adds input validation everywhere |
| Testability         | 3     | 3     | 4       | Charlie's interfaces make mocking easy |
| Extensibility       | 2     | 2     | 4       | Charlie designed for future features |
| Feasibility         | 4     | 3     | 3       | Alpha is most realistic about scope |
```

Score 1-4:
- 4 = Best approach across all plans
- 3 = Solid approach
- 2 = Adequate but another plan does it better
- 1 = Weak or missing

This matrix drives the synthesis. For each dimension, pull from the highest-scoring plan.

---

## Phase 5: Synthesize

This is where the real value lives. Do not just pick a winner -- combine the strongest elements from each plan into something better than any individual.

### Synthesis Process

1. **Pick the architectural backbone.** Start with the highest-scoring architecture. This provides the structural skeleton.

2. **Layer in error handling.** Take the best error handling strategy and graft it onto the chosen architecture. If Bravo's retry logic is superior, adopt it even if using Charlie's architecture.

3. **Merge edge case coverage.** Union all edge cases identified across plans. Each agent notices different failure modes. Combine them.

4. **Resolve conflicts.** When plans contradict (e.g., Alpha uses a flat structure, Charlie uses nested modules), decide based on the specific project's needs and constraints. Document why you chose one over another.

5. **Simplify.** After merging, look for complexity that crept in. The merged plan may have redundant layers from combining multiple approaches. Strip what does not earn its keep.

6. **Verify completeness.** Walk through the brief's requirements one by one. Ensure the synthesized plan addresses every requirement and constraint.

### Synthesis Output Format

```markdown
## Synthesized Plan: [Feature Name]

### Source Attribution
- Architecture: from Plan Charlie (clean module boundaries)
- Error handling: from Plan Bravo (retry logic, graceful degradation)
- Implementation order: from Plan Alpha (pragmatic phasing)
- Edge cases: merged from all three plans

### Architecture Overview
[Combined architecture drawing from the strongest source]

### Implementation Phases
[Ordered phases, pulling the most practical sequencing]

#### Phase 1: [Name]
- Files: [specific files to create/modify]
- Changes: [what each file needs]
- Tests: [what to test at this phase]
- From: [which plan this phase draws from and why]

#### Phase 2: [Name]
...

### Data Model
[Best data model from the competing plans]

### API Design
[Best API design, with error responses from the defensive plan]

### Error Handling Strategy
[Comprehensive strategy, usually Bravo-influenced]

### Edge Cases
[Union of all edge cases from all plans]
| Edge Case | Source | Handling |
|-----------|--------|----------|
| [case 1]  | Bravo  | [how]    |
| [case 2]  | Charlie| [how]    |
| [case 3]  | Alpha  | [how]    |

### Testing Strategy
[Combined testing approach]

### Risks and Mitigations
[Merged risk register from all plans]

### Rejected Alternatives
[Document what was NOT adopted from each plan and why.
This is valuable context for future decisions.]
```

### The "Rejected Alternatives" Section

This section matters. When you reject Plan Alpha's flat file structure in favor of Charlie's module hierarchy, document it:

> **Rejected:** Alpha proposed a flat file structure with all handlers in one directory.
> **Why:** With 8+ endpoints, flat structure makes navigation harder. Charlie's module-per-domain approach scales better.
> **When to reconsider:** If scope shrinks to 3 or fewer endpoints, flat is simpler.

This prevents rehashing the same debate later and preserves the reasoning.

---

## Phase 6: Present

Show the user:

1. **The scoring matrix** -- so they see why certain elements won
2. **The synthesized plan** -- the complete merged plan
3. **Key tradeoff decisions** -- the 2-3 most consequential choices made during synthesis

Then ask:

```
The synthesized plan combines:
- [Strongest element 1] from Plan [X]
- [Strongest element 2] from Plan [Y]
- [Strongest element 3] from Plan [Z]

Options:
A. Accept this plan and proceed to implementation
B. See the full competing plans for comparison
C. Adjust specific sections (tell me which)
D. Re-run with different constraints or additional persona
```

---

## Integration with Other Skills

The synthesized plan output is designed to feed into downstream workflows:

- **compound-plan** -- Convert the synthesized plan into prd.json for compounding-ralph-tui execution
- **compound-review** -- After implementation, review the code against the original competing plans
- **tdd** -- Use the testing strategy from the synthesized plan to drive TDD workflow

---

## Example

**User:** "Plan a notification system for our app"

**Phase 1 (Clarify):**
- In-app only or also email/push? -> In-app + email
- Real-time required? -> Yes, WebSocket for in-app
- Existing patterns? -> Found existing WebSocket setup in `src/ws/`

**Phase 2 (Brief):** Write brief covering notification types, delivery channels, real-time requirements, existing WebSocket infrastructure.

**Phase 3 (Compete):** Three agents return plans.
- Alpha: Simple event emitter, notifications table, React hook. 6 files.
- Bravo: Same core but adds delivery retry queue, notification deduplication, rate limiting per user, dead letter handling. 11 files.
- Charlie: NotificationService abstraction, Channel interface (WebSocket, Email implement it), NotificationRepository, Observer pattern. 14 files.

**Phase 4 (Evaluate):**
- Architecture: Charlie (4) -- cleanest separation
- Simplicity: Alpha (4) -- least code
- Error handling: Bravo (4) -- retry queue is valuable for email delivery
- Edge cases: Bravo (4) -- rate limiting prevents notification spam

**Phase 5 (Synthesize):**
- Use Charlie's Channel interface (clean extension point for future push notifications)
- Use Alpha's simple notifications table (skip Charlie's over-normalized schema)
- Use Bravo's retry queue for email channel (but not for WebSocket -- unnecessary)
- Use Bravo's rate limiting (Alpha and Charlie both missed this)
- Result: 9 files, clean architecture, robust email delivery, practical scope

---

## Checklist

Before presenting the synthesized plan:

- [ ] Brief written and reviewed (no solution bias)
- [ ] All competing agents returned complete plans
- [ ] Scoring matrix filled for all dimensions
- [ ] Synthesis pulls from multiple plans (not just picking a winner)
- [ ] Conflicts between plans explicitly resolved with rationale
- [ ] Edge cases merged (union of all plans)
- [ ] Rejected alternatives documented
- [ ] Synthesized plan covers every requirement from the brief
- [ ] Result is simpler than the most complex competing plan
