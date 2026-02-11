# MUI to Radix UI Migration — Feasibility Assessment

**Verdict: Feasible, but significant effort.**

## Current State

- **75 files** with MUI imports
- **203 `styled()` instances** using MUI/Emotion
- **4 MUI packages** + 2 Emotion packages (`@mui/material`, `@mui/icons-material`, `@emotion/react`, `@emotion/styled`)
- Custom theme with `createTheme()`, design tokens, and component overrides
- Radix UI is **not installed** yet — greenfield migration
- Tailwind CSS is installed but minimally used

## Most Used MUI Components

| Component | Uses | Radix Equivalent |
|-----------|------|-------------------|
| `Box` | 57 | Plain `div` + Tailwind |
| `Typography` | 52 | Custom component + Tailwind |
| `Grid`/`Container` | 20 | CSS Grid/Flexbox + Tailwind |
| `TextField`/`Select`/`FormControl` | ~15 | `@radix-ui/react-select`, custom input |
| `IconButton`/`Button` | 12 | Custom button component |
| `Avatar` | 6 | `@radix-ui/react-avatar` |
| `TextField` | 5 | Custom input component |
| `Dialog` | 4 | `@radix-ui/react-dialog` |
| `Collapse` | 3 | `@radix-ui/react-collapsible` |
| `CircularProgress` | 3 | Custom spinner + Tailwind |
| `Slider` | 2 | `@radix-ui/react-slider` |
| `Paper` | 2 | `div` + Tailwind |
| `Tabs` | 1 | `@radix-ui/react-tabs` |
| `Checkbox` | 1 | `@radix-ui/react-checkbox` |
| `Autocomplete` | 1 | `@radix-ui/react-popover` + custom |
| `Pagination` | 1 | Custom component |

## MUI Icons (16 unique)

Edit, CheckCircle, Check, Visibility, VisibilityOff, Star, Shield, LocationOn, KeyboardArrowDown, ExpandMore, ExpandLess, CreditCard, Close, AutoAwesome, Error

These need a replacement library (e.g., Lucide, React Icons).

## Theme & Styling

- **Theme file** (`src/theme/index.ts`): Full `createTheme()` config with custom palette, typography, spacing, shadows, and component overrides (`MuiButton`, `MuiCard`, `MuiTextField`, `MuiInputBase`, `MuiChip`).
- **Design tokens file**: Comprehensive color palette, typography scales, spacing, border-radius, shadows, durations, easing functions.
- **26 files** use MUI's `styled()` function (203 total instances).
- Heaviest styled usage: `DateTimeSelectionModal.tsx` (22), `PaymentConfirmationPage.tsx` (20), `BookingSummary.tsx` (17), `FilterSidebar.tsx` (16), `PaymentForm.tsx` (14), `TaskerCard.tsx` (12).

## Key Challenges

1. **`styled()` refactor (203 instances)** — The biggest effort. Convert to Tailwind classes or CSS modules since Radix is unstyled.
2. **Theme system** — MUI's `createTheme()` has no Radix equivalent. Design tokens need to map to CSS variables / Tailwind config.
3. **`Box` and `Typography` (109 uses)** — No direct Radix equivalent; replace with native HTML + Tailwind.
4. **Icons (16 unique)** — Need a separate library (Lucide, React Icons, etc.).
5. **Form components** — MUI's TextField/Select have built-in labels, validation states, and styling that Radix doesn't provide out of the box.

## What Makes It Feasible

- Tailwind is already installed (can replace Emotion/styled)
- Radix covers most interactive components (Dialog, Select, Tabs, Slider, etc.)
- The codebase is moderately sized (~75 files)
- Design tokens are already extracted into a separate file

## Recommendation

The migration is feasible but non-trivial. The main cost isn't swapping components — it's **rebuilding 203 styled instances** and the theme system. Consider using **shadcn/ui** (built on Radix + Tailwind) which would give pre-styled Radix components and significantly reduce the styling effort.

---

## On customization of MUI

A common misconception is that MUI is not very customizable. In reality, MUI provides multiple layers of deep customization:

- **`createTheme()`** — Global theming with full control over palette, typography, spacing, shadows, breakpoints, and per-component default props and style overrides.
- **Component overrides** — Every MUI component can be globally restyled via `theme.components.MuiXxx.styleOverrides`, affecting all instances at once.
- **`sx` prop** — Inline, theme-aware styling on any MUI component without creating a separate styled wrapper.
- **`styled()` API** — Full CSS-in-JS with access to theme tokens, props-based conditional styling, and composition.
- **`slotProps` / `slots`** — Override internal sub-components (e.g., the input element inside TextField) without wrapping or forking.
- **CSS class overrides** — Every component exposes granular CSS classes (`classes` prop) for traditional CSS targeting.

The reason MUI gets a reputation for being hard to customize is not a lack of flexibility — it's the effort required to **override opinionated defaults**. MUI ships with Material Design baked in, so projects with a custom design system often spend significant time undoing those defaults before applying their own. This is the trade-off: MUI gives you a polished look out of the box, but that polish has a cost when your design diverges from Material Design.

Radix UI (and shadcn/ui) take the opposite approach — unstyled primitives with no visual opinions — which means less to override but more to build from scratch.

---

## Slack Summary

*MUI → Radix UI Migration — Feasibility Summary*

We assessed migrating our UI layer from MUI/Emotion to shadcn/ui (Radix + Tailwind).

*Current state:*
• 75 files with MUI imports, 203 `styled()` instances
• 42 reusable UI primitives in `src/components/ui/` — pages reuse these across booking flows
• Custom theme with design tokens, palette, typography, and component overrides
• Tailwind already installed; Radix not yet present

*Component mapping:*
• `Box` (57 uses), `Typography` (52) → native HTML + Tailwind
• `Dialog`, `Select`, `Tabs`, `Slider`, `Checkbox`, `Avatar` → direct Radix equivalents
• `Button`, `Card`, `TextField`, form components → shadcn/ui primitives
• 16 MUI icons → Lucide

*Key challenges:*
• Converting 203 `styled()` instances to Tailwind
• Replacing MUI's `createTheme()` with CSS variables + Tailwind config
• Form components (TextField, Select) lose built-in labels/validation — need to rebuild

*What makes it feasible:*
• Tailwind is already in the project
• Design tokens are already extracted into a separate file
• Pages are compositions of the same core components — migrate the core set and the rest follows
• shadcn/ui provides pre-built Radix + Tailwind components, reducing the build-from-scratch effort

*Verdict:* Feasible. The migration is primarily a styling refactor, not a logic rewrite.

