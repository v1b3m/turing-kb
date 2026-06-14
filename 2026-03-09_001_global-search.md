---
title: 2026-03-09_001_global-search
ref: feat/global-search
depends_on: none
ready: true
tags:
  - search
  - global
picked_at: 2026-03-09T06:23:22Z
picked_mode: execute
---
### Context
We need to implement global search:

#### 1. Typing in the header search input

Entering text inside the search input in the header simply show a "View results" popover(drop-down) below the input like so:
![[Pasted image 20260309085833.png]]

The Link has link styling i.e hovering reveals an underline. It then navigates to

#### Header center button update

![[Pasted image 20260309090713.png]]

The "center button" updates to the search results content `Search Results - [query] ☆`. `[query]` is the search query in this case such as 'an'

The 'star' indicates we should be able to star the search results.]

![[Pasted image 20260309090806.png]]

#### The search results

![[Pasted image 20260309090840.png]]

I've also dumped the html from the search results page at `/home/v1b3m/Dev/Turing/ServiceNow/pg/search-results.html`

This is simply to be used as a guide and is in now way supposed to be used as a copy/paste reference for implementation

### Acceptance Criteria
- [ ] Typing in the header search input shows a "View results" popover/dropdown below the input
- [ ] "View results" link navigates to the search results page at `/search?q=<query>`
- [ ] The header center pill updates to show `Search Results - <query> ☆` when on the search results page
- [ ] Star icon in the center pill toggles favorite for the search results (uses existing CreateFavoriteDialog / RemoveFavoriteDialog)
- [ ] Search results page shows a heading: `N results for "<query>"`
- [ ] Results are grouped by category sections (see sidebar categories below)
- [ ] Each section header shows: section name, count `(showing of total)`, "View all [section]" link, "Go to list view" icon
- [ ] Each result card displays: title (h3), metadata fields as label-value pairs (Number, Opened, Caller, Priority, State, Category, Assignment group), and a description snippet
- [ ] Right sidebar lists all result categories with badge counts; clicking scrolls to section
- [ ] Pressing Enter in the search input also navigates to search results page
- [ ] Search input clear button (×) clears the query

### Constraints
- No backend API exists — all data is static JSON (contacts.json, knowledge-articles.json, etc.)
- Search must be client-side only, filtering across all available static datasets
- Must use existing UI component library (Radix UI + Tailwind + Lucide icons)
- Must match ServiceNow NowService visual style (dark header, branded colors)
- Scope dropdown in SearchWithScopeDropdown currently exists but is non-functional; this task does NOT need to implement scope filtering (can be a follow-up)

### Resources
- Reference HTML dump: `/home/v1b3m/Dev/Turing/ServiceNow/pg/search-results.html`
- Reference screenshots in Obsidian assets (linked above)

---

## Discovery Findings

### Current State

