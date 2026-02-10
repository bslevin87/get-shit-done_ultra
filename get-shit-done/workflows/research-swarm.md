<purpose>
Execute a 4-perspective research swarm for a phase. Spawns 4 specialized researcher agents in parallel, collects their outputs, and synthesizes into a single RESEARCH.md.

Replaces GSD's single-researcher model with multi-perspective research: Pattern Analyst, Domain Expert, Risk Analyst, and UX Investigator each contribute a unique lens.
</purpose>

<core_principle>
Four perspectives are better than one. Each researcher has a different "attack surface" — patterns, domain expertise, risks, and UX. The synthesis combines them into a single research document that captures insights no single researcher would find alone.
</core_principle>

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
# Parse role models from Ultra config based on profile
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
</step>

<step name="prepare_research_dir">
```bash
mkdir -p "${PHASE_DIR}/research"
```
</step>

<step name="build_research_prompt">
Construct shared context for all 4 researchers:

```bash
STATE_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).state_content || ''")
ROADMAP_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).roadmap_content || ''")
CONTEXT_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).context_content || ''")
PHASE_DESC=$(node ~/.claude/get-shit-done/bin/gsd-tools.js roadmap get-phase "${PHASE_ARG}" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).section")
```

Shared context block (passed to all researchers):
```markdown
<phase_context>
**Phase:** {phase_number} - {phase_name}
**Goal:** {phase goal from roadmap}
**Description:** {phase_desc}

**User Decisions (from CONTEXT.md):**
{context_content}

**Project State:**
{state_content}
</phase_context>
```
</step>

<step name="spawn_researchers">
Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► RESEARCH SWARM — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Spawning 4 researchers in parallel...
  → Pattern Analyst   ({PATTERN_MODEL})
  → Domain Expert     ({DOMAIN_MODEL})
  → Risk Analyst      ({RISK_MODEL})
  → UX Investigator   ({UX_MODEL})
```

Spawn all 4 in parallel using Task():

```
Task(
  prompt="First, read the ultra-pattern-analyst agent definition.
  Then research patterns for Phase {phase}: {phase_name}.
  {shared_context}
  Write to: {phase_dir}/research/pattern-analysis.md",
  subagent_type="general-purpose",
  model="{PATTERN_MODEL}",
  description="Research patterns Phase {phase}"
)

Task(
  prompt="First, read the ultra-domain-expert agent definition.
  Then research domain expertise for Phase {phase}: {phase_name}.
  {shared_context}
  Write to: {phase_dir}/research/domain-expertise.md",
  subagent_type="general-purpose",
  model="{DOMAIN_MODEL}",
  description="Research domain Phase {phase}"
)

Task(
  prompt="First, read the ultra-risk-analyst agent definition.
  Then research risks for Phase {phase}: {phase_name}.
  {shared_context}
  Write to: {phase_dir}/research/risk-analysis.md",
  subagent_type="general-purpose",
  model="{RISK_MODEL}",
  description="Research risks Phase {phase}"
)

Task(
  prompt="First, read the ultra-ux-investigator agent definition.
  Then research UX for Phase {phase}: {phase_name}.
  {shared_context}
  Write to: {phase_dir}/research/ux-investigation.md",
  subagent_type="general-purpose",
  model="{UX_MODEL}",
  description="Research UX Phase {phase}"
)
```

All 4 run in parallel. Wait for all to complete.
</step>

<step name="verify_outputs">
Verify all 4 perspective files were written:

```bash
for f in pattern-analysis domain-expertise risk-analysis ux-investigation; do
  [ -f "${PHASE_DIR}/research/${f}.md" ] && echo "✓ ${f}.md" || echo "✗ ${f}.md MISSING"
done
```

If any missing: report which researcher failed, offer retry or continue with available perspectives.
</step>

<step name="synthesize">
Display:
```
◆ Synthesizing 4 perspectives into RESEARCH.md...
```

Read all 4 perspective files. Synthesize into a single RESEARCH.md:

```markdown
# Phase {X}: {Name} — Research (Ultra 4-Perspective)

**Researched:** {timestamp}
**Perspectives:** 4 (Pattern Analysis, Domain Expertise, Risk Analysis, UX Investigation)

## Synthesis Summary
{2-3 paragraphs combining key findings from all 4 perspectives}

## User Constraints
{from CONTEXT.md if exists — copied verbatim}

## Standard Stack
{from domain-expertise.md}

## Architecture Patterns
{from pattern-analysis.md + domain-expertise.md}

## Code Patterns
{from pattern-analysis.md}

## Risk Assessment
{from risk-analysis.md — critical/high risks}

## Edge Cases
{from risk-analysis.md}

## UX Requirements
{from ux-investigation.md — states, accessibility, responsive}

## Don't Hand-Roll
{from domain-expertise.md}

## Common Pitfalls
{merged from all perspectives}

## Cross-Perspective Insights
{findings that emerged from combining perspectives — things no single researcher would have caught}

## Open Questions
{gaps from all perspectives}

## Sources
{consolidated source list}
```

Write to: `${PHASE_DIR}/${PADDED_PHASE}-RESEARCH.md`
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
 ULTRA ► RESEARCH SWARM COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Phase {X}: {Name}** — 4 perspectives synthesized

| Perspective | Confidence | Key Finding |
|-------------|------------|-------------|
| Pattern Analyst | {level} | {1-line} |
| Domain Expert | {level} | {1-line} |
| Risk Analyst | {level} | {1-line} |
| UX Investigator | {level} | {1-line} |

Research: {phase_dir}/{padded_phase}-RESEARCH.md

───────────────────────────────────────────────────────

## ▶ Next Up

**Adversarial Planning** — Builder/Critic debate loop

/ultra:adversarial-plan {X}

<sub>/clear first → fresh context window</sub>
```
</offer_next>

<success_criteria>
- [ ] All 4 researcher agents spawned in parallel
- [ ] All 4 perspective files written to research/ directory
- [ ] Perspectives synthesized into single RESEARCH.md
- [ ] Cross-perspective insights identified
- [ ] RESEARCH.md committed to git
- [ ] User knows next step (adversarial-plan)
</success_criteria>
