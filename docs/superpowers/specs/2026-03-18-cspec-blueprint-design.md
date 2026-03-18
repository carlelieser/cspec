# C-Spec Blueprint Design: Implementation Plan Artifacts

## Overview

`/cspec-blueprint` bridges behavioral specs to implementation. It reads specs (from C-Spec or any well-structured source) and produces two artifact types:

- **Backend system design** — Class contracts: names, responsibilities, relationships, and public method signatures
- **Frontend component definitions** — Component contracts: names, purpose, hierarchy, props interfaces, and key state descriptions

Every invocation produces both backend and frontend artifacts. The skill runs in three stages: analyze, skeleton, then per-slice. Output lives in `.cspec/plans/`.

## Position in the C-Spec Pipeline

`/cspec-blueprint` serves dual roles:

- **Phase 5 of C-Spec** — After `/cspec-review` passes, the natural next step to translate specs into implementation plans
- **Standalone** — Works with any well-structured spec files, prompting the user for missing information as needed

## Input Handling

### C-Spec Mode (Phase 5)

When `.cspec/` exists with a foundation and slice specs:

- Reads `.cspec/foundation.md` for architecture, tech stack, shared data models, and conventions
- Reads all slice specs under `.cspec/<domain>/<slice>.md`
- Reads `.cspec/manifest.md` for ordering and dependencies

**Prerequisites:** Foundation must exist and at least some slices must be in `reviewed` status. If not, directs user to run the appropriate earlier phase.

### Standalone Mode

When no `.cspec/` directory exists (or user points to other spec files):

- Accepts any well-structured spec files the user provides
- Scans for the information it needs: data models, API contracts, user flows, business rules
- Prompts the user for anything missing that's required to produce class or component contracts (e.g., "I don't see data model definitions — can you describe the key entities?")
- No manifest or foundation required — the skill builds its own understanding from whatever it's given

## Stage 1: Read & Analyze

Before producing any artifacts, the skill reads everything and builds a mental model:

1. **Read all inputs** — Foundation + all slice specs (C-Spec mode) or user-provided spec files (standalone mode)
2. **Identify backend candidates** — Entities that need classes: data models, services, repositories, controllers/handlers. Each candidate gets a preliminary role assignment (entity, service, repository, controller, utility)
3. **Identify frontend candidates** — Components implied by user flows: pages/screens, forms, lists, detail views, navigation elements, shared UI patterns
4. **Map dependencies** — Which classes depend on which. Which components contain or communicate with which others
5. **Identify shared patterns** — Base classes, interfaces, or abstract types that multiple slices will need. Shared component patterns (e.g., a standard form layout, a common list/detail pattern)
6. **Present findings** — Summarize to the user: "I found N backend class candidates across M domains and N frontend component candidates. Here's the breakdown..." The user can adjust before the skill proceeds to skeleton generation

This stage produces no files — it's analysis only, presented in conversation for validation.

## Stage 2: Skeleton

After the user validates the analysis, the skill produces two skeleton files.

### Backend Skeleton (`.cspec/plans/backend-skeleton.md`)

Captures the system-wide backend architecture:

- **Architecture Pattern** — The structural pattern being used (e.g., layered, hexagonal) and how layers relate
- **Shared Interfaces** — Interfaces/abstract types that multiple slices implement (e.g., a base repository interface, a service interface pattern)
- **Base Classes** — Abstract classes that provide shared behavior with their public method signatures
- **Shared Services** — Classes that serve multiple slices (auth service, validation, error handling) with their contracts
- **Dependency Graph** — How the major class categories relate: controllers → services → repositories → entities. Visual or tabular representation

Each entry includes: name, responsibility (one sentence), public method signatures (inputs and return types), and which slices use it.

### Frontend Skeleton (`.cspec/plans/frontend-skeleton.md`)

Captures the system-wide component architecture:

- **Component Hierarchy** — The top-level layout tree: app shell, navigation, page containers, shared layout regions
- **Shared Components** — Reusable components that appear in multiple slices (buttons, form fields, modals, list items) with their props interfaces
- **State Patterns** — Shared state management approach: what state is global vs. local, how components access shared state
- **Routing Structure** — Page-level component mapping to routes

Each entry includes: name, purpose (one sentence), props interface, key state it manages, and which slices use it.

Both skeletons are presented to the user for review before proceeding to per-slice artifacts.

## Stage 3: Per-Slice Artifacts

After the user approves both skeletons, the skill works through each slice producing two files per slice.

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

In batch mode, the skill processes slices in manifest priority order (or dependency order in standalone mode). After each slice, it pauses for user review before continuing. The user can stop and resume — re-invoking picks up where it left off.

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

## Status Tracking

In C-Spec mode, the skill adds a `blueprint` status column to the manifest's Slice Inventory table:

| Status | Meaning |
|--------|---------|
| `unplanned` | No blueprint written yet |
| `planned` | Backend + frontend artifacts written |

In standalone mode, no status tracking — always regenerates from the provided specs.

## Re-run Behavior

- **Skeletons** — Re-running regenerates both skeletons from the current spec state, overwriting previous versions
- **Per-slice** — Re-running picks up `unplanned` slices. If invoked for a slice that already has artifacts, overwrites with new versions
- **Standalone mode** — No status tracking; always regenerates from the provided specs

## Completion

After all slices are processed:

> "Implementation blueprints complete. Backend and frontend artifacts written to `.cspec/plans/` for [N] slices. The specs are now ready to hand off to an implementation agent."
