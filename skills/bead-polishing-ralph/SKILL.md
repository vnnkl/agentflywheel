---
name: bead-polishing-ralph
description: "Iteratively refine beads for ralph-tui execution through 4-6 polishing rounds until convergence, ensuring each bead fits one agent context window, has quality gates appended, uses correct bd/br CLI syntax, and follows schema-backend-UI dependency ordering. Use when user wants to polish beads for ralph, refine ralph tasks, iterate on beads before running ralph-tui, review bead quality for ralph execution, or check if beads are ralph-ready. Triggers on: polish beads ralph, refine beads for ralph, ralph bead review, check beads, are beads ready, polish for ralph-tui, iterate ralph beads."
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

Check which beads CLI is available:

```bash
which br && echo "beads-rust" || (which bd && echo "beads-go" || echo "none")
```

Use `br` commands if beads-rust is installed, `bd` commands if beads-go. The polishing process is identical — only the CLI prefix changes.

### Gather Context

1. **Read the source plan/PRD** that the beads were created from. Polishing without the source is guessing.
2. **List existing beads**: `br list --json` or `bd list --json`
3. **Identify the epic**: Which epic are we polishing?
4. **Extract Quality Gates** from the PRD's "Quality Gates" section. If none exists, ask the user what commands should pass (e.g., `pnpm typecheck`, `pnpm lint`).

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

**Split oversized beads** into vertical slices that each produce a working increment. Use `br create` to make the new beads and `br close` + recreate if an existing bead needs splitting.

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
```

**Check for:**
- Beads missing quality gates entirely
- Beads with quality gates but missing story-specific criteria
- UI beads missing browser verification gates
- Quality gates that don't match the PRD's Quality Gates section

### Check 3: Dependency Integrity

ralph-tui respects dependencies strictly: it will never select a bead whose dependencies are still open. Broken dependencies block the entire pipeline.

**Verify:**
- No circular dependencies
- Correct ordering: schema -> backend -> UI -> integration
- No bead depends on something that should come after it
- No missing dependencies (bead B uses what bead A creates, but no dep link exists)
- No unnecessary dependencies (bead marked blocked when it could start immediately)

**Check with:** `br list --json` and examine the `dependsOn` arrays. Mentally simulate: "If ralph-tui executes these in dependency order, does each agent have everything it needs?"

**Fix with:** `br dep add <issue> <depends-on>` or `br dep remove <issue> <depends-on>`

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
Now that the structure is clean, verify the dependency graph and quality gate completeness. Run `br list --json` and trace the execution order. Append missing quality gates. This round typically fixes 3-5 dependency links and adds gates to 2-3 beads.

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
# Modify a bead description (beads-rust)
br update <bead-id> --description="$(cat <<'EOF'
[updated description with quality gates]
EOF
)"

# Add missing dependency
br dep add <issue> <depends-on>

# Remove incorrect dependency
br dep remove <issue> <depends-on>

# Create a new bead (from a split or gap)
br create --parent=<epic-id> \
  --title="[Story Title]" \
  --description="$(cat <<'EOF'
[description with acceptance criteria + quality gates]
EOF
)" \
  --priority=[1-4]

# Close a duplicate (superseded by another bead)
br close <bead-id>

# Sync to JSONL for git tracking (beads-rust only)
br sync --flush-only
```

For beads-go (`bd`), replace `br` with `bd` and skip `br sync --flush-only`.

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
Ready to run: ralph-tui run --tracker beads-rust --epic <epic-id>
```

---

## Anti-Patterns

**Polishing without the source PRD.** You cannot verify coverage without the original requirements.

**Beads that assume prior context.** ralph-tui agents start fresh. If bead 3 says "continue the work from bead 2," the agent will have no idea what bead 2 did. Each description must be self-contained.

**Vague quality gates.** "Tests pass" is not a quality gate. `pnpm typecheck && pnpm lint` is.

**Over-polishing.** If you're on round 7 and still finding issues, the PRD is underspecified. Fix the PRD, regenerate beads, then polish.

**Skipping the size check.** The most common ralph-tui failure is beads that are too large for one context window. When in doubt, split.

---

## Checklist

Before declaring beads ralph-ready:

- [ ] Ran at least 4 polishing rounds
- [ ] Convergence reached ~75%+
- [ ] Every bead fits one ralph-tui iteration (2-3 sentence scope)
- [ ] Quality gates appended to every bead
- [ ] UI beads have browser verification gate (if applicable)
- [ ] All acceptance criteria are verifiable (no vague language)
- [ ] Dependency graph has no cycles
- [ ] Dependencies follow schema -> backend -> UI ordering
- [ ] No bead assumes context from a previous iteration
- [ ] Every PRD requirement maps to at least one bead
- [ ] No orphan beads without a clear purpose
- [ ] Epic has external-ref linking back to source PRD
- [ ] Polishing log recorded with metrics per round
- [ ] `br sync --flush-only` run (beads-rust) or beads.jsonl verified (beads-go)
