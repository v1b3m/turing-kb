In this project we shall be cloning taskrabbit. The purpose of this is to create an RL (Reinforcement Learning) gym for training autonomous browser agents.

---

## Stack

- **Next.js** 14.2 (App Router) with **React 18** and **TypeScript 5**
- **MUI** v7.3 + Emotion for UI components and styling
- **Tailwind CSS** v3.3 (minimal usage — 4 files only, slated for removal)
- **better-sqlite3** for server-side persistence (SQLite with WAL mode)

## Project Structure

```
src/
├── app/              # Next.js App Router — pages and API routes
├── components/
│   ├── ui/           # 42 reusable UI primitives (Button, Card, Modal, Calendar, etc.)
│   ├── task/         # Multi-step booking flow sections
│   ├── recommendations/  # Tasker recommendation components
│   └── providers/    # ThemeProvider (MUI + CssBaseline)
├── config/           # Constants — categories, routes, step labels
├── data/             # Static tasker datasets (large JSON/TS files)
├── hooks/            # usePersistedState, useTaskerFilters
├── lib/server/       # SQLite database setup and prepared statements
├── theme/            # Design tokens + MUI theme
├── types/            # TypeScript definitions (AppState, form data, etc.)
└── utils/            # Utility functions
```

## Routing

**Core pages:** `/` (home/booktask), `/mytasks`, `/account`, `/getdiscount`, `/design-system`

**Task category routes** — most follow a 4-step booking flow:
1. Main category page (form/options)
2. `/details`
3. `/schedule`
4. `/confirm-details`

Implemented categories with full flows: yard-work, plumbing-help, trash-removal.
Partial/coming-soon: electrical-work, heavy-lifting, tv-mounting, furniture-assembly, cleaning, moving, and others.

**API routes:**
- `/api/health` — health check
- `/api/reset_state` — state reset for testing
- `/api/v1/run/*` — run management (init, verify, export)
- `/api/v1/kv/*` — key-value store (bulk-get, bulk-upsert, clear-run)

## State Management

Client-side localStorage via a custom `usePersistedState` hook. All app state lives under the key `task_rabbit_state` and follows a comprehensive `AppState` interface covering auth, bookings, taskers, payments, messaging, and UI state.

## Design System

Custom tokens extracted from real TaskRabbit design:
- Primary color: teal `#0a6b5a`
- 4px base spacing unit
- Inter font family
- Pill-shaped buttons, teal-tinted shadows
- Tokens are defined in `src/theme/design-tokens.ts` and wired into MUI via `src/theme/index.ts`

## Database

SQLite (better-sqlite3) with two tables:
- `runs` — run registry (run_id, created_at)
- `kv` — key-value store scoped per run

Located at `/tmp/app.db` (Vercel), `/data/app.db` (prod), `./app.db` (local).

## Notable Patterns

- Multi-step booking flow with per-category step counts and labels
- Large static tasker datasets (~200KB+ each) for realistic UI
- Prepared SQL statements and idempotent `INSERT OR IGNORE` patterns
- Heavy use of `'use client'` for interactive components
- Props-driven controlled form inputs across booking sections
