**Ref:** feat/knowledge-articles
**Depends:** 2026-03-02_011_favorites
### Context
If I click "All", search for and click "My Knowledge Articles", it should reload and move me to the default "My Knowledge Articles" view.

### Acceptance Criteria
- [ ] Clicking "My Knowledge Articles" behaves as expected

### Constraints
-

### Resources
- None

---

### Handoff

| Field | Details |
|---|---|
| **What was done** | Made `/my-knowledge-articles` page call `refreshList()` on mount, resetting search, filters, group-by, and active view to defaults whenever navigating to that route. |
| **Commit hash(es)** | `c394584` |
| **Files touched** | `src/app/my-knowledge-articles/page.tsx` |
| **Decisions made** | Used `useEffect` on mount in the page component rather than hooking into AllMenuDropdown's onClick — this covers both menu navigation and direct URL access. |
| **Known limitations** | Pre-existing build error in `KnowledgeArticlesHeader.tsx` (missing `useMemo` import) — not related to this task. |
| **How to verify** | 1. Go to `/knowledge`, apply filters/search/change view. 2. Click "All" in header, search "My Knowledge Articles", click it. 3. Verify the table resets to default view with no filters/search. |
