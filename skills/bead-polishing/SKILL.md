---
name: bead-polishing
description: "Iteratively refine task breakdowns (beads, issues, stories, tickets) through 4-6 polishing rounds until convergence. Based on the 'Check N Times, Implement Once' principle. Use when user wants to polish beads, refine tasks, review task breakdown quality, improve issue definitions, iterate on a plan's task list, check task coverage, or deduplicate tasks. Triggers on: polish beads, refine tasks, polish tasks, review breakdown, check coverage, deduplicate issues, iterate tasks, polish issues, converge tasks."
---

# Bead Polishing

Iteratively refine a task breakdown through multiple review rounds until the tasks converge. Works with any task format: beads, GitHub issues, user stories, PRD tasks, todo lists.

## Why This Matters

The Three Reasoning Spaces cost model:

| Space | Rework Cost | Example |
|-------|-------------|---------|
| Plan | 1x | Fix a markdown document |
| Task definitions | 5x | Rewrite and re-link beads/issues |
| Code | 25x | Debug, refactor, retest |

Fixing a vague task description takes 30 seconds. Fixing the code that a vague description produced takes 30 minutes. Polishing task definitions until they are airtight is the single highest-ROI activity in the entire workflow.

A task breakdown that "looks done" after one pass almost always contains duplicates, gaps, broken dependencies, and underspecified acceptance criteria. These defects are invisible until an agent starts coding and produces the wrong thing. Polishing exposes them while the fix is still cheap.

---

## Process Overview

```
[Task breakdown exists]
       |
       v
  Round 1: Structural audit
       |
       v
  Round 2: Coverage + dependency check
       |
       v
  Round 3: Description depth pass
       |
       v
  Round 4: Adversarial review
       |
       v
  Round 5+: Only if convergence < 75%
       |
       v
  [Polished, converged task set]
```

Run 4-6 rounds. Each round applies ALL checks but emphasizes a different lens. Track convergence after each round. Stop when convergence reaches ~75% (diminishing returns appear around 90%).

---

## Before Starting

Identify the task source. Ask the user if unclear:

- **Beads:** `.beads/beads.jsonl` or `bd list --json`
- **GitHub Issues:** `gh issue list` with a milestone or label filter
- **PRD user stories:** A markdown file with `US-XXX` entries
- **prd.json:** A JSON file with `userStories` array
- **Other:** Any structured list of tasks with titles and descriptions

Read the **source plan** (the PRD or design doc the tasks were derived from). Polishing without the source plan is guessing. If no source plan exists, ask the user for the intent document or feature description.

---

## Round Structure

Every round applies the same five checks. The emphasis shifts per round, but run all five every time.

### Check 1: Duplicates and Overlaps

Look for tasks that:
- Cover the same functionality with different wording
- Partially overlap (one task's scope is a subset of another's)
- Were split unnecessarily (two tasks that are always done together)

**Action:** Merge duplicates. For partial overlaps, redraw the boundary so each task owns a distinct piece. For unnecessary splits, combine into one task.

### Check 2: Coverage Against Source Plan

Compare the task list against every section, feature, and requirement in the source plan.

- Walk through the source plan section by section
- For each requirement, confirm at least one task addresses it
- Flag requirements with no corresponding task as **gaps**
- Flag tasks with no corresponding requirement as **orphans** (may indicate scope creep or may be legitimate infrastructure work)

**Action:** Create tasks for gaps. Investigate orphans -- either link them to a requirement or remove them.

### Check 3: Dependency Integrity

Verify dependency links are correct and complete:
- No circular dependencies
- No task depends on something that should come after it
- No missing dependencies (task B uses what task A creates, but no link exists)
- No unnecessary dependencies (task marked as blocked when it could start immediately)

Walk through the execution order that dependencies imply. Mentally simulate: "If I execute these in dependency order, does each task have everything it needs when it starts?"

**Action:** Add missing dependency links. Remove false dependencies. Fix reversed links.

### Check 4: Description Completeness

Each task needs enough context for an agent (or developer) to execute without asking questions. Check for:

- **Vague acceptance criteria:** "works correctly" or "good UX" are not verifiable
- **Missing technical context:** Which files to modify, which APIs to call, which patterns to follow
- **Missing scope boundaries:** What this task explicitly does NOT include
- **Underspecified inputs/outputs:** What the task receives and what it produces
- **Missing quality gates:** Test commands, lint requirements, type-check obligations

**Action:** Rewrite vague criteria into verifiable statements. Add technical context. Add explicit scope boundaries for ambiguous tasks.

### Check 5: Complexity and Sizing

Each task should be completable in a single focused session (one agent context window, one PR, one afternoon).

- Flag tasks that require touching more than 3-4 files across different domains
- Flag tasks with more than 5-6 acceptance criteria (likely too big)
- Flag tasks with descriptions under 2 sentences (likely too thin -- missing context that will cause the implementer to guess)

**Action:** Split oversized tasks into vertical slices. Expand underspecified tasks with context from the source plan.

---

## Round-by-Round Emphasis

While all five checks run every round, each round leads with a different lens to prevent tunnel vision.

### Round 1: Structural Audit (lead with Checks 1 + 5)

First pass focuses on shape. Are there obvious duplicates? Are tasks the right size? This round produces the most changes -- task merges, splits, and removals.

