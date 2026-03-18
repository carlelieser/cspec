# C-Spec Design: Complete Specification Methodology

## Overview

C-Spec is a set of four skills that produce complete, interconnected specifications for any MVP. The specs contain no implementation code — they define *what* to build with enough precision that an agent (or separate implementation skill) can execute each slice independently.

The distinguishing factor is not a single long spec but a series of **self-contained vertical slices** — each one describing a user story end-to-end — plus a **foundation spec** that captures shared infrastructure derived from those slices.

## Core Principles

- **Vertical slices at user-story granularity** — Each spec describes a single user flow that can be built end-to-end.
- **Self-contained specs** — Each slice is fully buildable when paired only with the foundation spec. No cross-slice knowledge required.
- **Foundation is derived, not designed** — Shared concerns (data models, tech stack, conventions) are extracted from slices after they're written, not imposed top-down.
- **Intentional duplication** — Slices may describe the same entity differently. This is expected and reconciled during foundation derivation.
- **No implementation details** — Specs describe what the user experiences, not how the system delivers it. The test: could you explain this slice to a non-technical user and have it make sense? A single user action is a single slice — even if the system uses multiple technical approaches to fulfill it. Code snippets, library choices, framework patterns, and algorithm descriptions are all banned.
- **Phased user control** — The user decides when to move between phases and can re-run any phase independently.

## Skill Architecture

### The Four Skills

| Skill | Input | Output | Purpose |
|-------|-------|--------|---------|
| `/cspec-discover` | Product idea (interview or document) | `.cspec/specs/manifest.md`, `.cspec/specs/user-stories.md` | Identify all vertical slices, group by domain, define ordering, write user stories |
| `/cspec-write` | Manifest + user guidance | `.cspec/specs/<domain>/<slice>.md` | Write full specs for individual slices (single or batch) |
| `/cspec-foundation` | All written slice specs | `.cspec/specs/foundation.md` | Derive shared infrastructure: tech stack, project structure, conventions, data models, shared services |
| `/cspec-review` | All specs (foundation + slices) | Validation report + fixes | Cross-validate consistency, completeness, and gaps across all specs |

### Execution Flow

```
/cspec-discover  →  /cspec-write  →  /cspec-foundation  →  /cspec-review
     ↑                                                          │
     └──────────── (iterate if gaps found) ─────────────────────┘
```

The review skill identifies which phase to revisit based on the type of issue — missing slices require `/cspec-discover`, incomplete specs require `/cspec-write`, foundation gaps require `/cspec-foundation`.

## Output Structure

All output lives under `.cspec/` in the directory where Claude is invoked. Spec artifacts live under `.cspec/specs/`, organized by domain:

```
.cspec/
  specs/
    manifest.md
    user-stories.md
    foundation.md
    review-report.md
    auth/
      signup.md
      login.md
      password-reset.md
    billing/
      checkout.md
      subscription-management.md
    content/
      create-post.md
      edit-post.md
```

## Phase 1: Discovery (`/cspec-discover`)

### Input Modes

The skill infers its mode from context: if the user provides a document (file path or pasted content), it enters document-driven or hybrid mode. Otherwise, it defaults to interactive.

- **Interactive** — The skill interviews the user about their product idea, asking questions one at a time. Progressively identifies slices as the conversation develops.
- **Document-driven** — The user provides an existing brief, PRD, or description. The skill analyzes it and extracts slices.
- **Hybrid** — Starts from a document, then asks follow-up questions to fill gaps.

### Process

1. Understand the product at a high level — what is it, who is it for, what problem does it solve
2. Identify user-facing features and break them into vertical slices (user-story granularity)
3. Group slices into domains
4. Determine ordering — which slices should be built first based on dependencies
5. Ask about tech preferences — language, framework, database, hosting constraints
6. Write user stories — for each slice, a story in "As a [user], I want to [action], so that [outcome]" format
7. Output the manifest and user stories document

### Manifest Structure (`.cspec/specs/manifest.md`)

The manifest is the single source of truth for what was discovered:

- **Product summary** — Brief description of the product, target users, core value proposition
- **Tech preferences** — Any tech stack preferences or constraints expressed during discovery
- **Domain listing** — Each domain with a brief description of its scope
- **Slice inventory** — For each slice:
  - Name and domain
  - One-line description
  - Priority/ordering (which slices come first)
  - Dependencies (e.g., "requires auth slices to be built first")
  - Status (unwritten / written / foundation-reconciled / reviewed)

### User Stories (`.cspec/specs/user-stories.md`)

A separate, reviewable document containing all user stories grouped by domain. Each story maps to a slice in the manifest. User stories are the **source of truth** for what each slice should do — editing a story should be followed by re-running `/cspec-write` for the affected slice.

The manifest is updated by each phase:
- `/cspec-write` marks slices as "written"
- `/cspec-foundation` marks slices as "foundation-reconciled" if it modified them during conflict resolution, and adds a `foundation_derived: true` flag to the manifest
- `/cspec-review` marks slices as "reviewed"

## Phase 2: Spec Writing (`/cspec-write`)

### Invocation Modes

- **Single slice** — User picks a specific slice to spec out. The skill reads the manifest, focuses on that slice, and may ask clarifying questions before writing.
- **Batch** — The skill works through all unwritten slices in manifest order. It pauses after each slice for user review before proceeding to the next. Each slice is a separate write operation, so the user can stop the batch at any point and resume later by re-invoking the skill (it picks up unwritten slices from the manifest).

### Slice Spec Template (`.cspec/specs/<domain>/<slice>.md`)

Each spec uses a hybrid format — prose for narrative, structured/tabular for technical details.

