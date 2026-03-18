---
name: cspec-foundation
description: Use when feature slice specs have been written and you need to derive the shared foundation spec. Activates after /cspec-write has produced slice specs, when user wants to extract shared models, conventions, and architecture decisions.
---

# C-Spec Foundation

Derives the shared foundation spec by reading all written slice specs, extracting common entities and patterns, reconciling conflicts, and synthesizing `.cspec/foundation.md`.

## Prerequisites

Requires `.cspec/manifest.md` with at least some slices in `written` status. If no manifest exists, direct the user to run `/cspec-discover` first. If no slices are written, direct to `/cspec-write`.

Warn the user if the manifest shows unwritten slices — the foundation will be incomplete.

## Implementation Decisions Enter Here

Slice specs describe what the user experiences and are deliberately implementation-free. The foundation is where implementation decisions are introduced — tech stack, architecture, project structure, and conventions. These decisions are made collaboratively with the user during derivation, not extracted from the (implementation-free) slices.

## Derivation Process

1. **Read all slice specs** — Read every `.md` file under `.cspec/` domain directories.
2. **Extract commonalities** — Identify entities, services, patterns, and conventions that appear in two or more slices. Look for: shared data models, repeated API patterns, common business rules, and terminology used across slices.
3. **Identify conflicts** — Find cases where the same entity is described differently across slices (different fields, types, constraints).
4. **Resolve conflicts** — For each conflict, follow the Conflict Resolution Protocol below.
5. **Update slice specs** — Write resolved definitions back to affected slice specs. Mark those slices as `foundation-reconciled` in the manifest (see Slice Status Lifecycle in `/cspec-discover`).
6. **Determine architecture and tech stack** — Read the manifest's Tech Preferences. If preferences were recorded, use them as the starting point. If "None specified" or incomplete, ask the user about: language, framework, database, auth approach, and hosting/deployment. Present recommendations with rationale based on the product's needs (e.g., real-time features suggest WebSocket support, CRUD-heavy apps suit relational databases). One question at a time. Prefer multiple choice when possible.
7. **Synthesize the foundation spec** — Write `.cspec/foundation.md` using the template below.
8. **Update manifest** — Set `foundation_derived: yes` under Tech Preferences in the manifest.

## Conflict Resolution Protocol

Check for five types of conflicts across slices:

1. **Data model conflicts** — Same entity described differently (different fields, types, constraints).
2. **API convention conflicts** — Inconsistent endpoint naming, error response shapes, or auth patterns.
3. **Business rule conflicts** — Contradictory rules for the same concept (e.g., one slice says passwords must be 8+ characters, another says 10+).
4. **Behavioral conflicts** — Contradictory system behavior (e.g., one slice says the system sends email on event X, another says SMS).
5. **Terminology conflicts** — Same concept called different names across slices (e.g., "workspace" vs. "organization").

For each conflict found:

1. **Present the conflict** to the user:
   - Conflict type
   - How each slice describes it
   - Which slice files are affected
2. **Propose a resolution** — For data models, a merged definition. For terminology, a canonical name. For business rules and behavior, the most restrictive or most complete version with rationale.
3. **Ask the user to approve or modify** the resolution.
4. **Write back** the resolved definition to each affected slice spec. If a rename is involved (field, entity, or term), propagate it across all sections of the affected slices — not just Data Requirements.

Slice specs are **not stripped** of their entity descriptions after reconciliation. They keep their own descriptions, but those descriptions are now consistent with the foundation and with each other.

## Foundation Spec Template

Write to `.cspec/foundation.md`:

```markdown
# [Product Name] — Foundation

## Product Overview

[Expanded from manifest's product summary, enriched with context gained from reading all slice specs]

## Architecture Overview

[High-level system shape: monolith vs. microservices, client-server split, key architectural decisions and rationale]

## Tech Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Language | [language] | [why] |
| Framework | [framework] | [why] |
| Database | [database] | [why] |
| Auth | [approach] | [why] |
| [other] | [technology] | [why] |

## Project Structure

```
project-root/
  [directory layout with descriptions]
```

## Shared Data Models

### [Entity Name]

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| ... | ... | ... | ... |

**Used by:** [comma-separated list of slices that reference this entity]

### [Entity Name]

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| ... | ... | ... | ... |

**Used by:** [comma-separated list of slices]

## Shared Services

Include the following sections as applicable to the product:

### Authentication & Authorization
[Auth middleware, session handling, token management]

### Error Handling
[Standard error response format, error codes, logging]

### API Conventions
[Base URL structure, versioning, request/response format, pagination]

### Validation
[Input validation patterns, shared validation rules]

## Environment & Configuration

| Variable | Purpose | Example |
|----------|---------|---------|
| ... | ... | ... |

[Config file structure, secrets management approach]

## Conventions

- **Code style:** [language-specific conventions]
- **Naming:** [file, variable, function, endpoint naming patterns]
- **Error response format:** [standard shape]
- **API versioning:** [strategy]
- **[Other cross-cutting standards]**
```

## Re-run Behavior

Re-derives the foundation from the current state of all slice specs. Overwrites the previous `.cspec/foundation.md`. Re-runs the conflict resolution process if new conflicts are found.

## Completion

After writing the foundation, inform the user:

> "Foundation spec written to `.cspec/foundation.md`. [N] shared entities extracted, [N] conflicts resolved. Run `/cspec-review` to validate all specs for consistency."
