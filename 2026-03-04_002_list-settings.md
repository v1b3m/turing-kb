---
title: 2026-03-04_002_list-settings
ref: main
depends_on: none
ready: true
tags:
  - frontend
  - discovery
  - knowledge-list
picked_at: 2026-03-04T12:41:50Z
picked_mode: execute
---

### Context

Align the **Knowledge > List Settings (Personalize List Columns)** flow to expected ServiceNow behavior and wire settings to actual table rendering.

Current discovery confirms these gaps:

- Dialog state is local-only; `OK` closes modal and does not update table columns.
- Move up/down affects only dialog list order, not rendered table order.
- `Reset to column defaults` is link-styled and always enabled.
- `Wrap column text` toggle does not control table wrapping/truncation behavior.
- Table columns currently come from static view config (`activeViewId -> getViewById(...)`).

### Acceptance Criteria
- [ ] `Reset to column defaults` is rendered as an outlined button on the left side of dialog footer and is disabled when there are no unsaved changes from defaults.
- [ ] Clicking `OK` applies selected columns (visibility and order) to the Knowledge table immediately.
- [ ] Reordering selected columns (up/down) changes rendered column order in the table.
- [ ] Toggling `Wrap column text` updates rendering behavior: off = single-line truncation with ellipsis for long text cells; on = wrapped multi-line text where applicable.
- [ ] `Cancel` (or close icon) does not apply pending dialog changes.
- [ ] Reset returns dialog state (selected columns + wrap/other supported toggles) to default values.
- [ ] Existing list features (search, sort, filters, group by, pagination, row selection) remain functional after applying settings.

### Constraints
- Discovery mode only for this task handoff; no production code changes in this pass.
- Follow `AGENTS.md` theming guidance: use `nowservice` tokens for any new styling updates in implementation.
- Keep current Knowledge list architecture intact unless explicitly approved:
  - View source: `src/config/knowledge-views.ts`
  - Table render: `src/components/knowledge/KnowledgeArticlesTable.tsx`
  - Settings dialog: `src/components/list-template/PersonalizeListColumnsDialog.tsx`
- Avoid breaking existing view selection behavior (`ViewSubmenu` / `activeViewId`) while introducing personalize settings.
- Favor deterministic column mapping by stable IDs/keys (not raw display strings alone) during implementation.

### Clarified Implementation Context
- Dialog component has no apply callback today; it only receives `{ allColumns, defaultSelected }` config.
- Knowledge table renders columns from `viewConfig.columns` and `cellRenderers`.
- Truncation is currently driven by renderer class (`truncated-text`, `html-truncated`) via `getTdClassName(...)`.
- Knowledge store is Zustand but currently non-persisted and has no list-settings slice.
- `allColumns` in `knowledge-articles.json` contains many fields not currently represented in view configs/cell renderers.

### Risks
- Column label mismatch risk: dialog labels may not map 1:1 to existing table field/render config.
- Scope risk: supporting every `allColumns` entry requires additional field + renderer definitions.
- UX consistency risk: wrap behavior can conflict with fixed-width columns (e.g., workflow visualization column).
- State risk: if settings are view-specific but stored globally, switching views can produce unexpected columns.
- Persistence risk: without explicit persistence rules, behavior may reset unexpectedly on refresh.

### Options
1. Minimal scope (recommended for first pass):
Implement personalize settings for currently supported view columns only, keep unknown columns unavailable/non-applicable.
2. Expanded scope:
Map all `allColumns` entries into concrete table fields/renderers (including empty placeholders), so dialog can control all listed columns.
3. Session-only vs persisted settings:
Session-only state in `useKnowledgeStore` vs persisted localStorage (new key and migration rules).

### Explicit User/Product Decisions Needed
- [ ] Should personalized columns apply only to the active view, or globally across all views?
- [ ] Should settings persist across page reloads/browser restarts?
- [ ] If a user removes all columns, should we block this or allow table with only utility columns (checkbox/info)?
- [ ] For unsupported columns from `allColumns`, should we hide them from dialog or show disabled/placeholder entries?
- [ ] Should `Wrap column text` affect all text-capable columns or only specific columns (e.g., Short description, Article body)?
- [ ] Confirm exact copy: user note says `"Rest to column defaults"` while current UI uses `"Reset to column defaults"`.

