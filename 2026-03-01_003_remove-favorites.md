**Ref:** main
**Depends:** 2026-03-01_001_more-ui-updates
### Context

Clicking a filled favorite star in the header reveals a remove favorite popover

![[Screenshot from 2026-03-02 00-14-04.png]]

### Acceptance Criteria
- [ ] Clicking the filled favorite star reveals the remove favorite popover
- [ ] The styling is pretty much similar to the add favorites popover
- [ ] The "More" button is styled the same as the "Cancel" button from the add favorites popover

### Constraints
- None

### Resources
- None

---

### Handoff

| Field | Details |
|---|---|
| **What was done** | Added a RemoveFavoriteDialog popover and a favorite star icon next to the Knowledge title. The star reflects isFavorited state (green filled when favorited, gray outline when not). Clicking the filled star opens a dark-themed confirmation popover matching the CreateFavoriteDialog style. |
| **Commit hash(es)** | f23cd6f |
| **Files touched** | `src/components/knowledge/RemoveFavoriteDialog.tsx` (new), `src/components/knowledge/KnowledgeArticlesHeader.tsx`, `src/components/layout/ListToolbar.tsx` |
| **Decisions made** | Added `titleExtra` prop to ListToolbar for the star button rather than embedding it in the title string. Cancel button styled like the "More" button (light blue text) per acceptance criteria. Popover anchors to the star button position. |
| **Known limitations** | The star only opens the remove dialog when favorited; no action when not favorited (create favorite is triggered via the hamburger menu). |
| **How to verify** | 1. Run `npm run dev` and navigate to /knowledge. 2. Use hamburger menu → Create Favorite to set favorited state. 3. Observe the green filled star next to "Knowledge". 4. Click the star — the remove favorite popover should appear. 5. Verify Cancel closes without removing, Remove clears the favorite. |
