### Context
What needs to be done and why. Include enough detail that any agent can start cold.

### Acceptance Criteria
- [x] A detailed overview of this codebase

### Constraints
- None

### Resources
- [[2026-02-24-service-now]]

---

## What was done
Created a comprehensive codebase overview document for the ServiceNow clone in the repository root: `CODEBASE_OVERVIEW.md`. It covers architecture, routing, component responsibilities, data schemas, theming tokens, utility classes, dependencies, and developer workflow details.

## Commit hash(es)
`bed3257`

## Files touched
- `CODEBASE_OVERVIEW.md` (created) — Detailed technical overview of the codebase
- `_SCRATCH.md` (modified) — task working notes

## Decisions made
- **Single-source overview doc**: Kept the output as a standalone root-level document so future contributors and agents can quickly understand the system without scanning many files.
- **Breadth + structure**: Included both high-level architecture and concrete file/data details to satisfy “detailed overview” while keeping sections navigable.

## Known limitations or follow-ups
- The overview reflects the codebase at commit `bed3257`; it should be updated as architecture, routes, or data models evolve.

## How to verify
1. Open `CODEBASE_OVERVIEW.md` in the repository root.
2. Confirm it includes at least: stack, directory structure, routing, component inventory, data schemas, and theme/system tokens.
3. Cross-check a few entries against source files (e.g., `src/app/layout.tsx`, `src/components/layout/Header.tsx`, `src/page-contents/home-page.json`, `tailwind.config.js`).
