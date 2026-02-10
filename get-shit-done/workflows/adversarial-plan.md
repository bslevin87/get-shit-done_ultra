<purpose>
Execute a Builder/Critic adversarial debate loop to create high-quality phase plans. The Builder creates plans, the Critic reviews them, and the loop continues until the Critic signals APPROVED (zero BLOCKERs) or max rounds (5) is reached.

Replaces GSD's planner → plan-checker → revision loop with a more rigorous adversarial debate where both roles are specialized for their purpose.
</purpose>

<core_principle>
Plans improve through adversarial pressure. The Builder creates the best plan they can, the Critic finds real problems, and the Builder revises. This produces plans that survive scrutiny before execution begins, catching issues that cost 10x to fix during execution.

Convergence = zero BLOCKERs. Warnings and suggestions are documented but don't block.
</core_principle>

<process>

<step name="initialize" priority="first">
Load all context:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init plan-phase "${PHASE_ARG}" --include state,roadmap,requirements,context,research)
```

Parse JSON for: `phase_dir`, `phase_number`, `phase_name`, `padded_phase`, `has_research`, `has_context`.

Resolve models for Builder and Critic:
```bash
ULTRA_CONFIG=$(cat gsd-ultra.json 2>/dev/null || echo '{}')
MODEL_PROFILE=$(cat .planning/config.json 2>/dev/null | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_profile || 'balanced'")

BUILDER_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.builder['$MODEL_PROFILE']")
CRITIC_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.critic['$MODEL_PROFILE']")
```

Fallback: both use `gsd-planner` model from gsd-tools.js.

Read file contents from INIT --include:
```bash
STATE_CONTENT, ROADMAP_CONTENT, REQUIREMENTS_CONTENT, CONTEXT_CONTENT, RESEARCH_CONTENT
```

Set MAX_ROUNDS=5, current_round=1.
</step>

<step name="detect_agent_teams" priority="after-init">
Check if Agent Teams is available for native multi-agent coordination:

1. Check env: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`
   ```bash
   AGENT_TEAMS_ENV=$(echo "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-0}")
   ```
2. Verify Claude Code supports Agent Teams (Opus 4.6+ with experimental flag)

Set coordination mode:
- `AGENT_TEAMS_MODE=true` → Builder/Critic debate via native teammate messaging
- `AGENT_TEAMS_MODE=false` → Sequential Task() subagent calls (existing behavior, default)

```bash
if [ "$AGENT_TEAMS_ENV" = "1" ]; then
  AGENT_TEAMS_MODE=true
else
  AGENT_TEAMS_MODE=false
fi
```

Display:
```
◆ Coordination: Agent Teams (native)    ← if AGENT_TEAMS_MODE=true
◆ Coordination: Task() subagents        ← if AGENT_TEAMS_MODE=false
```

**Both paths produce identical outputs** — same PLAN.md files, same approval flow, same convergence logic. Only the debate transport differs.
</step>

<step name="check_research">
If RESEARCH.md missing and research is enabled:
```
⚠ No RESEARCH.md found. Run /ultra:research-swarm {phase} first for best results.
   Proceeding without research...
```
</step>

<step name="build_context">
Construct shared planning context:

```markdown
<planning_context>
**Phase:** {phase_number} - {phase_name}
**Goal:** {phase goal from roadmap}

**Project State:** {state_content}
**Roadmap:** {roadmap_content}
**Requirements:** {requirements_content}

**User Decisions (CONTEXT.md):**
{context_content}

**Research (4-perspective):**
{research_content}

**Domains:** Read DOMAINS.md for domain decomposition and file ownership.
**Ultra Config:** Read gsd-ultra.json for pipeline configuration.
</planning_context>
```
</step>

<step name="debate_loop">
Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► ADVERSARIAL PLANNING — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Coordination: {Agent Teams (native) | Task() subagents}
```

### If AGENT_TEAMS_MODE=true — Native Teammate Debate

Lead spawns Builder and Critic as **persistent teammates**:

```
Teammate(
  name="builder",
  prompt="First, read the ultra-builder agent definition.
  You are the Builder for Phase {phase}: {phase_name}.
  {planning_context}
  Read CLAUDE.md, DOMAINS.md, and gsd-ultra.json from the project root.

  DEBATE PROTOCOL (Agent Teams mode):
  1. Create plans and write PLAN.md files to: {phase_dir}/
  2. Message the 'critic' teammate when plans are ready
  3. Critic will message you back with BLOCKER/WARNING/SUGGESTION findings
  4. Address all BLOCKERs, revise plans, message Critic again
  5. Continue until Critic messages 'APPROVED' or lead intervenes at max rounds

  Write PLAN.md files to: {phase_dir}/",
  model="{BUILDER_MODEL}"
)

Teammate(
  name="critic",
  prompt="First, read the ultra-critic agent definition.
  You are the Critic for Phase {phase}: {phase_name}.
  {planning_context}
  Read CLAUDE.md, DOMAINS.md, and gsd-ultra.json from the project root.

  DEBATE PROTOCOL (Agent Teams mode):
  1. Wait for Builder to message you with plans
  2. Read PLAN.md files from: {phase_dir}/
  3. Review with BLOCKER/WARNING/SUGGESTION classifications
  4. Message Builder directly with your findings
  5. If 0 BLOCKERs: message Builder with 'APPROVED' and message lead
  6. If BLOCKERs remain: message Builder with feedback for revision

  Maximum rounds: {MAX_ROUNDS}",
  model="{CRITIC_MODEL}"
)
```

**Debate happens via native messaging** — rounds proceed without orchestrator shuttling:
- Builder creates plans → messages Critic
- Critic reviews → messages Builder with BLOCKER/WARNING/SUGGESTION
- Builder revises → messages Critic again
- Loop until Critic messages "APPROVED" (0 BLOCKERs)

**Lead monitors** via message stream:
- Tracks round count
- Intervenes at max rounds ({MAX_ROUNDS})
- Convergence: Critic messages lead with "APPROVED"

**Both still write PLAN.md files** (same output format, same paths).

### If AGENT_TEAMS_MODE=false — Task() Subagents (existing behavior)

**Round 1: Initial Build**

```
◆ Round 1/{MAX_ROUNDS} — Builder creating plans...
```

```
Task(
  prompt="First, read the ultra-builder agent definition.
  Then create plans for Phase {phase}: {phase_name}.
  {planning_context}
  Read CLAUDE.md, DOMAINS.md, and gsd-ultra.json from the project root.
  Write PLAN.md files to: {phase_dir}/",
  subagent_type="general-purpose",
  model="{BUILDER_MODEL}",
  description="Build plans Phase {phase}"
)
```

**After Builder returns:**

Read created plans:
```bash
PLANS_CONTENT=$(cat "${PHASE_DIR}"/*-PLAN.md 2>/dev/null)
PLAN_COUNT=$(ls "${PHASE_DIR}"/*-PLAN.md 2>/dev/null | wc -l)
```

Display: `Builder created {PLAN_COUNT} plans. Sending to Critic...`

**Critic Review:**

```
◆ Round {N}/{MAX_ROUNDS} — Critic reviewing plans...
```

```
Task(
  prompt="First, read the ultra-critic agent definition.
  Then review these plans for Phase {phase}: {phase_name}.
  Round: {current_round}

  <plans>
  {plans_content}
  </plans>

  {planning_context}
  Read CLAUDE.md, DOMAINS.md, and gsd-ultra.json from the project root.",
  subagent_type="general-purpose",
  model="{CRITIC_MODEL}",
  description="Critique plans Phase {phase} R{N}"
)
```

**After Critic returns:**

Parse verdict: APPROVED or REVISE

**If APPROVED:**
```
✓ Critic APPROVED plans (Round {N}/{MAX_ROUNDS})
  BLOCKERs: 0 | Warnings: {N} | Suggestions: {N}
```
→ Proceed to finalize.

**If REVISE and current_round < MAX_ROUNDS:**
```
↻ Critic found {N} BLOCKERs. Sending back to Builder... (Round {N+1}/{MAX_ROUNDS})
```

Extract Critic feedback. Send to Builder for revision:

```
Task(
  prompt="First, read the ultra-builder agent definition.
  Then REVISE plans for Phase {phase}: {phase_name}.
  This is revision round {current_round + 1}.

  <current_plans>
  {plans_content}
  </current_plans>

  <critic_feedback>
  {critic_return}
  </critic_feedback>

  {planning_context}
  Address all BLOCKERs. Consider WARNINGs.
  Write updated PLAN.md files to: {phase_dir}/",
  subagent_type="general-purpose",
  model="{BUILDER_MODEL}",
  description="Revise plans Phase {phase} R{N}"
)
```

Increment current_round. Loop back to Critic Review.

### Max Rounds Handling (both modes)

**If REVISE and current_round >= MAX_ROUNDS:**
```
⚠ Max rounds ({MAX_ROUNDS}) reached. {N} BLOCKERs remain.

Remaining issues:
{list of unresolved BLOCKERs}

Options:
1. Force proceed (accept remaining issues)
2. Provide guidance and retry
3. Abort planning
```

Wait for user input.
</step>

<step name="finalize">
Read final plans:
```bash
PLANS_CONTENT=$(cat "${PHASE_DIR}"/*-PLAN.md 2>/dev/null)
```

Commit:
```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs(ultra): adversarial plan for phase ${PHASE} — approved round ${ROUND}" --files "${PHASE_DIR}"/*-PLAN.md
```

Update ROADMAP.md with plan count and objectives.
</step>

</process>

<offer_next>
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► ADVERSARIAL PLANNING COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Phase {X}: {Name}** — {N} plans, approved round {R}/{MAX}

| Wave | Plans | Domains | Autonomous |
|------|-------|---------|------------|
| {wave} | {plans} | {domains} | {auto} |

Debate: {rounds} rounds, {total_blockers} blockers resolved
Warnings: {N} acknowledged | Suggestions: {N} incorporated

───────────────────────────────────────────────────────

## ▶ Next Up

**Parallel Execute** — Domain-owned execution

/ultra:parallel-execute {X}

<sub>/clear first → fresh context window</sub>
```
</offer_next>

<success_criteria>
- [ ] Builder created initial plans
- [ ] Critic reviewed plans with severity classification
- [ ] Debate loop ran until APPROVED or max rounds
- [ ] All BLOCKERs resolved (or user force-proceeded)
- [ ] Plans committed to git
- [ ] ROADMAP.md updated
- [ ] User knows next step (parallel-execute)
</success_criteria>
