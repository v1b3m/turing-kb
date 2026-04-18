# Feature Route Review And API Gap Analysis

## Scope

Reviewed the listed P1 and P2 features to determine:

1. how they work today
2. which browser routes and stores they use
3. which backend APIs are required to remove `localStorage` and client-only state as the source of truth

## Current Backend Reality

Current backend API coverage is limited to:

- `auth`
- `cases`
- `contacts`
- `accounts`
- `users`

Everything below in `knowledge`, `emails`, `templates`, `saved filters`, `entitlements`, `dashboards`, `reports`, `knowledge feedback`, and `report data sources` is currently frontend-owned state or generated JSON data.

## Key Findings

- Case CRUD is already partially backed by API in `frontend/src/lib/api/cases.ts`.
- Case list data can come from the backend, but filtering and sorting are still mostly client-side.
- Knowledge article list/detail/search is fully local today via `useKnowledgeStore` and generated seed data.
- Article attachment to case is stored on the case as `attachedKnowledgeArticleKeys`; there is no backend relation route today.
- Email composition and email thread history are fully local via `useEmailsStore`.
- Response templates are fully local via `useTemplateStore`.
- Global search is entirely client-side and searches local JSON/generated datasets.
- Saved filters are persisted in `localStorage` via `useAgentSavedFiltersStore`.
- Knowledge gap tasks and article feedback are fully local via `useKnowledgeFeedbackTaskStore` and `useKnowledgeArticleFeedbackStore`.
- Entitlements, dashboards, reports, and report data sources are all frontend stores backed by `localStorage`.

## P1 Features

### ROW_24 - Search Knowledge Articles

Current behavior:

- Browser routes:
  - `/knowledge`
  - `/knowledge/{index}`
  - from case context, attached knowledge uses `/now/nav/ui/classic/params/target/m2m_kb_task.do?sys_id={caseSysId}`
- Main components:
  - `frontend/src/components/knowledge/KnowledgeArticlesTable.tsx`
  - `frontend/src/components/case/CaseKnowledgeTable.tsx`
  - `frontend/src/stores/useKnowledgeStore.ts`
- Data source:
  - `generatedKnowledgeData` / `frontend/src/data/generated/knowledge-articles.seed.json`
- Search implementation:
  - client-side filtering in table components
  - no backend knowledge search route

API required:

- `GET /api/v1/knowledge-articles`
  - supports `skip`, `limit`, `q`
  - supports knowledge-specific filtering and sorting
- Optional convenience route if exact article-number lookup is needed:
  - `GET /api/v1/knowledge-articles/by-number/{number}`

### ROW_25 - View Knowledge Article

Current behavior:

- Browser routes:
  - `/knowledge/{index}`
  - `/now/nav/ui/classic/params/target/kb_view.do?sysparm_article={articleNumber}`
  - `/now/nav/ui/classic/params/target/kb_knowledge.do?sys_id={...}`
- Main components:
  - `frontend/src/app/knowledge/[id]/page.tsx`
  - `frontend/src/components/knowledge/KnowledgeArticleViewPage.tsx`
- Data source:
  - `useKnowledgeStore`
- Additional local article state:
  - field personalization in `LOCAL_STORAGE_KEYS.KNOWLEDGE_ARTICLE_VIEW`
  - feedback and views in `useKnowledgeArticleFeedbackStore`

API required:

- `GET /api/v1/knowledge-articles/{article_id}`
- If routes need article-number based resolution:
  - `GET /api/v1/knowledge-articles/by-number/{number}`

### ROW_26 - Attach Article to Case

Current behavior:

- Browser routes:
  - attached knowledge editor: `/now/nav/ui/classic/params/target/m2m_kb_task.do?sys_id={caseSysId}`
  - create new article from case: `/now/nav/ui/classic/params/target/kb_knowledge.do?sys_id=-1&case_sys_id={caseSysId}&source_task={caseNumber}`
- Main components:
  - `frontend/src/components/case/CaseKnowledgeTable.tsx`
  - `frontend/src/components/case/CaseKnowledgeCreatePage.tsx`
  - `frontend/src/components/case/CaseKnowledgeEditPage.tsx`
- Data model today:
  - attachment state is stored on the case via `attachedKnowledgeArticleKeys`
  - updated in `useCaseStore.setAttachedKnowledgeArticleKeys`
  - no API route is hit

Recommendation:

- model this as a relation resource, not as `attachedCaseIds` on the article

API required:

- `GET /api/v1/cases/{case_id}/knowledge-attachments`
- `POST /api/v1/cases/{case_id}/knowledge-attachments`
  - body: `article_id` or `article_number + version`
- `DELETE /api/v1/cases/{case_id}/knowledge-attachments/{attachment_id}`

