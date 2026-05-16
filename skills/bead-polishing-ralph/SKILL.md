---
name: bead-polishing-ralph
description: "Iteratively refine beads for ralph-tui execution through 4-6 polishing rounds until convergence, ensuring each bead fits one agent context window, has quality gates appended, uses correct bd CLI syntax, and follows schema-backend-UI dependency ordering. Use when user wants to polish beads for ralph, refine ralph tasks, iterate on beads before running ralph-tui, review bead quality for ralph execution, or check if beads are ralph-ready. Triggers on: polish beads ralph, refine beads for ralph, ralph bead review, check beads, are beads ready, polish for ralph-tui, iterate ralph beads."
---

# Bead Polishing for Ralph-TUI

Iteratively refine beads through 4-6 polishing rounds until convergence, enforcing all constraints that ralph-tui needs for successful autonomous execution.

This skill combines the convergence-tracked polishing loop from the Agentic Coding Flywheel with the specific format and sizing constraints required by ralph-tui's beads tracker.

## Why This Matters

ralph-tui spawns a **fresh agent instance per bead** with no memory of previous work. A bead that is too big, vague, or has broken dependencies will cause the agent to either run out of context, produce the wrong thing, or block the entire pipeline. Polishing beads before execution is the single highest-ROI activity because:

- Plan mistakes cost **1x** to fix (edit the markdown)
- Bead mistakes cost **5x** to fix (rewrite + re-link + re-order)
- Code mistakes cost **25x** to fix (debug + refactor + retest across files)

A 30-second fix to a bead description prevents a 30-minute failed ralph-tui iteration.

---

## Before Starting

### Detect CLI

Use `bd` (beads) as the CLI for all bead operations. The polishing process uses `bd` commands throughout.

### Gather Context

1. **Read the source plan/PRD** that the beads were created from. Polishing without the source is guessing.
2. **Extract Quality Gates** from the PRD's "Quality Gates" section. If none exists, ask the user what commands should pass (e.g., `pnpm typecheck`, `pnpm lint`).
3. **Check if beads exist**: `bd list --json`

### Create Beads if None Exist

If no beads exist yet, create them from the source PRD before polishing. This makes the separate "convert to tasks" step optional — bead-polishing-ralph can handle the full plan-to-polished-beads pipeline.

1. **Create an epic** linking back to the PRD:
   ```bash
   bd create --type=epic \
     --title="[Feature Name]" \
     --description="$(cat <<'EOF'
   [Feature description from PRD]
   EOF
   )" \
     --external-ref="prd:./path-to-prd.md"
   ```

2. **Create one bead per user story/requirement** from the PRD. For each story:
   - Title matches the story title
   - Description includes story context, acceptance criteria, quality gates, and the `<promise>COMPLETE</promise>` instruction
   - Priority reflects dependency order (schema=1, backend=2, UI=3, polish=4)
   - Pass `--parent=<epic-id>` at creation time so the parent link exists from the start

3. **Add dependencies** between beads: `bd dep add <issue> <depends-on>`. **NEVER** add `bd dep add <epic> <child>` ("epic depends on child"). The parent-child link already represents "epic completes when children complete" — adding a reverse dep creates a cycle with the parent-child relation and breaks `bd ready`.

4. **Identify the epic**: Which epic are we polishing?

> **CRITICAL — Epic type.** The epic MUST be `--type=epic` (not `task` / `feature` / `bug`). `bd` represents `--parent=<epic>` as a `parent-child` dependency on the child, and `bd ready` treats any open ancestor as a blocker UNLESS the ancestor is `type=epic`. A `feature`-typed parent will silently block every child from appearing in `bd ready`, which causes ralph-tui to report "no more tasks available after starting." Verify with `bd show <id>` — the line should read `[epic]`.

This initial creation is intentionally rough — it's the "first pass" that the polishing rounds will refine. Don't over-invest here; the convergence loop catches everything.

---

## The Five Ralph-Specific Checks

Every round applies all five checks. Each check is tuned for ralph-tui's execution model.

### Check 1: Size — One Context Window Per Bead

ralph-tui's #1 constraint: each bead must be completable in **one agent iteration** (~one context window). The agent starts fresh with no memory of previous work.

