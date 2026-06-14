---
title: 2026-03-05_000_outbound-mail
ref: main
depends_on: none
ready: true
tags:
  - communication
  - email
picked_at: 2026-03-05T12:33:24Z
picked_mode: execute
---

### Context
Goal: add first-step outbound email behavior from the Case related list area, aligned with ServiceNow classic patterns.

Current product context (from codebase):
- Case form renders related lists through `RelatedTablesTabGroup` in `src/components/case/CaseFormPage.tsx`.
- Tabs include `Emails` (`id: emails`) among other related tabs.
- Toolbar currently has a single shared `New` outline button rendered for every tab in `src/components/ui/related-tables-tab-group.tsx`.
- There is no existing outbound email compose popup flow yet.

Requested behavior from task intake:
1. In the `Emails` tab, rename toolbar button `New` -> `New Email`.
2. Restyle that button to a solid blue button with white text (no outline style).
3. Clicking it should open a popup-style browser window for composing email (ServiceNow-like compose window).

Reference artifacts provided:
- Screenshot: `![[Pasted image 20260305152532.png|500]]`
- Raw ServiceNow HTML dump included as visual/behavior reference only (no copy-paste).

### Discovery Summary (2026-03-05)
- `RelatedTablesTabGroup` is a reusable tab/table shell, but in this repo it is currently used by Case form related lists and not yet specialized per tab action.
- The `New` action is currently hardcoded once in shared toolbar markup, so a tab-specific `Emails` action needs either:
  - tab-aware conditional UI, or
  - configurable per-tab action metadata.
- Existing codebase primarily uses in-page dialogs (`Dialog`/Radix), not separate browser popup windows (`window.open`), so popup behavior will be a new interaction pattern.
- AGENTS guidance requires matching known ServiceNow behavior where design is known; this supports implementing popup-window semantics for this task if confirmed.

### Clarified Scope
In-scope for follow-up execution task:
- Emails-tab-specific CTA text/style parity (`New Email`, solid blue button, white text).
- Click handler to open compose experience in popup-style window.
- Base compose shell parity (window size/title/layout scaffolding), if needed for first increment.

Out-of-scope unless explicitly expanded:
- Full email delivery backend integration.
- Rich WYSIWYG parity, recipient autocomplete parity, attachments parity, keyboard shortcut parity (`Alt+S`, etc.).
- Full replication of entire ServiceNow legacy email client internals.

### Acceptance Criteria
- [x] In Case related lists, when `Emails` tab is active, toolbar primary action label is `New Email`.
- [x] `New Email` button uses solid blue background with white text and no outline treatment.
- [x] Non-Emails tabs retain current action behavior and are not regressed.
- [x] Clicking `New Email` opens a popup-style compose window from a direct user click event.
- [x] Popup opens with deterministic features (documented width/height and chrome settings) and does not trigger navigation away from case form.
- [x] If popup is blocked by browser, user receives a visible fallback message/instruction.
- [x] Implementation remains consistent with existing ServiceNow clone interaction patterns and does not break related-list toolbar layout on desktop/mobile widths.

### Execution Summary (2026-03-05)
- Implemented Emails-tab-specific toolbar action in `RelatedTablesTabGroup`:
  - `New` becomes `New Email` only for `tab.id === 'emails'`.
  - Style switched to solid `nowservice-form-submit` blue button with white text.
  - Non-email tabs continue to render the existing outline `New` button.
- Added direct-click popup behavior using `window.open(...)` with deterministic features:
  - Size: `920x780`
  - Window chrome/settings: `resizable=yes`, `scrollbars=yes`, `toolbar=no`, `menubar=no`, `location=no`, `status=no`
  - Centered via current browser viewport and screen offsets.
- Added blocked-popup fallback UX:
  - Shows visible inline alert message under the toolbar on Emails tab:
    `Allow popups for this site and click New Email again.`
- Added compose-shell route target and UI:
  - New target handled: `sn_customerservice_outbound_email.do`
  - New component: `src/components/case/OutboundEmailComposeWindow.tsx`
  - Provides ServiceNow-like first-step compose shell (To/Cc/Subject/Body + Send/Cancel/Close)
  - Uses case context via query param `case_number` to prefill subject/title context.

### Risks
- Shared-component risk: changing `RelatedTablesTabGroup` action rendering can unintentionally affect all related tabs.
- UX risk: popup blockers may prevent window opening if invocation is not directly tied to user gesture.
- Scope-creep risk: trying to fully replicate ServiceNow email client internals in one task may overrun scope.
- Styling consistency risk: introducing hardcoded blue values may conflict with current nowservice token strategy.

### Implementation Options
1. Minimal conditional in `RelatedTablesTabGroup` (recommended for speed)
- Approach: special-case `tab.id === 'emails'` for label/style/click action.
- Pros: smallest change set, fastest delivery.
- Cons: introduces feature-specific branching in shared component.

2. Config-driven tab actions via `RelatedTableTabConfig`
- Approach: extend tab config with `actionLabel`, `actionVariant`, `onAction`/`actionType`.
- Pros: cleaner architecture, avoids hardcoded tab id branching.
- Cons: slightly larger refactor.

3. Separate Emails related-list toolbar component
- Approach: keep generic component unchanged; inject a specialized toolbar for emails.
- Pros: strict separation of concerns.
- Cons: more wiring complexity for limited immediate need.

### Explicit User Decisions Needed
- [ ] Popup implementation target:
  - `browser window popup (window.open)` (recommended for ServiceNow parity)
  - `in-page modal/dialog`
- [ ] Compose scope for first iteration:
  - `UI shell only` (recommended)
  - `UI shell + draft persistence`
  - `UI shell + send simulation`
- [ ] Popup sizing baseline:
  - `920x780` (matches reference script; recommended)
  - `custom size`
- [ ] Button color source:
  - `existing nowservice token`
  - `introduce new nowservice token for email action blue` (recommended if no existing token matches)
  - `one-off utility color`
- [ ] Behavior when popup blocked:
  - `toast/banner guidance` (recommended)
  - `silent fail`

### Resource Paths
- `src/components/ui/related-tables-tab-group.tsx`
- `src/components/case/CaseFormPage.tsx`
- Task board: `/home/v1b3m/Dev/KnowledgeBase/turing-kb/Boards/ServiceNow Tasks.md`

### Notes
- Kept discovery focused on actionable implementation prep and decision closure.
- Production code was implemented in execute mode.