Optional:

- case detail response may include attached knowledge as derived data

### ROW_36 - Compose Email from Case

Current behavior:

- Browser routes:
  - `/compose-email?case_number={caseNumber}`
  - email detail: `/now/nav/ui/classic/params/target/sn_customerservice_email.do?sys_id={emailId}`
- Main components:
  - `frontend/src/components/case/OutboundEmailComposeWindow.tsx`
- Data source:
  - `useEmailsStore`
- Send behavior:
  - `sendEmail()` appends a local email record
  - uses `BroadcastChannel` for popup sync
  - no backend email send route

API required:

- `POST /api/v1/emails`
  - create/send outbound email
- Better case-scoped variant for clarity:
  - `POST /api/v1/cases/{case_id}/emails`

### ROW_37 - View Email Thread

Current behavior:

- Browser routes:
  - case record tabs render email list inside case
  - email detail uses `/now/nav/ui/classic/params/target/sn_customerservice_email.do?sys_id={emailId}`
- Main components:
  - `frontend/src/components/case/CaseEmailsTable.tsx`
  - `frontend/src/components/case/EmailDetailPage.tsx`
- Data source:
  - `useEmailsStore`
- Threading behavior:
  - case emails are filtered client-side by `email.caseNumber === caseNumber`

API required:

- `GET /api/v1/cases/{case_id}/emails`
  - preferred
- or:
  - `GET /api/v1/emails?caseNumber={caseNumber}`
- `GET /api/v1/emails/{email_id}`

### ROW_38 - Use Response Templates

Current behavior:

- Main components:
  - `frontend/src/components/template/template-bar.tsx`
  - `frontend/src/components/template/template-create-modal.tsx`
  - `frontend/src/components/case/EmailDetailPage.tsx`
- Data source:
  - `useTemplateStore`
- Persistence:
  - local only
- Behavior:
  - templates are filtered by table, primarily `Email [sys_email]`
  - applying a template mutates the currently open email record in frontend state

API required:

- `GET /api/v1/email-templates?table=Email%20[sys_email]`
- `GET /api/v1/email-templates/{template_id}`
- `POST /api/v1/email-templates`
- `PUT /api/v1/email-templates/{template_id}`
- `DELETE /api/v1/email-templates/{template_id}`

### ROW_41 - Global Search

Current behavior:

- Browser route:
  - `/search?q=...`
- Main component:
  - `frontend/src/app/search/page.tsx`
- Data sources:
  - local JSON:
    - incidents
    - change requests
    - problems
    - catalog items
  - local generated knowledge data
- Cases are not searched from backend here
- Entire feature is client-side aggregation

API required:

- `GET /api/v1/search?q={query}`

Suggested response shape:

- grouped categories with typed results, mirroring the current UI sections

Minimum categories needed for this scope:

- cases
- knowledge
- contacts
- accounts

Optional if the product keeps the broader search surface:

- incidents
- change requests
- problems
- catalog items

### ROW_42 - Filter Case List

Current behavior:

- Browser route:
  - `/cases`
  - plus optional `sysparm_query` and `tag`
- Main files:
  - `frontend/src/app/cases/page.tsx`
  - `frontend/src/config/case-filter-fields.ts`
  - `frontend/src/stores/useCaseListStore.ts`
  - `frontend/src/components/list-template/FilterPanel.tsx`
- Current data source split:
  - base list can come from backend `fetchCases()`
  - advanced filter logic is still client-side
  - filter UI state persists in local store

API required:

- expand `GET /api/v1/cases`
  - support field filtering, not only `q`
  - support structured filter payload or query params

Recommended options:

- simple query-param mode for common filters:
  - `state`
  - `priority`
  - `assigned_to`
  - `assignment_group`
  - `account`
  - `contact`
  - `consumer`
  - `channel`
  - `tag`
  - `active`
- advanced mode:
  - `POST /api/v1/cases/search`
  - body accepts filter conditions, sort rows, grouping

### ROW_43 - Sort Cases

Current behavior:

- Sort state lives in `useCaseListStore`
- sorting is still performed in frontend list/template components
- current backend `fetchCases()` supports only `skip`, `limit`, and `q`

API required:

- expand `GET /api/v1/cases`
  - `sort_by`
  - `sort_order`
- if multiple sort rows are needed:
  - `POST /api/v1/cases/search`

### ROW_44 - Saved Filters

Current behavior:

- Main files:
  - `frontend/src/stores/useAgentSavedFiltersStore.ts`
  - `frontend/src/components/agent/AgentListSubheader.tsx`
- Persistence:
  - `LOCAL_STORAGE_KEYS.AGENT_SAVED_FILTERS`
