# localStorage Usage Report — ServiceNow Frontend

**Date:** 2026-04-02  
**Scope:** `frontend/src/`

---

## Utility Layer

**`src/lib/localStorage.ts`** — Wrapper library providing safe localStorage access with JSON parsing, error handling, and SSR guards (`typeof window === 'undefined'`).

| Function | Operation |
|----------|-----------|
| `getLocalStorageItem<T>(key)` | GET with JSON parse |
| `setLocalStorageItem<T>(key, value)` | SET with JSON stringify |
| `removeLocalStorageItem(key)` | REMOVE |

---

## All Storage Keys

| Key | Constant | Mechanism | Purpose |
|-----|----------|-----------|---------|
| `redux_auth` | `LOCAL_STORAGE_KEYS.AUTH` | Zustand persist | Auth state: user profile, access_token, run_id |
| `run_id` | — | Direct set | Session run ID |
| `current_run_id` | — | Direct set | Duplicate of run ID |
| `nowservice-theme` | `THEME_STORAGE_KEY` | Direct get/set | Theme preference (`coral` / `polaris`) |
| `current-storage` | `LOCAL_STORAGE_KEYS.CURRENT` | Zustand persist | Current case ID, last accessed slug |
| `history-storage` | `LOCAL_STORAGE_KEYS.HISTORY` | Zustand persist | Visit history (capped at 50) |
| `user-storage` | `LOCAL_STORAGE_KEYS.USER` | Zustand persist | User profile (not actually persisted — always defaults) |
| `notification-list` | `LOCAL_STORAGE_KEYS.NOTIFICATION_LIST` | Zustand persist | Notification items and read state |
| `task-sla-list-storage` | `LOCAL_STORAGE_KEYS.TASK_SLA_LIST` | Zustand persist | SLA list sort column, filters |
| `knowledge-feedback-tasks-storage` | `LOCAL_STORAGE_KEYS.KNOWLEDGE_FEEDBACK_TASKS` | Zustand persist | Feedback tasks for knowledge articles |
| `knowledge-article-feedback-storage` | `LOCAL_STORAGE_KEYS.KNOWLEDGE_ARTICLE_FEEDBACK` | Zustand persist | Article feedback ratings by article ID |
| `followed-cases-storage` | `LOCAL_STORAGE_KEYS.FOLLOWED_CASES` | Direct wrapper | Array of followed/favorited case numbers |

---

## Usage by Domain

### Authentication

| File | Operations | Notes |
|------|------------|-------|
| `stores/useAuthStore.ts` | SET `run_id`, SET `current_run_id`, `localStorage.clear()` on logout, Zustand persist for `redux_auth` | Main auth persistence |
| `lib/auth/api.ts` | GET `redux_auth` | Extracts access_token for API calls |
| `components/auth/ProtectedRoute.tsx` | GET `redux_auth` | Checks for valid token on mount |
| `components/auth/UnauthHandler.tsx` | `localStorage.clear()` | Wipes everything on 401 response |

### Theme

| File | Operations | Notes |
|------|------------|-------|
| `components/layout/Header.tsx` | GET `nowservice-theme` | Loads saved theme on mount |
| `components/layout/preferences/ThemePanel.tsx` | GET/SET `nowservice-theme` | Reads and saves theme selection |

### Case Management

| File | Operations | Notes |
|------|------------|-------|
| `components/case/CasePageHeader.tsx` | GET/SET `followed-cases-storage` | Toggle follow/unfollow on cases |

### Zustand Persisted Stores

All use `createJSONStorage(() => localStorage)` with `skipHydration: typeof window === 'undefined'`.

| Store | Key | What's Stored |
|-------|-----|---------------|
| `useCurrentStore.ts` | `current-storage` | Current case ID, last slug |
| `useHistoryStore.ts` | `history-storage` | Visit history (capped at 50) |
| `useUserStore.ts` | `user-storage` | User profile (effectively unused — always defaults) |
| `useNotificationStore.ts` | `notification-list` | Notifications and read state |
| `useTaskSlaListStore.ts` | `task-sla-list-storage` | SLA list UI state |
| `useKnowledgeFeedbackTaskStore.ts` | `knowledge-feedback-tasks-storage` | Knowledge feedback tasks |
| `useKnowledgeArticleFeedbackStore.ts` | `knowledge-article-feedback-storage` | Feedback ratings |

### Debugging

| File | Operations | Notes |
|------|------------|-------|
| `app/localStorage/page.tsx` | Iterates all keys via `localStorage.key(i)` | Admin page at `/localStorage` that exports all storage as JSON |

---

## Destructive Operations

- **`localStorage.clear()`** is called in two places:
  1. `useAuthStore.ts` — on explicit logout
  2. `UnauthHandler.tsx` — on 401 response

Both wipe **all** keys, not just auth-related ones. This means theme preferences, followed cases, history, and notification state are all lost on logout or session expiry.

---

## Observations

1. **`run_id` duplication** — `useAuthStore` writes both `run_id` and `current_run_id` with the same value. Unclear why both exist.
2. **`user-storage` is a no-op** — The store exists but always uses defaults, making the persistence unnecessary.
3. **`localStorage.clear()` is aggressive** — Wiping all keys on logout destroys user preferences (theme, followed cases) that could reasonably survive across sessions.
4. **No key prefix/namespace** — Keys are bare strings with no app-level prefix, risking collisions if the app shares a domain with other apps.
5. **`/localStorage` debug page** — Exposes all stored data. Should be gated behind a dev/admin check in production.
