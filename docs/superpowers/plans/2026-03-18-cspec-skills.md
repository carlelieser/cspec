# C-Spec Skills Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build four Claude skills (`/cspec-discover`, `/cspec-write`, `/cspec-foundation`, `/cspec-review`) that produce complete, interconnected MVP specifications.

**Architecture:** Four independent skills, each in its own directory under `~/.claude/skills/`. Each skill has a `SKILL.md` with YAML frontmatter and follows the methodology defined in `docs/METHODOLOGY.md`. Skills read/write to `.cspec/specs/` in the user's working directory.

**Tech Stack:** Claude skills (markdown with YAML frontmatter), no code dependencies.

**References:**
- Design spec: `docs/superpowers/specs/2026-03-18-cspec-design.md`
- Methodology: `docs/METHODOLOGY.md`
- Skill authoring guide: superpowers:writing-skills

---

## Chunk 1: `/cspec-discover`

### Task 1: Write the `/cspec-discover` skill

**Files:**
- Create: `~/.claude/skills/cspec-discover/SKILL.md`

- [ ] **Step 1: Write the SKILL.md**

The skill must cover:

**Frontmatter:**
```yaml
---
name: cspec-discover
description: Use when starting a new product specification, gathering requirements for an MVP, or identifying feature slices for a project. Activates when user wants to define what to build before writing detailed specs.
---
```

**Content sections:**

1. **Overview** — One-line: discovers and inventories all vertical slices for an MVP, outputting a manifest.

2. **Input Mode Detection** — How the skill determines its mode:
   - If user provides a document (file path or pasted content) → document-driven or hybrid
   - Otherwise → interactive interview

3. **Interactive Process** — Step-by-step:
   - Understand the product (what, who, why)
   - Identify user-facing features
   - Break features into vertical slices (user-story granularity — one user flow per slice)
   - Group slices into domains
   - Determine ordering based on dependencies
   - Ask one question at a time, prefer multiple choice

4. **Document-Driven Process** — Step-by-step:
   - Read and analyze the provided document
   - Extract slices and domains
   - Present findings to user for validation
   - Ask follow-up questions to fill gaps (hybrid mode)

5. **Manifest Output** — Exact template for `.cspec/specs/manifest.md`:
   ```markdown
   # [Product Name] — Manifest

   ## Product Summary
   [Brief description, target users, core value proposition]

   ## Tech Preferences
   [Stack preferences or constraints, or "None specified"]

   ## Domains

   ### [Domain Name]
   [Brief description of domain scope]

   ## Slice Inventory

   | Slice | Domain | Description | Priority | Dependencies | Status |
   |-------|--------|-------------|----------|--------------|--------|
   | slice-name | domain | One-line description | 1 | none | unwritten |
   ```

6. **Re-run Behavior** — If manifest exists: present existing slices, ask whether to start fresh or amend.

7. **Completion** — After manifest is written, inform user to run `/cspec-write` next.

- [ ] **Step 2: Review the skill against the design spec**