**Right-sized:**
- Add a database column + migration
- Add a UI component to an existing page
- Update a server action with new logic
- Add a filter dropdown to a list

**Too big (split these):**
- "Build the entire dashboard" → schema, queries, UI components, filters
- "Add authentication" → schema, middleware, login UI, session handling
- "Refactor the API" → one story per endpoint or pattern

**Rule of thumb:** If you can't describe the change in 2-3 sentences, it's too big for ralph-tui.

**Split oversized beads** into vertical slices that each produce a working increment. Use `bd create` to make the new beads and `bd close` + recreate if an existing bead needs splitting.

### Check 2: Quality Gates Appended

Every bead's acceptance criteria must end with the quality gates from the PRD. This is not optional — ralph-tui agents check these criteria to verify their own work.

**Structure per bead:**
```
## Acceptance Criteria
- [ ] [Story-specific criterion 1]
- [ ] [Story-specific criterion 2]
- [ ] pnpm typecheck passes       <- universal gate
- [ ] pnpm lint passes             <- universal gate
- [ ] Verify in browser            <- UI gate (only for UI stories)

When all acceptance criteria are met, output <promise>COMPLETE</promise>
```

> **CRITICAL:** The `<promise>COMPLETE</promise>` signal is how ralph-tui detects that a bead is done. Exit code 0 alone does NOT trigger completion — only this explicit signal does. Every bead description MUST include this instruction, or the agent will finish its work but ralph-tui will not advance to the next bead.

**Check for:**
- Beads missing the `<promise>COMPLETE</promise>` instruction (most critical)
- Beads missing quality gates entirely
- Beads with quality gates but missing story-specific criteria
- UI beads missing browser verification gates
- Quality gates that don't match the PRD's Quality Gates section

### Check 3: Dependency Integrity + Parent Linkage

