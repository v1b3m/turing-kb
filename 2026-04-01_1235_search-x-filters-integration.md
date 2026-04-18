# Search & Filters Integration Plan

Repo: `/Users/v1b3m/Dev/Turing/ServiceNow`  
Purpose: move search and filtering behavior from frontend-local processing toward real backend-driven list queries across modules.

## Goal

Make search and filter behavior fully e2e where practical:

- backend APIs own search, filtering, sorting, and saved-filter persistence
- frontend list pages become query-driven instead of filtering fetched rows in memory
- localStorage is limited to optional UI convenience only, not source-of-truth query behavior

## Current State

There are two layers today:

1. Backend list APIs
- some modules already support `q`, filters, and sorting
- strongest examples: entitlements, emails, knowledge, reports

2. Shared frontend list framework
- `ListHeader` captures search input
- `ListPageTemplate` applies global search, column filters, advanced filters, grouping, and sorting client-side
- some list state is persisted in localStorage

This means the main integration task is not only backend expansion. It is replacing the shared frontend filtering model.

## Target Architecture

### Backend responsibilities

Each list API should support:

- pagination
- free-text search via `q`
- explicit field filters
- sorting via `sort_by` and `sort_order`
- optional grouped/aggregated responses where grouping must be server-driven

Shared additions:

- global search endpoint
- saved-filters resource

### Frontend responsibilities

The frontend should:

- keep current list UI
- build query params from header search, filter panel, sort state, and optional grouping state
- call backend list hooks with those params
- stop re-filtering full result sets in `ListPageTemplate`

### localStorage policy

Allowed:

- preserving draft UI preferences such as selected columns or last-used filter field

Not allowed as source of truth:

- active search query execution
- advanced filter results
- saved filters
- grouped data
- sorted dataset ordering

## Workstreams

## 1. Shared Query Model

### Objective

Define a shared frontend query shape that all list pages can use.

### Deliverables

- shared query DTO for:
  - `q`
  - `filters`
  - `sort_by`
  - `sort_order`
  - `page`
  - `page_size`
  - optional `group_by`
- mapping layer from list UI state to API query params

### Files to use as references

- [ListPageTemplate.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/list-template/ListPageTemplate.tsx)
- [ListHeader.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/list-template/ListHeader.tsx)
- [FilterListMenu.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/list-template/FilterListMenu.tsx)

### Output

- new shared utility or hook, for example:
  - `frontend/src/components/list-template/query-state.ts`
  - `frontend/src/components/list-template/query-mappers.ts`

## 2. Shared List Template Refactor

### Objective

Make `ListPageTemplate` request-driven instead of data-transform-driven.

### Current problem

`ListPageTemplate` currently:

- filters locally
- sorts locally
- groups locally

### Changes

1. Keep UI controls intact.
2. Lift query state up to page-level hooks.
3. Replace local `sortedAndFilteredData` logic with backend-provided rows.
4. Use server pagination consistently.
5. Keep local grouping only as a temporary fallback until server grouping exists.

### Phasing

Phase 1:
- backend search/filter/sort
- frontend no longer filters locally for modules that support backend params

Phase 2:
- remove most generic client-side filtering from `ListPageTemplate`

## 3. Cases API Expansion

### Objective

Bring cases closer to the richer query support already present in stronger modules.

### Add to `/api/v1/cases`

- `sort_by`
- `sort_order`
- more field filters:
  - `account`
  - `contact`
  - `consumer`
  - `product`
  - `asset`
  - `assigned_to`
  - `assignment_group`
  - `priority`
  - `channel`
  - `category`
  - `state_in`
  - `state_not_in`
  - `tag`

### Optional

- `active=true` as backend semantic
- grouped response support for:
  - `priority`
  - `assigned_to`
  - `category`

### Backend refs

- [backend/app/api/v1/endpoints/cases.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/cases.py)
- [backend/app/services/case.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/services/case.py)
- [backend/app/schemas/case.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/schemas/case.py)

### Frontend refs

- [frontend/src/hooks/cases/useCases.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/cases/useCases.ts)
- [frontend/src/app/cases/page.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/cases/page.tsx)
- [frontend/src/config/case-filter-fields.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/config/case-filter-fields.ts)

## 4. Strong Modules First

### Objective

Migrate modules that already have backend query support with minimal backend work.

### First-wave modules

- Entitlements
- Emails
- Knowledge Articles
- Reports

### Why first

These already expose:

- `q`
- multiple field filters
- `sort_by`
- `sort_order`

So the main work is frontend integration, not backend redesign.

### Tasks

1. Update hooks to accept full query state from list UI.
2. Update each page using `ListPageTemplate` to pass server query params.
3. Disable client-side duplicate filtering for migrated pages.

### Backend refs

