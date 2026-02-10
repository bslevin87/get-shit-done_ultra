# Knowledge Flywheel Reference

The knowledge flywheel is Ultra's compound learning mechanism. Each completed phase produces a RETROSPECTIVE.md that feeds conventions into CLAUDE.md and decisions into STATE.md, making every subsequent phase faster and higher quality.

## The Flywheel Effect

```
Phase 1 → RETROSPECTIVE → 3 conventions added to CLAUDE.md
Phase 2 → Research swarm reads those 3 conventions → finds patterns 20% faster
Phase 2 → RETROSPECTIVE → 2 more conventions + 4 DEC-{NNN} decisions
Phase 3 → Planner reads 5 conventions + 4 decisions → better first-draft plans
Phase 3 → Verifier checks against 5 conventions → finds issues faster
...
Phase 8 → Research 60% faster, verification 75% fewer findings, planning 40% better first drafts
```

The math: Each convention eliminates a class of errors. 5 conventions × 3 phases of compounding = exponential improvement in code quality and research speed.

## RETROSPECTIVE.md Template

```markdown
# Phase {X}: {Name} — Retrospective

**Date:** {timestamp}
**Verdict:** {from VERIFICATION.md}
**Plans:** {N} plans, {M} tasks
**Commits:** {count}

## What Delivered
- {artifact}: {description} ({file count} files, {line count} lines)

## What Went Well
- {pattern}: {why it worked, with evidence}

## What Went Wrong
- {issue}: {what happened, root cause, how resolved}

## What We Learned
- {lesson}: {how this changes future phases}

## Process Metrics
| Metric | Value | Benchmark |
|--------|-------|-----------|
| Plans created | {N} | — |
| Tasks executed | {M} | — |
| Verification verdict | {verdict} | PASS |
| Gap-close rounds | {N} | 0-1 |

## Patterns Identified
### Pattern 1: {Name}
**Observed in:** {files}
**Convention suggestion:** {proposed CLAUDE.md addition}

## Architectural Decisions Made
### Decision 1: {Title}
**Context:** {why}
**Decision:** {what}
**Rationale:** {why this over alternatives}

## Domain Boundary Changes
- {domain}: {what changed and why}

## Recommendations for Next Phase
- {recommendation}: {rationale}
```

## STATE.md Decision Protocol (DEC-XXX)

Decisions accumulate in STATE.md with sequential IDs:

```markdown
## Decisions

### DEC-001: Use Zustand over Redux for client state
**Phase:** 3 — State Management
**Date:** 2026-01-15
**Context:** Needed global state for theme, auth, notifications
**Decision:** Zustand — simpler API, smaller bundle, no boilerplate
**Rationale:** Project is medium-scale, doesn't need Redux's middleware ecosystem
**Alternatives:** Redux Toolkit (too heavy), Jotai (atomic model doesn't fit)

### DEC-002: Cursor-based pagination over offset pagination
**Phase:** 4 — API Integration
**Date:** 2026-01-18
**Context:** Product listing API needs pagination
**Decision:** Cursor-based with `after` parameter
**Rationale:** Handles real-time inserts without page drift
**Alternatives:** Offset (simpler but breaks with real-time data)
```

**Numbering:** DEC-{NNN} with zero-padded 3-digit numbers. Start at DEC-001.

**When to log a decision:**
- Technology choices (library, framework, pattern)
- Architecture choices (state management, data flow, API design)
- Convention choices (naming, file structure, export patterns)
- Trade-off decisions (performance vs. readability, DRY vs. explicit)

**When NOT to log:**
- Obvious choices (using React for a React project)
- Temporary decisions (will revisit next phase)
- Style preferences (tabs vs. spaces — goes in CLAUDE.md, not STATE.md)

## CLAUDE.md Evolution Protocol

Conventions are added to CLAUDE.md through the retrospective, never auto-generated:

1. **Pattern identified** during retrospective analysis
2. **Convention drafted** as 1-3 line rule
3. **User asked** to approve, modify, or reject
4. **If approved:** appended to the appropriate section of CLAUDE.md
5. **If modified:** user's version is used
6. **If rejected:** noted in RETROSPECTIVE.md but not added

**Good conventions (specific, actionable):**
```markdown
- Use `useCallback` for event handlers passed as props to memoized children
- API error responses always include `{ error: string, code: string }`
- Feature stores export a `use{Feature}Store` hook, never raw store access
```

**Bad conventions (vague, obvious):**
```markdown
- Write clean code (too vague)
- Use TypeScript (already implied)
- Test your code (not actionable enough)
```

## DOMAINS.md Update Protocol

Domain boundaries can shift during execution. The retrospective checks for:

1. **Files created outside expected domains** — might need domain expansion
2. **Shared files that are only used by one domain** — should be owned
3. **New integration points** — may need new shared paths
4. **Unused domains** — can be removed or consolidated

Updates require user approval. The retrospective presents:
```
Domain boundary changes detected:
1. auth domain expanded: added src/middleware/auth.ts
2. src/types/api.ts should be owned by api domain (only user)
Update DOMAINS.md? [Yes / No / Edit]
```

## Flywheel Math

**Research acceleration:**
- Phase 1: Baseline research time (100%)
- Phase 3: ~80% (conventions guide pattern search)
- Phase 5: ~60% (established patterns + decisions reduce exploration)
- Phase 8: ~40% (comprehensive conventions = focused research)

**Verification improvement:**
- Phase 1: Baseline findings count (100%)
- Phase 3: ~70% (conventions prevent known error classes)
- Phase 5: ~50% (decisions + conventions = fewer blind spots)
- Phase 8: ~25% (accumulated quality = minimal findings)

**Planning quality:**
- Phase 1: ~2.5 Builder/Critic rounds average
- Phase 3: ~2.0 rounds (decisions inform architecture)
- Phase 5: ~1.5 rounds (patterns established, less debate)
- Phase 8: ~1.2 rounds (conventions make first drafts good)

These numbers are aspirational benchmarks. Actual improvement depends on convention quality and project complexity.
