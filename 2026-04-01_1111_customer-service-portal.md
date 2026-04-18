# Customer Service Portal

Date: 2026-04-01
Repo inspected: `/Users/v1b3m/Dev/Turing/ServiceNow`

## Scope

Checked whether the frontend implementations of the portal-facing features below persist any feature data to `localStorage`.

Important distinction:
- API-backed data means the feature reads/writes through backend endpoints.
- localStorage persistence means the frontend stores feature state or feature data in the browser.
- Auth/session storage is global app behavior and is not the same as feature-specific persistence.

## Summary

- I did not find a distinct customer-facing portal shell or dedicated “Customer Service Portal” app/page in the frontend.
- Case creation and case status viewing are implemented through the shared cases UI and shared cases API.
- Knowledge search is implemented and is API-backed.
- I did not find a real Virtual Agent frontend implementation.
- I found a `PORTAL_CHAT` localStorage key constant, but no store or component using it.

## Findings

### Portal Home Page

Feature:
- Customer-facing self-service portal landing page

Status:
- No dedicated frontend implementation found

localStorage usage:
- None found for this feature

Notes:
- I did not find a route/component clearly acting as a customer portal landing page.
- Searches for `portal home`, `self-service portal`, `customer portal`, and `service portal` in frontend app/components did not return a concrete implementation.

Evidence:
- No dedicated route/component found under `frontend/src/app` or `frontend/src/components`

### Submit Case from Portal

Feature:
- Customer can create a new case via portal

Status:
- Case creation exists in frontend, but via the shared case UI rather than a dedicated portal flow

Backend/API:
- Uses the real cases API:
  - `POST /api/v1/cases`

Frontend usage:
- `frontend/src/hooks/cases/caseApi.ts`
- `frontend/src/hooks/cases/useCases.ts`
- `frontend/src/stores/useCaseStore.ts`
- `frontend/src/components/case/CaseFormTarget.tsx`
- `frontend/src/components/case/CaseFormPage.tsx`

localStorage usage:
- No feature data persistence found for case creation itself
- `useCaseStore` is not persisted with Zustand `persist`
- The case form data is kept in memory while the page is open

Related localStorage:
- Case list UI preferences and navigation state are persisted in `useCaseListStore` under `case-list-storage`
- Auth token/session is persisted in `useAuthStore` under `redux_auth`

What is persisted vs not:
- Persisted:
  - auth/session
  - case list UI state
- Not persisted:
  - submitted case form draft in `useCaseStore`
  - created case data itself in frontend localStorage

Evidence:
- `frontend/src/stores/useCaseStore.ts`
- `frontend/src/stores/useCaseListStore.ts`
- `frontend/src/stores/useAuthStore.ts`

### Track Case Status

Feature:
- Customer can view case progress and updates

Status:
- Implemented through shared case detail UI and real cases API

Backend/API:
- Uses the real cases API:
  - `GET /api/v1/cases/{id}`

Frontend usage:
- `frontend/src/hooks/cases/useCases.ts` via `useCaseQuery`
- `frontend/src/components/case/CaseFormTarget.tsx`
- `frontend/src/components/case/CaseFormPage.tsx`
- `frontend/src/components/case/CaseActivities.tsx`

Data shown:
- `state`
- timestamps
- `activities`
- `work_notes`
- `additional_comments`

localStorage usage:
- No feature data persistence found for case status/progress itself
- The displayed case detail data comes from API fetches, not from a persisted local store

Related localStorage:
- Case list UI state may persist independently in `useCaseListStore`
- Auth/session persists in `useAuthStore`

What is persisted vs not:
- Persisted:
  - auth/session
  - list UI state
- Not persisted:
  - case progress/status payload
  - activities/work notes/comments as a frontend cache in localStorage

Evidence:
- `frontend/src/hooks/cases/useCases.ts`
- `frontend/src/components/case/CaseFormTarget.tsx`
- `frontend/src/components/case/CaseFormPage.tsx`
- `frontend/src/components/case/CaseActivities.tsx`

