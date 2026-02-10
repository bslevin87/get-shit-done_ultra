# Knowledge Flywheel

The flywheel is Ultra's compound learning mechanism. Each completed phase makes the next one faster and higher quality. This is ~40% of Ultra's long-term value.

## How It Works

```
Phase 1 → RETROSPECTIVE → 3 conventions added to CLAUDE.md
Phase 2 → Research reads 3 conventions → finds patterns 20% faster
Phase 2 → RETROSPECTIVE → 2 more conventions + 4 DEC-{NNN} decisions
Phase 3 → Planner reads 5 conventions + 4 decisions → better first-draft plans
Phase 3 → Verifier checks against 5 conventions → finds issues faster
...
Phase 8 → Research 60% faster, verification 75% fewer findings, planning 40% better
```

## The Retrospective Command

```
/ultra:retrospective <N>
```

Runs after verification passes. Produces `RETROSPECTIVE.md` with:

### What Delivered
Concrete artifacts: components, stores, API endpoints, with file and line counts.

### What Went Well
Patterns that worked, with evidence. "Card-based layout with Zustand store pattern reduced plan iterations from 3 to 1."

### What Went Wrong
Issues encountered, root causes, how they were resolved. "Auth store wasn't exported correctly, causing 2 gap-close rounds."

### What We Learned
Lessons that change future phases. "Always export store hooks from feature index.ts."

### Process Metrics

| Metric | Value | Benchmark |
|--------|-------|-----------|
| Plans created | 3 | — |
| Tasks executed | 12 | — |
| Verification verdict | PASS | PASS |
| Gap-close rounds | 1 | 0-1 |

## CLAUDE.md Evolution

The retrospective identifies patterns and proposes CLAUDE.md conventions. Each convention goes through user approval:

1. **Pattern identified** during retrospective analysis
2. **Convention drafted** as a 1-3 line rule
3. **User asked** to approve, modify, or reject
4. **If approved:** appended to the appropriate CLAUDE.md section
5. **If modified:** user's version is used
6. **If rejected:** noted in RETROSPECTIVE.md but not added

**Good conventions (specific, actionable):**
```
- Use `useCallback` for event handlers passed as props to memoized children
- API error responses always include `{ error: string, code: string }`
- Feature stores export a `use{Feature}Store` hook, never raw store access
```

**Bad conventions (vague, obvious):**
```
- Write clean code (too vague)
- Use TypeScript (already implied)
- Test your code (not actionable enough)
```

## STATE.md Decision Log

Architectural decisions accumulate with sequential DEC-{NNN} IDs:

```markdown
### DEC-001: Use Zustand over Redux for client state
**Phase:** 3 — State Management
**Date:** 2026-01-15
**Context:** Needed global state for theme, auth, notifications
**Decision:** Zustand — simpler API, smaller bundle, no boilerplate
**Rationale:** Project is medium-scale, doesn't need Redux's middleware ecosystem
**Alternatives:** Redux Toolkit (too heavy), Jotai (atomic model doesn't fit)
```

**When to log a decision:**
- Technology choices (library, framework, pattern)
- Architecture choices (state management, data flow, API design)
- Convention choices (naming, file structure, export patterns)
- Trade-off decisions (performance vs. readability, DRY vs. explicit)

**When NOT to log:**
- Obvious choices (using React for a React project)
- Temporary decisions (will revisit next phase)
- Style preferences (tabs vs. spaces — goes in CLAUDE.md, not STATE.md)

## DOMAINS.md Updates

Domain boundaries shift during execution. The retrospective checks for:

1. **Files created outside expected domains** — might need domain expansion
2. **Shared files only used by one domain** — should be owned
3. **New integration points** — may need new shared paths
4. **Unused domains** — can be removed or consolidated

Updates require user approval.

## Flywheel Math

### Research Acceleration
| Phase | Research Time | Why |
|-------|-------------|-----|
| 1 | 100% (baseline) | No conventions to guide search |
| 3 | ~80% | Conventions guide pattern search |
| 5 | ~60% | Established patterns + decisions reduce exploration |
| 8 | ~40% | Comprehensive conventions = focused research |

### Verification Improvement
| Phase | Finding Count | Why |
|-------|--------------|-----|
| 1 | 100% (baseline) | No conventions prevent errors |
| 3 | ~70% | Conventions prevent known error classes |
| 5 | ~50% | Decisions + conventions = fewer blind spots |
| 8 | ~25% | Accumulated quality = minimal findings |

### Planning Quality
| Phase | Builder/Critic Rounds | Why |
|-------|----------------------|-----|
| 1 | ~2.5 average | Many unknowns |
| 3 | ~2.0 | Decisions inform architecture |
| 5 | ~1.5 | Patterns established, less debate |
| 8 | ~1.2 | Conventions make first drafts good |

## The Compounding Effect

Each stage's quality multiplies with every other stage:

```
Research quality × Planning quality × Execution quality × Verification quality

Good research → good plans (fewer Critic rounds)
Good plans → good execution (fewer deviations)
Good execution → good verification (fewer findings)
Good verification → good retrospective (better conventions)
Better conventions → better research next phase (flywheel!)
```

Skipping the retrospective doesn't just lose that stage — it breaks the compounding chain for all future phases.