- Saved payload contains:
  - conditions
  - sort rows
  - group by
  - permission
  - list id

API required:

- `GET /api/v1/saved-filters?list_id={listId}`
- `POST /api/v1/saved-filters`
- `PUT /api/v1/saved-filters/{filter_id}`
- `DELETE /api/v1/saved-filters/{filter_id}`

Suggested model fields:

- `id`
- `name`
- `permission`
- `list_id`
- `payload.conditions`
- `payload.sortRows`
- `payload.groupByField`

## P2 Features

### ROW_17 - Create Article from Case

Current behavior:

- Main component:
  - `frontend/src/components/case/CaseKnowledgeCreatePage.tsx`
- Behavior:
  - creates article directly in `useKnowledgeStore`
  - then attaches it to the case by updating `attachedKnowledgeArticleKeys` in `useCaseStore`
  - no backend route

API required:

- `POST /api/v1/knowledge-articles`
- `POST /api/v1/cases/{case_id}/knowledge-attachments`

Optional convenience endpoint if backend should do both atomically:

- `POST /api/v1/cases/{case_id}/knowledge-articles`

### ROW_18 - Report Knowledge Gap

Current behavior:

- Main files:
  - `frontend/src/components/knowledge/ReportKnowledgeGapDialog.tsx`
  - `frontend/src/stores/useKnowledgeFeedbackTaskStore.ts`
  - case form launches this flow
- Behavior:
  - submitting the dialog creates a local knowledge feedback task
  - nothing is sent to backend

API required:

- `POST /api/v1/knowledge-feedback-tasks`
- `GET /api/v1/cases/{case_id}/knowledge-feedback-tasks`
- `GET /api/v1/knowledge-feedback-tasks/{task_id}`
- `PUT /api/v1/knowledge-feedback-tasks/{task_id}`

### ROW_19 - Knowledge Feedback

Current behavior:

- Main files:
  - `frontend/src/components/knowledge/KnowledgeArticleViewPage.tsx`
  - `frontend/src/stores/useKnowledgeArticleFeedbackStore.ts`
- Local feedback stored by article id:
  - `starRating`
  - `helpfulVote`
  - `viewCount`
  - `comments`
- Persistence:
  - `LOCAL_STORAGE_KEYS.KNOWLEDGE_ARTICLE_FEEDBACK`

API required:

- `GET /api/v1/knowledge-articles/{article_id}/feedback`
- `PUT /api/v1/knowledge-articles/{article_id}/feedback`
  - for star rating / helpful vote
- `POST /api/v1/knowledge-articles/{article_id}/feedback/comments`
- `POST /api/v1/knowledge-articles/{article_id}/views`
  - or track views server-side in article detail route

### ROW_20 - View Entitlements

Current behavior:

- Browser routes:
  - `/entitlements`
  - `/now/nav/ui/classic/params/target/service_entitlement.do?sys_id={sysId}&sysparm_view=case`
- Main files:
  - `frontend/src/app/entitlements/page.tsx`
  - `frontend/src/components/entitlements/EntitlementFormPage.tsx`
  - `frontend/src/stores/useEntitlementStore.ts`
- Data source:
  - local store with persisted entitlements and related cases

API required:

- `GET /api/v1/entitlements`
- `GET /api/v1/entitlements/{sys_id}`

### ROW_21 - Validate Entitlement

Current behavior:

- No dedicated backend validation flow exists
- Cases can reference an entitlement text field
- entitlement selection is effectively lookup/local-data based

API required:

- `POST /api/v1/entitlements/validate`

Suggested request:

- `case_id` or
- `account`, `contact`, `consumer`, `product`, `asset`, `channel`, `entitlement_id`

Suggested response:

- `is_valid`
- matched entitlement
- validation reason / failure reason
- coverage window

### ROW_22 - View Entitlement Details

Current behavior:

- Entitlement form/detail page reads from `useEntitlementStore`
- related cases are embedded in the entitlement object

API required:

- `GET /api/v1/entitlements/{sys_id}`
- optionally:
  - `GET /api/v1/entitlements/{sys_id}/cases`

### ROW_35 - Case Dashboard

Current behavior:

- Browser routes:
  - `/now/platform-analytics-workspace/dashboard-library`
  - dashboard detail/editor routes under `/now/platform-analytics-workspace/dashboards/...`
- Main files:
  - `frontend/src/stores/useDashboardStore.ts`
  - `frontend/src/stores/useReportStore.ts`
  - `frontend/src/stores/useReportDataSourceStore.ts`
  - `frontend/src/components/dashboards/*`
- Data source:
  - dashboards, reports, and data sources are all local stores
