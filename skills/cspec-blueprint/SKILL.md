---
name: cspec-blueprint
description: Use when specs are complete and you need implementation blueprints — backend class contracts and frontend component contracts. Activates after /cspec-review passes, or standalone with any well-structured spec files.
---

# C-Spec Blueprint

Produces backend system designs (class contracts) and frontend component definitions from behavioral specs.

## Prerequisites

Check in this order:

**C-Spec mode:**

1. Check for `.cspec/manifest.md` — if missing, direct the user to run `/cspec-discover`.
2. Check for `.cspec/foundation.md` — if missing, direct the user to run `/cspec-foundation`.
3. Check for slices in `reviewed` status — if none, direct the user to run `/cspec-review`.

Warn if the manifest shows slices not yet in `reviewed` status — those will be skipped.

**Standalone mode:**

- Accepts any well-structured spec files the user provides. No manifest or foundation required.

## Input Handling

### C-Spec Mode (Phase 5)

- Reads `.cspec/foundation.md` for architecture, tech stack, shared data models, and conventions
- Reads all slice specs under `.cspec/<domain>/<slice>.md`
- Reads `.cspec/manifest.md` for ordering and dependencies
- Reads `.cspec/user-stories.md` for the purpose and outcome context behind each slice
- Individual slices must be in `reviewed` status to be blueprinted

### Standalone Mode

- Accepts any well-structured spec files the user provides
- Scans for: data models, API contracts, user flows, business rules
- Prompts the user for anything missing that is required to produce contracts
- Creates `.cspec/plans/` for output even when no `.cspec/` directory exists
- Minimum input for backend artifacts: data model definitions (entities, fields, types) and API contracts or service descriptions
- Minimum input for frontend artifacts: user flow descriptions and page/screen descriptions
- If neither minimum is met, prompt the user for the missing information before proceeding

## Artifact Type Detection

- **C-Spec mode:** reads the foundation's Architecture Overview and Tech Stack to determine if the product has a backend, frontend, or both
- **Standalone mode:** infers from the provided specs, or asks the user directly ("Does this product have a backend, frontend, or both?")
- **Per-slice:** if a slice has no frontend aspect (e.g., a webhook handler) or no backend aspect (e.g., a client-side preference toggle), only the applicable artifact is produced for that slice, with a note explaining the omission

## Stage 1: Read & Analyze

Before producing any artifacts, read everything and build a mental model.

1. **Read all inputs** — Foundation + all slice specs + user stories (C-Spec mode) or user-provided spec files (standalone mode).
2. **Identify backend candidates** — Entities that need classes: data models, services, repositories, controllers/handlers. Each candidate gets a preliminary role assignment (entity, service, repository, controller, utility).
3. **Identify frontend candidates** — Components implied by user flows: pages/screens, forms, lists, detail views, navigation elements, shared UI patterns.
4. **Map dependencies** — Which classes depend on which. Which components contain or communicate with which others.
5. **Identify shared patterns** — Base classes, interfaces, or abstract types that multiple slices will need. Shared component patterns (e.g., a standard form layout, a common list/detail pattern). Prefer flat hierarchies — one level of abstraction over deep inheritance chains.
6. **Present findings** — Summarize to the user: "I found N backend class candidates across M domains and N frontend component candidates. Here's the breakdown..." The user can adjust before the skill proceeds to skeleton generation.

This stage produces no files — it is analysis only, presented in conversation for validation.

> "Analysis complete: [N] backend class candidates and [N] frontend component candidates identified across [M] domains. Review the breakdown above and let me know if you'd like to adjust anything before I generate the skeletons."

## Stage 2: Skeleton

After the user validates the analysis, produce skeleton files (one or both depending on artifact type detection).

### Backend Skeleton (`.cspec/plans/backend-skeleton.md`)

Captures the system-wide backend architecture:

