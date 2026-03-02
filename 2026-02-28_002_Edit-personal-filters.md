![[Pasted image 20260228154921.png]]

Hitting "Edit personal filters" from the hamburger menu's second item results in this view

---

## What was done
Added a new "Edit personal filters" page at `/knowledge/personal-filters` that renders a filters list table matching the ServiceNow reference screenshot. Wired the "Edit personal filters" menu item in the Filters submenu to navigate to this page.

## Commit hash(es)
`e22cddc`

## Files touched
- `src/components/knowledge/PersonalFiltersTable.tsx` (created) — Main component with toolbar, breadcrumb, table columns, empty state, footer
- `src/app/knowledge/personal-filters/page.tsx` (created) — Next.js route page
- `src/components/knowledge/FiltersSubmenu.tsx` (modified) — Wired onClick to navigate to the new route

## Decisions made
- **Separate route** rather than inline view switching — the personal filters view has completely different columns, breadcrumb, and data from the knowledge articles table, so a dedicated route at `/knowledge/personal-filters` is cleaner
- **Reused existing toolbar** (`ListToolbar`) and hamburger menu components to maintain visual consistency
- **Matched empty state SVG** from `KnowledgeArticlesTable` for visual consistency

## Known limitations
- The search dropdown and search input are non-functional (no data to filter)
- The "New" button is non-functional (no create form implemented)
- No data source for personal filters — currently always shows empty state

## How to verify
1. Navigate to `/knowledge` page
2. Click the hamburger menu → Filters → "Edit personal filters"
3. Verify it navigates to `/knowledge/personal-filters`
4. Verify the page matches the reference screenshot: toolbar with Filters title, breadcrumb, column headers, empty state, footer info icon