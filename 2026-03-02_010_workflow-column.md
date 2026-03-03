**Ref:** main
**Depends:** 2026-03-02_011_favorites

### Context

The circular "progress indicators" in the workflow column need to remain vertically centered. On the Group By "Article ID" view, I can see them aligned to the top of the cell as shown below:

![[Pasted image 20260302110708.png]]

### Acceptance Criteria
- [x] Circular workflow progress indicator always centered vertically in the cell

### Constraints
- None

### Resources
- None

---

### Handoff

| Field | Details |
|---|---|
| **What was done** | Removed `align-top` from the workflow-stages td class in `getTdClassName()`, allowing the base `align-middle` class to take effect and center indicators vertically in taller grouped cells. |
| **Commit hash(es)** | `0129be2` |
| **Files touched** | `src/components/knowledge/cell-renderers.tsx` |
| **Decisions made** | Removed `align-top` rather than adding explicit `align-middle`, since the table already applies `align-middle` as the default — removing the override is cleaner. |
| **Known limitations** | None |
| **How to verify** | Open the Knowledge Articles table, group by "Article ID", and confirm the circular workflow progress indicators are vertically centered in cells that span multiple rows. |
