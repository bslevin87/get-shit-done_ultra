# Scaling and Cost Reference

Ultra scales from solo developer to multi-domain team execution. This reference covers team sizing, coordination overhead, cost modeling, and when to use each mode.

## Team Sizes per Phase

| Phase Complexity | Plans | Teammates | Researchers | Verifiers | Total Agents |
|-----------------|-------|-----------|-------------|-----------|-------------|
| Tiny (1 file) | 1 | 1 | 2 | 3 | 6 |
| Small (2-3 files) | 1-2 | 1-2 | 2 | 3 | 6-7 |
| Medium (4-8 files) | 2-3 | 2-3 | 3 | 3 | 8-9 |
| Large (10+ files) | 3-5 | 3-5 | 4 | 3 | 10-12 |
| Greenfield (new project) | 4-6 | 4-5 | 4 | 3 | 11-12 |

**Sweet spot:** 4-5 teammates for execution, 4 researchers, 3 verifiers. This maximizes parallelism while keeping coordination manageable.

## Coordination Overhead Curve

```
Teammates  | Speedup | Overhead | Net Gain
-----------|---------|----------|----------
1          | 1.0x    | 0%       | 1.0x (baseline)
2          | 1.8x    | ~5%      | 1.7x
3          | 2.5x    | ~10%    | 2.3x
4          | 3.0x    | ~15%    | 2.5x (sweet spot)
5          | 3.3x    | ~20%    | 2.6x
6          | 3.4x    | ~30%    | 2.4x (diminishing)
7+         | 3.3x    | ~40%+   | 2.0x (split the phase)
```

Overhead comes from: file ownership resolution, integration wiring, hub-spoke communication, and deferred integration time.

**Agent Teams overhead adjustment:** When Agent Teams mode is enabled, add ~5-15% token overhead per workflow for mailbox messaging, persistent teammate context, and shared task list management. The coordination is faster (real-time messaging vs file serialization) but uses slightly more tokens per agent due to mailbox routing.

## Anti-Patterns

1. **The Kitchen Sink** — Throwing 8 teammates at a 3-file phase. More agents = more coordination overhead. Match team size to phase complexity.

2. **The Lone Wolf** — Running Ultra pipeline on a single-file bug fix. Use `/gsd:quick` for small tasks; Ultra's pipeline has fixed overhead that doesn't pay off for trivial work.

3. **The Debater** — Running 5 rounds of Builder/Critic debate on a straightforward phase. If the first plan is good enough, don't force 5 rounds. Convergence at round 1-2 is fine.

4. **The Perfectionist** — Chasing 100% compliance when 92% is PASS. Diminishing returns on the last few percent. Ship and learn.

5. **The Skip-Verifier** — Skipping adversarial verify "to save time." Verification catches 60-80% of bugs before users do. The cost of verification is always less than the cost of fixing bugs in production.

6. **The No-Learner** — Skipping `/ultra:retrospective`. The knowledge flywheel is ~40% of Ultra's long-term value. Without it, Phase 8 is as expensive as Phase 1.

7. **The Over-Communicator** — Letting teammates communicate peer-to-peer instead of hub-spoke. N*(N-1)/2 communication channels = chaos. Hub-spoke keeps it at N channels.

## Decision Matrix: When to Use What

| Criterion | Raw Claude Code | GSD | GSD Ultra |
|-----------|----------------|-----|-----------|
| **Files touched** | 1-2 | 3-10 | 5-30+ |
| **Domains** | 1 | 1-2 | 2-6 |
| **Quality needs** | Prototype | Production | Production+ |
| **Verification** | Manual | Single verifier | Triple-check debate |
| **Learning** | None | STATE.md | Full flywheel |
| **Research** | Ad hoc | Single researcher | 4-perspective swarm |
| **Planning** | Inline | Single planner | Builder/Critic debate |
| **Execution** | Single thread | Wave-parallel | Domain-parallel + deferred integration |
| **Cost** | $0.50-2 | $2-8 | $5-20 |
| **Time** | 5-15 min | 15-45 min | 30-90 min |
| **Best for** | Quick fixes, prototypes | Feature phases, medium projects | Complex features, multi-domain, critical quality |

## Cost Model per Profile

Estimated token costs per phase (medium complexity, 4-8 files):

### Quality Profile
| Stage | Agents | Model | Tokens (est.) | Cost (est.) |
|-------|--------|-------|---------------|-------------|
| Research | 4 researchers + 1 synthesizer | Opus×4 + Sonnet×1 | ~400K | ~$6.00 |
| Planning | Builder + Critic (2-3 rounds) | Opus×2 | ~300K | ~$4.50 |
| Execution | 3-4 executors | Opus×4 | ~600K | ~$9.00 |
| Verification | Defender + Attacker + Auditor + Debate | Sonnet×3 + Opus×1 | ~400K | ~$4.50 |
| Gap Close | 1-2 fixers | Opus×2 | ~200K | ~$3.00 |
| **Total** | | | **~1.9M** | **~$27.00** |

### Balanced Profile (Default)
| Stage | Agents | Model | Tokens (est.) | Cost (est.) |
|-------|--------|-------|---------------|-------------|
| Research | 4 researchers + 1 synthesizer | Sonnet×3 + Haiku×1 + Sonnet×1 | ~400K | ~$2.50 |
| Planning | Builder + Critic (2-3 rounds) | Opus×2 | ~300K | ~$4.50 |
| Execution | 3-4 executors | Sonnet×4 | ~600K | ~$3.00 |
| Verification | 3 verifiers + Debate | Sonnet×3 + Sonnet×1 | ~400K | ~$2.00 |
| Gap Close | 1-2 fixers | Sonnet×2 | ~200K | ~$1.00 |
| **Total** | | | **~1.9M** | **~$13.00** |

### Budget Profile
| Stage | Tokens (est.) | Cost (est.) |
|-------|---------------|-------------|
| Research | ~400K | ~$1.00 |
| Planning | ~300K | ~$2.50 |
| Execution | ~600K | ~$3.00 |
| Verification | ~400K | ~$0.80 |
| Gap Close | ~200K | ~$0.50 |
| **Total** | **~1.9M** | **~$7.80** |

### Eco Profile
| Stage | Tokens (est.) | Cost (est.) |
|-------|---------------|-------------|
| Research | ~400K | ~$0.40 |
| Planning | ~300K | ~$1.50 |
| Execution | ~600K | ~$1.50 |
| Verification | ~400K | ~$0.40 |
| Gap Close | ~200K | ~$0.30 |
| **Total** | **~1.9M** | **~$4.10** |

*Costs are approximate, based on Anthropic's published pricing as of early 2026.*

## The Multiplicative Effect

Ultra's value compounds across stages:

```
Research quality × Planning quality × Execution quality × Verification quality

Good research → good plans (fewer Critic rounds)
Good plans → good execution (fewer deviations)
Good execution → good verification (fewer findings)
Good verification → good retrospective (better conventions)
Better conventions → better research next phase (flywheel!)

10% improvement per stage:
1.1 × 1.1 × 1.1 × 1.1 = 1.46× overall improvement

20% improvement per stage (mature flywheel):
1.2 × 1.2 × 1.2 × 1.2 = 2.07× overall improvement
```

This multiplicative effect is why skipping stages (especially research and verification) destroys more value than it saves — you're not just losing that stage's benefit, you're losing its compounding effect on all downstream stages.