- **Architecture Pattern** — The structural pattern being used (e.g., layered, hexagonal) and how layers relate
- **Shared Interfaces** — Interfaces/abstract types that multiple slices implement (e.g., a base repository interface, a service interface pattern)
- **Base Classes** — Abstract classes that provide shared behavior with their public method signatures
- **Shared Services** — Classes that serve multiple slices with their contracts (auth service, validation, error handling)
- **Dependency Graph** — Visual or tabular representation of how the major class categories relate: controllers → services → repositories → entities

Each entry includes: name, responsibility (one sentence), public method signatures (inputs and return types), and which slices use it.

### Frontend Skeleton (`.cspec/plans/frontend-skeleton.md`)

Captures the system-wide component architecture:

- **Component Hierarchy** — The top-level layout tree: app shell, navigation, page containers, shared layout regions
- **Shared Components** — Reusable components (buttons, form fields, modals, list items) that appear in multiple slices with their props interfaces
- **State Patterns** — What state is global vs. local, how components access shared state
- **Routing Structure** — Page-level component-to-route mapping

Each entry includes: name, purpose (one sentence), props interface, key state it manages, and which slices use it.

> "Skeletons written to `.cspec/plans/`. Review the backend and frontend architecture above, then confirm to proceed with per-slice blueprints."

## Stage 3: Per-Slice Artifacts

After the user approves the skeletons, work through each slice producing artifacts (one or both depending on artifact type detection and whether the individual slice has backend/frontend aspects).

### Backend Slice (`.cspec/plans/backend/<domain>/<slice>.md`)

- **Classes** — Each class this slice introduces, with:
  - Name and responsibility (one sentence)
  - Which skeleton class it extends or interface it implements (if any)
  - Public method signatures: method name, parameters with types, return type, one-line description of what it does
  - Dependencies: which other classes it depends on (from this slice or from the skeleton)
- **Slice-Specific Business Logic** — Which business rules from the spec map to which class methods
- **Error Handling** — How error scenarios from the spec map to exceptions/error types and which class is responsible

### Frontend Slice (`.cspec/plans/frontend/<domain>/<slice>.md`)

- **Components** — Each component this slice introduces, with:
  - Name and purpose (one sentence)
  - Parent component (where it sits in the hierarchy)
  - Props interface: prop name, type, required/optional, description
  - Key state: state name, type, description of what triggers changes
  - Which skeleton components it uses or composes
- **User Flow Mapping** — How the steps in the spec's User Flow section map to component interactions (step 1 → user interacts with ComponentA, step 2 → ComponentB receives data and displays result)
- **Error States** — How error scenarios from the spec surface in the UI and which component is responsible for displaying each

### Batch Behavior

- Processes slices in manifest priority order (C-Spec mode) or dependency order (standalone mode)
- Pauses after each slice for user review before continuing
- Resumable — re-invoking picks up `unplanned` slices from the manifest

> "[Slice name] blueprinted — backend: [N] classes, frontend: [N] components. Review above, then confirm to continue with the next slice."

## Output Structure

```
.cspec/plans/
  backend-skeleton.md
  frontend-skeleton.md
  backend/
    auth/
      signup.md
      login.md
    billing/
      checkout.md
  frontend/
    auth/
      signup.md
      login.md
    billing/
      checkout.md
```

For backend-only or frontend-only products, only the applicable skeleton and subdirectory are produced.

## Status Tracking

In C-Spec mode, add a `Blueprint` column to the manifest's Slice Inventory table. On first run, backfill all existing slices as `unplanned`. Only slices in `reviewed` status can transition to `planned`.

| Status | Meaning |
|--------|---------|
| `unplanned` | No blueprint written yet |
| `planned` | Applicable artifacts written for this slice |

In standalone mode, no status tracking — always regenerates from the provided specs.

## Re-run Behavior

- **Skeletons** — Regenerates from the current spec state, overwrites previous versions
- **Per-slice** — Picks up `unplanned` slices; overwrites if re-run on a slice that already has artifacts
- **Standalone mode** — Always regenerates from the provided specs

## Completion

After all slices are processed:

> "Implementation blueprints complete. Artifacts written to `.cspec/plans/` for [N] slices ([N] backend, [N] frontend). The specs are now ready to hand off to an implementation agent."
