# Move Notifications & Task SLAs to Backend-Backed

## Description
Remove all localStorage persistence for notifications and task SLAs. Create backend API endpoints where missing and wire the frontend to fetch/mutate data via React Query hooks following the escalations pattern.

## Progress
- [x] Backend: Create Notification model, schema, service, endpoints, fixtures
- [x] Backend: Extend Task SLA endpoints from read-only to full CRUD (POST, PUT, DELETE, batch update by task)
- [x] Frontend: Create `useNotifications` React Query hooks
- [x] Frontend: Create `useTaskSlas` React Query hooks
- [x] Frontend: Strip localStorage persistence from notification and task SLA stores
- [x] Frontend: Update all consumers (NotificationsDropdown, CaseFormPage, useCaseStore, TaskSlaFormPage, TaskSlaFormTarget, useAgentListData, useAgentCaseFormsAndListData, AgentRecordInformationContent)
- [x] Remove `NOTIFICATION_LIST`, `TASK_SLA_LIST`, `TASK_SLA_FORM` from localStorage keys
- [x] Backend: Scope notifications to authenticated user (auto-set `user_id` on create, filter on list/mark-all-read)
- [x] Frontend: Singleton QueryClient + cache invalidation in fire-and-forget helpers
- [x] Frontend: Single-item task SLA fetch (`useTaskSlaQuery`, `fetchTaskSlaApi`) — eliminates form edit race condition

## Commits
- `a69f6fb` — Add notifications backend and task SLA write endpoints
- `204c0e1` — Add React Query hooks and remove localStorage from notification/SLA stores
- `85dcefe` — Wire all notification and task SLA consumers to backend API
- `b5e9c51` — Scope notifications to the authenticated user
- `cb7b67b` — Invalidate query cache after fire-and-forget API calls
- `f4a3be8` — Add single-item task SLA fetch and use it for form edit

## Decisions
- **Fire-and-forget with cache invalidation**: `useCaseStore` is a Zustand store whose actions can't use React hooks. Created standalone async functions that call `sendJson` directly and then invalidate the relevant query cache via a singleton `QueryClient` exported from `query-client.ts`.
- **No Alembic migrations**: Tables created via `Base.metadata.create_all`. The new `Notification` model is picked up by importing in `models/__init__.py`.
- **`TaskSlaListItem` type kept in store**: Re-exported from `useTaskSlaListStore` for backward compatibility. Canonical source is now the React Query hook types.
- **User-scoped notifications via backend auth**: The backend auto-extracts `user_id` from the auth token — no frontend changes needed. List, create, and mark-all-read all scope by user.
- **Single-item fetch for form edit**: `TaskSlaFormTarget` and agent form store now use `fetchTaskSlaApi(id)` / `useTaskSlaQuery(id)` instead of searching the full list, avoiding the race condition where the form stayed empty until the list query resolved.

## Watch Out For
- **No delete UI for task SLAs**: `useDeleteTaskSlaMutation` hook exists but no UI wires to it yet.
- **Fixture seeding**: `notifications.json` seeds 5 records matching the old `DEFAULT_NOTIFICATIONS`. Seeded notifications have no `user_id` — they won't appear for authenticated users. Re-seed or manually assign `user_id` if needed.
- **No backend test infrastructure**: Backend uses PostgreSQL-specific types (`UUID(as_uuid=True)`) and run-scoped databases, so integration tests need a conftest with test Postgres, mocked auth, and dependency overrides. This infrastructure doesn't exist yet.

## Next Steps
- [ ] Set up backend test infrastructure (conftest with test DB, mocked auth, TestClient)
- [ ] Write integration tests for notification and task SLA endpoints
- [ ] Wire delete UI for task SLAs if needed

## Outcome
All localStorage persistence removed for notifications and task SLAs. Both are now fully backend-backed with React Query hooks. Notifications are scoped to the authenticated user. Fire-and-forget helpers invalidate the query cache immediately on success. Single-item fetch eliminates form edit race condition. Six commits landed on `chore/more-integrations`.
