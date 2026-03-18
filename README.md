# C-Spec

AI agents build better software when they get better specs. But most specifications are either monolithic documents too large to fit in context, or scattered notes that leave agents guessing about data models, edge cases, and how features connect. The result: inconsistent implementations, missing error handling, and features that don't work together.

C-Spec is a methodology and set of five skills that produce complete, interconnected specifications for any MVP. Instead of one massive document, C-Spec generates a series of **self-contained vertical slices** — each describing a single user flow end-to-end — plus a **foundation spec** that captures shared infrastructure. An agent picks up one slice, pairs it with the foundation, and has everything it needs to build that feature without knowledge of any other slice.

## Install

```bash
npx skills add carlelieser/cspec
```

## Usage

C-Spec works in five phases, each invoked as a separate skill:

### 1. Discover

```
/cspec-discover
```

Interview-driven or document-driven discovery of your product idea. Identifies all vertical slices (user-story granularity), groups them into domains, writes user stories, and outputs a structured manifest to `.cspec/specs/manifest.md` and user stories to `.cspec/specs/user-stories.md`. Review and adjust the user stories before moving to the next phase — they're the source of truth for what each slice should do.

### 2. Write

```
/cspec-write
```

Writes detailed specs for each slice — one at a time or in batch. Each spec includes purpose, user flow, data models, API endpoints, business rules, error scenarios, and acceptance criteria. Every spec is fully self-contained.

### 3. Foundation

```
/cspec-foundation
```

Reads all written slice specs, extracts shared entities and patterns, reconciles conflicts, and synthesizes `.cspec/specs/foundation.md` — the tech stack, project structure, shared data models, conventions, and services that all slices depend on.

### 4. Review

```
/cspec-review
```

Cross-validates every spec for completeness, consistency, coverage, and dependency integrity. Produces a report at `.cspec/specs/review-report.md` with categorized findings and tells you exactly which phase to re-run for each issue.

### 5. Blueprint

```
/cspec-blueprint
```

Reads the foundation and reviewed slice specs, then produces implementation blueprints — backend class contracts and frontend component contracts. Works in three stages: analysis (identifies class and component candidates), skeleton (shared architecture), then per-slice artifacts. Also works standalone with any well-structured spec files.

## Output

All specs live under `.cspec/specs/` in your project, organized by domain:

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
  plans/
    backend-skeleton.md
    frontend-skeleton.md
    backend/
      auth/
        signup.md
      billing/
        checkout.md
    frontend/
      auth/
        signup.md
      billing/
        checkout.md
```

## No Implementation Details

C-Spec deliberately excludes implementation details from every spec it produces. This is the most important rule in the methodology.

**The test:** Could you explain this slice to a non-technical user and have it make sense? If you need to explain a technical concept for the slice to make sense, it contains implementation details.

**Why:**

- **Separation of concerns.** Specs define *what* the user experiences. Implementation defines *how* the system delivers it. Mixing them couples decisions that should be independent.
- **No premature decisions.** Implementation choices made during spec writing are uninformed — you don't yet know what shared patterns will emerge, what the foundation will look like, or what the tech stack constrains.
- **Context efficiency.** Implementation detail bloats specs, eating context window that agents need for the actual build.

**In practice**, this means slices describe what the user does, not what the system does under the hood:

| Allowed (user experience) | Banned (implementation detail) |
|---------------------------|-------------------------------|
| "User searches their items" | "Full-text search" and "Semantic search" as separate slices |
| "User records audio, transcription starts automatically" | "Use Whisper API for transcription" |
| "User signs up with email" | "Hash password with bcrypt" |
| "User chats with an AI assistant" | "RAG pipeline with vector embeddings" |

A single user action should be a single slice — even if the system uses multiple technical approaches to fulfill it. Splitting "user searches" into "full-text search" and "semantic search" is an architectural decision disguised as a spec. The spec should say the user can search; implementation decides how.

## Principles

- **Self-contained slices** — Each spec is buildable with only the foundation. No cross-slice knowledge required.
- **Foundation derived, not designed** — Shared infrastructure is extracted from slices after they're written, not imposed upfront.
- **User-story granularity** — One user flow per spec. "User signs up with email," not "authentication system."
- **No implementation details** — Specs describe what the user experiences, not how the system delivers it. See above.
- **Intentional duplication** — Slices describe their own entities independently. The foundation phase reconciles them.
