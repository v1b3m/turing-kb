---
title: 2026-06-07_002_email-templates
ref: communication
depends_on: none
ready: true
tags:
picked_at: 2026-03-08T06:22:49Z
picked_mode: execute
---

### Context

We need to update the email details view to include a templates section at the very bottom of the page. It's like stuck at the bottom,the rest of the page remains behaving the same, scrolling etc.

**No templates are available view**

![[Screenshot from 2026-03-08 08-14-43.png]]

The layout:
1. Everything is in one row
	1. Left items
		1. No templates are available
		2. Create A New One?
			1. This is a link item
	2. Right items
		1. Icon buttons, outline style
			1. "+"
			2. "X"
The "+" button opens the template creation view
The "X" will hide the templates view

The templates bar is controlled mainly by a popover action item from the three dot menu in the header of the email details view. Note that other pages too will have their own templates, so this must be reusable for other pages.

![[Pasted image 20260308081931.png]]

The popover has two options:
1. Show/Hide Template Bar
	1. This uses a closing quote icon at the start
2. Show/Hide annotations
	1. This uses a question mark in a circle as icon
	2. We can disable this option for now

**Create a new one**

This opens a modal, full width and height with some padding around

**Loading state**
![[Pasted image 20260308082308.png]]

**Loaded state**

![[Pasted image 20260308082445.png]]

**HTML dump**

Email template html dump available at `/home/v1b3m/Dev/Turing/ServiceNow/pg/email-template-modal.html`

This is simply to be used as a guide and is in now way supposed to be used as a copy/paste reference for implementation

Most fields are dropdowns which can be changed to other fields
Template fields can be removed with the Red close buttons on the right

### Clarified Context
- Current email detail page is `src/components/case/EmailDetailPage.tsx` and currently has no template bar, no template modal, and no wired "more options" menu.
- The shared detail header supports injecting `moreControl` (`src/components/layout/DetailPageHeader.tsx`), so this task can add menu behavior without changing header internals.
- A similar "Show/Hide Template Bar" + disabled "Show/Hide annotations" interaction already exists in Knowledge detail page (`src/app/knowledge/[id]/page.tsx`), which is a strong reuse/reference candidate for interaction behavior and menu copy.
- There is already a compact menu variant with quote/help icons at `src/components/form/FormHeaderMoreMenu.tsx`; this is reusable as a pattern, but currently not stateful.
- The provided modal HTML dump is a shell that loads a form in an iframe and passes hidden params (`sys_table=sys_email`, editable fields list). In this app, we should mirror UI structure/flow only, not iframe-backed behavior.

### Acceptance Criteria
- [ ] Email detail header `More` control opens a menu with exactly:
  - `Show/Hide Template Bar` with leading quote icon.
  - `Show/Hide annotations` with leading question/help icon, disabled for now.
- [ ] Toggling `Show/Hide Template Bar` shows/hides a bottom templates bar on the email details page without changing existing page routing or email detail field behavior.
- [ ] Templates bar appears docked at the bottom of the email detail viewport while main content scroll behavior remains intact.
- [ ] Templates bar empty state layout matches requested one-row arrangement:
  - Left: `No templates are available` text + `Create A New One?` link action.
  - Right: outline icon buttons for `+` and `X`.
- [ ] `Create A New One?` and `+` trigger the same "Create New Template" experience.
- [ ] `X` hides the templates bar immediately.
- [ ] Create experience opens a full-viewport modal/sheet style UI with close affordance and loading-to-loaded states as defined in task references.
- [ ] Implementation is reusable for non-email pages (componentized template bar + creation modal with page-specific configuration, not email-only hardcoding).
- [ ] Annotations menu item stays disabled/non-functional in this iteration.
- [ ] UI styling follows existing nowservice theme patterns/tokens.

### Constraints
- Discovery mode only in this run; do not implement production UI/code changes.
- Reuse existing primitives/patterns where possible (`DetailPageHeader` `moreControl`, Popover, existing menu/button patterns).
- Keep route contract unchanged for email details (`sn_customerservice_email.do?sys_id=<id>`).
- Do not copy/paste from HTML dump; use it only as structural guidance.
- Avoid introducing persistence/API contracts for templates unless explicitly approved; default to UI-first behavior for this task.

### Risks
- State ownership risk: if template bar visibility is kept page-local, cross-page reuse may diverge quickly.
- Layout risk: bottom-docked bar can conflict with nested scroll containers (`overflow-y-auto`) and may overlap content on short viewports.
- Interaction parity risk: existing knowledge-page menu style may drift from email-page implementation if duplicated instead of shared.
- Scope creep risk: "Create New Template" can expand into full template data model/editor work that is not currently defined.
- Accessibility risk: menu and modal keyboard handling/focus trap may regress if custom implementations bypass existing primitives.

