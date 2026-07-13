---
tags:
  - returnmax
  - conventions
---

# Conventions

Patterns, naming, and gotchas for this codebase.

## File Naming

- `_client.tsx` — client component (Next.js App Router convention). Every route folder has one.
- `page.tsx` — server component entry point, renders the `_client.tsx` component
- `layout.tsx` — layout wrapper for a route segment
- `route.ts` — API route handler
- `*-types.ts` — type definitions
- `*-store.ts` — Zustand store
- `*-wizard.ts` — multi-step form wizard flow definitions
- `*-flow.ts` — flow configuration/orchestration
- `use-*.ts` — custom React hook

## ID Generation

- **Deterministic:** `generateRecordId(entityType, ...parts)` — FNV-1a hash → UUID v5. Used for entities that must produce the same ID across sessions (W-2s, 1099s, dependents, deductions, credits). Seed = entity type string + semantic keys (e.g., employer name + tax year).
- **Random:** `generateId()` — `crypto.randomUUID()` with fallback. Used for runtime-generated records (OTS entries, ad-hoc forms).

## Store Pattern

Every Zustand store follows:
```typescript
create<XStore>()(
  persist(
    (set, get): XStore => ({
      state: initialXState,
      actions: { ... },
    }),
    {
      name: LOCAL_STORAGE_KEYS.X,
      storage: createJSONStorage(() => safeLocalStorage),
      version: N,
      migrate: ...,
      partialize: ...,
      merge: ...,
    },
  ),
);
```

- State + actions combined in one object
- Cross-store access via `useXxxStore.getState()` (never import hooks into stores)
- `safeLocalStorage` wraps `localStorage` with SSR guard + custom event dispatch
- `partialize` strips transient fields (toasts, isOpen, thumbnailDataUrl)
- `merge` resets transient fields on rehydrate

## Component Patterns

- One `_client.tsx` per route, default-exported
- Read state: `const x = useXxxStore((s) => s.state.field)`
- Read actions: `const { action } = useXxxStore((s) => s.actions)`
- URL params via `useSearchParams()` and `useRevisit()` hook
- Forms use react-hook-form with zod validation
- Navigation: `useRouter().push()` from `next/navigation`

## Route Conventions

- `?revisit=true` — re-entering a completed flow
- `?edit=true` — editing existing data
- `?returnTo=<path>` — redirect after completion

## Tax Return Store Specifics

- All mutations go through `useTaxReturnStore.actions`
- `partialize` hook normalizes deductions via `normalizeDeductions()`
- `merge` hook normalizes credits via `normalizeCredits()`
- 43 migration versions handle backward compatibility
- Deductions `method` field determines standard vs itemized path through `computeFilingSummary()`

## Styling

- Tailwind CSS v4 with custom `tt-blue` color
- `cn()` from `lib/utils.ts` merges Tailwind classes (clsx + tailwind-merge)
- Design tokens: TT Blue `#1972d2`, text `#2b3135`, muted `#6b6c72`, border `#e8ecef`

## RL Gym Integration

- `@turing-rlgym/cua-gym-utils` patches `localStorage.setItem` for state sync
- API routes at `/api/v1/*` serve as gym endpoints
- `StoreInitializer` calls `startEnvStateSync()` on mount
- `ResetListener` handles gym reset events via `reset-channel.ts`

## Key Gotchas

1. **Don't import store hooks into other stores** — use `getState()` for cross-store access
2. **TaxReturnState is deeply nested** — always spread when updating nested objects
3. **Deterministic IDs matter** — the gym diffs state and expects stable IDs
4. **Per-user isolation** — auth store saves/restores per-user data on login/logout
5. **defaultState.json** — must be regenerated if initial state shape changes
6. **Migration versions** — never decrement; always add new version with migrate function
7. **toasts are session-only** — `partialize` strips them, `merge` resets to `[]`
