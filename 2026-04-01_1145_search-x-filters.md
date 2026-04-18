Go through the search and filters features below and determine which gaps are there in making it fully api-backed (no localStorage unless necessary)

## Search & Filters

### Scope

This is a cross-module check, not just cases.

What I found:

- the app has a shared frontend list/search/filter framework
- many modules use that framework through `ListPageTemplate`
- the shared framework applies search, advanced filters, grouping, and most sorting in the frontend after data is already fetched
- backend support varies by module
- some modules are strongly API-backed for filtering/sorting
- some modules only support basic `q` search
- some modules are still local-store driven

## Shared Frontend Behavior

These are the main cross-module gaps.

### 1. Header search is frontend filtering, not backend search

The reusable list header search input only updates frontend state:

- [frontend/src/components/list-template/ListHeader.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/list-template/ListHeader.tsx#L152)

Pressing Enter sets:

- `filterField`
- `globalFilter`

It does not call a backend query method directly.

### 2. Advanced filters run client-side over fetched rows

The generic list template filters the already-fetched `data` array in memory:

- global filter: [frontend/src/components/list-template/ListPageTemplate.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/list-template/ListPageTemplate.tsx#L422)
- column filters: [frontend/src/components/list-template/ListPageTemplate.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/list-template/ListPageTemplate.tsx#L447)
- advanced filters: [frontend/src/components/list-template/ListPageTemplate.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/list-template/ListPageTemplate.tsx#L464)
- sort: [frontend/src/components/list-template/ListPageTemplate.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/list-template/ListPageTemplate.tsx#L511)

That means the shared list UX is not fully API-backed today, even when the underlying module API could support query params.

### 3. Grouping is frontend-only

Grouping in the generic list template is driven by selected UI state and rendered through grouped table components:

- group-by resolution: [frontend/src/components/list-template/ListPageTemplate.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/list-template/ListPageTemplate.tsx#L281)
- grouped render path: [frontend/src/components/list-template/ListPageTemplate.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/list-template/ListPageTemplate.tsx#L711)

There is no shared backend `group_by` contract for list pages.

### 4. List UI state is persisted to localStorage

Case list state persists sort/filter UI state locally:

- [frontend/src/stores/useCaseListStore.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/stores/useCaseListStore.ts#L68)

Persisted values include:

- `sortColumn`
- `sortDirection`
- `globalFilter`
- `filterField`
- `columnFilters`

This is convenience state, not strictly necessary for API-backed search/filter behavior.

## Module Snapshot

### Strong backend query support

These modules already expose meaningful backend search/filter/sort params:

- Entitlements
  - backend: [backend/app/api/v1/endpoints/entitlements.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/entitlements.py#L25)
  - frontend hook: [frontend/src/hooks/entitlements/useEntitlements.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/entitlements/useEntitlements.ts#L21)
- Emails
  - backend: [backend/app/api/v1/endpoints/emails.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/emails.py#L37)
  - frontend hook: [frontend/src/hooks/emails/useEmails.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/emails/useEmails.ts#L14)
- Knowledge Articles
  - backend: [backend/app/api/v1/endpoints/knowledge_articles.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/knowledge_articles.py#L31)
- Reports
  - backend: [backend/app/api/v1/endpoints/reports.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/reports.py#L30)

### Basic backend query support

These modules are API-backed, but query support is more limited:

- Cases
  - backend list params: [backend/app/api/v1/endpoints/cases.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/cases.py#L41)
  - frontend hook params: [frontend/src/hooks/cases/useCases.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/cases/useCases.ts#L32)
  - notable limitation: no backend `sort_by` / `sort_order`
- Accounts
  - backend exposes mostly `q`
- Contacts
  - backend exposes mostly `q`

### Not fully backend-backed

- Assets
  - assets page is still local-store driven, not backed by a backend assets resource
  - page: [frontend/src/app/assets/page.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/assets/page.tsx)

## Feature-by-Feature Gap Analysis

### Advanced Search

Status: Partial

What exists:

- generic advanced-filter UI exists in the shared list template
- some modules have backend endpoints that can support multi-criteria filtering well
  - entitlements
  - emails
  - knowledge
  - reports

Main gaps:

1. The shared advanced-filter UI does not translate filter rules into backend query params.
2. Filters are applied client-side in `ListPageTemplate`, not sent to module APIs.
3. There is no common backend contract for arbitrary multi-field filtering across modules.
4. Cases are still limited on the backend compared to the frontend filter vocabulary.

Case-specific note:

- the cases page defines a large frontend filter field map in [frontend/src/config/case-filter-fields.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/config/case-filter-fields.ts#L1)
- but the backend cases endpoint only supports a small subset of those fields in [backend/app/api/v1/endpoints/cases.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/cases.py#L41)

### Group Cases

Status: No backend support

What exists:

- generic grouping exists in the frontend list template

Main gaps:

1. No backend `group_by` parameter exists for case lists.
2. No grouped/aggregated response shape exists for the cases API.
3. Grouping is derived purely from the fetched row set in the browser.

If true API-backed grouping is required, the backend needs either:

- `GET /api/v1/cases?group_by=priority`

or a dedicated aggregation endpoint such as:

- `GET /api/v1/cases/grouped?group_by=priority`

### Hide Resolved Cases

Status: Partial

What exists:

- cases backend supports `state_not`
  - [backend/app/api/v1/endpoints/cases.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/cases.py#L49)
- case page also interprets `sysparm_query=active=true` in the frontend
  - [frontend/src/app/cases/page.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/cases/page.tsx#L21)

Main gaps:

1. The common hide-resolved behavior is not driven by a shared backend convention.
2. Cases only support excluding one state via `state_not`, not a multi-state exclusion list.
3. The frontend `active=true` behavior is implemented client-side after fetch.

For full API-backed behavior, cases need something like:

- `state_in`
- `state_not_in`
- or `active=true` as a real backend semantic

### Global Search

Status: Not implemented as a real backend-backed global feature

What exists:

- list-level header search inside `ListHeader`
- module-scoped search params like `q` on several endpoints

What is missing:

1. No true app-wide/global search endpoint across record types.
2. No shared header-bar search that queries cases, contacts, accounts, knowledge, etc. together.
3. No unified result schema for cross-record search results.

Needed backend shape:

- `GET /api/v1/search?q=...&types=cases,accounts,contacts,knowledge`

Needed response shape:

- grouped results by record type
- common label, id, href/route metadata

### Filter Case List

Status: Partial

What exists:

- case list is API-backed at base fetch level
- case list can pass limited filters to `/api/v1/cases`
- case page additionally applies:
  - `sysparm_query` filtering client-side
  - tag filtering client-side
  - advanced filter rules client-side via `ListPageTemplate`

References:

- backend list route: [backend/app/api/v1/endpoints/cases.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/cases.py#L41)
- frontend hook: [frontend/src/hooks/cases/useCases.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/cases/useCases.ts#L32)
- case page client-side filter layer: [frontend/src/app/cases/page.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/cases/page.tsx#L168)

Main gaps:

1. The case filter UI is richer than the backend filter contract.
2. `sysparm_query` is not converted into backend query params.
3. Tag filtering is local.
4. Generic advanced filter runs after fetch, not in SQL/API.

### Sort Cases

Status: Partial

What exists:

- frontend generic sort exists in `ListPageTemplate`
- case hook sorts results newest-first after fetch
  - [frontend/src/hooks/cases/useCases.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/cases/useCases.ts#L56)

Main gaps:

1. Cases backend does not expose `sort_by` / `sort_order`.
2. Case sorting by priority/date/assignee is currently frontend-driven.
3. Since sorting happens after fetch, pagination correctness would break if server pagination were introduced on cases without backend sort support.

Needed backend change:

- add `sort_by` and `sort_order` to `/api/v1/cases`

Candidate sortable fields:

- `priority`
- `opened_at`
- `last_activity_date`
- `assigned_to`
- `state`
- `number`

### Saved Filters

Status: Not backend-backed

What exists:

- the agent list subheader has a saved-filters UI
- those saved filters are frontend-only and in-memory

References:

- UI: [frontend/src/components/agent/AgentListSubheader.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentListSubheader.tsx#L448)
- store: [frontend/src/stores/useAgentWorkspaceUiStores.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/stores/useAgentWorkspaceUiStores.ts#L72)

Main gaps:

1. No backend saved-filter resource exists.
2. No cross-module saved-filter schema exists.
3. The main reusable `ListPageTemplate` does not have a backend-backed saved-filter model.
4. Current saved filters are not durable/shared through the API.

Needed backend shape:

- `GET /api/v1/saved-filters?module=cases`
- `POST /api/v1/saved-filters`
- `PUT /api/v1/saved-filters/{id}`
- `DELETE /api/v1/saved-filters/{id}`

Suggested saved-filter model:

- `id`
- `module`
- `name`
- `scope`
- `query`
- `sort`
- `group_by`
- `owner`

## Recommended Path To Fully API-Backed Search/Filters

### Shared work

1. Define a common frontend-to-backend filter translation layer instead of filtering inside `ListPageTemplate`.
2. Stop persisting active search/filter/sort state to localStorage by default for list pages unless there is a product reason to preserve view state.
3. Add a real global search API across record types.
4. Add a saved-filters backend resource.

### Cases

1. Add richer filter params to `/api/v1/cases`.
2. Add `sort_by` and `sort_order`.
3. Add multi-state include/exclude support.
4. Decide whether `sysparm_query` should be parsed server-side or replaced with explicit query params.

### Modules already close to API-backed

These are closest to full backend-backed behavior and mainly need frontend wiring changes so the shared list framework sends query params instead of filtering locally:

- entitlements
- emails
- knowledge articles
- reports

### Modules still behind

- assets need a real backend resource first
- accounts and contacts need richer filter/sort support if they should participate fully in advanced search/filter UX

## Bottom Line

The biggest gap is not one module. It is architectural:

- the app has backend list APIs for several modules
- but the shared list/search/filter UX is still primarily client-side

So “fully API-backed” requires both:

1. richer backend query contracts where missing
2. replacing the shared frontend filtering/grouping/sorting layer with request-driven behavior
