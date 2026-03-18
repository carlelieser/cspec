---
name: cspec-write
description: Use when writing detailed specifications for discovered feature slices. Activates after /cspec-discover has produced a manifest, when user wants to write or rewrite individual slice specs or batch-write all unwritten slices.
---

# C-Spec Write

Writes self-contained slice specs from the manifest. Each spec describes one vertical user flow with enough detail for an agent to build it independently.

## Prerequisites

Requires `.cspec/manifest.md`. If not found, direct the user to run `/cspec-discover` first.

## Invocation Modes

- **Single** — User specifies a slice name, or present the list of unwritten slices for selection.
- **Batch** — Work through all unwritten slices in manifest priority order. Pause after each slice for user review before proceeding to the next. Resumable — re-invoking picks up unwritten slices from the manifest.

If the user doesn't specify a mode, ask which they prefer.

## Spec Writing Process (per slice)

1. Read the manifest for this slice's context (description, domain, dependencies).
2. If the slice description is too brief to write a full spec, ask clarifying questions — one at a time, prefer multiple choice.
3. Write the spec to `.cspec/<domain>/<slice>.md` using the template below. Create the domain directory if it doesn't exist.
4. Update the slice's status to `written` in `.cspec/manifest.md`.
5. In batch mode: present the written spec to the user and wait for approval before moving to the next slice.

## Self-Containment Rule

Each slice spec must be fully understandable and buildable when paired only with `.cspec/foundation.md`. This means:

- **No cross-slice references.** Never write "see the signup spec for the User model." If this slice needs the User model, describe it here.
- **Duplication is intentional.** Multiple slices may describe the same entity. This is expected — the foundation derivation phase reconciles them later.
- **Include everything.** Data models, API shapes, business rules, error handling — if an agent needs it to build this slice, it must be in this spec.

## Slice Spec Template

Write to `.cspec/<domain>/<slice>.md`:

```markdown
# [Slice Name]

## Purpose

[What this slice does and why it exists, from the user's perspective]

## User Flow

[Step-by-step narrative of how a user moves through this feature]

1. User does [action]
2. System responds with [response]
3. ...

## Data Requirements

### [Entity Name]

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | uuid | PK | Unique identifier |
| ... | ... | ... | ... |

## API Endpoints

### [Endpoint Name]

- **Method:** GET / POST / PUT / DELETE
- **Path:** `/api/...`
- **Auth:** [required / public]
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
  - 404: [description]

## State Transitions

[If applicable — describe the states an entity moves through and what triggers each transition. Omit this section if no state machine applies.]

## Business Rules

1. [Specific rule with concrete values/thresholds]
2. [Specific rule with concrete values/thresholds]

## Error Scenarios

| Scenario | Cause | User Sees | System Does |
|----------|-------|-----------|-------------|
| [name] | [cause] | [error message or UI state] | [system action] |

## Acceptance Criteria

- [ ] [Concrete, testable condition]
- [ ] [Concrete, testable condition]
- [ ] [Concrete, testable condition]
```

### Template Notes

- **All sections are required** except State Transitions (include only when applicable).
- **Acceptance Criteria** must be concrete and testable — not vague statements like "should work well" or "handles errors gracefully."
- **Data Requirements** tables must include types and constraints, not just field names.
- **API Endpoints** must include request/response shapes, not just paths.

## Re-run Behavior

If a spec already exists for the target slice, overwrite it with the new version.

## Completion

After all target slices are written, inform the user:

> "[N] slice specs written. Run `/cspec-foundation` to derive the shared foundation spec."