**Header search input** (`src/components/layout/SearchWithScopeDropdown.tsx`):
- Input field exists with placeholder "Search", scope dropdown, and search icon
- Currently **non-functional**: no `onChange`, `value`, submission handler, or routing
- Background colors: `bg-nowservice-search-bg` (#66798A), dropdown: `bg-nowservice-search-dropdown-bg`

**Center pill** (`src/components/layout/Header.tsx`):
- Shows workspace label via `useHeaderWorkspaceLabel()` which returns "CSM/FSM Configurable Workspace" or "NowService" based on pathname
- Star icon toggles favorites using `useFavoritesStore`
- Existing `CreateFavoriteDialog` and `RemoveFavoriteDialog` components work

**Favorites store** (`src/stores/useFavoritesStore.ts`):
- Zustand store with `isFavorited`, `favoriteName`, `favoriteLocation`, `setFavorite()`, `removeFavorite()`
- In-memory only, no persistence

**Static data** available for search:
- `src/page-contents/contacts.json` — contact records
- `src/page-contents/knowledge-articles.json` — knowledge articles
- No incidents, change requests, problems, or catalog items data exists yet

**Routing**: Next.js App Router. No `/search` route exists today.

### Architecture — Recommended Approach

#### Option A: Single search results page (RECOMMENDED)
- Create `/src/app/search/page.tsx` — reads `?q=` from URL search params
- Client-side filtering across all static JSON datasets
- Group results by entity type (Contacts, Knowledge Articles, etc.)
- Simple, matches the reference UI

#### Option B: Store-based search with server component
- Use a Zustand search store to manage query state
- Overkill for static data, adds unnecessary complexity

**Recommendation: Option A** — URL-driven search page with `useSearchParams()`.

### Components to Create/Modify

| File | Action | Description |
|------|--------|-------------|
| `src/components/layout/SearchWithScopeDropdown.tsx` | Modify | Add controlled input, "View results" popover, navigation on Enter/click |
| `src/components/layout/Header.tsx` | Modify | Update `useHeaderWorkspaceLabel()` to return `Search Results - <query>` when on `/search` route |
| `src/app/search/page.tsx` | Create | Search results page with grouped results and sidebar |
| `src/components/search/SearchResultCard.tsx` | Create | Reusable result card (title, metadata fields, description) |
| `src/components/search/SearchResultSection.tsx` | Create | Section wrapper with header, count, "View all" link |
| `src/components/search/SearchSidebar.tsx` | Create | Right sidebar with category list and counts |

### Search Results Page Layout (from reference)

```
┌──────────────────────────────────────────────────────────────┐
│ ← ServiceNow    [Search Input: "an"]    [scope dropdown]    │
│            [ Search Results - an ☆ ]                        │
├──────────────────────────────────────────────────────────────┤
│ 73 results for "an"                                         │
│                                                             │
│ ┌─────────────────────────────────────┐ ┌─────────────────┐ │
│ │ Tasks - Incidents (10 of 18)        │ │ Sidebar         │ │
│ │ [View all] [↗]                      │ │                 │ │
│ │                                     │ │ Tasks - Incid…18│ │
│ │ • SAP Materials Management is slow…│ │ Tasks - Chang… 9│ │
│ │   Number: INC0000054  Opened: ...   │ │ Tasks - Probl… 3│ │
│ │   Caller: ...  Priority: 1-Critical │ │ K&C - Knowle…25│ │
│ │   State: On Hold  Category: Software│ │ K&C - Catalo…18│ │
│ │   Assignment group: Service Desk    │ │                 │ │
│ │   "Nothing loads in the app..."     │ │                 │ │
│ │                                     │ │                 │ │
│ │ • EMAIL is slow when an attachment…│ │                 │ │
│ │   ...                               │ │                 │ │
│ ├─────────────────────────────────────┤ │                 │ │
│ │ Tasks - Change Requests (9)         │ │                 │ │
│ │ ...                                 │ │                 │ │
│ ├─────────────────────────────────────┤ │                 │ │
│ │ Tasks - Problems (3)                │ │                 │ │
│ │ ...                                 │ │                 │ │
│ ├─────────────────────────────────────┤ │                 │ │
│ │ Knowledge & Catalog - Knowledge (25)│ │                 │ │
│ │ ...                                 │ │                 │ │
│ ├─────────────────────────────────────┤ │                 │ │
│ │ Knowledge & Catalog - Cat. Items(18)│ │                 │ │
│ │ ...                                 │ │                 │ │
│ └─────────────────────────────────────┘ └─────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### Sidebar Categories (from reference HTML)
1. **Tasks - Incidents** (18 results) — fields: Number, Opened, Caller, Priority, State, Category, Assignment group
2. **Tasks - Change Requests** (9 results)
3. **Tasks - Problems** (3 results)
4. **Knowledge & Catalog - Knowledge** (25 results) — fields: Number, Knowledge base, Workflow, Category, Article type, Published
5. **Knowledge & Catalog - Catalog Items** (18 results) — shows image thumbnails + description

### Data Gap — Decisions Needed

**DECISION REQUIRED:** The app currently only has static data for **Contacts** and **Knowledge Articles**. The ServiceNow reference shows search results for Incidents, Change Requests, Problems, and Catalog Items — none of which have data files.

**Options:**
1. **Mock all categories** — create static JSON files for incidents, change requests, problems, catalog items to match the reference screenshots
2. **Search only existing data** — only show Contacts and Knowledge Articles sections; add a breadcrumb note that more categories will come when data is available
3. **Hybrid** — search existing data but show hardcoded mock results for other categories as placeholders

**Recommendation:** Option 1 for fidelity to the ServiceNow reference, unless the user prefers a lighter approach.

### Risks
- **Data shape mismatch**: Each category shows different metadata fields. Without real incident/problem data, we need to define the schema for mock data.
- **Performance**: Client-side search across all datasets could be slow for very large mock datasets, but fine for current scale.
- **Scope dropdown**: Currently non-functional. This task should NOT implement scope-based filtering — just global search. Scope filtering can be a follow-up task.
- **No debounce**: The "View results" popover appears on typing — should we debounce? Reference shows it appears immediately.

### Implementation Notes
- The "View results" popover is a simple dropdown below the search input (like an autocomplete suggestion), not a full search preview. Only contains the text "View results" as a clickable link.
- The search results page URL should be `/search?q=<query>` to support bookmarking and direct linking.
- Each result card has an "Open in new tab" icon button (external link icon) per the reference HTML.
- The sidebar category links scroll to the corresponding section on the page (anchor-based navigation).
- Search appears to be case-insensitive text matching across record titles and descriptions.

## Handoff
- **What was done:** Implemented global search feature: (1) SearchWithScopeDropdown now has controlled input with View results popover dropdown on typing, Enter/click navigates to /search?q=query, clear button; (2) Header center pill shows Search Results - query when on /search route; (3) New /search page with grouped results across 5 categories (incidents, change requests, problems, knowledge articles, catalog items), each section with header/count/view-all links, result cards with metadata, and right sidebar with category counts and anchor links; (4) Mock data JSON files for incidents (18), change requests (9), problems (3), catalog items (18).
- **Commit:** 33fb408
- **Files touched:** src/components/layout/SearchWithScopeDropdown.tsx (modified), src/components/layout/Header.tsx (modified), src/app/search/page.tsx (created), src/page-contents/incidents.json (created), src/page-contents/change-requests.json (created), src/page-contents/problems.json (created), src/page-contents/catalog-items.json (created)
- **Decisions made:** (1) Option 1 - mocked all categories for full fidelity to reference. (2) Search is case-insensitive substring match across title/shortDescription/description fields. (3) Each section shows max 10 results with (X of Y) count. (4) No debounce on popover - appears immediately per reference. (5) Scope dropdown remains non-functional as specified in constraints. (6) Kept all components in single page.tsx file rather than separate files for simplicity.
- **Known limitations:** (1) Contacts data not included in search - its JSON format differs from the card-based display pattern; easy follow-up. (2) Star favorite toggle on search results center pill uses existing store (in-memory only). (3) View all and Go to list view buttons are non-functional placeholders. (4) No search result click-through navigation (cards are display-only).
- **How to verify:** Run pnpm dev, type in header search - popover appears. Press Enter or click View results - navigates to /search?q=query. Results grouped by category with sidebar. Center pill shows Search Results - query. Clear button clears input. pnpm build passes.
