# Knowledge Articles in All Menu Filter

## Description
Fix the "All" dropdown menu filter so that typing in the filter input shows results with matching descendants. Previously, parent nodes hid themselves when their own label didn't match, even if children matched.

## Progress
- [x] Add recursive `subtreeMatches` helper
- [x] Pass `hasMatchingDescendant` into TreeItem
- [x] Update visibility: show if own label matches OR descendant matches
- [x] Auto-expand parents when filter is active and descendants match
- [x] Add `HighlightMatch` component for bold match highlighting
- [x] Build passes with no errors

## Decisions
- Kept leaf visibility logic unchanged — it was already correct
- Auto-expand reverts to manual expanded state when filter is cleared
- Highlight uses `<strong>` tag matching ServiceNow screenshot behavior

## Outcome
Fixed in commit `4d15adb`. All menu filter now correctly shows parent nodes when descendants match, auto-expands them, and bold-highlights the matched portion of text.

\![[Pasted image 20260226141648.png|300]]