---
tags:
  - returnmax
  - index
  - about
---

# ReturnMax — Codebase Map

Private Next.js 16 app. Frontend for the Turing RL Gym — trains AI models on tax-filing workflows. Entirely client-side: no database, all state in localStorage via 13 Zustand stores. React 19, TypeScript strict, Tailwind CSS v4, pnpm.

**Repo:** `turing-rlgym/returnmax` | **Branch:** `ots_dev` | **Entry:** `http://localhost:3000` → `/index/tto`

## Quick Reference

| Need | Go To |
|------|-------|
| Understand data flow & store relationships | [[returnmax/About/Architecture]] |
| Find a Zustand store (state + actions) | [[returnmax/About/Stores]] |
| Find a type/interface/enum | [[returnmax/About/Types]] |
| Find a route or page component | [[returnmax/About/Routes]] |
| Understand tax calculations or find a lib utility | [[returnmax/About/Tax-Engine]] |
| Find a React component | [[returnmax/About/Components]] |
| Find seed data, initializers, or default state | [[returnmax/About/Data-Layer]] |
| Learn code patterns, naming, gotchas | [[returnmax/About/Conventions]] |

## File Map

```
returnmax/
├── app/                    # Next.js App Router: pages, layouts, API routes
│   ├── api/v1/             # RL Gym integration endpoints
│   ├── index/tto/          # Main tax filing flow (7 sections, ~160 routes)
│   ├── unified/[year]/     # Modern UI variant (sidebar nav, onboarding, vault)
│   ├── onboarding/         # First-time user flow
│   ├── signin/             # Landing/sign-in
│   └── state-diff*/        # Dev tools (state diff, localStorage inspect)
├── components/             # React components (app shell, unified, UI primitives)
│   ├── app/                # StoreInitializer, layouts, panels, toasts
│   ├── unified/            # UnifiedAppShell, sidebar, onboarding, vault
│   ├── ui/                 # Button, Modal, Accordion, CurrencyInput, etc.
│   ├── landing/            # Marketing/sign-in page
│   └── review/             # Review screen components
├── store/                  # 13 Zustand stores (all persisted to localStorage)
├── types/                  # TypeScript type definitions (tax-return-types.ts is ~2882 lines)
├── lib/                    # ~95 files: tax engine, wizards, normalizers, utilities
├── hooks/                  # 6 custom React hooks
├── data/                   # Static data: defaultState.json, initializers, seed JSON, content
├── rl_gym_schema/          # RL Gym database schema (JSON + DBML)
├── verifiers/              # Gym verifier assertion data
├── pg/                     # Debug/playground files
└── scripts/                # Utility scripts
```

## Architecture at a Glance

```
User Action → Component → Zustand Store Action → State Mutation → Persist → localStorage
                                                                         ↓
                                                               RL Gym sync
                                                         (cua-gym-utils patches
                                                          localStorage.setItem)
```

- **Single aggregate root:** `TaxReturnState` — one deeply nested object holding all tax data
- **13 Zustand stores** — all client-side, persisted to localStorage, deterministic IDs
- **Per-user isolation:** auth store saves/restores store data per user on login/logout
- **43 migrations:** backward-compatible schema evolution in `useTaxReturnStore`
- **`StoreInitializer`** seeds `defaultState.json` on first load, starts gym state sync

## Key Numbers

| Metric | Count |
|--------|-------|
| Zustand stores | 13 |
| Route pages | ~160 |
| Lib files | ~95 |
| TypeScript types file (largest) | ~2,882 lines |
| Tax return store (largest) | ~4,500 lines |
| Store migrations | 43 versions |
| Income form types | 15+ |
| Deduction categories | 25+ |
| Credit types | 20+ |
| OTS form buckets | 17+ |
| Seed users | 4 |
| Help articles | ~180 |

## Top 5 Files to Read First

1. `store/tax-return-store.ts` — the central store; understand the state shape
2. `types/tax-return-types.ts` — all domain types
3. `lib/tax-engine.ts` — core tax calculation
4. `components/StoreInitializer.tsx` — gym integration and state seeding
5. `app/index/tto/_client.tsx` — main TTO stepper and section ordering

## Project Rules

See `CLAUDE.md` in the repo root. 17 rules covering: think before coding, simplicity first, surgical changes, micro commits, React skill for web work, Obsidian vault context, no co-author on commits, live testing via Chrome CDP with `turing` profile.