Read `docs/superpowers/specs/2026-03-18-cspec-design.md` Phase 1 section and verify every requirement is addressed in the SKILL.md.

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/skills/cspec-discover/SKILL.md
git commit -m "feat: add /cspec-discover skill for MVP slice discovery"
```

---

## Chunk 2: `/cspec-write`

### Task 2: Write the `/cspec-write` skill

**Files:**
- Create: `~/.claude/skills/cspec-write/SKILL.md`

- [ ] **Step 1: Write the SKILL.md**

**Frontmatter:**
```yaml
---
name: cspec-write
description: Use when writing detailed specifications for discovered feature slices. Activates after /cspec-discover has produced a manifest, when user wants to write or rewrite individual slice specs or batch-write all unwritten slices.
---
```

**Content sections:**

1. **Overview** — Writes self-contained slice specs from the manifest.

2. **Prerequisites** — Must have `.cspec/specs/manifest.md`. If not found, direct user to run `/cspec-discover` first.

3. **Invocation Modes:**
   - **Single** — User specifies a slice name or the skill presents unwritten slices for selection.
   - **Batch** — Works through all unwritten slices in manifest priority order. Pauses after each slice for user review before proceeding. Resumable (picks up from manifest status).

4. **Spec Writing Process** (per slice):
   - Read manifest for slice context (description, domain, dependencies)
   - Ask clarifying questions if needed (one at a time)
   - Write the spec following the template
   - Update manifest status to "written"

5. **Slice Spec Template** — Exact template for `.cspec/specs/<domain>/<slice>.md`:
   ```markdown
   # [Slice Name]

   ## Purpose
   [What this slice does and why, from the user's perspective]

   ## User Flow
   [Step-by-step narrative of how a user moves through this feature]

   ## Data Requirements

   ### [Entity Name]
   | Field | Type | Constraints | Description |
   |-------|------|-------------|-------------|
   | id | uuid | PK | Unique identifier |

   ## API Endpoints

   ### [Endpoint Name]
   - **Method:** GET/POST/PUT/DELETE
   - **Path:** `/api/...`
   - **Request Body:**
     ```json
     {}
     ```
   - **Response (200):**
     ```json
     {}
     ```
   - **Error Responses:**
     - 400: [description]
     - 401: [description]

   ## State Transitions
   [If applicable — states, triggers, diagram]

   ## Business Rules
   1. [Rule with specific values/thresholds]
   2. [Rule with specific values/thresholds]

   ## Error Scenarios
   | Scenario | Cause | User Sees | System Does |
   |----------|-------|-----------|-------------|
   | [name] | [cause] | [message] | [action] |

   ## Acceptance Criteria
   - [ ] [Concrete, testable condition]
   - [ ] [Concrete, testable condition]
   ```

6. **Self-Containment Rule** — Explicit instruction: each spec must be fully buildable paired only with foundation.md. No cross-slice references. If this slice needs an entity described in another slice, describe it again here.

7. **Re-run Behavior** — If spec exists for a slice, overwrite with new version.

8. **Completion** — After all slices written, inform user to run `/cspec-foundation` next.

- [ ] **Step 2: Review the skill against the design spec**

Read `docs/superpowers/specs/2026-03-18-cspec-design.md` Phase 2 section and verify every requirement is addressed.

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/skills/cspec-write/SKILL.md
git commit -m "feat: add /cspec-write skill for slice spec authoring"
```

---

## Chunk 3: `/cspec-foundation`

### Task 3: Write the `/cspec-foundation` skill

**Files:**
- Create: `~/.claude/skills/cspec-foundation/SKILL.md`

- [ ] **Step 1: Write the SKILL.md**

**Frontmatter:**
```yaml
---
name: cspec-foundation
description: Use when all feature slice specs have been written and you need to derive the shared foundation spec. Activates after /cspec-write has completed all slices, when user wants to extract shared infrastructure, data models, and conventions.
---
```

**Content sections:**

1. **Overview** — Derives the foundation spec by reading all slice specs, extracting shared concerns, and reconciling conflicts.

2. **Prerequisites** — Must have `.cspec/specs/manifest.md` with at least some slices in "written" status. Warn if not all slices are written.

3. **Derivation Process:**
   - Read all slice specs under `.cspec/specs/`
   - Extract entities, services, and patterns that appear in multiple slices
   - Identify conflicts (same entity with different fields across slices)
   - For each conflict: flag to user, propose resolution, get approval
   - Update affected slice specs to match resolved definitions
   - Mark updated slices as "foundation-reconciled" in manifest
   - Synthesize the foundation spec
   - Set `foundation_derived: true` in manifest

4. **Foundation Spec Template** — Exact template for `.cspec/specs/foundation.md`:
   ```markdown
   # [Product Name] — Foundation

   ## Product Overview
   [Expanded from manifest summary with context from slice specs]

   ## Architecture Overview
   [System shape: monolith/microservices, client-server, etc.]

   ## Tech Stack
   | Layer | Technology | Rationale |
   |-------|-----------|-----------|
   | Language | | |
   | Framework | | |
   | Database | | |
   | Auth | | |

   ## Project Structure
   ```
   project-root/
     src/
       ...
   ```

   ## Shared Data Models

   ### [Entity Name]
   | Field | Type | Constraints | Description |
   |-------|------|-------------|-------------|
   | | | | |

   **Used by:** [list of slices that reference this entity]

   ## Shared Services
   [Auth middleware, error handling, logging, API conventions, validation]

   ## Environment & Configuration
   [Env vars, config structure, secrets management]

   ## Conventions
   [Code style, naming, error response format, API versioning]
   ```

5. **Conflict Resolution Protocol** — Step-by-step:
   - Present the conflict (entity name, differing fields, which slices)
   - Propose a merged definition
   - Ask user to approve or modify
   - Write back to affected slice specs
   - Note: slice specs keep their entity descriptions (not stripped), but descriptions are now consistent

6. **Re-run Behavior** — Re-derives from current slice state, overwrites previous foundation.

7. **Completion** — After foundation written, inform user to run `/cspec-review` next.

- [ ] **Step 2: Review the skill against the design spec**

Read `docs/superpowers/specs/2026-03-18-cspec-design.md` Phase 3 section and verify every requirement is addressed.

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/skills/cspec-foundation/SKILL.md
git commit -m "feat: add /cspec-foundation skill for shared infrastructure derivation"
```

---

## Chunk 4: `/cspec-review`

### Task 4: Write the `/cspec-review` skill

**Files:**
- Create: `~/.claude/skills/cspec-review/SKILL.md`

- [ ] **Step 1: Write the SKILL.md**

**Frontmatter:**
```yaml
---
name: cspec-review
description: Use when all specs (slices and foundation) have been written and you need to cross-validate for consistency, completeness, and gaps. Activates after /cspec-foundation, when user wants to verify specs are ready for implementation.
---
```

**Content sections:**

1. **Overview** — Cross-validates the entire spec suite for consistency, completeness, coverage, and dependency integrity.

2. **Prerequisites** — Must have `.cspec/specs/foundation.md` and at least one slice spec. Warn if manifest shows unwritten slices.

3. **Validation Checks** — Four categories, each with specific checks:

   **Completeness:**
   - Every slice in manifest has a written spec file
   - Every spec has all required sections filled (Purpose, User Flow, Data Requirements, API Endpoints, Business Rules, Error Scenarios, Acceptance Criteria)
   - No empty stubs or TODO placeholders
   - Acceptance criteria are concrete and testable (not vague like "should work well")

   **Consistency:**
   - Data models in slice specs match foundation definitions (field names, types, constraints)
   - API conventions uniform across slices (naming, error format, auth patterns)
   - Terminology consistent (same concept not called different names across specs)

   **Coverage:**
   - No orphaned entities in foundation (defined but unused by any slice)
   - No missing slices (slice references behavior not covered by any spec, e.g., "sends notification" but no notification slice)
   - Edge cases and error scenarios addressed, not just happy paths
   - No implicit features (behavior assumed but never specified)

   **Dependency Integrity:**
   - Slice ordering in manifest has no circular dependencies
   - Each slice's declared dependencies exist in the manifest
   - Dependencies would be built before the slices that need them

4. **Report Template** — Exact template for `.cspec/specs/review-report.md`:
   ```markdown
   # Spec Review Report

   **Date:** [date]
   **Status:** PASS / ISSUES FOUND

   ## Summary
   - Slices reviewed: [N]
   - Foundation reviewed: yes/no
   - Issues found: [N]

   ## Issues

   ### [Category: Completeness/Consistency/Coverage/Dependency]

   **[Issue title]**
   - **Location:** `.cspec/specs/domain/slice.md`, section [name]
   - **Finding:** [specific description]
   - **Suggested fix:** [concrete recommendation]
   - **Re-run:** `/cspec-write` for [slice] | `/cspec-foundation` | `/cspec-discover`

   ## Passed Checks
   [List of checks that passed]
   ```

5. **Issue Routing** — Each issue must indicate which phase to re-run:
   - Missing slices → `/cspec-discover`
   - Incomplete/incorrect slice specs → `/cspec-write` for specific slice
   - Foundation gaps or inconsistencies → `/cspec-foundation`

6. **Re-run Behavior** — Always runs a fresh validation, overwrites previous report.

7. **Completion:**
   - If PASS: "All specs validated. Ready for implementation."
   - If ISSUES: Present summary in conversation, direct user to the report for details and suggested re-runs.

- [ ] **Step 2: Review the skill against the design spec**

Read `docs/superpowers/specs/2026-03-18-cspec-design.md` Phase 4 section and verify every requirement is addressed.

- [ ] **Step 3: Commit**

```bash
git add ~/.claude/skills/cspec-review/SKILL.md
git commit -m "feat: add /cspec-review skill for cross-spec validation"
```