### Implementation Notes (2026-03-04)
- Implemented session-level per-view knowledge list settings in `useKnowledgeStore` (`selectedColumnLabels`, `wrapColumnText`) with apply-on-OK semantics.
- Added column utility helpers in `knowledge-views` for stable label-to-column mapping and per-view defaults.
- Updated `KnowledgePersonalizeColumns` to filter dialog options to supported/safely-renderable columns and wire dialog apply/cancel/reset behavior.
- Updated `KnowledgeArticlesTable` to render columns from active personalized settings (order + visibility) and use wrap preference for truncation behavior.
- Updated `KnowledgeArticlesHeader` search column options to follow the active personalized column set.
- Validation: `npm run lint` and `npm run build` pass (existing unrelated warnings remain).

### Resources
- Worktree: `/home/v1b3m/Dev/Turing/ServiceNow/.worktrees/task-2026-03-04_002_list-settings`
- Key files:
  - `src/components/list-template/PersonalizeListColumnsDialog.tsx`
  - `src/components/knowledge/KnowledgePersonalizeColumns.tsx`
  - `src/components/knowledge/KnowledgeArticlesTable.tsx`
  - `src/stores/useKnowledgeStore.ts`
  - `src/config/knowledge-views.ts`
  - `src/page-contents/knowledge-articles.json`
- Reference screenshots:
  - ![[Pasted image 20260304143422.png]]
  - ![[Pasted image 20260304143342.png]]
  - ![[Pasted image 20260304143940.png]]

### Execution Handoff (2026-03-04)

- **What was done**
  - Wired `Personalize List Columns` dialog to active Knowledge list state with apply-on-OK behavior.
  - Applied selected columns to table rendering (visibility + order), including grouped and ungrouped row rendering.
  - Implemented `Wrap column text` behavior in table cells for truncation-capable renderers.
  - Made `Reset to column defaults` an outlined footer button, reset-aware, and disabled when already at defaults.
  - Kept pending dialog edits local until `OK`; `Cancel` and close icon discard pending changes.
  - Synced search-column choices in header with personalized visible columns.
  - Fixed subscription path so header/table re-render immediately when list settings are applied.

- **Commit hash(es)**
  - `a73d69d`

- **Files touched**
  - `src/components/list-template/PersonalizeListColumnsDialog.tsx`
  - `src/components/knowledge/KnowledgePersonalizeColumns.tsx`
  - `src/components/knowledge/KnowledgeArticlesTable.tsx`
  - `src/components/knowledge/KnowledgeArticlesHeader.tsx`
  - `src/components/knowledge/cell-renderers.tsx`
  - `src/config/knowledge-views.ts`
  - `src/stores/useKnowledgeStore.ts`

- **Decisions made**
  - Applied settings per active view (`listSettingsByView`) rather than globally to avoid cross-view column mismatch.
  - Scoped selectable dialog columns to supported columns already mapped in `knowledge-views` and existing renderers.
  - Enforced at least one selected column in the dialog to prevent an unusable empty data table.
  - Implemented session-level settings in Zustand (non-persisted), matching current store architecture.

- **Known limitations / follow-ups**
  - List settings are session-only and reset on page refresh.
  - Unsupported entries from `allColumns` are filtered out from dialog options (not shown disabled).
  - Non-list toggles in dialog (`Compact rows`, etc.) remain local UI state and are not applied to table behavior yet.

- **How to verify**
  - `npm run lint`
  - `npm run build`
  - Manual UI checks on `/knowledge`:
    - Open list settings, reorder columns, click `OK`, confirm table header/cell order updates.
    - Remove/add columns, click `OK`, confirm visible columns match selected set.
    - Toggle `Wrap column text`, click `OK`, confirm truncation vs wrapping changes in text-heavy columns.
    - Open dialog, make changes, click `Cancel` or close icon, reopen and confirm no pending edits were applied.
    - Use `Reset to column defaults` and confirm it restores defaults and becomes disabled when at defaults.
    - Confirm search/sort/filter/group/pagination/row-select still work after applying settings.
