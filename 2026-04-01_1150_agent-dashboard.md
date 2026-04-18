## Agent Dashboard

### Location

The agent dashboard is implemented in the frontend at:

- [frontend/src/app/agent/page.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/agent/page.tsx)
- [frontend/src/app/now/cwf/agent/home/page.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/now/cwf/agent/home/page.tsx)
- [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx)

Both routes render the same `AgentHomeDashboard` component.

### Backend API

The dashboard backend API from the referenced spec is present:

- `GET /api/v1/agent-dashboard/my-cases`
- `GET /api/v1/agent-dashboard/my-team-cases`
- `GET /api/v1/agent-dashboard/case-statistics`

Backend files:

- [backend/app/api/v1/endpoints/agent_dashboard.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/agent_dashboard.py)
- [backend/app/services/agent_dashboard.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/services/agent_dashboard.py)
- [backend/app/schemas/agent_dashboard.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/schemas/agent_dashboard.py)
- [backend/app/api/v1/__init__.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/__init__.py)

Frontend API hooks:

- [frontend/src/hooks/agent-dashboard/useAgentDashboardCaseListQueries.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/agent-dashboard/useAgentDashboardCaseListQueries.ts)
- [frontend/src/hooks/agent-dashboard/useAgentDashboardCaseStatistics.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/agent-dashboard/useAgentDashboardCaseStatistics.ts)

### Feature Check

#### Agent Home Dashboard

Status: Implemented  
API-connected: Yes

Evidence:

- `AgentHomeDashboard` is the actual dashboard shell in [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L361)
- It loads:
  - statistics from `useAgentDashboardCaseStatisticsQuery('team')` in [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L366)
  - my cases from `useAgentDashboardMyCasesQuery(...)` in [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L367)
  - team cases from `useAgentDashboardMyTeamCasesQuery(...)` in [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L368)

Conclusion:

- The personalized dashboard exists and is connected to the dedicated backend dashboard API.

#### My Open Cases Widget

Status: Implemented, with naming mismatch  
API-connected: Yes

Current implementation:

- The dashboard renders a cases panel titled `My active cases` in [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L518)
- That panel is backed by `myCasesQuery.data?.cases` in [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L395)
- The frontend fetches from `GET /api/v1/agent-dashboard/my-cases` in [frontend/src/hooks/agent-dashboard/useAgentDashboardCaseListQueries.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/agent-dashboard/useAgentDashboardCaseListQueries.ts#L10) and [frontend/src/hooks/agent-dashboard/useAgentDashboardCaseListQueries.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/agent-dashboard/useAgentDashboardCaseListQueries.ts#L58)
- The backend route is implemented in [backend/app/api/v1/endpoints/agent_dashboard.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/agent_dashboard.py#L24)

Important nuance:

- The backend endpoint returns all cases assigned to the signed-in agent.
- It does not filter to only open/non-terminal cases.
- The UI title says `My active cases`, but the endpoint logic in [backend/app/services/agent_dashboard.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/services/agent_dashboard.py#L70) only filters by `assigned_to_id == user_id`.

Conclusion:

- There is a real agent-cases widget and it is API-backed.
- It is not strictly a "My Open Cases" widget yet unless terminal states are excluded.

#### Unassigned Cases Widget

Status: Partially implemented  
API-connected: Yes, but only as a count

Current implementation:

- The dashboard `Important items` cards include `Unassigned cases` in [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L24)
- That card is populated from the statistics response mapping in [frontend/src/hooks/agent-dashboard/useAgentDashboardCaseStatistics.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/agent-dashboard/useAgentDashboardCaseStatistics.ts#L45)
- `unassignedCases` comes from `GET /api/v1/agent-dashboard/case-statistics` in [frontend/src/hooks/agent-dashboard/useAgentDashboardCaseStatistics.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/agent-dashboard/useAgentDashboardCaseStatistics.ts#L37)
- The backend count is calculated in [backend/app/services/agent_dashboard.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/services/agent_dashboard.py#L178)

Gap versus requested feature:

- The requested feature says: `Widget showing cases in queue needing assignment`
- Current implementation only shows the count
- There is no dedicated unassigned-cases list widget
- There is no frontend call to `GET /api/v1/agent-dashboard/my-team-cases?state=...` for unassigned-only rows
- There is also no dedicated backend endpoint that returns only unassigned queue cases

Conclusion:

- Unassigned cases exist as a real backend-backed metric.
- The list/widget described in the requirement is not fully implemented.

#### SLA At Risk Widget

Status: Partially implemented  
API-connected: Yes, but as a proxy count

Current implementation:

- The dashboard `Important items` cards include `SLA breached or due today` in [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L24)
- That value comes from `slaBreached` in the statistics hook mapping in [frontend/src/hooks/agent-dashboard/useAgentDashboardCaseStatistics.ts](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/agent-dashboard/useAgentDashboardCaseStatistics.ts#L45)
- Backend computation is in [backend/app/services/agent_dashboard.py](/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/services/agent_dashboard.py#L188)

Important nuance:

- The backend explicitly says there is no dedicated SLA table
- The current `slaBreached` is a proxy:
  - high/critical priority
  - non-terminal case
  - no activity in the last 24 hours

This is not the same as:

- real SLA timers
- real countdowns
- real "at risk" thresholding from SLA records

Conclusion:

- There is a backend-connected SLA-risk style widget.
- It is only an approximation, not a true SLA-at-risk implementation.

### Additional Current Dashboard Panels

The dashboard also includes:

- `My team's cases` list panel in [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L528)
- `Performance` cards in [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L538)

Notes:

- `My team's cases` is API-backed via `GET /api/v1/agent-dashboard/my-team-cases`
- Performance cards are placeholders with hardcoded zero values in [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L32) and [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L546)

### Summary

| Feature | Implemented | API-connected | Notes |
| --- | --- | --- | --- |
| Agent Home Dashboard | Yes | Yes | Real dashboard route and backend API hooks exist |
| My Open Cases Widget | Partial | Yes | Implemented as `My active cases`, but backend does not currently restrict to open/non-terminal cases |
| Unassigned Cases Widget | Partial | Yes | Count exists via statistics API, but no dedicated unassigned list widget |
| SLA At Risk Widget | Partial | Yes | Count exists via statistics API, but it is an SLA proxy, not real SLA data |

### Gap List Against The Referenced Spec

1. `my-cases` should probably exclude terminal states if the intended widget is "My Open Cases".
2. Unassigned cases need a dedicated list widget or filtered queue panel, not just a summary count.
3. SLA at risk needs real SLA-backed semantics if the product expectation is actual SLA countdown/risk logic.
4. Refresh behavior in the dashboard currently invalidates `['cases']` in [frontend/src/components/agent/AgentHomeDashboard.tsx](/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/agent/AgentHomeDashboard.tsx#L375), not the `agent-dashboard` query keys directly.

### Context

- [2026-04-01_1153_agent-dashboard-spec](/Users/v1b3m/Dev/KnowledgeBase/turing-kb/2026-04-01_1153_agent-dashboard-spec.md)