- Chart values:
  - current preview/report data is derived client-side from local case, entitlement, account, and knowledge stores

API required:

- `GET /api/v1/dashboards`
- `GET /api/v1/dashboards/{sys_id}`
- `GET /api/v1/reports/{sys_id}`
- `GET /api/v1/report-data-sources`

If dashboards remain editable:

- `POST /api/v1/dashboards`
- `PUT /api/v1/dashboards/{sys_id}`
- `PATCH /api/v1/dashboards/{sys_id}/bookmark`
- `PATCH /api/v1/dashboards/{sys_id}/deactivate`

### ROW_36 - Agent Performance

Current behavior:

- implemented as predefined dashboards + linked reports/data sources in local stores
- no backend metrics route

API required:

- either reuse generic dashboards/reports routes:
  - `GET /api/v1/dashboards/{agent-performance-dashboard-id}`
  - linked `GET /api/v1/reports/{report_id}`
- or expose dedicated metrics endpoints if backend aggregation should not be report-driven:
  - `GET /api/v1/analytics/agent-performance`

Given the current UI structure, report/dashboard routes are the more natural fit.

### ROW_37 - SLA Compliance Report

Current behavior:

- same pattern as above
- currently modeled as report-backed dashboard content in local stores

API required:

- `GET /api/v1/dashboards/{sla-dashboard-id}`
- `GET /api/v1/reports/{report_id}`
- if backend computes SLA metrics directly:
  - `GET /api/v1/analytics/sla-compliance`

### ROW_38 - Case Volume Trends

Current behavior:

- also implemented as local dashboard/report/data source configuration
- chart data is currently derived in frontend preview code

API required:

- `GET /api/v1/dashboards/{case-volume-dashboard-id}`
- `GET /api/v1/reports/{report_id}`
- optionally direct metric route:
  - `GET /api/v1/analytics/case-volume-trends`

## Recommended API Work Breakdown

### Highest Priority

- Expand case list API for filtering and sorting:
  - `GET /api/v1/cases`
  - optionally `POST /api/v1/cases/search`
- Knowledge article APIs:
  - `GET /api/v1/knowledge-articles`
  - `GET /api/v1/knowledge-articles/{id}`
  - `POST /api/v1/knowledge-articles`
  - `PUT /api/v1/knowledge-articles/{id}`
- Case/article attachment relation APIs:
  - `GET /api/v1/cases/{case_id}/knowledge-attachments`
  - `POST /api/v1/cases/{case_id}/knowledge-attachments`
  - `DELETE /api/v1/cases/{case_id}/knowledge-attachments/{attachment_id}`
- Email APIs:
  - `GET /api/v1/cases/{case_id}/emails`
  - `GET /api/v1/emails/{id}`
  - `POST /api/v1/cases/{case_id}/emails`
- Email template APIs:
  - `GET /api/v1/email-templates`
  - `POST /api/v1/email-templates`
  - `PUT /api/v1/email-templates/{id}`
  - `DELETE /api/v1/email-templates/{id}`
- Saved filter APIs:
  - `GET /api/v1/saved-filters`
  - `POST /api/v1/saved-filters`
  - `PUT /api/v1/saved-filters/{id}`
  - `DELETE /api/v1/saved-filters/{id}`

### Next Priority

- Global search:
  - `GET /api/v1/search`
- Knowledge feedback:
  - `GET /api/v1/knowledge-articles/{id}/feedback`
  - `PUT /api/v1/knowledge-articles/{id}/feedback`
  - `POST /api/v1/knowledge-articles/{id}/feedback/comments`
- Knowledge gap tasks:
  - `POST /api/v1/knowledge-feedback-tasks`
  - `GET /api/v1/knowledge-feedback-tasks/{id}`
  - `PUT /api/v1/knowledge-feedback-tasks/{id}`
- Entitlements:
  - `GET /api/v1/entitlements`
  - `GET /api/v1/entitlements/{id}`
  - `POST /api/v1/entitlements/validate`

### Analytics / Dashboard Priority

- `GET /api/v1/dashboards`
- `GET /api/v1/dashboards/{id}`
- `GET /api/v1/reports`
- `GET /api/v1/reports/{id}`
- `GET /api/v1/report-data-sources`

## Bottom Line

To fully move away from `localStorage`, the missing API work is not just CRUD for the new domains. It also requires:

- server-backed relationship routes for case/article attachments
- server-backed list filtering/sorting for cases
- server-backed saved filters
- server-backed feedback/task flows for knowledge
- server-backed report/dashboard/data-source retrieval
- a single backend search endpoint for header/global search

The only feature area in this list that is already meaningfully wired to backend API routes is case CRUD and basic case list retrieval. Everything else is still frontend-owned state today.
