---
name: ultra:research-swarm
description: Spawn 4-perspective research swarm (Pattern Analyst, Domain Expert, Risk Analyst, UX Investigator) for a phase
argument-hint: "<phase-number>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - WebSearch
  - WebFetch
---
<objective>
Run a 4-perspective research swarm for a phase. Spawns Pattern Analyst, Domain Expert, Risk Analyst, and UX Investigator in parallel. Synthesizes findings into RESEARCH.md.

Context budget: ~20% orchestrator, 100% fresh per researcher agent.
</objective>

<execution_context>
@get-shit-done/workflows/research-swarm.md
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
Execute the research-swarm workflow from @get-shit-done/workflows/research-swarm.md end-to-end.

1. Initialize with gsd-tools.js
2. Spawn 4 researcher agents in parallel
3. Verify all 4 perspective files written
4. Synthesize into single RESEARCH.md
5. Commit and present results

Preserve all workflow gates (parallel spawning, synthesis, commit).
</process>
