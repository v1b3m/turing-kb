# Number Dropdown

## Description
Replace the hardcoded "Number" button next to the search input in ListToolbar with a dropdown that lists all column names. Selecting a column changes the button text to the selected value.

![[Pasted image 20260228125340.png]]

## Progress
- [x] Add `searchColumns`, `selectedSearchColumn`, `onSearchColumnChange` props to ListToolbar
- [x] Replace hardcoded button with DropdownMenu (with "for text" + column names)
- [x] Highlight selected item with blue background
- [x] Wire up state in KnowledgeArticlesHeader from knowledge store
- [x] Verify type-check passes

## Decisions
- Kept ListToolbar generic: when `searchColumns` is not provided, falls back to the original static "Number" button
- Reused existing `DropdownMenu`/`DropdownMenuItem` components already in use in the file
- "for text" is a special option rendered at the top of the dropdown, before the column list

## Outcome
Implemented and committed. Commit: 9e4224d