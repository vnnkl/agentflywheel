# Agentic Coding Flywheel Skills Bundle

A collection of Claude Code skills extracted from Jeffrey Emanuel's [Agentic Coding Flywheel](https://agent-flywheel.com/complete-guide) methodology. These skills capture the key workflows from the 9-step process for building software with AI agent swarms.

## Skills

### `/deep-planning` - 85% Planning First
Enforce comprehensive upfront planning before any implementation. Produces a markdown plan covering architecture, data flow, error handling, edge cases, and security. Based on the principle that plan mistakes cost 1x to fix, while code mistakes cost 25x.

### `/competing-plans` - Parallel Plan Synthesis
Spawn 3 parallel agents that independently plan the same feature, then synthesize a "best-of-all-worlds" plan. Each agent has a different persona (Pragmatist, Defender, Architect) that emphasizes different concerns. The synthesis combines the strongest elements from each.

### `/idea-wizard` - Structured Feature Ideation
A divergent-convergent ideation pipeline: generate 30 ideas grounded in the existing codebase, winnow to 5 best, re-expand to 15 refined ideas around the strongest themes. Avoids both tunnel vision and unfocused brainstorming.

### `/bead-polishing` - Iterative Task Refinement
Run 4-6 polishing rounds on a task breakdown until convergence. Each round: deduplicate, verify coverage against the plan, fix dependencies, fill gaps, check for oversimplification. Stop at ~75% convergence. Works with any task format (beads, GitHub issues, PRD stories).

### `/multi-mode-review` - Three-Mode Code Review
Three review modes that catch different bug categories:
- **Fresh Eyes**: Self-review for logic errors, edge cases, off-by-ones
- **Cross-Agent**: Independent agent reviews for integration/boundary issues
- **Random Exploration**: Trace execution flows for architectural drift and systemic bugs

## Recommended Flow

```
/idea-wizard               "What should I build?"
      |
/deep-planning             Comprehensive markdown plan (85% of effort)
      |
/competing-plans           3 parallel plans -> best-of-all-worlds synthesis
      |
/bead-polishing-ralph      Create beads (if none exist) + 4-6 polish rounds
      |                    until ~75% convergence. Enforces ralph-tui constraints:
      |                    one-context-window sizing, quality gates, completion signal.
      |
[implementation]           ralph-tui run --tracker beads-rust
      |
/multi-mode-review         Fresh Eyes + Cross-Agent + Random Exploration
```

> **Note:** `/bead-polishing-ralph` can create beads from the plan if none exist yet,
> or you can use `/ralph-tui-create-beads` first and then polish separately.
> `/bead-polishing` is the generic (non-ralph) variant for GitHub issues or other formats.

## Installation

Copy the skill directories into your Claude Code skills directory:

```bash
cp -r skills/* ~/.claude/skills/
```

Or symlink them:

```bash
for skill in skills/*/; do
  ln -s "$(pwd)/$skill" ~/.claude/skills/$(basename "$skill")
done
```

## Source

Based on [The Complete Flywheel Guide](https://agent-flywheel.com/complete-guide) by Jeffrey Emanuel.

The original methodology covers a 9-step workflow for building software with coordinated agent swarms, including concepts like beads (self-contained work units), Agent Mail (inter-agent communication), and convergence-based refinement. These skills extract the most actionable workflows into Claude Code slash commands.

## License

MIT
