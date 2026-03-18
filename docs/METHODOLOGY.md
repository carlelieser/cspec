# C-Spec Methodology

C-Spec produces complete, interconnected specifications for MVPs. No implementation code — only specs precise enough for an agent to build from independently.

## Core Principles

1. **Vertical slices at user-story granularity** — Each spec is a single user flow, buildable end-to-end.
2. **Self-contained specs** — Each slice is buildable with only the foundation spec. No cross-slice knowledge.
3. **Foundation derived last** — Shared infrastructure is extracted from slices, not designed upfront.
4. **Intentional duplication** — Slices describe their own entities. Conflicts reconciled during foundation derivation.
5. **Hybrid format** — Prose for narrative (purpose, user flow), structured/tabular for technical details (data models, APIs, rules).

## Four Phases

| Phase | Skill | Input | Output |
|-------|-------|-------|--------|
| 1. Discovery | `/cspec-discover` | Product idea or document | `.cspec/manifest.md` |
| 2. Spec Writing | `/cspec-write` | Manifest | `.cspec/<domain>/<slice>.md` |
| 3. Foundation | `/cspec-foundation` | All slice specs | `.cspec/foundation.md` |
| 4. Review | `/cspec-review` | All specs | `.cspec/review-report.md` |

## Output Structure

```
.cspec/
  manifest.md              # Slice inventory, domains, ordering, statuses
  foundation.md             # Tech stack, shared models, conventions
  review-report.md          # Validation results
  <domain>/
    <slice>.md              # Individual slice specs
```

## Manifest

Single source of truth for discovered slices. Contains: product summary, tech preferences, domain listing, and slice inventory (name, domain, description, priority, dependencies, status).

**Statuses:** unwritten → written → foundation-reconciled → reviewed

Updated by each phase. `/cspec-foundation` also sets `foundation_derived: true`.

## Slice Spec Template

**Prose:** Purpose, User Flow

**Structured:** Data Requirements, API Endpoints/Interfaces, State Transitions, Business Rules, Error Scenarios, Acceptance Criteria

Each slice must be fully buildable when paired only with the foundation spec.

## Foundation Spec Template

**Prose:** Product Overview, Architecture Overview

**Structured:** Tech Stack, Project Structure, Shared Data Models, Shared Services, Environment & Configuration, Conventions

Derived by reading all slices, extracting commonalities, and reconciling conflicts. Foundation phase has write access to slice specs to resolve inconsistencies.

## Review Checks

- **Completeness** — All manifest slices have specs, all sections filled, acceptance criteria testable
- **Consistency** — Models match foundation, conventions uniform, terminology consistent
- **Coverage** — No orphaned entities, no missing slices, edge cases addressed
- **Dependency integrity** — No circular dependencies, ordering achievable

## Re-run Behavior

- `/cspec-discover` — Amend or replace existing manifest (preserves written specs unless slices removed)
- `/cspec-write` — Picks up unwritten slices; overwrites if re-run on existing slice
- `/cspec-foundation` — Re-derives from current slice state
- `/cspec-review` — Fresh validation pass

Version history tracked by git.

## Skill Locations

```
~/.claude/skills/
  cspec-discover/SKILL.md
  cspec-write/SKILL.md
  cspec-foundation/SKILL.md
  cspec-review/SKILL.md
```

## Design Reference

Full design spec: `docs/superpowers/specs/2026-03-18-cspec-design.md`
