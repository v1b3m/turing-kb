---
title: 2026-03-07_000_email-preview
ref: communication
depends_on: none
ready: true
tags:
  - communication
  - email
picked_at: 2026-03-07T19:12:34Z
picked_mode: execute
---

### Context

Clicking the info icon button (the one after the checkbox) in the email list should:

1. Start with a loading pointer (as the preview loads)
2. Show an email preview

We need to hook this up in the table itself cause other pages will have their own different previews that are not necessarily emails.

**Email preview demo**

![[Screenshot from 2026-03-07 21-11-56 1.png|500]]

I've attached a dump of the preview popup html at `/home/v1b3m/Dev/Turing/ServiceNow/pg/email-preview.html`. 

Disclaimer: This html dump is just supposed to guide on what elements are expected but is in no way supposed to be used as a copy/paste reference for implementation.
### Acceptance Criteria
- [x] In the Email list table, clicking the row info icon (column after checkbox) starts a visible loading state immediately.
- [x] Loading state is scoped to the clicked row/action and ends when preview content is ready or fails.
- [x] After loading, an email preview appears for that selected row.
- [x] Preview is rendered from table-level email data/lookup logic (not global/shared preview logic) so other tables can implement different preview experiences.
- [x] Preview supports at least: title (`Email`), key content summary/body area, and `Open Record` action to the full email record.
- [x] `Open Record` preserves current routing pattern to `sn_customerservice_email.do?sys_id=<id>`.
- [x] Only one preview is open at a time in the same table.
- [x] Preview can be dismissed (outside click and/or explicit close) without breaking table selection, sorting, grouping, pagination, or search state.
- [x] If preview data fails to load, user gets a non-blocking error/fallback state and the loading indicator clears.

### Constraints
- Discovery mode only in this run; no production implementation in this task handoff.
- Implement in email table component scope (`CaseEmailsTable`) because preview behavior is domain-specific and should not be forced into generic related-table primitives.
- Follow AGENTS theming guidance: prefer `nowservice` tokens and existing UI primitives; avoid new hardcoded colors where tokens exist.
- HTML dump (`/home/v1b3m/Dev/Turing/ServiceNow/pg/email-preview.html`) is structural/expectation reference only, not copy source.
- Must work for both grouped and ungrouped table render paths (info icon exists in both markup branches).
- Must remain compatible with current email flows already in place (row subject link navigation and compose popup flow).

### Clarified Context
- Existing behavior today:
  - Info icon buttons are present in `CaseEmailsTable` but currently have no click handler; they only render hover styling.
  - The component renders two row paths (grouped and ungrouped), both with separate info-button markup that must stay behaviorally consistent.
- Scope boundary:
  - This task is specifically preview-from-list behavior for the email table.
  - Full email detail page (`EmailDetailPage`) remains the canonical record view and is out of scope for redesign here.
- UX reference:
  - Screenshot + `email-preview.html` indicate a lightweight popover-style preview with `Email` header and `Open Record` action.

### Risks
- Duplicate logic risk if preview trigger/render is added in one row path (grouped or ungrouped) but not the other.
- Interaction conflict risk with row hover/select mechanics (checkbox visibility, group toggles, pagination controls).
- Positioning risk for anchored preview in scrollable table container.
- Accessibility risk if preview is mouse-only (needs keyboard/focus handling).
- State-reset risk when search/filter/pagination changes while preview is open.

### Options
1. Anchored popover from info icon (recommended)
Pros: closest to provided reference, least disruptive to existing table layout.
Cons: requires careful positioning in scrollable container.

2. Inline expandable row preview
Pros: simpler placement logic.
Cons: diverges from ServiceNow-like popover pattern and complicates grouped rows.

3. Right-side panel preview
Pros: stable layout and easier responsive behavior.
Cons: less faithful to reference and heavier UI footprint.

### Explicit User Decisions Needed Before Execute
- [ ] Preview container pattern:
  - `Anchored popover` (recommended)
  - `Inline row expansion`
  - `Side panel`
- [ ] Loading indicator style for “loading pointer”:
  - `Cursor progress + icon disabled`
  - `Inline spinner on info icon` (recommended for clarity/accessibility)
  - `Both cursor and spinner`
- [ ] Preview content scope:
  - `Minimal` (subject, to/cc, sent, excerpt/body)
  - `Expanded` (more metadata fields from HTML dump)
- [ ] Open behavior:
  - `Single-click info icon opens preview` (recommended)
  - `Hover-trigger preview`
- [ ] Dismiss behavior baseline:
  - `Outside click + Esc + close button` (recommended)
  - `Close button only`

### Resources
- Screenshot: `![[Screenshot from 2026-03-07 21-11-56 1.png|500]]`
- HTML guidance: `/home/v1b3m/Dev/Turing/ServiceNow/pg/email-preview.html`
- Table component target: `/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-07_000_email-preview/src/components/case/CaseEmailsTable.tsx`
- Email row renderers: `/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-07_000_email-preview/src/components/case/email-cell-renderers.tsx`
- Email detail route target: `/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-07_000_email-preview/src/app/now/nav/ui/classic/params/target/[target]/page.tsx`

### Discovery Handoff (2026-03-07)
- Discovery confirmed this should be implemented in `CaseEmailsTable` and not generalized in `RelatedTablesTabGroup`.
- The info icon exists in both grouped and ungrouped table rows; execution should unify behavior for both paths.
- Recommended implementation direction for execute phase: anchored popover + per-row loading state + single-open-preview state machine + resilient dismiss and error states.

### Execution Handoff (2026-03-07)
- What was done:
  - Implemented table-scoped email preview behavior directly in `CaseEmailsTable` with an anchored popover triggered by the info icon.
  - Added row-scoped preview state machine (`idle/loading/ready/error`) and request guards so only one preview is active at a time.
  - Added immediate loading feedback on click (spinner + `cursor-progress`) and graceful fallback state with retry when preview lookup fails.
  - Added preview content block with `Email` title, metadata (`To`, `Cc`, `Subject`, `Sent`), body excerpt area, and `Open Record` link to the full record.
  - Unified grouped and ungrouped row behavior by routing both render paths through a shared `renderPreviewCell`.
  - Preserved existing table mechanics (selection, sorting, grouping, pagination, search) and added auto-close when previewed rows leave the visible page/group.
- Files touched:
  - `src/components/case/CaseEmailsTable.tsx`
- Verification:
  - `npm run lint` (passes; only pre-existing warnings outside this task).
  - `npm run build` (passes; compile + type check + production build completed).
  - `npm test` fails due to missing script target file (`tests/run.ts`) in this worktree.
