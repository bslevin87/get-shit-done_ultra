# Adversarial Planning

Plans that survive scrutiny before execution rarely fail during implementation. The Builder/Critic debate loop ensures every plan is challenged, revised, and approved before code is written.

## The Debate Loop

```
Round 1: Builder creates plan → Critic reviews
Round 2: Builder revises → Critic re-reviews
Round N: Builder revises → Critic re-reviews
Exit: Critic signals APPROVED (0 BLOCKERs) OR max rounds reached
```

## Builder

The Builder creates plans with:

- **Atomic tasks** — XML-structured, each executable in a fresh context window
- **File ownership declarations** — OWN / SHARED / READ-ONLY / DO NOT TOUCH per domain
- **Interface contracts** — For phases with 3+ domains, explicit contracts between domains
- **Domain-to-wave assignment** — Which domains execute in parallel, which are sequential
- **Verification steps** — How to verify each task completed correctly

The Builder reads STATE.md decisions (DEC-{NNN}) to incorporate past architectural choices.

## Critic

The Critic reviews plans with three severity levels:

| Level | Meaning | Action Required |
|-------|---------|----------------|
| **BLOCKER** | Plan cannot proceed as-is | Builder MUST fix |
| **WARNING** | Plan is weak in this area | Builder SHOULD fix |
| **SUGGESTION** | Plan could be better | Builder MAY fix |

**Convergence** = zero BLOCKERs remaining. Warnings and suggestions are documented but non-blocking.

## Max Rounds

Default: 5 rounds. Configurable in `gsd-ultra.json`:

```json
{
  "pipeline": {
    "adversarial_plan": { "max_rounds": 5 }
  }
}
```

When max rounds reached without convergence:
1. Remaining BLOCKERs are displayed
2. User chooses: force-proceed, provide guidance, or abort
3. If force-proceeded, BLOCKERs are documented as known risks

## Planning Auction (Advanced)

For critical architecture phases, spawn 3 competing Builders:

```
/ultra:adversarial-plan 3 --auction
```

The Critic evaluates all 3 independent plans, selects the best foundation, then enters the standard Builder/Critic refinement loop with the winning plan.

**When to use:**
- Major architectural decisions
- Multiple valid approaches exist
- Quality matters more than speed

**When NOT to use:**
- Straightforward phases with obvious approaches
- Budget-constrained work
- Phases where the architecture is already decided (from previous DEC-{NNN} decisions)

## Interface Contracts

For phases touching 3+ domains, the Builder produces interface contracts:

```markdown
## Interface Contracts

| Provider | Consumer | Contract | Type |
|----------|----------|----------|------|
| auth domain | tasks domain | `useAuth()` returns `{ user, token }` | Hook |
| api domain | auth domain | `/api/auth/login` returns `{ token, refreshToken }` | REST |
| tasks domain | notifications domain | `TaskCreated` event with `{ taskId, userId }` | Event |
```

These contracts become verification criteria — the Attacker checks that every contract is fulfilled during adversarial verification.

## Plan Output

Each approved plan includes:

```markdown
---
phase: 3
plan: 1
type: execute
wave: 1
depends_on: []
files_modified: [src/features/auth/LoginForm.tsx, ...]
autonomous: true
---

<objective>
Implement user authentication with email/password login.

FILE OWNERSHIP:
- OWN: src/features/auth/
- SHARED (append-only): src/types/
- DO NOT TOUCH: src/features/tasks/, src/features/dashboard/
</objective>

<tasks>
<task type="auto">
  <name>Create auth store</name>
  <files>src/features/auth/store.ts</files>
  <action>...</action>
  <verify>...</verify>
  <done>...</done>
</task>
</tasks>
```

## Typical Convergence

- **Phase 1 of new project:** 2-3 rounds (many unknowns)
- **Phase 3+ with conventions:** 1-2 rounds (patterns established)
- **Phase 8+ with mature flywheel:** 1 round (first draft is good)

The knowledge flywheel makes planning faster over time. Conventions and decisions eliminate classes of debate.
