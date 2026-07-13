---
tags:
  - returnmax
  - architecture
---

# Architecture

## Project Summary

**ReturnMax** is a private Next.js 16 application serving as the frontend for the Turing RL Gym — an environment for training AI models on tax-filing workflows. It is entirely client-side: no real database, all state in localStorage via Zustand stores. Styled with Tailwind CSS v4, forms via react-hook-form + zod.

**Stack:** Next.js 16 (Turbopack), TypeScript strict, Zustand 5, React 19, pnpm, Docker standalone output.

## Data Flow

```
User Action → React Component → Zustand Store Action → State Mutation → Persist Middleware → localStorage
                                                                                              ↓
                                                                              storage-events (custom event)
                                                                                              ↓
                                                                              RL Gym backend sync
                                                                        (@turing-rlgym/cua-gym-utils)
```

1. Components read/write state exclusively through Zustand hooks (`useXxxStore`)
2. Every store uses `persist` middleware with `safeLocalStorage` (SSR-safe wrapper)
3. `StoreInitializer` (mounted in root layout) seeds `defaultState.json` on first load and starts RL Gym env state sync
4. Custom `storage-events` dispatch on every `localStorage.setItem` for cross-tab sync

## Store Dependency Graph

```
useAuthStore
  ├── useUsersStore         (CRUD, authentication)
  ├── useTaxReturnStore     (profile sync, per-user reset)
  ├── useDocumentsStore     (per-user reset)
  ├── useLinkedAccountsStore(per-user reset)
  ├── useFiledReturnsStore  (per-user reset)
  └── useOrderReceiptStore  (per-user reset)

useTaxReturnStore
  ├── useFiledReturnsStore  (save on e-file submit)
  └── useOrderReceiptStore  (save on payment confirm)

useTaxUIStore ←→ useNotificationsStore  (mutual panel close)

StoreInitializer → all 13 stores (seeds defaultState.json on first load)
```

## Per-User Data Isolation

`useAuthStore.login()` / `logout()` implements a multi-user save/restore system:
- On logout: current user's per-user store data saved under `{key}_user_{userId}`
- On login: new user's data restored from their per-user key
- Non-onboarded users always get a blank slate via `resetPerUserStores()`

Per-user stores: `rm_tax_return`, `rm_documents`, `rm_linked_accounts`, `rm_filed_returns`, `rm_order_receipt`.

## localStorage Keys

| Key | Store |
|-----|-------|
| `auth` | useAuthStore |
| `users` | useUsersStore |
| `uiModal` | useModalStore |
| `rm_tax_return` | useTaxReturnStore |
| `rm_tax_ui` | useTaxUIStore |
| `rm_settings` | useSettingsStore |
| `rm_documents` | useDocumentsStore |
| `rm_linked_accounts` | useLinkedAccountsStore |
| `rm_filed_returns` | useFiledReturnsStore |
| `rm_notifications` | useNotificationsStore |
| `rm_order_receipt` | useOrderReceiptStore |
| `rm_articles` | useArticlesStore |
| `rm_search` | useSearchStore |

Session storage: `rm_tto_sections_visited`

## RL Gym Integration

- API routes at `/api/v1/*` (health, scope, env state, state resolvers)
- `@turing-rlgym/cua-gym-utils` monkey-patches `localStorage.setItem` to debounce-push full state snapshots
- State schema validated against `lib/contracts/state-schema.json`
- Deterministic IDs (FNV-1a hash → UUID) ensure reproducible state across sessions

## Migration Strategy

`useTaxReturnStore` has 43 persist migrations (v3→v43). Every schema change adds a migration entry. The `partialize`/`merge` hooks normalize deductions and credits on every read/write. The codebase strongly favors backward compatibility for persisted user data.
