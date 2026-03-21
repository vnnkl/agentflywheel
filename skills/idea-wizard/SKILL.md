---
name: idea-wizard
description: Structured divergent-convergent ideation pipeline for generating feature ideas for existing projects. Follows a specific 30 → 5 → 15 expansion/contraction pattern grounded in the current codebase to avoid both tunnel vision and unfocused brainstorming. Use this skill whenever the user wants to brainstorm features, generate ideas for a project, come up with new functionality, explore what to build next, think about improvements, wants a feature ideation session, mentions "idea wizard", asks "what should I build", says "feature ideas", or wants to figure out what to add to an existing codebase. Also use when user mentions wanting to adapt ideas from an external project or system into their own.
---

# Idea-Wizard

A structured ideation pipeline that alternates between divergent thinking (generate many ideas) and convergent thinking (select the strongest) to produce high-quality feature ideas grounded in an existing codebase.

## Why This Structure Matters

Unstructured brainstorming produces two failure modes:

1. **Tunnel vision** — jumping to the first decent idea and elaborating it without exploring alternatives. The result is a local optimum: a perfectly refined version of an idea that was never the best option.

2. **Unfocused sprawl** — generating dozens of ideas with no winnowing. Everything stays at surface level. Nothing gets developed enough to evaluate seriously.

The expansion/contraction pattern solves both problems. First you go wide (30 ideas) to escape tunnel vision. Then you go narrow (5 best) to identify the strongest themes. Then you re-expand around those themes (15 ideas) to explore the most promising directions at higher resolution. The human picks from a curated set that has been both broadly explored and deeply developed.

The specific ratios (30 → 5 → 15) are not arbitrary. 30 forces you past the obvious ideas that come first — the genuinely novel ones tend to appear after idea 15-20 when the easy options are exhausted. Winnowing to 5 creates real selection pressure and surfaces the underlying themes worth pursuing. Re-expanding to 15 around those 5 themes gives 3 variations per theme on average, which is enough to find the specific angle that fits the project best.

## Process

### Step 0: Understand the Project

Before generating ideas, build a mental model of the project. Read:

- Project root files (package.json, Cargo.toml, go.mod, etc.) to understand the stack
- README or docs for stated purpose and goals
- Directory structure to understand architecture
- Any existing task/issue files, TODO lists, or roadmap documents
- AGENTS.md or CLAUDE.md if present

Summarize the project back to the user in 3-5 sentences: what it does, who it serves, what stack it uses, and its current state (early prototype, production app, side project, etc.). Ask the user to correct anything wrong before proceeding.

This grounding step prevents the most common ideation failure: generating ideas that duplicate existing functionality, contradict architectural decisions, or don't fit the project's scope. You cannot generate good feature ideas for a codebase you do not understand.

### Step 1: Diverge — Generate 30 Ideas

Generate 30 feature ideas. Present them as a numbered list with one-line descriptions. Prioritize quantity and range over polish.

Guidelines for idea generation:

- **Cover different dimensions**: new user-facing features, developer experience improvements, performance optimizations, integrations, data/analytics capabilities, accessibility, security hardening, UX refinements
- **Vary in scope**: mix quick wins (hours) with medium efforts (days) and ambitious features (weeks)
- **Push past the obvious**: the first 10-12 ideas that come to mind are the ones everyone would think of. The value lives in ideas 15-30 where you exhaust the obvious and reach genuinely novel territory
- **Stay grounded**: every idea must be technically feasible given the current stack and architecture. No ideas that require rewriting the project from scratch
- **No duplicates of existing functionality**: reference Step 0 — if the project already has it, do not suggest it again

Format:

```
1. [Short title] — [One sentence explaining what it does and why it matters]
2. [Short title] — [One sentence]
...
30. [Short title] — [One sentence]
```

Do NOT elaborate on ideas at this stage. One line each. Resist the urge to explain.

### Step 2: Converge — Winnow to 5

Select the 5 strongest ideas from the 30. For each, explain in 2-3 sentences:

