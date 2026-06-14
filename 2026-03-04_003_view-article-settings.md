---
title: 2026-03-04_003_view-article-settings
ref: main
depends_on: none
ready: true
tags:
picked_at: 2026-03-04T13:04:53Z
picked_mode: execute
---

### Context
The settings icon should open a popover for customizing the page. This popup enables/disabled what fields are rendered by the page as seen in the screenshot below.

![[Pasted image 20260304155926.png]]

**Popup Structure**

![[Pasted image 20260304160132.png]]

### Discovery Summary (2026-03-04)
- Target page is [src/app/knowledge/[id]/page.tsx](/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-04_003_view-article-settings/src/app/knowledge/[id]/page.tsx), where read-only article fields are currently hardcoded and always rendered.
- Header shell is [src/components/layout/DetailPageHeader.tsx](/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-04_003_view-article-settings/src/components/layout/DetailPageHeader.tsx); it supports `actions` but has no built-in settings/personalize control yet.
- Closest existing pattern is [src/components/form/PersonalizeDropdown.tsx](/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-04_003_view-article-settings/src/components/form/PersonalizeDropdown.tsx), which already provides checkbox-based field visibility in a dropdown menu.
- No knowledge-detail-specific persisted personalization exists today in [src/stores/useKnowledgeStore.ts](/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-04_003_view-article-settings/src/stores/useKnowledgeStore.ts) or [src/stores/local-storage-keys.ts](/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-04_003_view-article-settings/src/stores/local-storage-keys.ts).

### Clarified Scope
- Add a settings trigger on Knowledge article **view** page only (`/knowledge/[id]`).
- Opening the trigger shows a popover-style personalization UI.
- Toggling entries controls field visibility in the article metadata/body layout.
- Discovery excludes production code implementation in this task.


### Acceptance Criteria
- [ ] A settings trigger (gear/sliders icon) exists in the article view header and is keyboard accessible.
- [ ] Trigger opens a popover aligned to the trigger and closable via outside click + `Esc`.
- [ ] Popover lists configurable fields that map 1:1 to visible blocks on the page.
- [ ] Toggling a field immediately shows/hides that field block in the rendered article page.
- [ ] Required/system fields (if designated) cannot be disabled and are visually marked.
- [ ] Existing article actions (Checkout/Retire/Delete) keep current behavior and layout.
- [ ] Mobile behavior is defined (popover placement, wrapping, and scroll behavior).
- [ ] Personalization state behavior is explicitly defined (session-only vs persisted).

### Constraints
- Discovery mode only; no production code changes in this task.
- Must follow `nowservice` theme tokens and existing UI patterns (AGENTS.md).
- Prefer existing shared primitives (`dropdown-menu`/`popover`, `FormHeaderPersonalizeDropdown`) over one-off behavior.
- Must preserve current read-only article rendering semantics and data source.
- Avoid regressions in header responsiveness and keyboard navigation.

### Risks
- Field-to-UI mapping drift: hardcoded JSX sections can desync from popover configuration.
- Persistence ambiguity: unclear whether settings should reset per page load, per browser, or per user profile.
- Required-field policy not yet specified (which fields are mandatory visible).
- Header crowding risk on smaller widths if settings trigger competes with existing action buttons.
- Potential mismatch with screenshot if control type (dropdown vs popover) differs from expected ServiceNow behavior.

### Implementation Options
- Option A: Page-local state in `knowledge/[id]/page.tsx` with inline popover config.
  - Pros: fastest and lowest change surface.
  - Cons: logic duplication and weaker reuse.
- Option B: Reuse/adapt shared `FormHeaderPersonalizeDropdown` pattern for knowledge detail fields.
  - Pros: consistent UX and reusable behavior.
  - Cons: requires adaptation for article-specific sections/labels.
- Option C: Add persisted preferences (localStorage/Zustand) for article detail personalization.
  - Pros: survives reload/navigation; better realism.
  - Cons: extra state modeling, migration/default handling.

### Explicit User Decisions Needed
- [ ] Decide trigger icon style in header: `gear` (screenshot-like) or existing `sliders`.
- [ ] Decide state lifetime: `session only` or `persist across reloads`.
- [ ] Decide required fields that cannot be hidden (recommended minimum: `Number`, `Short description`, `Explanation`).
- [ ] Decide scope of configurable items:
  - `metadata fields only`
  - `metadata + Introduction + Explanation + Related Links + footer tabs block`
- [ ] Decide whether settings are global for all article records or per-article.
- [ ] Decide expected empty-state behavior if all optional fields are hidden.

### Out Of Scope (for this task)
- Backend/API persistence beyond current client-side app model.
- Redesigning read-only field components.
- Changes to list page (`/knowledge`) personalization behavior.

### Resources
- [src/app/knowledge/[id]/page.tsx](/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-04_003_view-article-settings/src/app/knowledge/[id]/page.tsx)
- [src/components/layout/DetailPageHeader.tsx](/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-04_003_view-article-settings/src/components/layout/DetailPageHeader.tsx)
- [src/components/form/PersonalizeDropdown.tsx](/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-04_003_view-article-settings/src/components/form/PersonalizeDropdown.tsx)
- [src/stores/useKnowledgeStore.ts](/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-04_003_view-article-settings/src/stores/useKnowledgeStore.ts)
- [src/stores/local-storage-keys.ts](/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-04_003_view-article-settings/src/stores/local-storage-keys.ts)

### Implementation Notes (2026-03-04)
- Added article-view personalization on `/knowledge/[id]` using a header settings trigger and a Shadcn `Popover` checklist.
- Added configurable field mapping for metadata fields plus `Short description`, `Introduction`, `Explanation`, `Related Links`, and `Footer tabs`.
- Required fields are enforced in UI and logic: `Number`, `Short description`, `Explanation`.
- Field visibility updates are applied immediately and conditionally render the corresponding blocks.
- Preferences are persisted via localStorage key `knowledge-article-view-storage` and reused across records/reloads.
- Header action buttons (`Checkout`, `Retire`, `Delete`) remain present and unchanged in behavior.
- Mobile handling: actions allow wrapping, popover uses viewport-bounded width and scroll (`w-[min(92vw,340px)]`, `max-h-[70vh]`).

### Execution Handoff (2026-03-04)

- **What was done**
  - Implemented a keyboard-accessible settings trigger in the knowledge detail header.
  - Implemented a popover-based field personalization panel with required-field locking.
  - Wired field toggles to immediate visibility control for each mapped page block.
  - Added persisted personalization state using existing localStorage utilities.

- **Commit hash(es)**
  - `e973203`

- **Files touched**
  - `_SCRATCH.md`
  - `src/app/knowledge/[id]/page.tsx`
  - `src/stores/local-storage-keys.ts`

- **Decisions made**
  - Trigger icon: `SlidersHorizontal`.
  - State lifetime: persisted across reloads.
  - Scope: metadata + introduction + explanation + related links + footer tabs.
  - Scope granularity: global preference for article view (not per-article).

- **Known limitations / follow-ups**
  - Preference scope is global for all knowledge detail pages; no per-user/per-article partitioning beyond browser storage.
  - Existing repository lint warnings remain in unrelated files.

- **How to verify**
  - `npm run lint`
  - `npm run build`
  - Manual checks on `/knowledge/[id]`:
    - Open settings popover from header icon; close via outside click and `Esc`.
    - Toggle optional fields and confirm immediate show/hide behavior.
    - Confirm required fields cannot be disabled and are marked `(Required)`.
    - Refresh page or open a different article and confirm personalization persists.
    - Confirm `Checkout/Retire/Delete` remain available in header and body action rows.
