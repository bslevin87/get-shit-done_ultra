# Research Swarm

The research swarm replaces GSD's single researcher with four parallel perspectives. Each perspective catches what the others miss.

## The Four Perspectives

### Pattern Analyst

Investigates existing implementations, library options, and ecosystem patterns. Answers: "How have others solved this?"

- Framework-specific patterns and idioms
- Library comparisons (bundle size, maintenance, API quality)
- Common implementation approaches
- Anti-patterns to avoid

### Domain Expert

Brings deep domain knowledge, best practices, and common pitfalls. Answers: "What does this domain require?"

- Domain-specific requirements and constraints
- Regulatory or compliance considerations
- Data modeling patterns
- Integration requirements

### Risk Analyst

Identifies risks, trade-offs, security concerns, and performance implications. Answers: "What could go wrong?"

- Security vulnerabilities and attack vectors
- Performance bottlenecks and scalability concerns
- Dependency risks (maintenance, licensing, breaking changes)
- Migration and rollback considerations

### UX Investigator

Examines UX patterns, accessibility, responsive design, and interaction models. Answers: "How should users experience this?"

- Component patterns and interaction models
- Accessibility requirements (WCAG, ARIA, keyboard navigation)
- Responsive breakpoints and mobile considerations
- Loading states, error states, empty states

## RELAY Protocol

When a researcher discovers information relevant to another perspective, they flag it.

### Task() Subagent Mode (default)

Researchers write RELAY sections in their output files:

```markdown
## RELAY → Risk Analyst
The auth library we recommend (jose) had a critical CVE in v4.x.
Recommend Risk Analyst evaluate v5.x security posture.
```

RELAYs are processed during synthesis. They ensure cross-pollination without creating chaotic peer-to-peer communication.

### Agent Teams Mode (experimental)

Researchers **message each other directly** via the native mailbox:

```
pattern-analyst → risk-analyst: "jose v4.x has CVE — flag JWT v5+ pinning"
```

No file sections needed — the mailbox IS the relay. Researchers still write their output files (same format), but cross-perspective discoveries travel instantly via messaging instead of waiting for synthesis.

### When to RELAY

**When to RELAY:** Only for discoveries that significantly affect another perspective. Not every finding needs relaying.

**When NOT to RELAY:** If the discovery is tangential or the synthesizer will naturally catch it during cross-referencing.

## Conflict Resolution

Researchers may disagree. The synthesizer resolves conflicts using these rules:

1. **Safety > Convenience** — If one researcher flags a security risk and another recommends the risky approach for convenience, safety wins.
2. **Evidence > Opinion** — Specific benchmarks, CVE reports, and documentation beat general preferences.
3. **Flag When Uncertain** — If a conflict can't be resolved objectively, document both positions and flag for human review.

Conflict resolution is documented in RESEARCH.md:

```markdown
## Conflicts and Resolutions

### Conflict 1: Auth Library Choice
- **Pattern Analyst:** Recommends passport.js (most popular, many strategies)
- **Risk Analyst:** Flags passport.js middleware chain as attack surface, recommends jose
- **Resolution:** jose — security concern outweighs popularity (Safety > Convenience)
```

## Scaling Guidance

Not every phase needs four researchers:

| Phase Size | Files | Researchers | Skip |
|-----------|-------|-------------|------|
| Small | 2-3 | 2 | UX Investigator, Risk Analyst (unless security-relevant) |
| Medium | 4-8 | 3 | UX Investigator (unless UI phase) |
| Large | 10+ | 4 | None |
| Greenfield | New project | 4 | None |

**Decision logic:**
- Always include Pattern Analyst and Domain Expert
- Add Risk Analyst if: security, data handling, external APIs, or infrastructure
- Add UX Investigator if: user-facing UI, forms, navigation, or accessibility requirements

## Output Structure

```
.planning/phases/XX-name/
├── research/
│   ├── pattern-analysis.md      # From Pattern Analyst
│   ├── domain-expertise.md      # From Domain Expert
│   ├── risk-analysis.md         # From Risk Analyst
│   └── ux-investigation.md      # From UX Investigator
└── XX-RESEARCH.md               # Synthesized from all perspectives
```

## RESEARCH.md Template

The synthesized research document includes:

- **Research Summary** — Key findings across all perspectives
- **Perspectives Included** — Which researchers ran (and why any were skipped)
- **RELAYs Processed** — Cross-perspective discoveries
- **Conflicts and Resolutions** — Disagreements and how they were resolved
- **Recommendations** — Unified recommendations for the planner
- **Risks and Mitigations** — Consolidated risk register