- [backend/app/api/v1/endpoints/entitlements.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/entitlements.py)
- [backend/app/api/v1/endpoints/emails.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/emails.py)
- [backend/app/api/v1/endpoints/knowledge_articles.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/knowledge_articles.py)
- [backend/app/api/v1/endpoints/reports.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/reports.py)

## 5. Accounts and Contacts Query Upgrade

### Objective

Improve modules that are API-backed but still limited to basic search.

### Additions

Accounts:

- `sort_by`
- `sort_order`
- field filters such as:
  - `name`
  - `number`
  - `city`
  - `primary_contact`

Contacts:

- `sort_by`
- `sort_order`
- field filters such as:
  - `first_name`
  - `last_name`
  - `email`
  - `account`
  - `department`

### Outcome

These modules become compatible with the same query-driven list framework as the stronger modules.

## 6. Assets Prerequisite

### Objective

Bring assets into the search/filter architecture.

### Dependency

Assets need a real backend resource first.

Use:

- [2026-04-01_1157_assets-spec](/Users/v1b3m/Dev/KnowledgeBase/turing-kb/2026-04-01_1157_assets-spec.md)

### Plan

1. Add `/api/v1/assets`
2. Add list filters and sort support from the start
3. Replace local asset list/store usage with API hooks

## 7. Saved Filters Backend

### Objective

Move saved filters from frontend-only state into a reusable backend resource.

### Proposed routes

- `GET /api/v1/saved-filters?module=cases`
- `POST /api/v1/saved-filters`
- `PUT /api/v1/saved-filters/{id}`
- `DELETE /api/v1/saved-filters/{id}`

### Suggested model

- `id`
- `module`
- `name`
- `scope`
- `owner`
- `filters`
- `sort`
- `group_by`
- `created_at`
- `updated_at`

### Frontend integration points

- agent subheader saved-filters UI
- reusable list pages once standardized

### Current frontend refs

- [frontend/src/components/agent/AgentListSubheader.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentListSubheader.tsx)
- [frontend/src/stores/useAgentWorkspaceUiStores.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/stores/useAgentWorkspaceUiStores.ts)

## 8. Global Search

### Objective

Create real app-wide search across record types.

### Proposed route

- `GET /api/v1/search?q=...&types=cases,accounts,contacts,knowledge,entitlements`

### Response shape

```json
{
  "query": "vpn",
  "results": {
    "cases": [],
    "accounts": [],
    "contacts": [],
    "knowledge": [],
    "entitlements": []
  }
}
```

### Minimum fields per result

- `id`
- `type`
- `title`
- `subtitle`
- `href`

### Notes

- this is separate from list-page header search
- implement after per-module list search is stabilized

## Rollout Order

## Phase 1. Shared foundation

1. Define shared query state model.
2. Refactor `ListPageTemplate` to support server-driven mode cleanly.
3. Stop treating client-side filter/sort/group as the default architecture.

## Phase 2. Quick wins

1. Entitlements
2. Emails
3. Knowledge
4. Reports

Reason:
- lowest backend lift
- validates the query-driven UI pattern

## Phase 3. Cases

1. Expand cases backend query contract.
2. Replace case page `sysparm_query` and local post-filtering where possible.
3. Add backend sort support.
4. Add multi-state visibility controls.

## Phase 4. Accounts and Contacts

1. Expand backend filter/sort support.
2. Move list pages to full server-driven queries.

## Phase 5. Assets

1. Ship assets backend
2. integrate into shared list/search/filter framework

## Phase 6. Saved Filters and Global Search

1. backend saved filters
2. backend global search
3. wire shared UX to both

## Definition of Done

A module is considered fully integrated when:

1. Search input triggers backend query params, not local row filtering.
2. Advanced filters are sent to the backend or translated into supported backend params.
3. Sort order is produced by the backend.
4. Pagination is server-correct under active search/filter/sort.
5. Grouping is either explicitly documented as client-only UI or implemented server-side where required.
6. Saved filters, if supported, persist through the backend.
7. localStorage is not the source of truth for active query behavior.

## Risks

### 1. Shared frontend refactor can break many list pages

Mitigation:

- add a server-driven mode incrementally
- migrate module by module

### 2. Backend contracts differ by module

Mitigation:

- standardize on a common subset:
  - `q`
  - `skip`
  - `limit`
  - `sort_by`
  - `sort_order`
- add module-specific filters on top

### 3. Feature expectations differ between “search”, “advanced filters”, and “global search”

Mitigation:

- treat them as separate deliverables
- do not conflate list search with global search

## Immediate Next Steps

1. Refactor the shared list framework to support server-driven query state.
2. Migrate entitlements as the first reference implementation.
3. Expand the cases API with backend sorting and richer filters.
4. Design the saved-filters backend resource.
