# Configuration and Cost

Control how Ultra runs: model profiles, pipeline settings, and cost optimization.

## Model Profiles

Four profiles control which Claude model each agent uses:

### GSD Agents

| Agent | quality | balanced | budget | eco |
|-------|---------|----------|--------|-----|
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

| Agent | quality | balanced | budget | eco |
|-------|---------|----------|--------|-----|
| ultra-pattern-analyst | opus | sonnet | haiku | haiku |
| ultra-domain-expert | opus | sonnet | haiku | haiku |
| ultra-risk-analyst | opus | sonnet | haiku | haiku |
| ultra-ux-investigator | sonnet | haiku | haiku | haiku |
| ultra-builder | opus | opus | sonnet | sonnet |
| ultra-critic | opus | opus | sonnet | sonnet |
| ultra-defender | sonnet | sonnet | haiku | haiku |
| ultra-attacker | opus | sonnet | sonnet | haiku |
| ultra-auditor | sonnet | sonnet | haiku | haiku |

### Profile Philosophy

**quality** — Maximum reasoning power. Opus for all decision-making agents. Use when quota is available and you're doing critical architecture work.

**balanced** (default) — Smart allocation. Opus only for planning (where architecture decisions happen). Sonnet for execution and verification. Good balance of quality and cost.

**budget** — Minimal Opus usage. Sonnet for anything that writes code. Haiku for research and verification. Use for high-volume, less critical work.

**eco** — Maximum efficiency. Haiku for everything except planning and execution (Sonnet). Best for well-established projects with strong CLAUDE.md conventions. The knowledge flywheel compensates for cheaper models. Don't use for Phase 1 of new projects.

### Switching Profiles

```
/gsd:set-profile eco
```

Or set in `.planning/config.json`:
```json
{
  "model_profile": "balanced"
}
```

## Cost Model

Estimated costs per phase (medium complexity, 4-8 files):

### Quality Profile (~$27/phase)

| Stage | Model Mix | Tokens | Cost |
|-------|-----------|--------|------|
| Research | Opus×4 + Sonnet×1 | ~400K | ~$6.00 |
| Planning | Opus×2 (2-3 rounds) | ~300K | ~$4.50 |
| Execution | Opus×4 | ~600K | ~$9.00 |
| Verification | Sonnet×3 + Opus×1 | ~400K | ~$4.50 |
| Gap Close | Opus×2 | ~200K | ~$3.00 |
| **Total** | | **~1.9M** | **~$27.00** |

### Balanced Profile (~$13/phase)

| Stage | Model Mix | Tokens | Cost |
|-------|-----------|--------|------|
| Research | Sonnet×3 + Haiku×1 + Sonnet×1 | ~400K | ~$2.50 |
| Planning | Opus×2 (2-3 rounds) | ~300K | ~$4.50 |
| Execution | Sonnet×4 | ~600K | ~$3.00 |
| Verification | Sonnet×4 | ~400K | ~$2.00 |
| Gap Close | Sonnet×2 | ~200K | ~$1.00 |
| **Total** | | **~1.9M** | **~$13.00** |

### Budget Profile (~$8/phase)

| Stage | Tokens | Cost |
|-------|--------|------|
| Research | ~400K | ~$1.00 |
| Planning | ~300K | ~$2.50 |
| Execution | ~600K | ~$3.00 |
| Verification | ~400K | ~$0.80 |
| Gap Close | ~200K | ~$0.50 |
| **Total** | **~1.9M** | **~$7.80** |

### Eco Profile (~$4/phase)

| Stage | Tokens | Cost |
|-------|--------|------|
| Research | ~400K | ~$0.40 |
| Planning | ~300K | ~$1.50 |
| Execution | ~600K | ~$1.50 |
| Verification | ~400K | ~$0.40 |
| Gap Close | ~200K | ~$0.30 |
| **Total** | **~1.9M** | **~$4.10** |

## gsd-ultra.json

Project-level Ultra configuration:

```json
{
  "domains": {
    "auth": {
      "paths": ["src/features/auth/"],
      "shared_paths": ["src/types/auth.ts"]
    },
    "tasks": {
      "paths": ["src/features/tasks/"],
      "shared_paths": ["src/types/task.ts"]
    }
  },
  "shared_paths": ["src/types/", "src/lib/", "src/components/ui/"],
  "model_routing": {
    "roles": {
      "ultra-builder": {
        "quality": "opus",
        "balanced": "opus",
        "budget": "sonnet",
        "eco": "sonnet"
      }
    }
  },
  "pipeline": {
    "adversarial_plan": { "max_rounds": 5 },
    "adversarial_verify": { "max_debate_rounds": 3 }
  }
}
```

**Not required.** Without this file, Ultra uses smart defaults and auto-discovers domains from your codebase.

### Model Routing Resolution

1. `gsd-ultra.json` → `model_routing.roles.{role}[{profile}]`
2. GSD `gsd-tools.js` MODEL_PROFILES table (fallback)

## Scaling Guidance

### Team Sizes

| Phase Complexity | Plans | Teammates | Researchers | Verifiers | Total |
|-----------------|-------|-----------|-------------|-----------|-------|
| Tiny (1 file) | 1 | 1 | 2 | 3 | 6 |
| Small (2-3 files) | 1-2 | 1-2 | 2 | 3 | 6-7 |
| Medium (4-8 files) | 2-3 | 2-3 | 3 | 3 | 8-9 |
| Large (10+ files) | 3-5 | 3-5 | 4 | 3 | 10-12 |
| Greenfield | 4-6 | 4-5 | 4 | 3 | 11-12 |

### Anti-Patterns

1. **The Kitchen Sink** — 8 teammates on a 3-file phase
2. **The Lone Wolf** — Ultra pipeline on a single-file bug fix (use `/gsd:quick`)
3. **The Debater** — 5 rounds when round 1 was fine
4. **The Perfectionist** — Chasing 100% when 92% is PASS
5. **The Skip-Verifier** — Skipping verify "to save time"
6. **The No-Learner** — Skipping retrospective (costs ~40% of long-term value)
7. **The Over-Communicator** — Peer-to-peer instead of hub-spoke
