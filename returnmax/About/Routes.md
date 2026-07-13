---
tags:
  - returnmax
  - routes
---

# Routes

Next.js App Router. All routes under `/app/`. Root `/` redirects to `/index/tto`.

## Top-Level Pages

| Path | Purpose |
|------|---------|
| `/signin` | Landing/sign-in with `HomeRouter` |
| `/onboarding` | First-time user flow (situations, verify, prior year) |
| `/personal-taxes` | Pricing tiers page |
| `/myreturnmax` | User dashboard (tax years, filed returns) |
| `/index/tto` | **TTO Tax Home** — main stepper dashboard |
| `/index/vault` | Document vault |

## Dev/Test Pages

| Path | Purpose |
|------|---------|
| `/scope` | RL Gym scope config |
| `/verify-ls`, `/verify_raw` | localStorage verification |
| `/state-diff`, `/state-diff-2` | State diff viewer (defaultState vs actual) |
| `/localStorage` | Auto-downloads localStorage as JSON |

## API Routes (`/api/v1/`)

| Route | Purpose |
|-------|---------|
| `health` | Health check |
| `scope` | Scope info |
| `get_actual_state` | Returns runtime state |
| `get_expected_state` | Returns expected/spec state |
| `env/reset`, `env/defaultState`, `env/spec`, `env/state`, `env/schema` | RL Gym env management |

## TTO Tax Flow (7 Sections)

The stepper sequence at `/index/tto`:
```
1. Personal Info   → /index/tto/my-info
2. Wages & Income  → /index/tto/income
3. Deductions & Credits → /index/tto/deductions
4. Other Tax Situations → /index/tto/other
5. State Taxes     → /index/tto/state
6. Expert Review   → /index/tto/expert-review
7. Finish & File   → /index/tto/file
```

### Section 1: My Info (`/index/tto/my-info/`)

Flow: my-info → (military?) → household → contact → ssn → filing-status → spouse? → dependents

Key sub-pages: `profile`, `household`, `spouse`, `dependents`, `contact`, `ssn`, `filing-status`, `filing-status-optimizer`, `military`

### Section 2: Income (`/index/tto/income/`)

Dashboard with 15+ form types: `w2`, `1099-int`, `1099-div`, `1099-b`, `1099-nec`, `1099-misc`, `1099-k`, `1099-r`, `1099-g`, `ssa-1099`, `crypto`, `k-1`, `rental`, `self-employment`, `other`, `other-investments`

### Section 3: Deductions & Credits (`/index/tto/deductions/`)

~40 sub-pages covering: `standard-deduction`, `mortgage`, `charitable`, `medical`, `ira`, `hsa`, `student-loan`, `child-tax-credit`, `earned-income-credit`, `child-care`, `education-credits`, `energy`, `clean-vehicle-credit`, `ev-credit`, `sales-tax`, `property-taxes`, `estimated-tax-payments/*`, `salt`, `educator`, `adoption`, `retirement-savings`, `foreign-tax-credit`, `other-credits`, and more.

### Section 4: Other Tax Situations (`/index/tto/other/`)

~20 sub-pages: `amt`, `ip-pin`, `underpayment`, `identity-theft/*`, `apply-refund`, `amend-return`, `file-extension`, `injured-spouse`, `change-of-address`, `foreign-financial-assets`, `household-employment`, `w4-estimated-taxes`, etc.

### Section 5: State Taxes (`/index/tto/state/`)

`returns`, `income`, `adjustments`, `credits`, `complete`, `review`, `multi-state`

### Section 6: Expert Review (`/index/tto/expert-review/`)

`consent`, `check-summary`, `issues`, `tackle-issues`, `connect-expert`, `max`, `max/confirmed`

### Section 7: Finish & File (`/index/tto/file/`)

Flow: file → order-summary → pay → service-code → tax-payment → refund-split → refund-advance → ready-to-file → verify → submit → confirmation

Also: `federal-review/*` (8 sub-pages), `final-review/*` (9 sub-pages including fix-hub), `tools/*` (print-center, tax-summary, delete-form, my-fees)

## URL Parameter Conventions

- `?revisit=true` — re-entering a completed flow (used by `useRevisit` hook)
- `?edit=true` — editing existing data (shows validation state)
- `?returnTo=<path>` — where to redirect after completing the flow

## Route Patterns

- Each route folder has `page.tsx` (server component that renders the client component)
- Client logic in `_client.tsx` (convention: underscore-prefixed = client component)
- Shared UI/utils in sibling files (e.g., `deduction-flow-ui.tsx`, `fix-routes.ts`)