- Why this idea is strong (user value, technical leverage, strategic fit)
- What makes it feasible given the current codebase
- How it connects to or amplifies existing functionality

Present the 5 selections and your reasoning. Explicitly name the themes or patterns you see across the top 5 — these themes guide the next expansion.

The winnowing criteria, in priority order:

1. **User impact** — Does this solve a real problem or create genuine delight?
2. **Architectural fit** — Does this work with the current codebase, or does it fight it?
3. **Leverage** — Does this create compounding value (enables future features, attracts users, generates data)?
4. **Distinctiveness** — Is this something that differentiates the project, or is it table-stakes that every competitor already has?
5. **Feasibility** — Can this be built without heroic effort or deep domain expertise the team lacks?

### Step 3: Re-Diverge — Expand to 15

Take the 5 selected ideas and the themes identified in Step 2. Generate 15 refined ideas — roughly 3 variations per theme, though the distribution can be uneven if some themes are richer than others.

These 15 ideas are more developed than the original 30. For each, provide:

- **Title**
- **Description** (3-5 sentences): what it does, how it works, why it matters
- **Scope estimate**: Small (hours), Medium (days), Large (weeks)
- **Dependencies**: what existing code/infrastructure it builds on
- **Risk**: the single biggest risk or unknown

This re-expansion is where the real value emerges. You are no longer brainstorming from scratch — you are exploring the most promising territory identified through the previous two steps. Each idea here benefits from both the breadth of Step 1 and the focused selection of Step 2.

### Step 4: Present for Human Review

Present the 15 ideas grouped by theme. Ask the user:

> "Which of these ideas interest you? Pick any number — you can select individual ideas, entire themes, or ask me to develop any idea further before deciding."

Wait for the user's response. Do not proceed until they select.

### Step 5: Convert to Tasks

For each idea the user selects, produce a structured task specification:

```
## [Idea Title]

**Goal**: [One sentence — what success looks like]

**User story**: As a [user type], I want [capability] so that [benefit].

**Scope**: [Small / Medium / Large]

**Key requirements**:
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

**Technical approach**:
- [How this integrates with existing code]
- [Key files/modules affected]
- [New files/modules needed]

**Acceptance criteria**:
- [ ] [Testable criterion 1]
- [ ] [Testable criterion 2]
- [ ] [Testable criterion 3]

**Open questions**:
- [Anything that needs human decision before implementation]
```

After presenting all task specs, ask the user if they want to refine any of them, adjust scope, or add/remove requirements.

### Step 6: Save Output

Save the final task specifications to a file the user specifies. If no path is given, suggest saving to `.claude/plans/idea-wizard-<date>.md` or a project-appropriate location.

---

## Variant: Research-Driven Reimagining

When the user wants to adapt ideas from an external project or system, follow this modified pipeline.

This applies when the user says something like "I want to bring ideas from X into my project" or "look at how Y works and adapt that for us" or references an external tool, library, or product.

### Research Phase

1. Study the external system. Use WebSearch/WebFetch to understand its architecture, core features, and design philosophy. If the user provides docs or code, read those instead.

2. Extract the 5-8 most interesting architectural ideas or capabilities from the external system. Focus on concepts, not implementation details — the goal is to understand *what problems it solves and how it thinks about them*, not to copy its code.

3. Present these extracted ideas to the user and ask which ones resonate.

### Adaptation Phase

4. For each selected idea, generate 3-5 adaptations that reinterpret the concept through your project's unique capabilities, stack, and user base. The best adaptations are not ports — they are ideas that could only exist in your project.

5. Present the adaptations and let the user pick. Then proceed to Step 5 (Convert to Tasks) from the main pipeline.

---

## Interaction Style

- Be direct. Present ideas without hedging or excessive caveats.
- Be opinionated during winnowing. The user wants a strong read on which ideas are best, not a diplomatic menu where everything is equally good.
- Ask questions when you lack context that would change your recommendations. Do not guess at business constraints or user demographics.
- Keep the momentum. Each step should flow into the next without requiring the user to prompt you to continue — except at Step 4 where you must wait for human selection.
