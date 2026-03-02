![[Pasted image 20260228161940.png]]

Above is the create favorite behavior.

1. It stars the current page
2. We may need to add state for this in zustand, a different store just for this

---

## What was done
Added a "Create Favorite" dialog that opens when clicking the "Create Favorite" button in the hamburger menu. The dialog shows "Favorite added" title, a required Name input (pre-filled with "Knowledge"), a Location dropdown ("Top level (default)"), and action buttons (More, Remove, Done). Created a separate `useFavoritesStore` for favorite state.

## Commit hash(es)
`14e6034`

## Files touched
- `src/components/knowledge/CreateFavoriteDialog.tsx` (created) — Dialog with Name input, Location select, More/Remove/Done buttons
- `src/components/knowledge/HamburgerMenuDropdown.tsx` (modified) — Added dialog state, wired "Create Favorite" onClick, added optional `onCreateFavorite` prop to `KnowledgeMenuItems`
- `src/stores/useFavoritesStore.ts` (created) — Zustand store with `isFavorited`, `favoriteName`, `favoriteLocation`, `setFavorite`, `removeFavorite`

## Decisions made
- **Dialog vs Popover**: Used Dialog (Radix) instead of Popover since popover-inside-dropdown has focus-trapping issues. Dialog renders in a portal and works cleanly.
- **Separate store**: Created a dedicated `useFavoritesStore` as suggested in the task, rather than adding to the knowledge store.
- **KnowledgeMenuItems prop**: Made `onCreateFavorite` optional so existing usages (titleMenu) don't break. The HamburgerMenuDropdown manages the dialog state and passes the callback.

## Known limitations
- "More" button is non-functional (placeholder)
- Location dropdown has only one option ("Top level (default)")
- The star icon in the nav bar is not updated when a favorite is set (out of scope)
- When `KnowledgeMenuItems` is used in the title dropdown (not the hamburger), "Create Favorite" click does nothing since no `onCreateFavorite` is passed

## How to verify
1. Navigate to `/knowledge`
2. Click hamburger menu → "Create Favorite" (blue button)
3. Verify dialog appears with "Favorite added" title, Name="Knowledge", Location="Top level (default)"
4. Edit the name → click Done → dialog closes
5. Reopen → click Remove → dialog closes