# NowService Pages Audit Tracker

Generated from `frontend/src/app/**/page.tsx` on 2026-03-31.

Use this checklist to track dead-link cleanup page by page.

Legend:
- `[ ]` not audited yet
- `[-]` in progress
- `[x]` audited and updated

## Core App

- [x] `/` — `frontend/src/app/page.tsx`
- [x] `/502` — `frontend/src/app/502/page.tsx`
- [x] `/coming-soon` — `frontend/src/app/coming-soon/page.tsx`
- [x] `/[...slug]` — `frontend/src/app/[...slug]/page.tsx`
- [x] `/login` — `frontend/src/app/login/page.tsx`
- [x] `/sessionid` — `frontend/src/app/sessionid/page.tsx`
- [x] `/search` — `frontend/src/app/search/page.tsx`
- [x] `/profile` — `frontend/src/app/profile/page.tsx`
- [x] `/lookup-popup` — `frontend/src/app/lookup-popup/page.tsx`
- [x] `/localStorage` — `frontend/src/app/localStorage/page.tsx`
- [x] `/verify_raw` — `frontend/src/app/verify_raw/page.tsx`
- [x] `/demo/panel-entry` — `frontend/src/app/demo/panel-entry/page.tsx`

## Main Records

- [x] `/cases` — `frontend/src/app/cases/page.tsx`
- [x] `/contacts` — `frontend/src/app/contacts/page.tsx`
- [x] `/accounts` — `frontend/src/app/accounts/page.tsx`
- [x] `/assets` — `frontend/src/app/assets/page.tsx`
- [x] `/entitlements` — `frontend/src/app/entitlements/page.tsx`
- [x] `/compose-email` — `frontend/src/app/compose-email/page.tsx`
- [x] `/special-handling-notes` — `frontend/src/app/special-handling-notes/page.tsx`
- [x] `/activity-overrides` — `frontend/src/app/activity-overrides/page.tsx`
- [x] `/escalations` — `frontend/src/app/escalations/page.tsx`
- [x] `/escalation-severity` — `frontend/src/app/escalation-severity/page.tsx`
- [x] `/escalation-templates` — `frontend/src/app/escalation-templates/page.tsx`
- [x] `/task_sla.do` — `frontend/src/app/task_sla.do/page.tsx`
- [x] `/task_sla_list.do` — `frontend/src/app/task_sla_list.do/page.tsx`

## Knowledge

- [x] `/knowledge` — `frontend/src/app/knowledge/page.tsx`
- [x] `/knowledge/[id]` — `frontend/src/app/knowledge/[id]/page.tsx`
- [x] `/knowledge/personal-filters` — `frontend/src/app/knowledge/personal-filters/page.tsx`
- [x] `/my-knowledge-articles` — `frontend/src/app/my-knowledge-articles/page.tsx`

## Agent Workspace

- [x] `/agent` — `frontend/src/app/agent/page.tsx`
- [x] `/now/cwf/agent/home` — `frontend/src/app/now/cwf/agent/home/page.tsx`
- [x] `/now/cwf/agent/list/params/list-id/[listId]` — `frontend/src/app/now/cwf/agent/list/params/list-id/[listId]/page.tsx`
- [x] `/now/cwf/agent/record/interaction/[id]` — `frontend/src/app/now/cwf/agent/record/interaction/[id]/page.tsx`
- [x] `/now/cwf/agent/record/sn_customerservice_case/[id]` — `frontend/src/app/now/cwf/agent/record/sn_customerservice_case/[id]/page.tsx`
- [x] `/now/cwf/agent/record/sn_customerservice_case/[id]/tasks` — `frontend/src/app/now/cwf/agent/record/sn_customerservice_case/[id]/tasks/page.tsx`
- [x] `/now/cwf/agent/record/sn_customerservice_case/[id]/tasks/new` — `frontend/src/app/now/cwf/agent/record/sn_customerservice_case/[id]/tasks/new/page.tsx`
- [x] `/now/cwf/agent/record/sn_customerservice_case/[id]/tasks/[taskId]` — `frontend/src/app/now/cwf/agent/record/sn_customerservice_case/[id]/tasks/[taskId]/page.tsx`

## Analytics And Reports