### Options
1. Local implementation in `EmailDetailPage` only
- Pros: fastest path.
- Cons: weak reusability; likely rework when other pages adopt templates.

2. Reusable UI components with local page wiring (recommended)
- Build shared `TemplateBar` + `TemplateCreateModal` + `TemplateMenuItems` primitives.
- Keep data handling local/UI-only for now.
- Pros: satisfies "other pages too" requirement with bounded scope.
- Cons: moderate up-front wiring.

3. Global template UI store + shared components
- Pros: single visibility/modal state across pages.
- Cons: unnecessary coupling for current scope; higher regression risk.

### Explicit User Decisions Needed Before Execute
- [ ] Default template bar visibility on first load:
  - Hidden by default (Recommended)
  - Visible by default
- [ ] Create action behavior for this iteration:
  - UI shell only (open modal with scaffold/loading states, no persistence) (Recommended)
  - Persist a local template record
  - Integrate with backend/API
- [ ] Reuse strategy:
  - Build shared reusable template UI primitives now (Recommended)
  - Ship email-only implementation first
- [ ] Menu implementation source:
  - Reuse/adapt existing Knowledge-style `Popover` menu pattern (Recommended)
  - Reuse/adapt `FormHeaderMoreMenu` `DropdownMenu` pattern
- [ ] Close behavior after create modal submit/cancel:
  - Close modal and keep template bar visible (Recommended)
  - Close modal and auto-hide template bar
- [ ] Annotation item behavior for now:
  - Disabled with tooltip/title only (Recommended)
  - Hidden entirely

#### User Feedback

1. Use the recommended options
2. 
### Discovery Handoff (2026-03-08)
- This task is primarily a UI/state orchestration task in the email detail shell, not a data-model task.
- Strongest low-risk path is Option 2: reusable components with page-local state and UI-only template creation flow.
- The key unresolved product decisions are default visibility, scope of create behavior (UI shell vs persistence), and which existing menu pattern to standardize on.
### Resources
- Task board: `/home/v1b3m/Dev/KnowledgeBase/turing-kb/Boards/ServiceNow Tasks.md`
- Task note: `/home/v1b3m/Dev/KnowledgeBase/turing-kb/2026-06-07_002_email-templates.md`
- Email details page target: `/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-06-07_002_email-templates/src/components/case/EmailDetailPage.tsx`
- Shared header: `/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-06-07_002_email-templates/src/components/layout/DetailPageHeader.tsx`
- Similar menu precedent: `/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-06-07_002_email-templates/src/app/knowledge/[id]/page.tsx`
- Alternate menu precedent: `/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-06-07_002_email-templates/src/components/form/FormHeaderMoreMenu.tsx`
- HTML guide (reference only): `/home/v1b3m/Dev/Turing/ServiceNow/pg/email-template-modal.html`

## Handoff (Execute)
- **What was done:** Implemented reusable template bar, create modal, and more menu components. Wired them into EmailDetailPage with local state management.
- **Commit:** 354f463
- **Files touched:**
  - `src/components/template/template-bar.tsx` (new) — Bottom-docked bar with empty state, create link, +/X buttons
  - `src/components/template/template-create-modal.tsx` (new) — Full-viewport dialog with loading→loaded transition, removable template fields
  - `src/components/template/template-more-menu.tsx` (new) — Popover menu with Show/Hide Template Bar + disabled annotations
  - `src/components/case/EmailDetailPage.tsx` (modified) — Added template state, wired TemplateMoreMenu as moreControl, conditional TemplateBar + TemplateCreateModal
- **Decisions made:**
  - Option 2 (reusable components with page-local state) as recommended
  - Template bar hidden by default
  - UI shell only for create (no persistence)
  - Used Popover pattern matching Knowledge page precedent
  - Annotations disabled with tooltip
  - pb-16 padding on scroll container when bar is visible to prevent content overlap
- **Known limitations:**
  - No persistence/API for templates — UI-only scaffold
  - Annotations menu item is disabled/non-functional
  - Template bar state is page-local, not persisted across navigation
- **How to verify:**
  1. Navigate to an email detail page
  2. Click the ⋯ (More) button in the header → see Show/Hide Template Bar + disabled annotations
  3. Click Show/Hide Template Bar → bottom bar appears with 'No templates are available' + 'Create A New One?'
  4. Click 'Create A New One?' or '+' → full-screen modal with loading then field list
  5. Click 'X' on bar → bar hides
  6. TypeScript compiles cleanly: npx tsc --noEmit
