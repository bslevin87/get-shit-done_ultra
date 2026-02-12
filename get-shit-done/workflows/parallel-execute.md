<purpose>
Execute plans using domain-owned parallel execution with deferred integration and two-layer parallelism. Teammates build domains independently, lead integrates serially after all complete.
</purpose>

<context_budget>
Lead stays under 50% context. Enforced by:
1. **Context-by-reference** — ownership maps and plan refs passed as file paths, not inline content
2. **Spawn brevity** — executor prompts are ownership block + plan path + protocol ref
3. **Protocols by reference** — file ownership template, hub-spoke, scaling guidance live in `ultra-conventions.md`
</context_budget>

<process>

<step name="initialize" priority="first">
Load all context:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init execute-phase "${PHASE_ARG}")
```

Parse JSON for: `phase_dir`, `phase_number`, `phase_name`, `executor_model`, `plans`.

Read DOMAINS.md and gsd-ultra.json for file ownership mappings.

```bash
ULTRA_CONFIG=$(cat gsd-ultra.json 2>/dev/null || echo '{}')
DOMAINS=$(cat DOMAINS.md 2>/dev/null || echo '')
```
</step>

<step name="detect_agent_teams" priority="after-init">
```bash
AGENT_TEAMS_ENV=$(echo "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-0}")
AGENT_TEAMS_MODE=$( [ "$AGENT_TEAMS_ENV" = "1" ] && echo true || echo false )
```

Display: `Coordination: {Agent Teams (native) | Task() subagents}`

Both paths produce identical SUMMARY.md files and deferred integration.
</step>

<step name="discover_and_group_plans">
Discover plans and group by wave:

```bash
PLAN_INDEX=$(node ~/.claude/get-shit-done/bin/gsd-tools.js phase-plan-index "${PHASE_ARG}")
```

Parse into wave groups. Plans in the same wave execute in parallel; waves execute sequentially.

Check for `--gaps-only` flag: if set, filter to plans with `gap_closure: true`.
</step>

<step name="build_ownership_map">
For each plan, determine file ownership from DOMAINS.md + gsd-ultra.json:

```
Plan 01 (Domain: auth):
  OWN: src/features/auth/
  SHARED: src/types/auth.ts
  DO NOT TOUCH: src/features/tasks/, src/features/dashboard/
```

If DOMAINS.md doesn't exist, derive ownership from plan `files_modified` frontmatter.

Build ownership block per plan using template from `ultra-conventions.md` (File Ownership Spawn Template).
</step>

<step name="execute_waves">
Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA > PARALLEL EXECUTE — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Wave 1: {N} teammates in parallel
  > {Domain A} ({model}) — OWN: {path}
  > {Domain B} ({model}) — OWN: {path}
Coordination: {Agent Teams (native) | Task() subagents}
```

**Executor spawn spec** (per plan in wave):

| Field | Value |
|-------|-------|
| Agent Def | gsd-executor |
| Plan Path | {phase_dir}/{plan_id}-PLAN.md |
| Summary Path | {phase_dir}/{plan_id}-SUMMARY.md |
| Ownership | From build_ownership_map step |
| Model | {EXECUTOR_MODEL} |

**Prompt template:**
```
Read the gsd-executor agent definition.
Execute plan at: {plan_path}
Phase {phase}: {phase_name}.
{file_ownership_block}
Write summary to: {summary_path}
LAYER 2: You may spawn subagents within YOUR owned files.
Cross-domain discoveries: report in summary (see Hub-Spoke in ultra-conventions.md).
```

**If AGENT_TEAMS_MODE=true:** Lead creates shared task list from plans, spawns domain Teammate()s who self-claim. Hub-spoke via messaging. Lead exits delegate mode after all idle.

**If AGENT_TEAMS_MODE=false:** Task(subagent_type="gsd-executor") per plan. All wave plans spawn in parallel. Wait for wave completion before next wave.
</step>

<step name="deferred_integration">
After all domain waves complete:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA > INTEGRATION PHASE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Integrating {N} domains serially...
```

The lead performs integration serially:

1. **Conflict Check** — verify no files modified by multiple teammates
2. **Cross-Domain Wiring** — connect domains through shared types, routes, imports
3. **Build Verify** — `npm run build 2>/dev/null || npx tsc --noEmit 2>/dev/null`
4. **Cross-Domain Discovery** — process discoveries from summaries (see Hub-Spoke in `ultra-conventions.md`)
5. **Integration Commit** — commit integration wiring separately

If integration is complex, spawn a dedicated integration executor:
```
Task(
  prompt="Wire together completed domains: {list}
  Connect shared state, routing, imports. Do NOT modify domain-internal code.",
  subagent_type="gsd-executor",
  description="Integrate domains Phase {phase}"
)
```
</step>

<step name="aggregate_results">
Verify all summaries:

```bash
for plan in $PLANS; do
  [ -f "${PHASE_DIR}/${plan}-SUMMARY.md" ] && echo "ok ${plan}" || echo "MISSING ${plan}"
done
```
</step>

</process>

<offer_next>
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA > PARALLEL EXECUTE COMPLETE

Phase {X}: {Name}
Waves: {N} | Plans: {M} | Integration: done
Commits: {N} domain + {M} integration

 > Next: /ultra:adversarial-verify {X}
   /clear first for fresh context
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</offer_next>
