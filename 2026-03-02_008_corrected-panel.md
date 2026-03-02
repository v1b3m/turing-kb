**Ref:** main
**Depends:** 2026-03-01_001_more-ui-updates

### Context
We did [[2026-03-02_002_panel-entry]] in the wrong branch without the prerequisites i.e the panel implementation. This branch is to correct that. We need to move that logic into this worktree. 

### Acceptance Criteria
- [ ] Panel entry logic is replicated into this worktree

### Constraints
- None

### Resources
- [[2026-03-02_002_panel-entry]]

---

### Handoff

| Field | Details |
|---|---|
| **What was done** | Replicated all 4 source files from task 2026-03-02_002_panel-entry (commit de705e1, wrong branch) into this worktree: RecordStreamEntry component, types, sample data, and demo page. Exact copy — no modifications needed. |
| **Commit hash(es)** | `f89e35c` |
| **Files touched** | `src/components/activity/types.ts`, `src/components/activity/sample-data.ts`, `src/components/activity/RecordStreamEntry.tsx`, `src/app/demo/panel-entry/page.tsx` |
| **Decisions made** | Direct copy with no changes — the original code was self-contained and compatible with this worktree's dependencies. |
| **Known limitations** | Same as original: no real data fetching, uses sample data. Component needs integration with the activity stream panel (which exists in this worktree from task 001). |
| **How to verify** | `npm run build` passes. Visit `/demo/panel-entry` to see the component rendered with sample data. |
