---
name: ultra:full-pipeline
description: Chain all Ultra stages — research-swarm → adversarial-plan → parallel-execute → adversarial-verify → gap-close
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
  - WebSearch
  - WebFetch
---
<objective>
Execute the full Ultra pipeline for a phase end-to-end. Chains all 5 stages with checkpoints between them. Each stage runs in a fresh context window via subagent.

Pipeline: Research Swarm → Adversarial Plan → Parallel Execute → Adversarial Verify → Gap Close (if needed)

Context budget: ~10% orchestrator per stage, 100% fresh per stage agent.
</objective>

<context>
Phase: $ARGUMENTS

@.planning/ROADMAP.md
@.planning/STATE.md
@CLAUDE.md
@DOMAINS.md
@gsd-ultra.json
</context>

<process>

## Coordination Mode Detection

Before starting the pipeline, detect whether Agent Teams is available:

```bash
AGENT_TEAMS_ENV=$(echo "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-0}")
if [ "$AGENT_TEAMS_ENV" = "1" ]; then
  AGENT_TEAMS_MODE=true
else
  AGENT_TEAMS_MODE=false
fi
```

Display:
```
◆ Coordination: Agent Teams (native)    ← if enabled
◆ Coordination: Task() subagents        ← if fallback
```

The mode is detected once here and passed through to each stage. Individual workflows also detect it independently (idempotent), so stages run correctly whether invoked from the pipeline or standalone.

## Pipeline Overview

```
┌─────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│ Research Swarm   │────▶│ Adversarial Plan  │────▶│ Parallel Execute │
│ (4 perspectives) │     │ (Builder/Critic)  │     │ (domain-owned)   │
└─────────────────┘     └──────────────────┘     └──────────────────┘
                                                          │
                         ┌──────────────────┐             │
                         │ Gap Close        │◀────────────┤
                         │ (if needed)      │     ┌──────────────────┐
                         └──────────────────┘◀────│ Adversarial      │
                                                  │ Verify (3-role)  │
                                                  └──────────────────┘
```

## Stage 1: Research Swarm

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA PIPELINE ► STAGE 1/5: RESEARCH SWARM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Execute research-swarm workflow. Pass phase argument.

**Checkpoint:** Verify RESEARCH.md exists before proceeding.

## Stage 2: Adversarial Plan

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA PIPELINE ► STAGE 2/5: ADVERSARIAL PLAN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Execute adversarial-plan workflow. Pass phase argument.

**Checkpoint:** Verify PLAN.md files exist and were approved before proceeding.

## Stage 3: Parallel Execute

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA PIPELINE ► STAGE 3/5: PARALLEL EXECUTE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Execute parallel-execute (enhanced execute-phase) workflow. Pass phase argument.

**Checkpoint:** Verify SUMMARY.md files exist and spot-check commits before proceeding.

## Stage 4: Adversarial Verify

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA PIPELINE ► STAGE 4/5: ADVERSARIAL VERIFY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Execute adversarial-verify workflow. Pass phase argument.

**Checkpoint:** Read verdict. If PASS → done. If CONDITIONAL_PASS → ask user. If FAIL → stage 5.

## Stage 5: Gap Close (Conditional)

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA PIPELINE ► STAGE 5/5: GAP CLOSE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Only runs if verdict is FAIL or CONDITIONAL_PASS (and user approves).

Execute gap-close workflow. Pass phase argument.

After gap-close, re-run adversarial-verify to confirm fix:
```
◆ Re-verifying after gap closure...
```

## Pipeline Complete

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA PIPELINE ► COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Phase {X}: {Name} — Full pipeline executed

| Stage | Status | Duration |
|-------|--------|----------|
| Research Swarm | ✓ 4 perspectives | {time} |
| Adversarial Plan | ✓ Approved R{N} | {time} |
| Parallel Execute | ✓ {N} plans | {time} |
| Adversarial Verify | ✓ {VERDICT} | {time} |
| Gap Close | {✓ closed / — skipped} | {time} |

Total Duration: {total}
```

## Error Handling

If any stage fails:
1. Report which stage failed and why
2. Offer options: retry stage, skip to next, abort pipeline
3. If user skips: continue but note in final report
4. Partial completion is tracked — pipeline can be resumed from the failed stage

</process>
