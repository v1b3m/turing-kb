Portal path in current branch: `/now/cwf/agent/home`

This is not the old `/csm` portal branch. It is the current agent workspace home. For the scope below, only the interaction / virtual-agent-like flow is materially local-first. The other listed features are already using backend APIs.

## Customer Service Portal

### Portal Home Page

Status: Mostly backend-backed, with some local UI state.

What this page is:
- `frontend/src/app/now/cwf/agent/home/page.tsx`
- Main dashboard component: `frontend/src/components/agent/AgentHomeDashboard.tsx`

Backend-backed data on the home page:
- My cases: `GET /api/v1/agent-dashboard/my-cases`
- My team cases: `GET /api/v1/agent-dashboard/my-team-cases`
- Dashboard counters: `GET /api/v1/agent-dashboard/case-statistics`
- Supporting case cache invalidation also uses `/api/v1/cases`

Relevant frontend/backend files:
- `frontend/src/components/agent/AgentHomeDashboard.tsx`
- `frontend/src/hooks/agent-dashboard/useAgentDashboardCaseListQueries.ts`
- `frontend/src/hooks/agent-dashboard/useAgentDashboardCaseStatistics.ts`
- `backend/app/api/v1/endpoints/agent_dashboard.py`
- `frontend/src/hooks/cases/useCases.ts`

LocalStorage usage:
- Not for the dashboard metrics or case widgets themselves.
- The page persists open interaction tabs through `useAgentWorkspaceInteractionsStore`.
- Auth/session is persisted globally in `redux_auth`, but that is not a portal-feature data source.

Conclusion:
- Home dashboard content is backend-backed.
- Only workspace interaction tab state is localStorage-backed here.

### Submit Case from Portal

Status: Backend-backed.

Flow:
- Home tab header links `Create New Case` to `/now/cwf/agent/record/sn_customerservice_case/-1`
- Form component: `frontend/src/components/agent-case/AgentCreateCaseRecord.tsx`
- Save uses `useCreateCaseMutation()` and posts to `POST /api/v1/cases`

Relevant files:
- `frontend/src/app/now/cwf/agent/home/page.tsx`
- `frontend/src/components/agent-case/AgentTabHeader.tsx`
- `frontend/src/app/now/cwf/agent/routes.ts`
- `frontend/src/components/agent-case/AgentCreateCaseRecord.tsx`
- `frontend/src/hooks/cases/useCases.ts`
- `frontend/src/hooks/cases/caseApi.ts`
- `backend/app/api/v1/endpoints/cases.py`

LocalStorage usage:
- The form state itself uses `useAgentCaseStore`, which is not persisted.
- Temporary upload attachments use `useAttachmentStore`, which is client-only in-memory, not localStorage.
- The actual case record is created through the backend API.

Conclusion:
- Case submission is already backend-backed.
- The only client-only part in this flow is temporary attachment staging before save.

### Track Case Status

Status: Backend-backed.

Flow:
- Case record view route loads the case by id with `useCaseQuery(id)`
- That calls `GET /api/v1/cases/{id}`
- Emails tab also queries backend emails for the case number

Relevant files:
- `frontend/src/app/now/cwf/agent/record/sn_customerservice_case/[id]/AgentRecordViewPage.tsx`
- `frontend/src/hooks/cases/useCases.ts`
- `frontend/src/hooks/cases/caseApi.ts`
- `frontend/src/hooks/emails/useEmails.ts`
- `backend/app/api/v1/endpoints/cases.py`

LocalStorage usage:
- Not for the case record itself.
- Case progress, activities, and record details come from the backend case API.

Conclusion:
- Case status/progress tracking is backend-backed.

### Portal Knowledge Search

Status: Backend-backed.

Flow:
- Knowledge data in this workspace uses the knowledge API hooks, not generated local article data.
- Main fetchers call:
  - `GET /knowledge-articles`
  - `GET /knowledge-articles/{articleId}`

Relevant files:
- `frontend/src/lib/api/knowledge.ts`
- `frontend/src/components/knowledge/hooks/useAllKnowledgeArticles.ts`
- `frontend/src/components/knowledge/hooks/useKnowledgeArticleQuery.ts`
- `backend/app/api/v1/endpoints/knowledge_articles.py`

LocalStorage usage:
- I did not find portal knowledge search state persisted in localStorage for this route.
- There are separate feedback/task stores elsewhere in the app that persist UI/task state, but not the core knowledge search/data fetch itself.

Conclusion:
- Knowledge search and article retrieval are backend-backed.

### Virtual Agent

Status: Local-first / static-ui-like.

What exists in current branch:
- Interaction tabs opened from `/now/cwf/agent/home`
- Interaction details screen: `frontend/src/components/agent/AgentInteractionDetails.tsx`
- Interaction tab state and editable form values are stored in persisted Zustand stores

Relevant files:
- `frontend/src/components/agent/AgentInteractionDetails.tsx`
- `frontend/src/stores/useAgentWorkspaceInteractionsStore.ts`
- `frontend/src/stores/useAgentInteractionStore.ts`
- `frontend/src/app/now/cwf/agent/home/page.tsx`

LocalStorage usage:
- `agent-workspace-interactions-storage`
- `agent-interaction-storage`

What is backend-backed inside this screen:
- Account lookup options come from the accounts API
- Contact lookup options come from the contacts API

What is not backend-backed:
- The interaction record itself
- The interaction tab/session lifecycle
- Save action for the interaction form

Conclusion:
- This is the main feature in scope that is still localStorage-backed and would need a real backend resource if it is meant to be a true virtual agent / interaction system.

## Summary

Backend-backed already:
- Portal Home Page dashboard widgets
- Submit Case from Portal
- Track Case Status
- Portal Knowledge Search

LocalStorage-backed / client-only:
- Virtual Agent interaction session and saved interaction form state

Migration priority:
1. Virtual Agent / interactions
2. Optional attachment upload persistence for create-case if required

Bottom line:
- The note should not treat the whole portal as static UI.
- In the current `/now/cwf/agent/home` implementation, the only clearly local-first feature among the listed items is the virtual-agent / interaction flow.
