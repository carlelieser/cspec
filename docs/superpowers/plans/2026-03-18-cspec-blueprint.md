# `/cspec-blueprint` Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Claude skill (`/cspec-blueprint`) that produces backend class contracts and frontend component contracts from behavioral specs — either as Phase 5 of the C-Spec pipeline or standalone.

**Architecture:** Single skill in its own directory under `skills/`. SKILL.md with YAML frontmatter follows the patterns established by the four existing C-Spec skills. Reads from `.cspec/` (or user-provided spec files) and writes to `.cspec/plans/`.

**Tech Stack:** Claude skill (markdown with YAML frontmatter), no code dependencies.

**References:**
- Design spec: `docs/superpowers/specs/2026-03-18-cspec-blueprint-design.md`
- Methodology: `docs/METHODOLOGY.md`
- Existing skills for pattern reference: `skills/cspec-write/SKILL.md`, `skills/cspec-foundation/SKILL.md`, `skills/cspec-review/SKILL.md`

---

## Chunk 1: The `/cspec-blueprint` Skill

### Task 1: Write the `/cspec-blueprint` SKILL.md

**Files:**
- Create: `skills/cspec-blueprint/SKILL.md`

- [ ] **Step 1: Write the SKILL.md frontmatter and overview**

```yaml
---
name: cspec-blueprint
description: Use when specs are complete and you need implementation blueprints — backend class contracts and frontend component contracts. Activates after /cspec-review passes, or standalone with any well-structured spec files.
---
```

Follow with a one-line overview: produces backend system designs (class contracts) and frontend component definitions from behavioral specs.

- [ ] **Step 2: Write the Prerequisites section**

The skill must cover (check in this order):

- C-Spec mode: first check for `.cspec/manifest.md` — if missing, direct to `/cspec-discover`. Then check for `.cspec/foundation.md` — if missing, direct to `/cspec-foundation`. Then check for slices in `reviewed` status — if none, direct to `/cspec-review`.
- Standalone mode: accepts any well-structured spec files the user provides.
- Warn if manifest shows slices not yet in `reviewed` status — those will be skipped.

Pattern reference: `skills/cspec-review/SKILL.md` Prerequisites section (closest match — also requires foundation + reviewed slices).

- [ ] **Step 3: Write the Input Handling section**

Two subsections:

**C-Spec Mode (Phase 5):**
- Reads `.cspec/foundation.md` for architecture, tech stack, shared data models, conventions
- Reads all slice specs under `.cspec/<domain>/<slice>.md`
- Reads `.cspec/manifest.md` for ordering and dependencies
- Reads `.cspec/user-stories.md` for purpose and outcome context behind each slice
- Individual slices must be in `reviewed` status to be blueprinted

**Standalone Mode:**
- Accepts any well-structured spec files
- Scans for: data models, API contracts, user flows, business rules
- Prompts for missing information required to produce contracts
- Creates `.cspec/plans/` for output even when no `.cspec/` exists
- Minimum input: data models + API contracts for backend; user flows + page/screen descriptions for frontend

- [ ] **Step 4: Write the Artifact Type Detection section**

- C-Spec mode: reads foundation's Architecture Overview and Tech Stack to determine backend, frontend, or both
- Standalone mode: infers from specs or asks the user directly
- Per-slice: if a slice has no frontend aspect (e.g., webhook handler) or no backend aspect (e.g., client-side toggle), only the applicable artifact is produced with a note explaining the omission

- [ ] **Step 5: Write Stage 1 — Read & Analyze**

Six numbered steps:
1. Read all inputs (foundation + slice specs + user stories, or user-provided files)
2. Identify backend candidates — data models, services, repositories, controllers/handlers with preliminary role assignments (entity, service, repository, controller, utility)
3. Identify frontend candidates — pages/screens, forms, lists, detail views, navigation, shared UI patterns
4. Map dependencies — class-to-class and component-to-component
5. Identify shared patterns — base classes, interfaces, shared component patterns. Prefer flat hierarchies (one level of abstraction)
6. Present findings — summarize candidate counts and breakdown, let user adjust before proceeding

Stage completion message:
> "Analysis complete: [N] backend class candidates and [N] frontend component candidates identified across [M] domains. Review the breakdown above and let me know if you'd like to adjust anything before I generate the skeletons."

This stage produces no files.

- [ ] **Step 6: Write Stage 2 — Skeleton**

Two subsections with templates:

**Backend Skeleton** (`.cspec/plans/backend-skeleton.md`):
- Architecture Pattern — structural pattern and how layers relate
- Shared Interfaces — interfaces/abstract types multiple slices implement
- Base Classes — abstract classes with public method signatures
- Shared Services — classes serving multiple slices with contracts
- Dependency Graph — how class categories relate (controllers → services → repositories → entities)
- Each entry: name, responsibility (one sentence), public method signatures (inputs and return types), which slices use it

**Frontend Skeleton** (`.cspec/plans/frontend-skeleton.md`):
- Component Hierarchy — top-level layout tree (app shell, navigation, page containers, shared regions)
- Shared Components — reusable components with props interfaces
- State Patterns — global vs. local state, access patterns
- Routing Structure — page-level component-to-route mapping
- Each entry: name, purpose (one sentence), props interface, key state, which slices use it

Stage completion message:
> "Skeletons written to `.cspec/plans/`. Review the backend and frontend architecture above, then confirm to proceed with per-slice blueprints."

Only applicable skeletons produced (based on artifact type detection).

- [ ] **Step 7: Write Stage 3 — Per-Slice Artifacts**

Two subsections with templates:

**Backend Slice** (`.cspec/plans/backend/<domain>/<slice>.md`):
- Classes — each class with: name, responsibility, skeleton class/interface it extends (if any), public method signatures (name, params with types, return type, one-line description), dependencies
- Slice-Specific Business Logic — business rules mapped to class methods
- Error Handling — error scenarios mapped to exceptions/error types and responsible class

**Frontend Slice** (`.cspec/plans/frontend/<domain>/<slice>.md`):
- Components — each with: name, purpose, parent component, props interface (name, type, required/optional, description), key state (name, type, change triggers), skeleton components it uses
- User Flow Mapping — spec user flow steps mapped to component interactions
- Error States — error scenarios mapped to UI states and responsible component

**Batch Behavior:**
- Processes slices in manifest priority order (C-Spec) or dependency order (standalone)
- Pauses after each slice for user review
- Resumable — re-invoking picks up `unplanned` slices

Per-slice completion message:
> "[Slice name] blueprinted — backend: [N] classes, frontend: [N] components. Review above, then confirm to continue with the next slice."

- [ ] **Step 8: Write the Output Structure section**

Show the directory tree:
```
.cspec/plans/
  backend-skeleton.md
  frontend-skeleton.md
  backend/
    <domain>/
      <slice>.md
  frontend/
    <domain>/
      <slice>.md
```

Note: for backend-only or frontend-only products, only applicable skeleton and subdirectory are produced.

- [ ] **Step 9: Write the Status Tracking section**

- C-Spec mode: adds a `Blueprint` column to manifest's Slice Inventory table
- On first run, backfills all existing slices as `unplanned`
- Only `reviewed` slices can transition to `planned`
- Standalone mode: no status tracking, always regenerates

| Status | Meaning |
|--------|---------|
| `unplanned` | No blueprint written yet |
| `planned` | Applicable artifacts written for this slice |

- [ ] **Step 10: Write the Re-run Behavior section**

- Skeletons: regenerates from current spec state, overwrites previous
- Per-slice: picks up `unplanned` slices; overwrites if re-run on existing
- Standalone: always regenerates

Pattern reference: `skills/cspec-write/SKILL.md` Re-run Behavior section.

- [ ] **Step 11: Write the Completion section**

Final message after all slices processed:
> "Implementation blueprints complete. Artifacts written to `.cspec/plans/` for [N] slices ([N] backend, [N] frontend). The specs are now ready to hand off to an implementation agent."

- [ ] **Step 12: Review the skill against the design spec**

Read `docs/superpowers/specs/2026-03-18-cspec-blueprint-design.md` and verify every requirement is addressed in the SKILL.md. Check:
- All three stages present with correct content
- Both artifact type templates complete (backend and frontend)
- Artifact type detection logic present
- Both input modes covered (C-Spec and standalone)
- Status tracking, re-run behavior, and completion messages match spec
- Stage completion messages present for all three stages
- Standalone mode specifics: `.cspec/plans/` directory creation when no `.cspec/` exists, minimum input requirements for each artifact type, prompting behavior for missing information
- Prerequisites check manifest → foundation → reviewed slices in that order

- [ ] **Step 13: Commit**

```bash
git add skills/cspec-blueprint/SKILL.md
git commit -m "feat: add /cspec-blueprint skill for implementation blueprints"
```

---

## Chunk 2: Update Existing Documentation

### Task 2: Update the Slice Status Lifecycle in `/cspec-discover`

