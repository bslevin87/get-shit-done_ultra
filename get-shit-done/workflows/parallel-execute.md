<purpose>
Execute plans using domain-owned parallel execution with deferred integration and two-layer parallelism.

This workflow replaces GSD's standard execute-phase with Ultra's enhanced model:
- **Deferred Integration:** Teammates build domains independently → lead integrates serially after all complete
- **Two-Layer Parallelism:** Layer 1 = teammates work domains in parallel; Layer 2 = teammates spawn subagents within their domain
- **Hub-Spoke Communication:** All cross-domain discovery goes through lead, not peer-to-peer
</purpose>

<core_principle>
Parallel execution fails when agents step on each other's files. Ultra prevents this with three mechanisms:

1. **File Ownership** — Each agent is told exactly which files it owns (OWN), can append to (SHARED), can read (READ-ONLY), and must not touch (DO NOT TOUCH)
2. **Deferred Integration** — Cross-domain wiring happens serially after all domains complete, eliminating merge conflicts
3. **Hub-Spoke** — Lead handles all cross-domain coordination, preventing the N*(N-1)/2 communication explosion of peer-to-peer

The result: teammates work at full speed without coordination overhead, and integration is clean.
</core_principle>

<spawn_template>
Every teammate executor prompt MUST include this ownership block:

```markdown
<file_ownership>
**Domain:** {domain_name}

YOU OWN (full read/write):
- {owned_path_1}/
- {owned_path_2}/

SHARED (append-only — add new items, don't modify existing):
- {shared_path_1}
- {shared_path_2}

READ-ONLY (import and reference, don't modify):
- {readonly_path_1}/
- {readonly_path_2}/

DO NOT TOUCH (off-limits — other domain directories):
- {forbidden_path_1}/
- {forbidden_path_2}/

CRITICAL: If you need to modify a file outside your OWN or SHARED paths,
STOP and report it as a deviation. Do NOT modify it.

CROSS-DOMAIN DISCOVERY: If you discover something that affects another domain,
report it in your summary. The lead will handle cross-domain coordination.
</file_ownership>
```
</spawn_template>

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

<step name="discover_and_group_plans">
Discover plans and group by wave (same as GSD):

```bash
PLAN_INDEX=$(node ~/.claude/get-shit-done/bin/gsd-tools.js phase-plan-index "${PHASE_ARG}")
```

Parse into wave groups. Plans in the same wave execute in parallel; waves execute sequentially.

Check for `--gaps-only` flag: if set, filter to plans with `gap_closure: true`.
</step>

<step name="build_ownership_map">
For each plan, determine file ownership from DOMAINS.md + gsd-ultra.json:

```markdown
Plan 01 (Domain: auth):
  OWN: src/features/auth/
  SHARED: src/types/auth.ts
  READ-ONLY: src/components/, src/lib/
  DO NOT TOUCH: src/features/tasks/, src/features/dashboard/

Plan 02 (Domain: tasks):
  OWN: src/features/tasks/
  SHARED: src/types/task.ts
  READ-ONLY: src/components/, src/lib/
  DO NOT TOUCH: src/features/auth/, src/features/dashboard/
```

If DOMAINS.md doesn't exist, derive ownership from plan `files_modified` frontmatter. Each plan owns the directories of its listed files.
</step>

<step name="execute_waves">
Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► PARALLEL EXECUTE — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Wave 1: {N} teammates in parallel
  → {Domain A} ({model}) — OWN: {path}
  → {Domain B} ({model}) — OWN: {path}
```

For each wave, spawn all plan executors in parallel (Layer 1):

```
Task(
  prompt="First, read the gsd-executor agent definition.
  Execute plan {plan_id} for Phase {phase}: {phase_name}.

  {file_ownership_block}

  Read the plan at: {plan_path}
  Write summary to: {summary_path}

  LAYER 2: If a task is complex, you may spawn subagents (Task tool)
  for subtasks within YOUR owned files. Do not spawn subagents that
  modify files outside your ownership.",
  subagent_type="gsd-executor",
  model="{EXECUTOR_MODEL}",
  description="Execute {domain} Phase {phase}"
)
```

Wait for all teammates in the wave to complete before starting the next wave.
</step>

<step name="deferred_integration">
After all domain waves complete:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► INTEGRATION PHASE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Integrating {N} domains serially...
```

