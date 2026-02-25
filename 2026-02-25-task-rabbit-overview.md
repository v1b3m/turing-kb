# Task Rabbit — Project Overview

A **Next.js 14 RL-Gym environment** that simulates the TaskRabbit marketplace for training and evaluating AI models on multi-step hiring workflows.

## What It Does

Simulates a full home-services marketplace where AI agents navigate service discovery, tasker filtering, booking flows, and payment — then verifies correctness via an assertion engine.

**Core user flows:**
1. Browse/search 15+ services (furniture assembly, moving, cleaning, etc.)
2. Filter taskers by location, rate, rating, elite status
3. Complete a 4-step booking (details → schedule → hire → confirm)
4. IKEA-specific product search and assembly booking
5. View task history on a dashboard

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript |
| UI | Radix UI + Tailwind CSS 3.3 |
| State | Zustand 5 (localStorage-persisted) |
| Data Fetching | TanStack React Query 5 |
| Database | better-sqlite3 (server-side KV for runs) |
| Data | Static JSON (chunked taskers, locations, IKEA catalog) |
| Verification | @turing-rlgym/cua-gym-utils (JSONPath assertions) |
| Container | Docker (Node 20, standalone build) |

## Directory Structure

```
src/
├── app/                # Next.js App Router
│   ├── (pages)/        # Route groups (booktask, locations, book, dashboard, near-me)
│   ├── api/            # REST endpoints (taskers, locations, ikea, services, verification)
│   ├── ikea/           # IKEA search pages
│   └── verify_raw/     # Verification UI
├── components/         # React components by domain
│   ├── booking/        # Booking flow forms
│   ├── ikea/           # IKEA search, results, cards
│   ├── services/       # Service browsing, breadcrumbs
│   ├── profile/        # Tasker profiles
│   ├── ui/             # Shared primitives (forms, inputs, buttons)
│   └── verifier/       # Verification display
├── stores/             # Zustand stores
│   ├── useBookingStore.ts
│   ├── useTasksStore.ts
│   └── useUserStore.ts
├── hooks/              # React Query hooks (useIkeaSearch, useTaskerFilters, etc.)
├── data/               # Static JSON (taskers, locations, services, IKEA products, assertions)
├── lib/                # Utilities
│   └── server/         # Server-only (database.ts, tasker search/filtering)
├── types/              # TypeScript definitions (Tasker, Task, AppState, Service)
└── config/             # Constants, service IDs
```

## Key API Endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET /api/taskers` | Search taskers (serviceId, city, state, rate, elite) |
| `GET /api/taskers/[id]` | Tasker detail |
| `GET /api/taskers/recommended` | Featured taskers (one per service) |
| `GET /api/locations` | Address autocomplete |
| `GET /api/locations/target-cities` | Service areas by state |
| `GET /api/locations/services` | Services at a location |
| `GET /api/ikea/search` | Search IKEA subcategories & products |
| `GET /api/ikea/products` | Paginated IKEA products with filters |
| `GET /api/services/pricing` | Service pricing info |
| `POST /api/v1/get_actual_state` | Run verification assertions |
| `POST /api/v1/run/init` | Initialize a verification run |

## Architectural Patterns

**State persistence**: Zustand stores serialize to localStorage with custom keys (`current-booking-state`, `tasks-storage`, `user-storage`). The verification system reads this state to assert correctness.

**Data layer**: No traditional database for domain data. Taskers (~9,300) are split across 4 chunked JSON files loaded at server startup. IKEA catalog is a 10.7MB JSON file. SQLite is only used for run-based KV storage during verification.

**Booking flow**: Multi-step form persisted in `useBookingStore`. Each step is a nested route under `/book/[taskId]/`. On confirmation, booking moves from store to `useTasksStore` as a completed task.

**Verification**: Assertions defined in `assertions.json` use JSONPath queries + operators (STRING_MATCH, ARRAY_LENGTH, etc.) against localStorage dumps. The `@turing-rlgym/cua-gym-utils` package provides the assertion engine.

## Data Pipeline

See [[2026-02-25-data-schema]] for schema details.

- `npm run generate-data` — Generate seed taskers/locations with faker.js
- `npm run apply-service-ids` — Map booking IDs to service IDs
- `npm run bootstrap-ikea-data` — Initialize IKEA product catalog
- `npm run build-all-taskers` — Rebuild chunked tasker files

## Development

```bash
npm run dev          # Start dev server on :3000
npm run build        # Production build (standalone)
npm run test         # Run test suite
npm run reset-db     # Reset SQLite database
```

## Deployment

Docker multi-stage build → standalone Next.js output → Node 20-slim runtime.
- Port: 3000
- Healthcheck: `GET /api/health`
- Secret: `GITHUB_TOKEN` for private `@turing-rlgym` packages

## Related Notes

- [[2026-02-12-features]] — Full feature list
- [[2026-02-12-feature-summarised]] — Feature summary
- [[2026-02-25-data-schema]] — Data schema documentation
- [[2026-02-12-service-category]] — Service categories
- [[2026-02-12-radix-migration]] — Radix UI migration
- [[2026-02-12-current-state]] — Project state snapshot
- [[About]] — About page content
