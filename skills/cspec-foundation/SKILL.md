---
name: cspec-foundation
description: Use when all feature slice specs have been written and you need to derive the shared foundation spec. Activates after /cspec-write has completed all slices, when user wants to extract shared infrastructure, data models, and conventions.
---

# C-Spec Foundation

Derives the shared foundation spec by reading all written slice specs, extracting common entities and patterns, reconciling conflicts, and synthesizing `.cspec/foundation.md`.

## Prerequisites

Requires `.cspec/manifest.md` with at least some slices in `written` status. If no manifest exists, direct the user to run `/cspec-discover` first. If no slices are written, direct to `/cspec-write`.

Warn the user if the manifest shows unwritten slices — the foundation will be incomplete.

## Derivation Process

1. **Read all slice specs** — Read every `.md` file under `.cspec/` domain directories.
2. **Extract commonalities** — Identify entities, services, patterns, and conventions that appear in two or more slices.
3. **Identify conflicts** — Find cases where the same entity is described differently across slices (different fields, types, constraints).
4. **Resolve conflicts** — For each conflict, follow the Conflict Resolution Protocol below.
5. **Update slice specs** — Write resolved definitions back to affected slice specs. Mark those slices as `foundation-reconciled` in the manifest.
6. **Synthesize the foundation spec** — Write `.cspec/foundation.md` using the template below.
7. **Update manifest** — Set `foundation_derived: true` in the manifest.

## Conflict Resolution Protocol

When two or more slices describe the same entity differently:

1. **Present the conflict** to the user:
   - Entity name
   - How each slice describes it (differing fields, types, constraints)
   - Which slice files are affected
2. **Propose a merged definition** that incorporates all fields/constraints from both slices.
3. **Ask the user to approve or modify** the merged definition.
4. **Write back** the resolved definition to each affected slice spec's Data Requirements section.

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
