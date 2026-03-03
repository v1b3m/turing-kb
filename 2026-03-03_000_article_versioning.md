**Ref:** feat/knowledge-articles
**Depends:** none

### Context

![[Screenshot from 2026-03-03 15-52-03.png]]

On the articles view, editing should be disabled for articles that are no the latest versioning. 

**This means we need to have robust versioning implemented.**

Clicking the "An updated..." link should open the latest version in a read-only view.

There will be an additional "Checkout" button which can then checkout this version for editing.

![[Pasted image 20260303155642.png]]

### Acceptance Criteria
- [ ] Robust versioning is implemented
- [ ] Editing is only allowed on the latest versions of articles
- [ ] 

### Constraints
- None

### Resources
- None
-

## Handoff
- **What was done:** Implemented article versioning on the knowledge article detail page. Added version display in header, info banner for non-latest versions with link to latest, "Make this Current" button, "Checkout" button, and a fully functional Article Versions tab. Added store helpers (getArticleVersions, getLatestVersionIndex, isLatestVersion) to query version data from the existing mock dataset which already contains multi-version articles.
- **Commit:** 3c45a00
- **Files touched:** src/stores/useKnowledgeStore.ts, src/app/knowledge/[id]/page.tsx
- **Decisions made:** Versioning determined by articleId grouping — rows with the same articleId are versions of the same article. Latest version = highest numeric version value. "Make this Current" navigates to latest version (UI stub). "Checkout" is a UI-only button placeholder.
- **Known limitations:** Checkout and Make this Current are UI stubs — no backend logic. Editing is not yet implemented (page is fully read-only), so the edit-gating for non-latest versions will need wiring once editing is added.
- **How to verify:** Navigate to a multi-version article like KB0005001 (indices vary — find v1.0). Confirm: banner appears, links to latest, header shows version, Article Versions tab lists all versions, Checkout button visible. Navigate to the latest version — confirm no banner, no Make this Current button, no Checkout button.