**Prose sections:**

- **Purpose** — What this slice does and why it exists, from the user's perspective.
- **User Flow** — Step-by-step narrative of how a user moves through this feature.

**Structured sections:**

- **Data Requirements** — Tables defining entities, fields, types, constraints. Only entities this slice needs — the foundation reconciles shared ones later.
- **API Endpoints** — Method, path, request/response shapes, status codes.
- **State Transitions** — If applicable, the states an entity moves through and what triggers transitions.
- **Business Rules** — Enumerated rules and edge cases (e.g., "password must be 8+ characters," "free tier limited to 3 projects").
- **Error Scenarios** — What can go wrong, how it should be handled, what the user sees.
- **Acceptance Criteria** — Concrete, testable conditions that define "done" for this slice.

### Self-Containment Rule

Each slice spec must be fully understandable and buildable when paired only with the foundation spec:

- No references like "see the signup spec for the User model" — if this slice needs the User model, it describes it.
- Duplication across slices is expected and intentional at this stage.
- The foundation derivation step later reconciles these into single shared definitions.

## Phase 3: Foundation Derivation (`/cspec-foundation`)

### Process

1. **Read all written slice specs** — Parse every spec under `.cspec/specs/`.
2. **Extract commonalities** — Identify entities, services, patterns, and conventions that appear across multiple slices.
3. **Reconcile conflicts** — Identify conflicts across five types (data models, API conventions, business rules, behavior, terminology). For each: present the conflict, propose a resolution, get user approval, then write back.
4. **Determine architecture and tech stack** — Use manifest tech preferences as starting point. Interview the user for any missing decisions (language, framework, database, auth, hosting).
5. **Synthesize the foundation spec.**

### Foundation Spec Template (`.cspec/specs/foundation.md`)

**Prose sections:**

- **Product Overview** — Pulled from the manifest's product summary, expanded with context gained from the slice specs.
- **Architecture Overview** — High-level description of the system's shape (e.g., monolith vs. microservices, client-server split).

**Structured sections:**

- **Tech Stack** — Languages, frameworks, databases, key libraries, with rationale.
- **Project Structure** — Directory layout, naming conventions, file organization.
- **Shared Data Models** — Unified entity definitions extracted from across slices, with full field tables.
- **Shared Services** — Auth middleware, error handling, logging, API conventions, validation patterns — anything multiple slices rely on.
- **Environment & Configuration** — Environment variables, config file structure, secrets management approach.
- **Conventions** — Code style, naming, error response format, API versioning strategy, any other cross-cutting standards.

### Conflict Resolution

When the skill finds conflicting descriptions across slices (data models, API conventions, business rules, behavior, or terminology):

1. Present the conflict to the user with affected files.
2. Propose a resolution.
3. Ask the user to approve or modify the resolution.
4. Write back the resolved definition to affected slice specs and mark them as "foundation-reconciled" in the manifest. If a rename is involved, propagate across all sections of affected slices.

Note: `/cspec-foundation` has write access to slice spec files. After reconciliation, slice specs retain their self-contained descriptions (they are not stripped), but those descriptions are now consistent with the foundation.

## Phase 4: Review (`/cspec-review`)

### Validation Checks

**Completeness:**
- Every slice in the manifest has a written spec.
- Every spec has all required sections filled out (no empty stubs or TODOs).
- Acceptance criteria are concrete and testable, not vague.

**Consistency:**
- Data models in slice specs match their foundation definitions.
- API conventions (naming, error format, auth patterns) are uniform across slices.
- Terminology is consistent — the same concept isn't called "user" in one spec and "account" in another.

**Coverage:**
- No orphaned entities in the foundation that no slice actually uses.
- No missing slices — if a slice references behavior that doesn't exist in any spec (e.g., "user receives a notification" but there's no notification slice), flag it.
- Edge cases and error scenarios are addressed, not just happy paths.

**Implementation leakage:**
- No code snippets, pseudocode, algorithm descriptions, database queries/DDL, library choices, framework patterns, or internal function/class/method designs in any spec.
- No slices that split a single user action into technical approaches.

**Dependency integrity:**
- Slice ordering in the manifest is achievable — no circular dependencies.
- Each slice's declared dependencies actually exist and would be built before it.

### Output

The review produces a validation report written to `.cspec/specs/review-report.md` and summarized in the conversation:

- **Pass** — Everything checks out, specs are ready for implementation.
- **Issues** — Categorized findings (completeness, consistency, coverage, implementation leakage, dependency) with specific locations and suggested fixes. Each issue indicates which phase to re-run (e.g., "re-run `/cspec-write` for auth/login" or "re-run `/cspec-foundation`").

If issues are found, the user can fix them manually or re-run the indicated phases, then run `/cspec-review` again.

## Re-run Behavior

Each skill is idempotent when re-invoked:

- **`/cspec-discover`** — If a manifest already exists, the skill presents the existing slices and asks the user whether to start fresh or amend (add/remove/modify slices and user stories). Existing written specs are not deleted unless the user explicitly removes a slice.
- **`/cspec-write`** — Picks up unwritten slices from the manifest. If invoked for a slice that already has a spec, it overwrites with a new version.
- **`/cspec-foundation`** — Re-derives from the current state of all slice specs, overwriting the previous foundation.
- **`/cspec-review`** — Always runs a fresh validation pass, overwriting the previous report.

Version history is tracked by git — specs should be committed after each phase.

## Methodology Reference

The `docs/METHODOLOGY.md` file serves as the working reference for building the four skills. It captures the design decisions and principles documented here in a form useful during skill implementation. It is not consumed by the skills at runtime.
