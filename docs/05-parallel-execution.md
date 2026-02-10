# Parallel Execution

Ultra's execution engine runs domains in parallel with strict file ownership, then integrates results serially. This is the two-layer parallelism model.

## Two-Layer Parallelism

**Layer 1: Domain-Level** — Multiple teammates work on separate domains simultaneously. Each teammate owns specific directories and cannot touch others.

**Layer 2: Task-Level** — Within their domain, each teammate can spawn subagents for independent tasks (e.g., writing tests while implementing features).

## File Ownership Protocol

Every executor receives explicit ownership declarations:

| Level | Permission | Example |
|-------|-----------|---------|
| **OWN** | Full read/write | `src/features/auth/` |
| **SHARED (append-only)** | Add new items, don't modify existing | `src/types/` |
| **READ-ONLY** | Import and reference, don't modify | `src/components/` |
| **DO NOT TOUCH** | No access | Other domain directories |

Ownership is enforced at the prompt level — each executor's prompt includes its ownership block. This is convention-based, relying on the agent following instructions.

```
YOU OWN: src/features/auth/ (full read/write)
SHARED (append-only): src/types/ (add new types, don't modify existing)
DO NOT TOUCH: src/features/tasks/, src/features/dashboard/, src/features/notifications/
```

## Deferred Integration

Teammates build their domains independently. After all complete, the lead integrates serially:

1. **Conflict Check** — Verify no teammates modified the same files
2. **Cross-Domain Wiring** — Connect domains (imports, event subscriptions, route registration)
3. **Build Verify** — Everything compiles with all domains combined
4. **Integration Commit** — Single commit for the wiring work

This avoids merge conflicts and race conditions. Each domain is self-contained until integration.

## Hub-Spoke Communication

All cross-domain discovery goes through the lead, never peer-to-peer:

```
teammate-1 → lead → teammate-2    (correct: hub-spoke)
teammate-1 → teammate-2           (wrong: peer-to-peer)
```

### Task() Subagent Mode (default)

When a teammate discovers something affecting another domain:
1. Teammate reports the discovery in their SUMMARY.md
2. Lead processes discoveries after the wave completes
3. Lead injects relevant context into the next wave or backfills

### Agent Teams Mode (experimental)

When a teammate discovers something affecting another domain:
1. Teammate **messages lead** via native mailbox (real-time, no file wait)
2. Lead **messages affected teammates** with relevant context immediately
3. Teammates can react within the same wave (faster feedback loop)

The shared task list also enables **self-claiming** — teammates pick up tasks matching their domain without orchestrator assignment. Tasks with unmet dependencies stay locked until predecessors complete.

**Why hub-spoke in both modes?** N teammates with peer-to-peer = N*(N-1)/2 communication channels. Hub-spoke keeps it at N channels.

## Scaling Guidance

| Teammates | Speedup | Overhead | Net Gain |
|-----------|---------|----------|----------|
| 1 | 1.0x | 0% | 1.0x (baseline) |
| 2 | 1.8x | ~5% | 1.7x |
| 3 | 2.5x | ~10% | 2.3x |
| 4 | 3.0x | ~15% | 2.5x (sweet spot) |
| 5 | 3.3x | ~20% | 2.6x |
| 6+ | 3.4x | ~30%+ | Diminishing — split the phase |

**Sweet spot:** 4-5 teammates. Interface contracts required for 3+ domains.

## Wave Execution

Plans are grouped into waves based on dependencies:

```
Wave 1: Plans with no dependencies (parallel)
Wave 2: Plans depending on Wave 1 (parallel)
Wave 3: Plans depending on Wave 2 (parallel)
```

Within each wave, all plans execute simultaneously. Between waves, the lead checks results and resolves any hub-spoke discoveries.

## Spawn Template

Each executor is spawned with:

```markdown
You are executing Phase {X}: {phase_name}
Plan: {plan_id}

FILE OWNERSHIP:
- OWN: {owned_paths}
- SHARED (append-only): {shared_paths}
- READ-ONLY: {read_only_paths}
- DO NOT TOUCH: {forbidden_paths}

INTERFACE CONTRACTS:
{contracts relevant to this domain}

Execute all tasks in the plan. Commit per task.
Write summary to: {phase_dir}/{plan_id}-SUMMARY.md
```

## Anti-Patterns

**The Kitchen Sink** — 8 teammates on a 3-file phase. More agents = more coordination overhead. Match team size to complexity.

**The Over-Communicator** — Letting teammates communicate peer-to-peer. Use hub-spoke.

**No Ownership** — Running parallel execution without file ownership declarations. Teammates will inevitably modify the same files, creating conflicts.
