# PR Title

refactor: migrate from MUI/Emotion to Tailwind CSS + Radix UI primitives

---

# PR Description

## Summary

Complete migration of the UI layer from Material UI (MUI) + Emotion to Tailwind CSS, Class Variance Authority (CVA), Lucide icons, and Radix UI primitives across 84 files — a net reduction of ~7,400 lines of code.

## Motivation

### Why migrate away from MUI?

An earlier feasibility assessment identified that while MUI is highly customizable, the project was paying a significant cost for that flexibility:

- **Runtime overhead**: MUI + Emotion generate styles at runtime via CSS-in-JS. Every component render involves style computation, serialization, and injection. Tailwind compiles to static CSS at build time with zero runtime JavaScript for styling.
- **Bundle bloat**: `@mui/material`, `@mui/icons-material`, `@emotion/react`, and `@emotion/styled` together added substantial weight to the client bundle. The migration removes all four dependencies.
- **Fighting defaults**: MUI ships with Material Design opinions baked in. Our custom design system required significant effort to override those defaults — 203 `styled()` instances across the codebase, many of which existed solely to undo MUI's visual opinions before applying our own. Radix UI primitives are unstyled by design, so there are no defaults to fight.
- **Styling fragmentation**: The codebase had three co-existing styling approaches — MUI `sx` props, `styled()` wrappers, and Tailwind utility classes. This migration unifies everything under Tailwind, eliminating cognitive overhead.
- **Developer experience**: Tailwind classes are co-located with markup. No more jumping between styled component definitions and JSX, or reasoning about theme token resolution through MUI's theming layer.

### Why was the migration feasible?

- Tailwind was already installed in the project (used in 4 files)
- Design tokens were already extracted into a standalone file, making the Tailwind config mapping straightforward
- Pages are compositions of ~42 shared UI primitives — migrating the core component set cascaded to all pages
- shadcn/ui patterns (Radix + Tailwind + CVA) provided proven component recipes, reducing build-from-scratch effort
- The migration is purely a styling refactor — no business logic changes

## What changed

### Phase 0 — Foundation
- Extended `tailwind.config.js` with full design token color scales, shadows, border-radii, transitions
- Added CSS custom properties to `globals.css` for theming
- Created `src/lib/utils.ts` with `cn()` utility (clsx + tailwind-merge)

### Phase 1 — Core components (Button, Card, Input, Modal)
- `Button` — CVA-based with variant/size props, loading state via Lucide `Loader2`
- `Card` — div + CVA with elevated/outlined/filled variants
- `Input` — Native `<input>`/`<textarea>` with Tailwind focus/error/disabled states
- `Modal` — Radix `Dialog` primitive with size variants via `max-w-*`

### Phase 2 — Form components
- `RadioButton` — Radix `RadioGroup`
- `Checkbox` — Radix `Checkbox` + Lucide `Check` indicator
- `TimeSelector` — Custom popover replacing native `<select>`
- `LocationSelector` — Combobox with Radix Popover
- `FilterSidebar` — Custom popovers, dual-thumb range sliders

### Phase 3 — Layout primitive replacement
- `Box` (57 uses) → `<div>` + Tailwind
- `Typography` (52 uses) → semantic HTML (`<h1>`–`<h6>`, `<p>`, `<span>`) + Tailwind
- `Grid`/`Container` (20 uses) → CSS grid/flexbox + Tailwind
- `Divider` → `<hr>` + Tailwind
- Converted all 203 `styled()` instances to Tailwind-classed elements

### Phase 4 — Remaining components + icons
- Alert, Loading, Chip, Tabs, Avatar, Calendar, Badge, Rating, and all composite components
- All 16 `@mui/icons-material` imports replaced with Lucide equivalents (`Close`→`X`, `Star`→`Star`, `Edit`→`Pencil`, `CheckCircle`→`CheckCircle`, `Visibility`→`Eye`, `VisibilityOff`→`EyeOff`, `Shield`→`Shield`, `LocationOn`→`MapPin`, `KeyboardArrowDown`→`ChevronDown`, `ExpandMore`→`ChevronDown`, `ExpandLess`→`ChevronUp`, `CreditCard`→`CreditCard`, `AutoAwesome`→`Sparkles`, `Error`→`AlertCircle`)

### Phase 5 — Full MUI removal
- Removed MUI ThemeProvider from app layout
- Deleted `src/theme/index.ts` (MUI `createTheme()` config)
- Uninstalled `@mui/material`, `@mui/icons-material`, `@emotion/react`, `@emotion/styled`
- Zero `@mui` or `@emotion` imports remain

### UI refinements
- Replaced all native `<select>` elements with custom popover dropdowns (sort, time, country selectors)
- Implemented dual-thumb range sliders for price filters across all filter sidebars
- Matched original site colors (`#00b894` for category chips/search focus, `rgb(13,122,95)` for actions)
- Consistent popover behavior: hover (`bg-gray-100`), active item (`bg-[#e6f5f0]`), click-outside dismiss
- Fixed button text centering, footer padding, font sizes on mytasks page

## Impact

| Metric | Before | After |
|--------|--------|-------|
| Styling dependencies | 4 (MUI + Emotion) | 0 runtime CSS-in-JS |
| `styled()` instances | 203 | 0 |
| Lines changed | — | +5,056 / -12,441 (net -7,385) |
| Files touched | — | 84 |
| Icon library | `@mui/icons-material` | `lucide-react` |

## Dependencies removed

```
- @mui/material
- @mui/icons-material
- @emotion/react
- @emotion/styled
```

## Dependencies added/retained

```
+ lucide-react
+ class-variance-authority
+ clsx
+ tailwind-merge
+ @radix-ui/react-dialog
+ @radix-ui/react-checkbox
+ @radix-ui/react-radio-group
```

## Test plan

- [ ] `npm run build` passes with zero errors
- [ ] All pages render correctly: booktask, electrical-work, heavy-lifting, tv-mounting, plumbing-help (details → recommendations → schedule → confirm-details), yard-work (all steps), trash-removal (all steps), mytasks, account, getdiscount, design-system
- [ ] Interactive elements work: popover dropdowns, dual-thumb range sliders, calendar date selection, time selection, checkbox/radio inputs, modals
- [ ] Responsive layouts verified at mobile, tablet, and desktop breakpoints
- [ ] `grep -r "@mui\|@emotion" src/` returns zero results
- [ ] Visual comparison against original site confirms parity