Two independent graph properties must both be correct for ralph-tui to schedule anything. They look similar but answer different questions: parent linkage answers *"which beads belong to this epic?"* (ralph-tui's `--epic <id>` filter), and dependencies answer *"in what order can they run?"*. Missing parent links and missing deps fail in different ways.

#### Parent linkage (ralph-tui's epic filter)

> **CRITICAL:** `ralph-tui --epic <id>` filters by `parent`, NOT by the dependency graph. Beads that block the epic via `bd dep add <epic> <child>` are invisible to ralph-tui unless they also carry `parent=<epic>`. Symptom: ralph-tui reports "no tasks or epic found" despite a populated `bd ready` queue. This is the #2 cause of failed ralph runs (after missing `<promise>COMPLETE</promise>` signals).

**Verify:**
- The epic itself has `type=epic` (see `bd show <epic-id>` — the header should read `[epic]`). A `feature`/`task`-typed parent silently blocks every child from `bd ready`.
- Every child bead has `parent = <epic-id>` set explicitly.
- `bd show <epic-id>` lists every child under a `CHILDREN` section. If a bead you expect to see is missing, its `--parent` was never set.
- `bd ready` returns at least the schema/leading bead. If it returns "No ready work found" despite open children, suspect (1) wrong epic type and/or (2) a redundant `bd dep add <epic> <child>` creating a cycle with the parent-child relation.

**Check with:** `bd show <epic-id>` and confirm the CHILDREN list matches the intended child set. Then `bd ready --explain` to see why anything is reported as blocked — if a child is "blocked by [the epic]", the epic is mistyped or the cycle exists.

**Fix with:**
- `bd update <epic-id> --type=epic` if the epic is mistyped.
- `bd update <child-id> --parent=<epic-id>` for every child missing the link.
- `bd dep remove <epic-id> <child-id>` for any redundant "epic depends on child" entry that creates a cycle with parent-child.

#### Dependency ordering

ralph-tui respects dependencies strictly within an epic: it will never select a bead whose dependencies are still open. Broken dependencies block the entire pipeline.

**Verify:**
- No circular dependencies
- Correct ordering: schema -> backend -> UI -> integration
- No bead depends on something that should come after it
- No missing dependencies (bead B uses what bead A creates, but no dep link exists)
- No unnecessary dependencies (bead marked blocked when it could start immediately)

**Check with:** `bd list --json` and examine the `dependsOn` arrays. Mentally simulate: "If ralph-tui executes these in dependency order, does each agent have everything it needs?"

**Fix with:** `bd dep add <issue> <depends-on>` or `bd dep remove <issue> <depends-on>`

### Check 4: Description Completeness

Each bead description must give a fresh agent (with zero prior context) everything it needs to execute. ralph-tui includes the bead description in the agent prompt — that description is all the agent gets.

**Must include:**
- What to change and where (specific files, tables, components)
- Why (enough context to make reasonable decisions)
- Acceptance criteria (verifiable, not vague)
- Quality gates (appended)

**Red flags:**
- "Works correctly" — not verifiable
- "Good UX" — not verifiable
- "Handle edge cases" — which ones?
- Missing file paths or component names
- Assumes knowledge from a previous bead (the agent won't have it)

**The key test:** Could an agent with access to the codebase but no other context complete this bead using only its description?

### Check 5: Coverage Against Source Plan

Compare beads against every section, feature, and requirement in the source PRD.

- Walk through the PRD section by section
- For each requirement, confirm at least one bead addresses it
- Flag requirements with no bead as **gaps** -> create new beads
- Flag beads with no corresponding requirement as **orphans** -> investigate

---

## Round-by-Round Emphasis

All five checks run every round, but each round leads with a different lens.

### Round 1: Structural Audit (Checks 1 + 5)
Focus on shape. Are beads the right size for one context window? Are there obvious duplicates? Does the bead set cover the PRD? This round produces the most changes — splits, merges, new beads.

### Round 2: Dependencies + Quality Gates (Checks 2 + 3)
Now that the structure is clean, verify the dependency graph and quality gate completeness. Run `bd list --json` and trace the execution order. Append missing quality gates. This round typically fixes 3-5 dependency links and adds gates to 2-3 beads.

### Round 3: Description Depth (Check 4)
With the right beads identified, invest in making each one self-contained. Rewrite vague criteria. Add file paths, API references, table names. Remember: a fresh agent with no memory will read this description and nothing else.

### Round 4: Adversarial Review (All Checks)
Read every bead as if you are a ralph-tui agent encountering it for the first time with zero context beyond the description. Ask:
- "Could I complete this in one context window?"
- "Do I have enough information to start without asking questions?"
- "Would a different agent interpret this description the same way I do?"
- "If I run the quality gates after implementing, will they actually pass?"

### Rounds 5-6: Only if Needed
Run additional rounds only if convergence is below 75% after round 4.

---

## Convergence Tracking

After each round, record metrics:

```markdown
## Polishing Log

### Round 1
- Beads added: 3
- Beads removed: 1 (merged duplicate)
- Beads modified: 6
- Beads split: 2 (too big for one iteration)
- Dependencies changed: 3
- Quality gates added: 4
- Total changes: 19
- Notes: Split "build dashboard" into 4 slices, merged duplicate schema beads

### Round 2
- Beads added: 1
- Beads modified: 5
- Dependencies changed: 4
- Quality gates added: 2
- Total changes: 12
- Change velocity: -37% (converging)

### Round 3
- Beads modified: 4
- Quality gates added: 0
- Total changes: 4
- Change velocity: -67% (converging)
- Notes: Expanded descriptions with file paths and component names

### Round 4
- Beads modified: 1
- Total changes: 1
- Convergence: ~85%
- Notes: Clarified one ambiguous acceptance criterion
```

### When to Stop

- **~75% convergence (4 rounds):** Good default. Further polishing yields diminishing returns.
- **~90% convergence:** Maximum useful refinement.
- **Change velocity near zero for 2 consecutive rounds:** Stop.
- **Changes increasing:** Something is structurally wrong — step back and check the source PRD.

---

## Applying Changes

After each round, apply changes using the appropriate CLI:

```bash
# Modify a bead description
bd update <bead-id> --description="$(cat <<'EOF'
[updated description with quality gates]
EOF
)"

# Add missing dependency
bd dep add <issue> <depends-on>

# Remove incorrect dependency
bd dep remove <issue> <depends-on>

# Create a new bead (from a split or gap)
bd create --parent=<epic-id> \
  --title="[Story Title]" \
  --description="$(cat <<'EOF'
[description with acceptance criteria + quality gates]
EOF
)" \
  --priority=[1-4]

# Close a duplicate (superseded by another bead)
bd close <bead-id>
```

> **CRITICAL:** Always use `<<'EOF'` (single-quoted) for HEREDOCs. This prevents shell interpretation of backticks, `$variables`, and `()` in descriptions.

---

## Presenting Results

After the final round, present:

1. **Summary:** Rounds run, beads before/after, key structural changes
2. **The polishing log** (all rounds with metrics)
3. **Ralph-readiness assessment:**
   - Are all beads one-context-window sized?
   - Are quality gates on every bead?
   - Is the dependency graph clean?
   - Are all descriptions self-contained?
4. **Remaining concerns:** Anything needing human input

Then suggest:
```
Ready to run: ralph-tui run --tracker beads --epic <epic-id>
```

---

## Anti-Patterns

**Polishing without the source PRD.** You cannot verify coverage without the original requirements.

**Beads that assume prior context.** ralph-tui agents start fresh. If bead 3 says "continue the work from bead 2," the agent will have no idea what bead 2 did. Each description must be self-contained.

**Vague quality gates.** "Tests pass" is not a quality gate. `pnpm typecheck && pnpm lint` is.

**Over-polishing.** If you're on round 7 and still finding issues, the PRD is underspecified. Fix the PRD, regenerate beads, then polish.

**Missing completion signal.** Every bead must end with `When all acceptance criteria are met, output <promise>COMPLETE</promise>`. Without this, ralph-tui cannot detect that the agent finished and the loop stalls. This is the #1 cause of "stuck" ralph-tui runs.

**Missing parent linkage.** Wiring children only via `bd dep add <epic> <child>` is not enough — ralph-tui's `--epic <id>` filter reads the `parent` field, not the dependency graph. Symptom: ralph-tui reports "no tasks or epic found" even though `bd ready` shows work. Always run `bd show <epic-id>` and confirm every intended child appears under CHILDREN before suggesting the ralph-tui command.

**Mistyped epic.** Creating the epic with `--type=feature` (because `bd prime` only lists `task|bug|feature` in its quick-reference) causes bd to treat the parent-child link from each child to the epic as a regular blocking dependency. `bd ready` returns "No ready work found", ralph-tui reports "no more tasks available after starting", and the session sits at iteration 0. Fix: `bd update <epic-id> --type=epic`. The `epic` type is supported by `bd update --type=` even when `bd prime` doesn't advertise it.

**Cyclic epic-child deps.** Adding `bd dep add <epic> <child>` "so the epic closes when children close" creates a cycle: parent-child says child depends on epic, `bd dep add` says epic depends on child. `bd ready --explain` will show every node "blocked by" something in the cycle. Fix: remove the explicit `bd dep add` entries — parent-child + closing children already gives you "epic closes when children close" semantically.

**Skipping the size check.** The most common ralph-tui failure is beads that are too large for one context window. When in doubt, split.

---

## Checklist

Before declaring beads ralph-ready:

- [ ] Ran at least 4 polishing rounds
- [ ] Convergence reached ~75%+
- [ ] Every bead fits one ralph-tui iteration (2-3 sentence scope)
- [ ] Every bead includes `<promise>COMPLETE</promise>` instruction
- [ ] Quality gates appended to every bead
- [ ] UI beads have browser verification gate (if applicable)
- [ ] All acceptance criteria are verifiable (no vague language)
- [ ] Dependency graph has no cycles
- [ ] Dependencies follow schema -> backend -> UI ordering
- [ ] The epic has `type=epic` (verify with `bd show <epic-id>` — header reads `[epic]`)
- [ ] Every child bead has `parent=<epic-id>` set (verify with `bd show <epic-id>` — every expected child appears in the CHILDREN list)
- [ ] No redundant `bd dep add <epic> <child>` entries (would create a cycle with parent-child)
- [ ] `bd ready` returns at least one ready bead (sanity-check the entry point exists)
- [ ] No bead assumes context from a previous iteration
- [ ] Every PRD requirement maps to at least one bead
- [ ] No orphan beads without a clear purpose
- [ ] Epic has external-ref linking back to source PRD
- [ ] Polishing log recorded with metrics per round
- [ ] beads.jsonl verified