### Round 2: Coverage + Dependencies (lead with Checks 2 + 3)

Now that the structure is clean, verify completeness. Walk the source plan end-to-end. Trace the dependency graph. This round typically adds 2-5 missing tasks and fixes 3-4 dependency links.

### Round 3: Description Depth (lead with Check 4)

With the right set of tasks identified, invest in making each one self-contained. Rewrite acceptance criteria. Add file paths, API references, and pattern guidance. This round makes task descriptions noticeably longer and more specific.

### Round 4: Adversarial Review (all checks, skeptical posture)

Read every task as if you are an agent encountering it for the first time with zero context beyond what the task provides. Ask:
- "Could I start working on this right now without asking any questions?"
- "Would two different agents interpret this the same way?"
- "If this task fails, will the error be in the task or in the code?"

This round catches subtle ambiguities that survived earlier passes.

### Rounds 5-6: Only if Needed

Run additional rounds only if convergence is below 75% after round 4. Each additional round should produce fewer changes than the previous one. If a round produces MORE changes than the prior round, something is structurally wrong -- step back and re-examine the source plan.

---

## Convergence Tracking

After each round, record these metrics:

```markdown
## Polishing Log

### Round 1
- Tasks added: 3
- Tasks removed: 2
- Tasks modified: 8
- Dependencies changed: 4
- Total changes: 17
- Notes: Merged 2 duplicate DB tasks, split the monolithic "build dashboard" into 4 slices

### Round 2
- Tasks added: 2
- Tasks removed: 0
- Tasks modified: 5
- Dependencies changed: 3
- Total changes: 10
- Change velocity: -41% (converging)
- Notes: Found 2 gaps in auth flow, fixed circular dep between US-007 and US-009

### Round 3
- Tasks added: 0
- Tasks removed: 0
- Tasks modified: 6
- Dependencies changed: 0
- Total changes: 6
- Change velocity: -40% (converging)
- Notes: Expanded all acceptance criteria, added file references

### Round 4
- Tasks added: 0
- Tasks removed: 0
- Tasks modified: 2
- Dependencies changed: 1
- Total changes: 3
- Change velocity: -50% (converging)
- Convergence: ~82%
- Notes: Clarified 2 ambiguous criteria, one missing dep link
```

### Convergence Signals

**Converging (good):**
- Total changes decreasing round over round
- No new tasks being added
- Modifications are wording tweaks, not structural changes
- Dependency graph is stable

**Not converging (investigate):**
- Changes increasing or holding steady
- New tasks still appearing in round 3+
- Dependency graph keeps shifting
- Tasks being split AND merged in the same round

### When to Stop

- **~75% convergence (4 rounds):** Good default stopping point. Further polishing yields diminishing returns.
- **~90% convergence:** Maximum useful refinement. Beyond this, you are wordsmithing, not improving.
- **Change velocity near zero for 2 consecutive rounds:** Stop even if you haven't hit a target number of rounds.

---

## Presenting Results

After the final round, present the user with:

1. **Summary:** How many rounds ran, total tasks before and after, key structural changes
2. **The polishing log** (all rounds)
3. **Remaining concerns:** Any unresolved ambiguities or decisions that need human input
4. **The polished task list** in its original format

Ask the user:
- Are the tasks at the right granularity?
- Do the dependencies look correct?
- Any requirements I may have missed?
- Ready to proceed to implementation?

---

## Format-Specific Notes

### Beads (`.beads/beads.jsonl`)

Use `bd` commands to modify beads. After each round, run `bd list` to verify the state. Use `bd update` to modify descriptions and `bd dep add`/`bd dep remove` to fix dependencies.

### GitHub Issues

Use `gh issue edit` to update titles, bodies, and labels. Use issue comments to log polishing changes (creates an audit trail). Add/remove "blocked-by" references in issue bodies.

### PRD User Stories (markdown)

Edit the markdown file directly. Keep the `US-XXX` numbering stable -- renumber only if tasks are removed. Update the dependency references inline.

### prd.json

Edit the JSON file. Update `userStories` array entries. Adjust `dependsOn` arrays. Keep `passes: false` on all stories.

---

## Anti-Patterns

**Polishing without a source plan.** The whole point is to verify tasks against intent. Without the source document, you are just rearranging tasks based on vibes.

**Endless polishing.** If you are on round 7 and still finding significant issues, the source plan itself is likely underspecified. Stop polishing tasks and go fix the plan.

**Polishing one task at a time.** Each round should review ALL tasks. Tasks interact through dependencies and coverage -- reviewing in isolation misses gaps.

**Treating the first pass as final.** A task breakdown that "looks complete" after one pass is the most dangerous kind. It has just enough structure to feel done while hiding the defects that will cost 25x to fix in code.

---

## Checklist

Before declaring polishing complete:

- [ ] Ran at least 4 rounds
- [ ] Convergence reached ~75%+ (change velocity decreasing)
- [ ] No duplicate or overlapping tasks remain
- [ ] Every source plan requirement maps to at least one task
- [ ] No orphan tasks without a clear purpose
- [ ] Dependency graph has no cycles and no missing links
- [ ] Every task has verifiable acceptance criteria (not vague)
- [ ] Every task is completable in one focused session
- [ ] Polishing log recorded with metrics for each round
- [ ] User reviewed and approved the final task set
