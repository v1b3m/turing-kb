---
title: 2026-03-05_001_mail-attachments
ref: communication
depends_on: none
ready: true
tags:
  - communication
  - email
picked_at: 2026-03-05T13:58:41Z
picked_mode: execute
---

### Context

![[Pasted image 20260305152950.png]]

Goal: replicate ServiceNow classic outbound-email attachment behavior in the compose popup flow.

Current product context (repo):
- Compose popup exists at target route `sn_customerservice_outbound_email.do` and renders `OutboundEmailComposeWindow`.
- Current compose window has `Send` and `Cancel` only; no attachments UI in this component yet.
- Existing attachment dialog primitives already exist:
  - `src/components/AttachmentsDialog.tsx` (more complete: choose file, mock progress, list, remove)
  - `src/components/form/AttachmentsModal.tsx` (older/minimal variant)
- Existing `AttachmentsDialog` is local-state only and currently uses hardcoded colors that should be aligned to `nowservice` theme tokens during execution.

Observed ServiceNow behavior from provided captures:
- Attachments button appears before `Send`.
- `Choose file` opens native file picker.
- Uploaded file appears in list and can be selected via checkbox.
- `Remove` button is enabled when one or more rows are selected.
- Per-row actions include `View` (open attachment window) and `Rename` (inline edit filename).

Clicking "Choose file" should open the native file picker

**Uploading attachment:**

![[Screenshot from 2026-03-05 15-30-55.png]]

**Uploaded attachment:**

![[Screenshot from 2026-03-05 15-30-59.png]]

1. View should open yet again another window with the attachment loaded
2. Rename will make the attachment text editable
3. The check mark can be selected to enable the "Remove" button in the bottom right

**Checkbox clicked:**

![[Screenshot from 2026-03-05 15-31-09.png]]

**Uploading another attachment:**
![[Screenshot from 2026-03-05 15-31-17.png]]

**View attachment:**

![[Screenshot from 2026-03-05 15-31-31.png]]

**Attachments added to mail window:**

![[Screenshot from 2026-03-05 15-31-56.png]]

They will appear in the attachments section

### Acceptance Criteria
- [x] Compose popup shows an `Attachments` action positioned before `Send`, matching ServiceNow-like order and visual hierarchy.
- [x] Clicking `Attachments` opens an attachments modal/dialog within the compose popup without navigating away.
- [x] In attachments modal, `Choose file` triggers native file chooser.
- [x] Selected file is added to attachment list and displayed by filename.
- [x] Row checkbox selection works and controls `Remove` enabled/disabled state.
- [x] `Remove` deletes only selected rows.
- [x] `Rename` allows filename editing (at least client-side display update).
- [x] `View` opens attachment in a new window/tab (or deterministic mock preview fallback if blob URL unavailable).
- [x] Added attachments are visible in compose window attachment area after modal closes.
- [x] Behavior is resilient for repeated uploads in one session and does not regress existing popup send/cancel flow.

### Constraints
- Discovery mode only for this task handoff; no production code changes in this step.
- Keep parity with known ServiceNow behavior from screenshots; avoid over-expanding to full email client parity.
- Use existing stack and patterns (Next.js + Shadcn + nowservice theme tokens).
- No backend upload endpoint currently exists; implementation should use client-side/mock attachment state for now unless scope is expanded.
- Maintain popup-window experience already established by outbound-mail flow.

### Risks
- Competing attachment components (`AttachmentsDialog` vs `AttachmentsModal`) may cause duplication unless one path is selected.
- Browser security limits for local files can impact `View` behavior; preview may differ by file type.
- Large files/multiple files can expose performance gaps if state shape is naive.
- Hardcoded colors in existing attachment dialog can drift from theme consistency if reused without refactor.
- Ambiguity about persistence scope (session-only vs durable) can cause rework.

### Implementation Options
1. Reuse and adapt `src/components/AttachmentsDialog.tsx` in compose popup (recommended).
   - Pros: fastest path, existing remove/select/progress behaviors already present.
   - Cons: needs theming cleanup and action label parity (`View` vs `[download]`).
2. Build a compose-specific attachments dialog from scratch.
   - Pros: exact behavior fit for outbound mail only.
   - Cons: duplicates existing logic and increases maintenance.
3. Consolidate both existing attachment components into one shared primitive before feature wiring.
   - Pros: cleaner architecture long-term.
   - Cons: larger scope and higher risk for this task.

### Explicit User Decisions Needed
- [ ] Single vs multiple upload per chooser interaction:
  - `single file only` (recommended for first increment)
  - `multiple files at once`
- [ ] Attachment persistence scope:
  - `popup session only` (recommended)
  - `persist to store/localStorage`
- [ ] `View` behavior for non-previewable files:
  - `download/open via blob URL` (recommended)
  - `show unsupported preview message`
- [ ] Rename commit behavior:
  - `save on checkmark/enter` (recommended)
  - `save on blur`
- [ ] Component strategy:
  - `reuse and adapt AttachmentsDialog` (recommended)
  - `new compose-specific dialog`

### Resources
- Codebase:
  - `src/components/case/OutboundEmailComposeWindow.tsx`
  - `src/components/AttachmentsDialog.tsx`
  - `src/components/form/AttachmentsModal.tsx`
  - `src/app/now/nav/ui/classic/params/target/[target]/page.tsx`
- Task board:
  - `/home/v1b3m/Dev/KnowledgeBase/turing-kb/Boards/ServiceNow Tasks.md`

### Execution Handoff (2026-03-05)
- What was done:
  - Added session-scoped attachments support to outbound compose popup with an `Attachments` button before `Send`.
  - Reused and upgraded `AttachmentsDialog` to support native file picking, upload simulation, per-row selection + remove, rename, and view in a new tab.
  - Wired compose window to render attached files in an `Attachments` section after modal close.
- Commit hash(es):
  - `fba0782`
- Files touched:
  - `src/components/AttachmentsDialog.tsx`
  - `src/components/case/OutboundEmailComposeWindow.tsx`
- Decisions made:
  - Reused `AttachmentsDialog` (instead of creating a new component) and extended it to support controlled attachments so compose window can own popup-session state.
  - Chose single-file chooser interaction per upload action (first increment).
  - Implemented `View` via blob URL `window.open(...)` with deterministic fallback content if blob URL is unavailable.
  - Implemented rename commit on Enter/checkmark for predictable UX.
- Known limitations or follow-ups:
  - Attachment state is popup-session only and not persisted to store/localStorage.
  - No backend upload/send integration yet; attachments are client-side only.
  - Multi-file upload in one chooser interaction is not enabled in this increment.
- How to verify:
  - Run: `npm run lint`
  - Open case form -> `Emails` related tab -> `New Email`.
  - In compose popup: click `Attachments`, choose a file, confirm it appears in dialog and in compose attachment area after closing.
  - Select one or more rows, verify `Remove` enables and removes only selected files.
  - Click `[rename]`, edit filename, press Enter/checkmark and verify updated display name.
  - Click `[view]`, verify a new tab/window opens attachment preview (or fallback message).
