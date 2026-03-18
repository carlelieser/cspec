# C-Spec Methodology

C-Spec produces complete, interconnected specifications for MVPs. No implementation code — only specs precise enough for an agent to build from independently.

## Core Principles

1. **Vertical slices at user-story granularity** — Each spec is a single user flow, buildable end-to-end.
2. **Self-contained specs** — Each slice is buildable with only the foundation spec. No cross-slice knowledge.
3. **Foundation derived last** — Shared infrastructure is extracted from slices, not designed upfront.
4. **Intentional duplication** — Slices describe their own entities. Conflicts reconciled during foundation derivation.
5. **No implementation details** — Specs describe what the user experiences, not how the system delivers it. The test: could a non-technical user understand this slice? Code, library choices, and framework patterns are banned. A single user action is a single slice — never split into technical approaches.
6. **Hybrid format** — Prose for narrative (purpose, user flow), structured/tabular for technical details (data models, APIs, rules).

## Five Phases

| Phase | Skill | Input | Output |
|-------|-------|-------|--------|
| 1. Discovery | `/cspec-discover` | Product idea or document | `.cspec/manifest.md`, `.cspec/user-stories.md` |
| 2. Spec Writing | `/cspec-write` | Manifest | `.cspec/<domain>/<slice>.md` |
| 3. Foundation | `/cspec-foundation` | All slice specs | `.cspec/foundation.md` |
| 4. Review | `/cspec-review` | All specs | `.cspec/review-report.md` |
| 5. Blueprint | `/cspec-blueprint` | Foundation + reviewed slice specs | `.cspec/plans/backend-skeleton.md`, `.cspec/plans/frontend-skeleton.md`, per-slice artifacts |

## Output Structure

```
.cspec/
  manifest.md              # Slice inventory, domains, ordering, statuses
  user-stories.md           # User stories grouped by domain (source of truth for slices)
  foundation.md             # Tech stack, shared models, conventions
  review-report.md          # Validation results
  plans/
    backend-skeleton.md
    frontend-skeleton.md
    backend/
      <domain>/
        <slice>.md
    frontend/
      <domain>/
        <slice>.md
  <domain>/
    <slice>.md              # Individual slice specs
```

## Manifest

Single source of truth for discovered slices. Contains: product summary, tech preferences, domain listing, and slice inventory (name, domain, description, priority, dependencies, status).

## User Stories

Separate reviewable document (`.cspec/user-stories.md`) containing all user stories grouped by domain. Each story maps to a slice in the manifest and uses the format "As a [user], I want to [action], so that [outcome]." User stories are the source of truth for what each slice should do — editing a story should be followed by re-running `/cspec-write` for the affected slice.

**Statuses:** `unwritten` → `written` → `reviewed` → `unplanned` → `planned`. Slices modified during foundation conflict resolution pass through `foundation-reconciled` before `reviewed`. `/cspec-blueprint` sets `unplanned` on slices that have not yet been blueprinted, and `planned` once skeleton artifacts are generated. See the authoritative Slice Status Lifecycle in `/cspec-discover`.

Updated by each phase. `/cspec-foundation` also sets `foundation_derived: yes` in the manifest.

## Slice Spec Template

Written from the corresponding user story in `.cspec/user-stories.md`.

**Prose:** Purpose, User Flow

**Structured:** Data Requirements, API Endpoints, State Transitions, Business Rules, Error Scenarios, Acceptance Criteria

Each slice must be fully buildable when paired only with the foundation spec.

## Foundation Spec Template

**Prose:** Product Overview, Architecture Overview

**Structured:** Tech Stack, Project Structure, Shared Data Models, Shared Services, Environment & Configuration, Conventions

Derived by reading all slices, extracting commonalities, and reconciling conflicts. Foundation phase has write access to slice specs to resolve inconsistencies.

## Review Checks

- **Completeness** — All manifest slices have specs, all sections filled, acceptance criteria testable
- **Consistency** — Models match foundation, conventions uniform, terminology consistent
- **Coverage** — No orphaned entities, no missing slices, edge cases addressed
- **Implementation leakage** — No code, library choices, framework patterns, or DDL in specs
- **Dependency integrity** — No circular dependencies, ordering achievable

## Re-run Behavior

- `/cspec-discover` — Amend or replace existing manifest and user stories (preserves written specs unless slices removed)
- `/cspec-write` — Picks up unwritten slices; overwrites if re-run on existing slice
- `/cspec-foundation` — Re-derives from current slice state
- `/cspec-review` — Fresh validation pass
- `/cspec-blueprint` — Re-generates skeletons from current specs. Picks up unplanned slices; overwrites if re-run on existing.

Version history tracked by git.

## Scalability

C-Spec is designed for MVPs with **5–20 slices**. At this scale, conflict resolution is manageable, the agent can hold all slice specs in context during foundation derivation, and review reports are actionable.

For larger projects (20+ slices), partition by domain — run a separate C-Spec process per domain, then write a meta-foundation that stitches the domain foundations together.

For any project, an optional early foundation checkpoint after every 5–7 slices can catch structural conflicts before writing more slices.

## Skill Locations

```
skills/
  cspec-discover/SKILL.md
  cspec-write/SKILL.md
  cspec-foundation/SKILL.md
  cspec-review/SKILL.md
  cspec-blueprint/SKILL.md
```

## Design Reference

Full design spec: `docs/superpowers/specs/2026-03-18-cspec-design.md`
