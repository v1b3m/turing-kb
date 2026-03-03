**Ref:** feat/knowledge-articles
**Depends:** 2026-03-02_011_favorites
### Context

![[Screenshot from 2026-03-02 11-29-02.png]]

The favorites popover is still not touching the "NowService ★" button, it should touch it
### Acceptance Criteria
- [x] favorites popover touches the "NowService ★" button
### Constraints
- None

### Resources
- None

### Handoff
| Field                 | Details                                                                                                                                                               |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **What was done**     | Reduced the `top` offset of both CreateFavoriteDialog and RemoveFavoriteDialog from 72px to 50px so the popover arrow touches the NowService pill button.             |
| **Commit hash(es)**   | `9408bac`                                                                                                                                                             |
| **Files touched**     | `src/components/knowledge/CreateFavoriteDialog.tsx`, `src/components/knowledge/RemoveFavoriteDialog.tsx`                                                              |
| **Decisions made**    | Set top to 50px based on: header=52px, pill bottom≈44px, arrow=6px → 50px total. Also fixed RemoveFavoriteDialog fallback positioning for consistency.                |
| **Known limitations** | Pre-existing build error in KnowledgeArticlesHeader.tsx (missing `useMemo` import) — not related to this task.                                                        |
| **How to verify**     | Open the app, trigger the favorites popover from the hamburger menu "Create Favorite" — the popover arrow should touch the bottom of the NowService pill with no gap. |