**Files:**
- Modify: `skills/cspec-discover/SKILL.md` (Slice Status Lifecycle table and Transitions note)

- [ ] **Step 1: Add blueprint statuses to the lifecycle table**

Add two rows to the Slice Status Lifecycle table in `skills/cspec-discover/SKILL.md`:

| Status | Set by | Meaning |
|--------|--------|---------|
| `unplanned` | `/cspec-blueprint` | Spec reviewed, no blueprint written yet |
| `planned` | `/cspec-blueprint` | Backend + frontend blueprint artifacts written |

- [ ] **Step 2: Append to the Transitions note**

Append to the existing Transitions description (do not replace the existing text). After the existing chain (`unwritten` → `written` → `reviewed`), add: `reviewed` → `unplanned` (set by `/cspec-blueprint` on first run, backfills all reviewed slices) → `planned` (after blueprint artifacts are written). The full chain should now read: `unwritten` → `written` → `reviewed` → `unplanned` → `planned`.

- [ ] **Step 3: Commit**

```bash
git add skills/cspec-discover/SKILL.md
git commit -m "feat: add blueprint statuses to slice status lifecycle"
```

---

### Task 3: Update `/cspec-review` completion message

**Files:**
- Modify: `skills/cspec-review/SKILL.md` (Completion section)

- [ ] **Step 1: Update the PASS completion message**

Change the existing PASS message from:
> "All specs validated. The spec suite is ready for implementation."

To:
> "All specs validated. Run `/cspec-blueprint` to generate implementation blueprints."

This follows the pipeline pattern where each skill's completion message directs users to the next phase (e.g., `/cspec-write` → `/cspec-foundation` → `/cspec-review` → `/cspec-blueprint`).

- [ ] **Step 2: Commit**

```bash
git add skills/cspec-review/SKILL.md
git commit -m "feat: update /cspec-review completion to point to /cspec-blueprint"
```

---

### Task 4: Update the Methodology doc

**Files:**
- Modify: `docs/METHODOLOGY.md`

- [ ] **Step 1: Update the title from "Four Phases" to "Five Phases"**

Change the `## Four Phases` heading to `## Five Phases`.

- [ ] **Step 2: Add Phase 5 to the phase table**

Add a row:

| Phase | Skill | Input | Output |
|-------|-------|-------|--------|
| 5. Blueprint | `/cspec-blueprint` | Foundation + reviewed slice specs | `.cspec/plans/backend-skeleton.md`, `.cspec/plans/frontend-skeleton.md`, per-slice artifacts |

- [ ] **Step 3: Add `.cspec/plans/` to the Output Structure**

Add the plans directory to the output structure tree:

```
.cspec/
  ...existing entries...
  plans/
    backend-skeleton.md
    frontend-skeleton.md
    backend/
      <domain>/
        <slice>.md
    frontend/
      <domain>/
        <slice>.md
```

- [ ] **Step 4: Add skill location**

Add `cspec-blueprint/SKILL.md` to the Skill Locations section.

- [ ] **Step 5: Add re-run behavior entry**

Add under Re-run Behavior:
- `/cspec-blueprint` — Re-generates skeletons from current specs. Picks up unplanned slices; overwrites if re-run on existing.

- [ ] **Step 6: Update the Statuses description**

The existing Statuses line (around line 43 of `docs/METHODOLOGY.md`) describes `unwritten` → `written` → `reviewed`. Extend it to include `unplanned` and `planned` statuses set by `/cspec-blueprint`.

- [ ] **Step 7: Commit**

```bash
git add docs/METHODOLOGY.md
git commit -m "docs: add Phase 5 (blueprint) to methodology"
```

---

### Task 5: Update the README

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Update introductory text**

Change "a methodology and set of four skills" to "a methodology and set of five skills" in the opening paragraph. Change "C-Spec works in four phases" to "C-Spec works in five phases" in the Usage section heading.

- [ ] **Step 2: Add Phase 5 usage section**

After the Phase 4 (Review) section, add:

```markdown
### 5. Blueprint

```
/cspec-blueprint
```

Reads the foundation and reviewed slice specs, then produces implementation blueprints — backend class contracts and frontend component contracts. Works in three stages: analysis (identifies class and component candidates), skeleton (shared architecture), then per-slice artifacts. Also works standalone with any well-structured spec files.
```

- [ ] **Step 3: Update the Output section**

Add `.cspec/plans/` directory to the output tree showing `backend-skeleton.md`, `frontend-skeleton.md`, and per-domain backend/frontend subdirectories.

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs: add /cspec-blueprint to README usage guide"
```
