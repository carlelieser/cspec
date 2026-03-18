---
name: cspec-review
description: Use when specs (slices and foundation) have been written and you need to cross-validate for consistency, completeness, and gaps. Activates after /cspec-foundation, when user wants to verify specs are ready for implementation.
---

# C-Spec Review

Cross-validates the entire spec suite for consistency, completeness, coverage, and dependency integrity. Produces a validation report at `.cspec/specs/review-report.md`.

## Prerequisites

Requires `.cspec/specs/foundation.md` and at least one slice spec. If the foundation doesn't exist, direct the user to run `/cspec-foundation` first.

Warn the user if the manifest shows unwritten slices — the review will be incomplete.

## Validation Checks

Run all five categories of checks against every spec file.

### Completeness

- Every slice in the manifest has a corresponding spec file at `.cspec/specs/<domain>/<slice>.md`.
- Every spec has all required sections filled: Purpose, User Flow, Data Requirements, API Endpoints, Business Rules, Error Scenarios, Acceptance Criteria.
- No empty stubs, placeholder text, or TODO markers.
- Acceptance criteria are concrete and testable — flag vague criteria like "should work well" or "handles errors gracefully."

### Consistency

- Data models in slice specs match their foundation definitions — field names, types, and constraints must align.
- API conventions are uniform across slices: naming patterns, error response format, authentication approach.
- Terminology is consistent — the same concept is not called "user" in one spec and "account" in another.

### Coverage

- No orphaned entities in the foundation that no slice references.
- No missing slices — if any spec references behavior not covered by another spec (e.g., "user receives a notification" but no notification slice exists), flag it.
- Edge cases and error scenarios are addressed, not just happy paths.
- No implicit features — behavior assumed but never specified in any spec.

### Implementation Leakage

- No code snippets, pseudocode, or algorithm descriptions in any spec.
- No specific library or package choices (e.g., "use bcrypt," "use Whisper API").
- No database queries or schema DDL.
- No framework-specific patterns (e.g., "use a React context provider").
- No internal function, class, or method designs.
- No slices that split a single user action into technical approaches (e.g., "full-text search" and "semantic search" instead of "user searches items"). Apply the test: could a non-technical user understand this slice name?
- Slice specs describe what the user experiences, not how the system delivers it.

### Dependency Integrity

- Slice ordering in the manifest has no circular dependencies.
- Each slice's declared dependencies exist in the manifest.
- Dependencies would be built before the slices that need them based on priority ordering.

## Review Report

Write the report to `.cspec/specs/review-report.md` and summarize findings in the conversation.

```markdown
# Spec Review Report

**Date:** [YYYY-MM-DD]
**Status:** PASS / ISSUES FOUND

## Summary

- Slices reviewed: [N]
- Foundation reviewed: yes / no
- Issues found: [N]

## Issues

### Completeness

**[Issue title]**
- **Location:** `.cspec/specs/[domain]/[slice].md`, section [name]
- **Finding:** [specific description of what's missing or inadequate]
- **Suggested fix:** [concrete recommendation]
- **Re-run:** `/cspec-write` for [slice-name]

### Consistency

**[Issue title]**
- **Location:** `.cspec/specs/[domain]/[slice].md` vs `.cspec/specs/foundation.md`
- **Finding:** [specific description of the inconsistency]
- **Suggested fix:** [concrete recommendation]
- **Re-run:** `/cspec-foundation`

### Coverage

**[Issue title]**
- **Location:** `.cspec/specs/[domain]/[slice].md`, section [name]
- **Finding:** [specific description of the gap]
- **Suggested fix:** [concrete recommendation]
- **Re-run:** `/cspec-discover` | `/cspec-write` for [slice-name]

### Implementation Leakage

**[Issue title]**
- **Location:** `.cspec/specs/[domain]/[slice].md`, section [name]
- **Finding:** [specific description of the implementation detail]
- **Suggested fix:** [concrete recommendation]
- **Re-run:** `/cspec-write` for [slice-name]

### Dependency

**[Issue title]**
- **Location:** `.cspec/specs/manifest.md`
- **Finding:** [specific description of the dependency issue]
- **Suggested fix:** [concrete recommendation]
- **Re-run:** `/cspec-discover`

## Passed Checks

- [List of checks that passed]
```

Omit any category section that has no issues.

## Issue Routing

Each issue must indicate which phase to re-run:

- **Missing slices** (not in manifest) → `/cspec-discover`
- **Incomplete or incorrect slice specs** → `/cspec-write` for the specific slice
- **Implementation details in specs** → `/cspec-write` for the specific slice
- **Foundation gaps or model inconsistencies** → `/cspec-foundation`
- **Dependency ordering problems** → `/cspec-discover`

## Re-run Behavior

Always runs a fresh validation pass. Overwrites the previous `.cspec/specs/review-report.md`.

## Completion

- **If PASS:** "All specs validated. Run `/cspec-blueprint` to generate implementation blueprints."
- **If ISSUES FOUND:** Present a summary of issue counts by category in the conversation. Direct the user to `.cspec/specs/review-report.md` for full details and suggested re-runs. After fixes, the user should run `/cspec-review` again.

After passing review, update all slice statuses to `reviewed` in the manifest.
