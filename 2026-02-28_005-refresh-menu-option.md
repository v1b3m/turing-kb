The refresh option in the hamburger menu is self-explanatory, we need to refresh the table's content.
1. Reload the table
2. Rerun any queries if present

---

## What was done
Wired the "Refresh List" hamburger menu item to call a `refreshList()` store action that resets: rows to original JSON data, search query/column, group-by field, and active view to defaults.

## Commit hash(es)
`1c00618`

## Files touched
- `src/stores/useKnowledgeStore.ts` (modified) — Added `refreshList()` action, `rowsPerPage`/`setRowsPerPage` (from concurrent task)
- `src/components/knowledge/HamburgerMenuDropdown.tsx` (modified) — Wired onClick on "Refresh List" to call `refreshList()`

## Decisions made
- **Reset scope**: Refresh resets rows (undoes deletes), clears search, resets group-by and view to defaults. Does not reset `rowsPerPage` since that is a user preference.
- Since data is static JSON, "refresh" means resetting to initial imported state.

## Known limitations
- In a real app, refresh would re-fetch from the server. Here it just resets to the static JSON data.

## How to verify
1. Delete some rows → apply search → select group-by → click hamburger → "Refresh List"
2. Verify: all 16 rows restored, search cleared, group-by cleared, default view restored