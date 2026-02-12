<purpose>
Execute a 4-perspective research swarm for a phase. Spawns specialized researcher agents in parallel, collects outputs, and delegates synthesis into RESEARCH.md.
</purpose>

<context_budget>
Lead stays under 50% context. Enforced by:
1. **Context-by-reference** — shared context written to `.context-brief.md`, not embedded in spawn prompts
2. **Delegated synthesis** — Task() subagent reads perspective files and writes RESEARCH.md; lead receives 3-line summary
3. **Spawn brevity** — each researcher prompt is role + file paths + protocol ref
4. **Protocols by reference** — RELAY, scaling, synthesis template live in `ultra-conventions.md`
</context_budget>

<process>

<step name="initialize" priority="first">
Load all context in one call:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init plan-phase "${PHASE_ARG}" --include state,roadmap,requirements,context)
```

Parse JSON for: `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`.

Also read Ultra config:
```bash
ULTRA_CONFIG=$(cat gsd-ultra.json 2>/dev/null || echo '{}')
MODEL_PROFILE=$(cat .planning/config.json 2>/dev/null | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_profile || 'balanced'")
```

Resolve models for each researcher role from gsd-ultra.json:
```bash
PATTERN_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.pattern_analyst['$MODEL_PROFILE']")
DOMAIN_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.domain_expert['$MODEL_PROFILE']")
RISK_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.risk_analyst['$MODEL_PROFILE']")
UX_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.ux_investigator['$MODEL_PROFILE']")
```

If gsd-ultra.json missing, fall back to gsd-tools.js model resolution:
```bash
PATTERN_MODEL=$(node ~/.claude/get-shit-done/bin/gsd-tools.js resolve-model gsd-phase-researcher)
# Use same model for all 4 as fallback
```

**Determine researcher count** using scaling guidance in `ultra-conventions.md`:
- Check phase scope (files count, UI involvement, sensitivity)
- Default: 4 researchers for medium+ phases
- Minimum: 2 researchers (Pattern + Domain) for small phases
</step>

<step name="detect_agent_teams" priority="after-init">
```bash
AGENT_TEAMS_ENV=$(echo "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-0}")
AGENT_TEAMS_MODE=$( [ "$AGENT_TEAMS_ENV" = "1" ] && echo true || echo false )
```

Display: `Coordination: {Agent Teams (native) | Task() subagents}`

Both paths produce identical outputs.
</step>

<step name="prepare_research_dir">
```bash
mkdir -p "${PHASE_DIR}/research"
```
</step>

<step name="write_context_brief">
Write shared context to `{phase_dir}/.context-brief.md` (format defined in `ultra-conventions.md`):

```bash
STATE_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).state_content || ''")
ROADMAP_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).roadmap_content || ''")
CONTEXT_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).context_content || ''")
PHASE_DESC=$(node ~/.claude/get-shit-done/bin/gsd-tools.js roadmap get-phase "${PHASE_ARG}" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).section")
```

Write `{phase_dir}/.context-brief.md` with phase info, user decisions, project state.
Include relay instructions: "If you discover cross-perspective info, see RELAY Protocol in ultra-conventions.md."

This file is written once and referenced by all spawned agents — never embedded inline.
</step>

<step name="spawn_researchers">
Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA > RESEARCH SWARM — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Spawning {N} researchers in parallel...
  > Pattern Analyst   ({PATTERN_MODEL})
  > Domain Expert     ({DOMAIN_MODEL})
  > Risk Analyst      ({RISK_MODEL})     {if included}
  > UX Investigator   ({UX_MODEL})       {if included}
Coordination: {Agent Teams (native) | Task() subagents}
```

**Researcher spawn spec** (table-driven, one entry per researcher):

| Role | Agent Def | Output File | Model |
|------|-----------|-------------|-------|
| Pattern Analyst | ultra-pattern-analyst | research/pattern-analysis.md | {PATTERN_MODEL} |
| Domain Expert | ultra-domain-expert | research/domain-expertise.md | {DOMAIN_MODEL} |
| Risk Analyst | ultra-risk-analyst | research/risk-analysis.md | {RISK_MODEL} |
| UX Investigator | ultra-ux-investigator | research/ux-investigation.md | {UX_MODEL} |

For each researcher, spawn with this prompt template:
```
Read the {agent_def} agent definition.
Research {role_focus} for Phase {phase}: {phase_name}.
Read context brief: {phase_dir}/.context-brief.md
Read ultra-conventions.md for RELAY protocol and scaling guidance.
Write output to: {phase_dir}/{output_file}
```

**If AGENT_TEAMS_MODE=true:** Spawn as Teammate() with name="{role-slug}". RELAY becomes native messaging. Lead enters delegate mode.

**If AGENT_TEAMS_MODE=false:** Spawn as Task(subagent_type="general-purpose"). All spawn in parallel. Wait for all to complete.
</step>

<step name="verify_outputs">
```bash
for f in pattern-analysis domain-expertise risk-analysis ux-investigation; do
  [ -f "${PHASE_DIR}/research/${f}.md" ] && echo "ok ${f}.md" || echo "MISSING ${f}.md"
done
```

If any missing: report which researcher failed, offer retry or continue with available perspectives.
</step>

<step name="delegate_synthesis">
Display: `Synthesizing {N} perspectives into RESEARCH.md...`

Spawn a synthesis subagent — lead does NOT read the perspective files:

```
Task(
  prompt="You are the research synthesizer for Phase {phase}: {phase_name}.
  Read these perspective files from {phase_dir}/research/:
  - pattern-analysis.md, domain-expertise.md, risk-analysis.md, ux-investigation.md
  Read ultra-conventions.md for the RESEARCH.md Synthesis Template and Synthesis Rules.
  Read the context brief: {phase_dir}/.context-brief.md
  Write synthesized output to: {phase_dir}/{padded_phase}-RESEARCH.md
  Return a 3-line summary: perspectives count, conflicts count, key insight.",
  subagent_type="general-purpose",
  description="Synthesize research Phase {phase}"
)
```

Lead receives only the 3-line summary. RESEARCH.md is written by the subagent.
</step>

<step name="commit">
```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs(ultra): research swarm for phase ${PHASE}" --files "${PHASE_DIR}/research/pattern-analysis.md" "${PHASE_DIR}/research/domain-expertise.md" "${PHASE_DIR}/research/risk-analysis.md" "${PHASE_DIR}/research/ux-investigation.md" "${PHASE_DIR}/${PADDED_PHASE}-RESEARCH.md"
```
</step>

</process>

<offer_next>
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA > RESEARCH SWARM COMPLETE

Phase {X}: {Name} — {N} perspectives synthesized
Conflicts: {N resolved}, {M unresolved}
Research: {phase_dir}/{padded_phase}-RESEARCH.md

 > Next: /ultra:adversarial-plan {X}
   /clear first for fresh context
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</offer_next>
