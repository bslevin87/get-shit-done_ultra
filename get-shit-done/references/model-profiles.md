# Model Profiles

Model profiles control which Claude model each GSD agent uses. This allows balancing quality vs token spend.

## Profile Definitions

| Agent | `quality` | `balanced` | `budget` | `eco` |
|-------|-----------|------------|----------|-------|
| gsd-planner | opus | opus | sonnet | sonnet |
| gsd-roadmapper | opus | sonnet | sonnet | haiku |
| gsd-executor | opus | sonnet | sonnet | sonnet |
| gsd-phase-researcher | opus | sonnet | haiku | haiku |
| gsd-project-researcher | opus | sonnet | haiku | haiku |
| gsd-research-synthesizer | sonnet | sonnet | haiku | haiku |
| gsd-debugger | opus | sonnet | sonnet | sonnet |
| gsd-codebase-mapper | sonnet | haiku | haiku | haiku |
| gsd-verifier | sonnet | sonnet | haiku | haiku |
| gsd-plan-checker | sonnet | sonnet | haiku | haiku |
| gsd-integration-checker | sonnet | sonnet | haiku | haiku |

### Ultra Agents

Ultra agents use `gsd-ultra.json` for model routing (project-level), falling back to these profiles.

| Agent | `quality` | `balanced` | `budget` | `eco` |
|-------|-----------|------------|----------|-------|
| ultra-pattern-analyst | opus | sonnet | haiku | haiku |
| ultra-domain-expert | opus | sonnet | haiku | haiku |
| ultra-risk-analyst | opus | sonnet | haiku | haiku |
| ultra-ux-investigator | sonnet | haiku | haiku | haiku |
| ultra-builder | opus | opus | sonnet | sonnet |
| ultra-critic | opus | opus | sonnet | sonnet |
| ultra-defender | sonnet | sonnet | haiku | haiku |
| ultra-attacker | opus | sonnet | sonnet | haiku |
| ultra-auditor | sonnet | sonnet | haiku | haiku |

## Profile Philosophy

**quality** - Maximum reasoning power
- Opus for all decision-making agents
- Sonnet for read-only verification
- Use when: quota available, critical architecture work

**balanced** (default) - Smart allocation
- Opus only for planning (where architecture decisions happen)
- Sonnet for execution and research (follows explicit instructions)
- Sonnet for verification (needs reasoning, not just pattern matching)
- Use when: normal development, good balance of quality and cost

**budget** - Minimal Opus usage
- Sonnet for anything that writes code
- Haiku for research and verification
- Use when: conserving quota, high-volume work, less critical phases

**eco** - Maximum efficiency
- Haiku for everything except planning and execution
- Sonnet for planning (architecture still matters) and code writing
- Use when: iterating fast on well-understood phases, budget-constrained, non-critical work
- Trade-off: verification quality is lower, so more gaps may slip through

## Resolution Logic

Orchestrators resolve model before spawning:

```
1. Read .planning/config.json
2. Get model_profile (default: "balanced")
3. Look up agent in table above
4. Pass model parameter to Task call
```

## Switching Profiles

Runtime: `/gsd:set-profile <profile>`

Per-project default: Set in `.planning/config.json`:
```json
{
  "model_profile": "balanced"
}
```

## Design Rationale

**Why Opus for gsd-planner?**
Planning involves architecture decisions, goal decomposition, and task design. This is where model quality has the highest impact.

**Why Sonnet for gsd-executor?**
Executors follow explicit PLAN.md instructions. The plan already contains the reasoning; execution is implementation.

**Why Sonnet (not Haiku) for verifiers in balanced?**
Verification requires goal-backward reasoning - checking if code *delivers* what the phase promised, not just pattern matching. Sonnet handles this well; Haiku may miss subtle gaps.

**Why Haiku for gsd-codebase-mapper?**
Read-only exploration and pattern extraction. No reasoning required, just structured output from file contents.

**Why eco exists?**
For well-established projects with strong CLAUDE.md conventions, the knowledge flywheel makes planning and verification easier. Eco leverages accumulated conventions to get good results from cheaper models. Don't use eco for Phase 1 of a new project.