### Portal Knowledge Search

Feature:
- Customer searches knowledge base from portal

Status:
- Knowledge browsing/search exists in frontend and is API-backed

Backend/API:
- Uses the real knowledge API:
  - `GET /api/v1/knowledge-articles?...`

Frontend usage:
- `frontend/src/app/search/page.tsx`
  Uses `useKnowledgeArticles` to include knowledge results in global search
- `frontend/src/app/knowledge/page.tsx`
  Knowledge listing page
- `frontend/src/components/knowledge/hooks/useKnowledgeArticles.ts`
- `frontend/src/lib/api/knowledge.ts`

localStorage usage:
- No localStorage persistence found for knowledge search itself
- `useKnowledgeStore` is an in-memory Zustand store and does not use `persist`

Related localStorage:
- Knowledge article feedback has its own persisted store:
  - `knowledge-article-feedback-storage`
- Knowledge feedback tasks also persist:
  - `knowledge-feedback-tasks-storage`
- Those are adjacent knowledge features, not the knowledge search feature itself

What is persisted vs not:
- Persisted:
  - knowledge feedback state
  - knowledge feedback task state
- Not persisted:
  - knowledge search query/results as localStorage state in the main search/list flow

Evidence:
- `frontend/src/app/search/page.tsx`
- `frontend/src/app/knowledge/page.tsx`
- `frontend/src/components/knowledge/hooks/useKnowledgeArticles.ts`
- `frontend/src/stores/useKnowledgeStore.ts`
- `frontend/src/stores/useKnowledgeArticleFeedbackStore.ts`
- `frontend/src/stores/useKnowledgeFeedbackTaskStore.ts`

### Virtual Agent

Feature:
- Chatbot for self-service case creation and info lookup

Status:
- No real Virtual Agent frontend implementation found

What was found:
- “Virtual Agent” appears as a selectable channel/label in the case UI
- `PORTAL_CHAT` exists as a localStorage key constant
- No persisted portal chat store or Virtual Agent component was found using that key

Frontend usage found:
- `frontend/src/components/case/CaseFormPage.tsx`
  Shows `Virtual Agent` as a channel option
- `frontend/src/stores/local-storage-keys.ts`
  Defines `PORTAL_CHAT: "portal-chat-storage"`

localStorage usage:
- No actual usage found

Notes:
- I found no `usePortalChatStore`, no component writing to `portal-chat-storage`, and no dedicated Virtual Agent/chatbot page.
- The nearest chat-like persisted feature is `useSidebarDiscussionsStore`, but that is internal discussion/thread functionality, not a customer Virtual Agent.

Evidence:
- `frontend/src/components/case/CaseFormPage.tsx`
- `frontend/src/stores/local-storage-keys.ts`
- `frontend/src/stores/useSidebarDiscussionsStore.ts`

## Storage Inventory Relevant to These Features

### Definitely persisted to localStorage

- Auth/session:
  - `redux_auth`
  - plus direct `run_id` and `current_run_id`
- Case list UI state:
  - `case-list-storage`
- Knowledge article feedback:
  - `knowledge-article-feedback-storage`
- Knowledge feedback tasks:
  - `knowledge-feedback-tasks-storage`

### Defined as keys but not confirmed in use for these portal features

- `portal-chat-storage`

### Not persisted as localStorage feature data

- Case create form state in `useCaseStore`
- Case detail/status/progress payloads from the API
- Knowledge search query/results in the main knowledge/search flows
- Virtual Agent/chat transcript, because no implementation was found

## Conclusion

For the features listed here, the frontend mostly does not persist the actual feature data to `localStorage`.

The current pattern is:
- Cases: real backend API for create/detail, with only UI/list state persisted locally
- Knowledge search: real backend API or in-memory store, not localStorage
- Virtual Agent: no concrete implementation found, and no active localStorage-backed chat store found
- Portal home page: no dedicated implementation found
