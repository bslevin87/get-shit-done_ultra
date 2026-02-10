---
name: ultra:retrospective
description: Knowledge flywheel — extract lessons, conventions, and decisions from a completed phase
argument-hint: "<phase-number>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - AskUserQuestion
---
<objective>
Run the knowledge flywheel for a completed phase. Reads verification and summary artifacts, creates a RETROSPECTIVE.md, extracts conventions for CLAUDE.md, logs decisions to STATE.md, and updates DOMAINS.md if domain boundaries shifted.

This is Ultra's compound learning mechanism. Each retrospective makes the next phase faster and higher quality.

Context budget: ~25% orchestrator (reads artifacts, asks user to approve convention additions).
</objective>

<execution_context>
@get-shit-done/workflows/retrospective.md
</execution_context>

<context>
Phase: $ARGUMENTS

@.planning/ROADMAP.md
@.planning/STATE.md
@CLAUDE.md
@DOMAINS.md
@gsd-ultra.json
</context>

<process>
Execute the retrospective workflow from @get-shit-done/workflows/retrospective.md end-to-end.

1. Initialize and load all phase artifacts (VERIFICATION.md, SUMMARY.md files, PLAN.md files)
2. Generate RETROSPECTIVE.md with structured sections
3. **Skill Extraction** — analyze completed code for patterns, suggest 2-3 CLAUDE.md convention additions
4. **Decision Logging** — append DEC-{NNN} entries to STATE.md for architectural decisions made
5. **Domain Review** — update DOMAINS.md if domain boundaries shifted during execution
6. Commit all artifacts
7. Present flywheel metrics (how much faster research/verification should be next phase)

The user approves each CLAUDE.md convention before it's added. This is not auto-pilot — it's collaborative learning.
</process>
