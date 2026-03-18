---
name: cspec-discover
description: Use when starting a new product specification, gathering requirements for an MVP, or identifying feature slices for a project. Activates when user wants to define what to build before writing detailed specs.
---

# C-Spec Discover

Discovers and inventories all vertical slices for an MVP, outputting a structured manifest to `.cspec/manifest.md` and user stories to `.cspec/user-stories.md`.

## Input Mode Detection

Determine mode from context:
- **User provides a document** (file path or pasted content) → document-driven or hybrid mode
- **No document provided** → interactive interview mode

## Re-run Behavior

If `.cspec/manifest.md` already exists, present the existing slices and ask the user:
- **Amend** — Add, remove, or modify slices in the existing manifest. Preserve statuses of existing slices.
- **Start fresh** — Discard existing manifest and user stories, begin discovery from scratch. Warn that this does not delete already-written slice spec files.

## Interactive Process

1. **Understand the product** — Ask what it is, who it's for, what problem it solves. One question at a time. Prefer multiple choice when possible.
2. **Identify features** — Ask about the major user-facing capabilities.
3. **Break into vertical slices** — Each slice is one user flow at user-story granularity (e.g., "user signs up with email" not "authentication system"). One flow per slice. **No implementation details in slices.** Apply the test: could you explain this slice to a non-technical user? If a slice name requires technical knowledge to understand, it's an implementation detail, not a user story. A single user action is a single slice — even if the system uses multiple technical approaches to fulfill it. Example: "user searches their items" is one slice, NOT "full-text search" and "semantic search" as separate slices.
4. **Group into domains** — Cluster related slices under domain names (e.g., auth, billing, content).
5. **Determine ordering** — Identify dependencies between slices and assign priority. Slices with no dependencies come first.
6. **Write user stories** — For each slice, write a user story in the format "As a [user], I want to [action], so that [outcome]."
7. **Present for validation** — Show the complete slice inventory and user stories to the user for review before writing.

## Document-Driven Process

1. **Read the document** — Analyze the provided brief, PRD, or product description.
2. **Extract slices and domains** — Identify vertical slices and group them.
3. **Present findings** — Show extracted slices to the user for validation.
4. **Fill gaps (hybrid)** — Ask follow-up questions one at a time for anything unclear or missing from the document.
5. **Write user stories** — For each extracted slice, write a user story.
6. **Finalize** — Confirm the complete inventory and user stories with the user before writing.

## Manifest Output

Write the manifest to `.cspec/manifest.md` using this template:

```markdown
# [Product Name] — Manifest

## Product Summary

[Brief description of the product, target users, and core value proposition]

## Tech Preferences

[Any tech stack preferences or constraints expressed during discovery, or "None specified"]

## Domains

### [Domain Name]

[Brief description of this domain's scope]

### [Domain Name]

[Brief description of this domain's scope]

## Slice Inventory

| Slice | Domain | Description | Priority | Dependencies | Status |
|-------|--------|-------------|----------|--------------|--------|
| [slice-name] | [domain] | [One-line description] | [1-N] | [comma-separated slice names, or "none"] | unwritten |
```

All slices start with status `unwritten`. Priority 1 is highest (build first).

## User Stories Output

Write user stories to `.cspec/user-stories.md` using this template:

```markdown
# [Product Name] — User Stories

## [Domain Name]

### [slice-name]
As a [user type], I want to [action], so that [outcome].

### [slice-name]
As a [user type], I want to [action], so that [outcome].

## [Domain Name]

### [slice-name]
As a [user type], I want to [action], so that [outcome].
```

User stories are the **source of truth** for what each slice should do. Editing a user story here should be followed by re-running `/cspec-write` for the affected slice to update its spec.

## Completion

After writing both files, inform the user:

> "Manifest written to `.cspec/manifest.md` and user stories to `.cspec/user-stories.md` with [N] slices across [N] domains. Review and adjust the user stories as needed, then run `/cspec-write` to begin writing slice specs."
