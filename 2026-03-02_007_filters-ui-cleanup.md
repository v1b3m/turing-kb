**Ref:** main
**Depends:** 2026-03-01_001_more-ui-updates
### Context

The filters UI needs to be cleaned up to match more closely the spec

1. The fields dropdowns such as the one showing "Workflow" need to be wider, the spec has them at 220px
2. The operator dropdowns too need adjustment, the spec has them at 150px
3. The values dropdowns too need to be at most 220px
4. The inputs (dropdowns) and buttons (AND, OR, X) have a height of 32px
5. Some rows start with "or", this "or" text should have color rgb(0, 128, 163)
	1. The buttons text too uses this same color
6. The filter section uses the same bg as the filter toolbar below the main header i.e rgb(226, 229, 231

**Demo:**

![[Pasted image 20260302010541.png]]
### Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2

### Constraints
- None

### Resources
- None

### Handoff

| Field | Details |
|---|---|
| **What was done** | Updated FilterPanel CSS to match spec: dropdown widths (220px field, 150px operator, max 220px value), 32px height on all inputs/buttons, teal rgb(0,128,163) color for "or" text and AND/OR button text, light gray rgb(226,229,231) background |
| **Commit hash(es)** | 01c6dbd |
| **Files touched** | `src/components/knowledge/FilterPanel.tsx` |
| **Decisions made** | Applied fixed widths (w-[220px], w-[150px]) instead of min-w to match spec exactly. Used max-w-[220px] for value inputs to cap their width. Used h-[32px] instead of padding for consistent heights. |
| **Known limitations** | None — purely CSS changes |
| **How to verify** | `npm run build` (passes). Open the filter panel and confirm: dropdowns are wider, buttons/inputs are 32px tall, "or" text is teal, background is light gray. |