- [x] `/reports` — `frontend/src/app/reports/page.tsx`
- [x] `/dashboards` — `frontend/src/app/dashboards/page.tsx`
- [x] `/now/platform-analytics-workspace/dashboard-library` — `frontend/src/app/now/platform-analytics-workspace/dashboard-library/page.tsx`
- [x] `/now/platform-analytics-workspace/dashboards/params/edit/[edit]/sys-id/[sys_id]` — `frontend/src/app/now/platform-analytics-workspace/dashboards/params/edit/[edit]/sys-id/[sys_id]/page.tsx`

## Playbook

- [x] `/playbook-content-items` — `frontend/src/app/playbook-content-items/page.tsx`
- [x] `/playbook-data-definitions` — `frontend/src/app/playbook-data-definitions/page.tsx`
- [x] `/playbook-experiences` — `frontend/src/app/playbook-experiences/page.tsx`
- [x] `/playbook-record-generators` — `frontend/src/app/playbook-record-generators/page.tsx`

## Target Resolver

- [x] `/now/nav/ui/classic/params/target/[target]` — `frontend/src/app/now/nav/ui/classic/params/target/[target]/page.tsx`

## Notes

- The goal is zero dead links or dead-end clickable controls on implemented pages.
- Any intentionally unavailable action should navigate to `/502`.
- When already on `/502`, further dead-end clicks should not create another `/502` history entry.

## Audit Matrix

Use this section for the real sweep. The checklist above answers "which routes exist"; this section answers "which shared surfaces must be validated on each route family."

### Shared Chrome

Validate these once per major shell, then only re-check when a page uses custom header/sidebar behavior.

- Global header
  - All
  - Favorites
  - History
  - Workspaces
  - Admin
  - Search
  - Application scope / update set
  - Sidebar discussions
  - Contact analytics
  - Menu / hamburger
  - Help center
  - Notifications
  - Profile
- Left navigation / app rail
- Global popovers
  - Favorites popover
  - Notifications popover
  - Help center popover
  - Workspaces / All / Admin menus

### Target Route Families

These are higher leverage than route-by-route clicking because many dead links resolve through the same target page.

- `sn_customerservice_case_list.do`
- `sn_customerservice_case.do`
- `sn_customerservice_email.do`
- `kb_view.do`
- `m2m_kb_task.do`
- `sys_report.do`
- `sys_report_template.do`
- `task_sla.do`
- `task_sla_list.do`
- Unknown / unsupported targets via `/now/nav/ui/classic/params/target/[target]`

### Page Family Passes

For each page family below, verify:
- page-level header actions
- table toolbar actions
- row actions / context menus
- inline banner links
- empty-state CTAs
- footer toolbars
- dialogs / popovers launched from the page

#### Core App

- `/`
- `/search`
- `/profile`
- `/lookup-popup`
- `/demo/panel-entry`

#### Record Lists

- `/cases`
- `/contacts`
- `/accounts`
- `/assets`
- `/entitlements`
- `/activity-overrides`
- `/escalations`
- `/escalation-severity`
- `/escalation-templates`
- `/special-handling-notes`

#### Knowledge

- `/knowledge`
- `/knowledge/[id]`
- `kb_view.do`
- `/knowledge/personal-filters`
- `/my-knowledge-articles`

#### Agent Workspace

- `/agent`
- `/now/cwf/agent/home`
- `/now/cwf/agent/list/params/list-id/[listId]`
- `/now/cwf/agent/record/interaction/[id]`
- `/now/cwf/agent/record/sn_customerservice_case/[id]`
- `/now/cwf/agent/record/sn_customerservice_case/[id]/tasks`
- `/now/cwf/agent/record/sn_customerservice_case/[id]/tasks/new`
- `/now/cwf/agent/record/sn_customerservice_case/[id]/tasks/[taskId]`

#### Analytics

- `/reports`
- `sys_report.do`
- `sys_report_template.do`
- `/dashboards`
- `/now/platform-analytics-workspace/dashboard-library`
- `/now/platform-analytics-workspace/dashboards/params/edit/[edit]/sys-id/[sys_id]`

#### Playbook

- `/playbook-content-items`
- `/playbook-data-definitions`
- `/playbook-experiences`
- `/playbook-record-generators`

### Recording Findings

Append findings in this format while sweeping:

- `[route or surface]` `implemented` `dead-end fixed` `verified`
- control:
  - label or aria-label
  - actual behavior before fix
  - final behavior after fix
  - file owning the fix

### Priority Order

Use this order for the remaining sweep:

1. Shared chrome and popovers
2. Target route families
3. List pages with shared toolbars / row menus
4. Detail pages with custom headers / related tables
5. Dialogs and secondary states opened from those pages
