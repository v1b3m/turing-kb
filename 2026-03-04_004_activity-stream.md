---
title: 2026-03-04_004_activity-stream
ref: main
depends_on: none
ready: true
tags:
picked_at: 2026-03-04T13:28:14Z
picked_mode: execute
execute_discovery_note_conflict_at: 2026-03-04T13:19:50Z
returned_at: 2026-03-04T13:19:50Z
---

### Context
Goal: make the activity stream on `view article` match the behavior and UI pattern already used in the `knowledge articles list`.

Current state observed:
- List page (`src/components/knowledge/KnowledgeArticlesTable.tsx`) can toggle a right-side `ActivityStreamPanel`.
- `ActivityStreamPanel` (`src/components/knowledge/ActivityStreamPanel.tsx`) has:
  - collapsed list of entries
  - per-entry `Open Entry`
  - expanded entry via `RecordStreamEntry`
  - close button (`ChevronsRight`)
- View article page (`src/app/knowledge/[id]/page.tsx`) currently has no activity stream toggle or right panel.
- Existing detail header (`src/components/layout/DetailPageHeader.tsx`) does not expose an activity-stream control.

Clarified parity target:
- "Behave just like list" should mean same interaction model and panel shell, not a brand-new variant.
- Keep first iteration data behavior aligned with existing list implementation (mock/static entries) unless reviewer chooses to wire per-article data now.

### Acceptance Criteria
- [ ] View article page has an activity stream toggle control with equivalent affordance to list behavior.
- [ ] Toggling opens a right-side activity panel without full-page navigation.
- [ ] Panel header uses same close affordance and title pattern as list (`Activity Stream` + close icon button).
- [ ] Panel supports both modes used in list:
  - entries list mode
  - single-entry mode (`RecordStreamEntry`) after `Open Entry`
- [ ] `Open Entry` from list mode opens detailed stream entry view with back navigation.
- [ ] Close behavior works from both list mode and single-entry mode.
- [ ] Opening/closing the panel does not break existing article detail layout, footer tabs, or scrolling.
- [ ] Styling remains consistent with current ServiceNow clone patterns and existing nowservice theme usage.
- [ ] No regressions introduced to knowledge list page activity stream behavior.

### Constraints
- Discovery mode only for this task handoff; no production implementation in this step.
- Reuse existing components where possible (`ActivityStreamPanel`, `RecordStreamEntry`) instead of duplicating UI.
- Preserve existing detail-page actions and layout structure.
- Follow project theme/tokens and existing UI patterns in AGENTS guidance.

### Resources
- View article page: `src/app/knowledge/[id]/page.tsx`
- List activity stream usage: `src/components/knowledge/KnowledgeArticlesTable.tsx`
- Activity stream panel: `src/components/knowledge/ActivityStreamPanel.tsx`
- Stream entry detail component: `src/components/activity/RecordStreamEntry.tsx`
- Detail header: `src/components/layout/DetailPageHeader.tsx`

### Risks
- Layout risk: adding a fixed-width right panel may cause overflow/squash on smaller widths if parent container is not converted to split layout.
- UX risk: if toggle location differs from list conventions, parity intent may be considered unmet.
- Data risk: list panel currently uses internal mock entries; wiring article-specific entries may be deferred unintentionally.
- Coupling risk: forcing activity control into `DetailPageHeader` could affect other detail pages using the same shared header.

### Options
1. Reuse `ActivityStreamPanel` directly in view article page (recommended)
- Pros: highest parity, lowest implementation cost, minimal new surface area.
- Cons: currently tied to mock entries and knowledge-specific assumptions.

2. Extract generic `ActivityStreamPanel` shell and share between list + detail
- Pros: cleaner architecture and easier future extension.
- Cons: larger refactor for this scope; higher regression risk in list.

3. Implement a separate detail-only panel component
- Pros: isolates risk from list page.
- Cons: duplicates behavior/UI and may drift from parity requirement.

### Explicit User Decisions Needed
1. Scope decision: parity only for interaction/UI shell, or also per-article data source now?
- Recommended: interaction/UI parity now; per-article data wiring in follow-up.

2. Toggle placement on view article:
- Recommended: place in detail header actions area, visually aligned with list affordance.

3. State persistence for panel open/closed on detail route:
- Recommended: no persistence initially (local component state), matching current list behavior.

4. Reuse strategy:
- Recommended: Option 1 (`ActivityStreamPanel` reuse) unless reviewer wants refactor investment now.

## Handoff (2026-03-04)
- **What was done:** Implemented activity stream parity on view article by adding an Activity toggle in the detail header and reusing the existing right-side `ActivityStreamPanel` in split layout mode. Panel opens/closes in place, supports list mode + `Open Entry` detail mode (`RecordStreamEntry`), and preserves page-level scrolling/layout.
- **Commit:** `6797bed`
- **Files touched:** `src/app/knowledge/[id]/page.tsx`
- **Decisions made:** Reused existing `ActivityStreamPanel` unchanged (mock/static entries) for behavior parity with knowledge list; placed toggle in header actions using existing toolbar-style icon affordance.
- **Known limitations:** Activity stream data is still static/mock from shared panel component; no per-article entry wiring in this task.
- **How to verify:**
  1. Open `/knowledge/[id]` for any valid article.
  2. Click the new Activity icon in header actions; confirm right-side panel appears with `Activity Stream` title + close icon.
  3. In panel list mode, click `Open Entry`; confirm detailed entry view opens with back navigation.
  4. Close from both list mode and entry mode; confirm panel closes and article layout/footer remain intact.
  5. Confirm knowledge list page activity stream behavior is unchanged.
- **Validation run:** `npm run lint` (passes; existing unrelated warnings remain in other files).
