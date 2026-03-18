# C-Spec

AI agents build better software when they get better specs. But most specifications are either monolithic documents too large to fit in context, or scattered notes that leave agents guessing about data models, edge cases, and how features connect. The result: inconsistent implementations, missing error handling, and features that don't work together.

C-Spec is a methodology and set of four skills that produce complete, interconnected specifications for any MVP. Instead of one massive document, C-Spec generates a series of **self-contained vertical slices** — each describing a single user flow end-to-end — plus a **foundation spec** that captures shared infrastructure. An agent picks up one slice, pairs it with the foundation, and has everything it needs to build that feature without knowledge of any other slice.

## Install

```bash
npx skills add carlelieser/cspec
```

## Usage

C-Spec works in four phases, each invoked as a separate skill:

### 1. Discover

```
/cspec-discover
```

Interview-driven or document-driven discovery of your product idea. Identifies all vertical slices (user-story granularity), groups them into domains, writes user stories, and outputs a structured manifest to `.cspec/manifest.md` and user stories to `.cspec/user-stories.md`. Review and adjust the user stories before moving to the next phase — they're the source of truth for what each slice should do.

### 2. Write

```
/cspec-write
```

Writes detailed specs for each slice — one at a time or in batch. Each spec includes purpose, user flow, data models, API endpoints, business rules, error scenarios, and acceptance criteria. Every spec is fully self-contained.

### 3. Foundation

```
/cspec-foundation
```

Reads all written slice specs, extracts shared entities and patterns, reconciles conflicts, and synthesizes `.cspec/foundation.md` — the tech stack, project structure, shared data models, conventions, and services that all slices depend on.

### 4. Review

```
/cspec-review
```

Cross-validates every spec for completeness, consistency, coverage, and dependency integrity. Produces a report at `.cspec/review-report.md` with categorized findings and tells you exactly which phase to re-run for each issue.

## Output

All specs live under `.cspec/` in your project, organized by domain:

```
.cspec/
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
```

## Principles

- **Self-contained slices** — Each spec is buildable with only the foundation. No cross-slice knowledge required.
- **Foundation derived, not designed** — Shared infrastructure is extracted from slices after they're written, not imposed upfront.
- **User-story granularity** — One user flow per spec. "User signs up with email," not "authentication system."
- **No implementation code** — Specs define behavior, data, and acceptance criteria. Implementation is a separate concern.
- **Intentional duplication** — Slices describe their own entities independently. The foundation phase reconciles them.
