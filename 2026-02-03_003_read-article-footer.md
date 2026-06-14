**Ref:** main
**Depends:** (none, or prior task ID e.g. 2026-02-28_003_incident-schema)

### Context

The bottom section on the read articles view needs some revamping

**Reference:**

![[Pasted image 20260303163817.png]]

**Current Version:**

![[Pasted image 20260303163854.png]]

- The reference view is in an elevated card-like structure
- Each tab is a table, you can reference the knowledge articles table (it's like a trimmed down version of it)

#### The table

**Header:**
- The header includes a hamburger menu
- The hamburger menu is followed by a filter icon-button
- This is then followed by the search input along with it's dropdown
- On the right, we have a settings icon
- It is then followed by a minus icon which collapses the table
- We then have the "New" and "Edit" buttons
- We can copy/paste whatever we need from the knowledge articles table to create this trimmed down version of the "footer tab tables"

**Tabs:**

- The tabs at the top also need to be updated to match the design in the ref
- The active indicator should also be at the top as in the ref
- The active tab looks like a "button", the inactive ones are dimmed and the active one is focused/highlighted
	- Read the ref carefully to understand this

**Collapsed Table:**

![[Pasted image 20260303165149.png]]

### Acceptance Criteria
- [x] Output matches spec

### Constraints
- None

### Resources
- None

---

### Handoff
| Field | Details |
|---|---|
| **What was done** | Replaced the read-article footer in `knowledge/[id]` with a ServiceNow-style related-lists card: top-indicator tabs, toolbar (menu, filter, dropdown + search, settings, collapse, New, Edit...), contextual row, table header, and empty-state panel. Added collapsed-state behavior matching the reference (gray strip with green outlined plus button to re-open). |
| **Commit hash(es)** | Not committed in this task |
| **Files touched** | `src/app/knowledge/[id]/page.tsx`, `src/components/knowledge/ReadArticleFooter.tsx` |
| **Decisions made** | Implemented as a dedicated reusable component (`ReadArticleFooter`) instead of inline JSX to keep `knowledge/[id]` page maintainable. Tab order and default active tab were aligned to the reference (`Affected Products` active at the right end). |
| **Known limitations** | Toolbar buttons are UI-only in this task (no behavior wired for menu/filter/settings/new/edit beyond collapse toggle). |
| **How to verify** | 1. Open `/knowledge/{id}`. 2. Scroll to bottom and confirm elevated footer card with styled tabs and toolbar. 3. Click different tabs and confirm table header label updates per tab. 4. Click minus icon to collapse; confirm gray strip + outlined plus button. 5. Click plus button to restore the list panel. |
