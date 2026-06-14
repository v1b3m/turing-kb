---
title: 2026-03-07_001_email-details-view
ref: communication
depends_on: none
ready: true
tags:
picked_at: 2026-03-08T06:21:47Z
picked_mode: execute
---

### Context

![[Pasted image 20260308073219.png]]

We need to update the full email record details view (`sn_customerservice_email.do?sys_id=<id>`) so it matches the provided ServiceNow-style structure.

#### Large screens

1. First Row
	1. Type
	2. UID
2. Second Row
	1. Target
	2. Created
3. Third Row
	1. User ID
	2. Deleted
4. Forth Row
	1. Notification type
	2. Weight
5. Fifth Row
	1. Importance
6. Sixth Row
	1. Subject
7. Seventh Row
	1. Recipients
8. Eighth Row
	1. Body
9. Ninth Row
	1. Content type
10. Tenth Row
	1. Headers
11. Footer
	1. Inline buttons
		1. Update
		2. Delete
	2. Small Heading
		1. Related Links
	3. Link text
		1. Preview Email
	4. Tabulated tables
		1. Email Log
		2. Email Attachments

**Tabulated tables**

- Use the same table as we use for the emails in the case details view
- For now the tables can both be in an empty state
- Email html dump is available at `/home/v1b3m/Dev/Turing/ServiceNow/pg/email-details.html`
	- This is simply to be used as a guide and is in now way supposed to be used as a copy/paste reference for implementation
	-

### Clarified Context
- Existing page already exists at `src/components/case/EmailDetailPage.tsx` but currently only shows `To`, `Cc`, `Subject`, `Sent`, `Body`, `Attachments`.
- Route wiring already exists in `src/app/now/nav/ui/classic/params/target/[target]/page.tsx` for `sn_customerservice_email.do`.
- Source email model (`useEmailsStore`) does not currently include most requested metadata fields (`UID`, `Target`, `Created`, `User ID`, `Deleted`, `Notification type`, `Weight`, `Importance`, `Content type`, `Headers`).
- Prior task `2026-03-07_000_email-preview` already routes `Open Record` to this page; this task is the follow-up record form fidelity pass.

### Acceptance Criteria
- [ ] Email detail page renders ServiceNow-style ordered field layout for large screens:
  - Row 1: `Type` + `UID`
  - Row 2: `Target` + `Created`
  - Row 3: `User ID` + `Deleted`
  - Row 4: `Notification type` + `Weight`
  - Row 5: `Importance` (single width/standalone)
  - Row 6: `Subject`
  - Row 7: `Recipients`
  - Row 8: `Body`
  - Row 9: `Content type`
  - Row 10: `Headers`
- [ ] Footer area includes inline actions `Update` and `Delete` styled/positioned consistent with existing classic detail pages.
- [ ] `Related Links` section includes `Preview Email` link.
- [ ] Related list tabs include `Email Log` and `Email Attachments`.
- [ ] Both related tabs support empty state and use existing in-app email table visual patterns (no bespoke table implementation).
- [ ] Missing field values render deterministic placeholders (for example `—`) rather than blank/undefined text.
- [ ] Existing record-not-found handling remains intact.
- [ ] Existing route contract remains unchanged (`/now/nav/ui/classic/params/target/sn_customerservice_email.do?sys_id=<id>`).

### Constraints
- Discovery mode only for this task run; no production implementation in this handoff.
- Reuse current app primitives and patterns (`DetailPageHeader`, existing table/list primitives, nowservice theme tokens).
- Do not copy/paste from HTML dump; use it only to confirm structure and relative ordering.
- Keep behavior compatible with existing preview flow from email list (`Open Record`).
- Favor `nowservice` theme tokens over introducing new hardcoded colors in new UI work.
- Preserve current client-side data source (`useEmailsStore`) unless a follow-up task explicitly expands the email schema.