The lead (orchestrator) performs integration serially:

1. **Conflict Check** — verify no files modified by multiple teammates
   ```bash
   # Extract files from all summaries, check for duplicates
   ```
2. **Cross-Domain Wiring** — connect domains through shared types, routes, imports
3. **Build Verify** — ensure everything compiles after integration
   ```bash
   npm run build 2>/dev/null || npx tsc --noEmit 2>/dev/null
   ```
4. **Cross-Domain Discovery** — process any discoveries teammates reported in summaries
5. **Integration Commit** — commit integration wiring separately

If integration is complex (many cross-domain connections), spawn a dedicated integration executor:
```
Task(
  prompt="Wire together the following completed domains:
  {list of domains with their outputs}
  {interface contracts from PLAN.md}
  Connect shared state, routing, imports.
  Do NOT modify domain-internal code.",
  subagent_type="gsd-executor",
  description="Integrate domains Phase {phase}"
)
```
</step>

<step name="handle_hub_spoke">
**Hub-Spoke Communication Protocol:**

During execution, teammates may report cross-domain discoveries in their summaries:

```markdown
## Cross-Domain Discoveries
- Domain B needs a `UserProfile` type that doesn't exist yet
- The API endpoint Domain A created returns paginated data, not a flat array
```

The lead processes these after the wave completes:
1. If it affects the next wave → inject into next wave's prompts
2. If it requires backfilling → create a mini integration task
3. If it's informational → note in the integration summary
</step>

<step name="aggregate_results">
Collect all summaries and verify:

```bash
for plan in $PLANS; do
  [ -f "${PHASE_DIR}/${plan}-SUMMARY.md" ] && echo "✓ ${plan}" || echo "✗ ${plan} MISSING"
done
```

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► PARALLEL EXECUTE COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Phase {X}: {Name}

| Wave | Plans | Domain | Status |
|------|-------|--------|--------|
| 1 | 01, 02 | auth, tasks | ✓ complete |
| 2 | 03 | dashboard | ✓ complete |
| INT | — | integration | ✓ wired |

Commits: {N} domain + {M} integration
Files: {count} created/modified

───────────────────────────────────────────────────────

## ▶ Next Up

**Adversarial Verification** — Defender/Attacker/Auditor triple-check

/ultra:adversarial-verify {X}

<sub>/clear first → fresh context window</sub>
```
</step>

</process>

<scaling_guidance>
**Team sizes:**

| Phase Complexity | Teammates | Notes |
|-----------------|-----------|-------|
| Small (1-2 plans) | 1-2 | Standard GSD execution, no integration needed |
| Medium (3-4 plans) | 3-4 | Sweet spot, manageable integration |
| Large (5-6 plans) | 4-5 | Maximum recommended; interface contracts required |
| Very Large (7+) | Split phase | Break into sub-phases; 7+ teammates = coordination overhead exceeds gains |

**Coordination overhead curve:**
- 2 teammates: ~5% overhead (near-linear speedup)
- 4 teammates: ~15% overhead (still strong gains)
- 6 teammates: ~30% overhead (diminishing returns)
- 8+ teammates: >50% overhead (net negative, split the phase)
</scaling_guidance>

<success_criteria>
- [ ] All plans discovered and grouped by wave
- [ ] File ownership mapped for every plan
- [ ] Teammates spawned with ownership blocks in prompts
- [ ] Layer 1 parallelism: all wave plans run simultaneously
- [ ] Layer 2 enabled: teammates can spawn subagents within their domain
- [ ] Deferred integration: lead wires domains after all complete
- [ ] Hub-spoke: cross-domain discoveries processed by lead
- [ ] All summaries collected
- [ ] Integration verified (builds, no conflicts)
- [ ] User knows next step (adversarial-verify)
</success_criteria>