### Risks
- Data availability risk: required fields are not present in store, which can lead to fake/static values unless placeholders are explicitly agreed.
- Scope creep risk: implementing fully functional `Update/Delete` on email records may imply persistence rules not yet defined for `sys_email`.
- UI fidelity risk: “same table as case details email table” is ambiguous between style-only reuse vs functional feature parity.
- Responsive behavior risk: requirements are explicit for large screens but not defined for smaller breakpoints.
- HTML content risk: body rendering can introduce formatting/sanitization differences (plain text vs rich HTML rendering).

### Options
1. Minimal fidelity pass (recommended for first execute pass)
- Keep current data model.
- Render required fields/sections with placeholders for unavailable metadata.
- Build static empty related tabs with shared table chrome and “No records to display”.
- Add non-destructive footer actions (UI-first behavior as defined in decisions).

2. Data-model expansion pass
- Extend `Email` store shape for missing metadata fields.
- Backfill seed and persisted data migrations.
- Render true values for most sections.
- Higher effort and regression risk; better as a separate execute task.

3. Hybrid
- Implement layout now.
- Only add a small subset of metadata derivable from current fields (`Created` from `sentAt`, `Recipients` from `to`/`cc`, `Content type` inferred).
- Leave the rest placeholdered.

### Explicit User Decisions Needed Before Execute
- [ ] Action semantics for footer buttons:
  - `UI only` (render buttons, no persistence side-effects) (Recommended)
  - `Functional update/delete` with real record mutation
- [ ] Missing metadata strategy:
  - `Show placeholder (—)` for unavailable fields (Recommended)
  - `Add temporary mocked values`
  - `Expand email store schema now`
- [ ] `Preview Email` link target behavior:
  - `Open preview modal/popover if available`
  - `Navigate to existing preview route/page` (Recommended if no modal spec)
- [ ] Related tab implementation depth:
  - `Header + empty table state only` (Recommended)
  - `Full toolbar/search/actions parity with case email table`
- [ ] Mobile/smaller screen behavior:
  - `Stack fields vertically in same order` (Recommended)
  - `Keep two-column layout until narrow breakpoint is explicitly reached`

### Discovery Handoff (2026-03-08)
- Primary delta is not routing; it is form fidelity and layout expansion in `EmailDetailPage`.
- Execute can proceed safely with Option 1 unless product direction requires real email-record mutation in this same task.
- If functional `Update/Delete` is required, split that behavior into a follow-up task to avoid mixing UI fidelity and persistence contracts.

### Resources
- HTML structure reference: `/home/v1b3m/Dev/Turing/ServiceNow/pg/email-details.html`
- Current email detail implementation: `/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-07_001_email-details-view/src/components/case/EmailDetailPage.tsx`
- Target route mapping: `/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-07_001_email-details-view/src/app/now/nav/ui/classic/params/target/[target]/page.tsx`
- Email data source: `/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-07_001_email-details-view/src/stores/useEmailsStore.ts`
- Related precedent task: `/home/v1b3m/Dev/KnowledgeBase/turing-kb/2026-03-07_000_email-preview.md`

## Handoff
- **What was done:** Reworked EmailDetailPage from simple 5-field layout to full ServiceNow-style 10-row record form with footer actions, Related Links, and related list tabs.
- **Commit:** 2f5ff51
- **Files touched:** src/components/case/EmailDetailPage.tsx
- **Decisions made:**
  - Footer buttons (Update/Delete) are UI-only — no persistence side-effects
  - Missing metadata fields use — placeholder
  - Content type inferred from body content (HTML tag detection)
  - Recipients derived by joining to/cc fields
  - Preview Email links to #email-body anchor on same page
  - Related tabs show empty state with column headers
  - Responsive: two-column on lg+, stacked on smaller screens
- **Known limitations:**
  - Most metadata fields (Type, User ID, Deleted, Notification type, Weight, Importance, Headers) show placeholders since the email store doesn't have these fields
  - Update/Delete buttons are non-functional (UI-only)
  - Related list tabs always show empty state
- **How to verify:** Navigate to any email detail page via Open Record from the case communications tab. Confirm 10-row field layout, footer buttons, Related Links section, and two related list tabs render correctly.

