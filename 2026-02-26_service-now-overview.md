# NowService — Project Overview

## What Is It

A pixel-accurate clone of the **ServiceNow IT Asset Management (ITAM) admin console**, built as a frontend-only Next.js app. The goal is to replicate the look, feel, and navigation of the real ServiceNow UI.

## Tech Stack

| Layer        | Technology                                              |
| ------------ | ------------------------------------------------------- |
| Framework    | Next.js 14 (App Router)                                 |
| Language     | TypeScript                                              |
| Styling      | Tailwind CSS 3 + custom `nowservice` design tokens    |
| UI Primitives| Shadcn UI (Radix under the hood) — `src/components/ui/`|
| State        | Zustand (installed, not yet heavily used)               |
| Data Fetching| TanStack React Query (installed, not yet heavily used)  |
| Icons        | Lucide React + custom SVGs                              |
| Fonts        | Inter (body), Lato (header), Open Sans (hero/cards)     |
| Testing      | Playwright + custom runner (`tsx tests/run.ts`)       |
| Linting      | ESLint + Husky pre-commit (lint-staged)                 |
| Package Mgr  | npm                                                     |

## Repo Layout

```
src/
├── app/
│   ├── layout.tsx            # Root layout — fonts, Header, body wrapper
│   ├── page.tsx              # Home page — hero banner + "Go Further" cards
│   ├── coming-soon/page.tsx  # Static coming-soon page
│   └── [...slug]/page.tsx    # Catch-all — renders "Coming Soon" for unknown routes
├── components/
│   ├── icons/                # Custom SVG components (hero banner, card illustration, open-link)
│   ├── layout/               # App chrome
│   │   ├── Header.tsx                 # Fixed top bar — logo, nav menus, search, icons, profile
│   │   ├── AllMenuDropdown.tsx        # "All" mega-menu — tree view with filter from all.json
│   │   ├── ListMenuDropdown.tsx       # Generic list dropdown (Favorites, History, Workspaces, Admin)
│   │   ├── ProfileDropdown.tsx        # User avatar + profile menu
│   │   └── SearchWithScopeDropdown.tsx# Search bar with scope selector
│   └── ui/                   # Shadcn primitives (button, dropdown-menu)
├── lib/
│   └── utils.ts              # cn() helper (clsx + tailwind-merge)
└── page-contents/
    ├── home-page.json        # Hero + Go Further cards content
    └── menus/                # JSON data for each header menu
        ├── all.json          # Hierarchical tree (items[].children[])
        ├── favorites.json
        ├── history.json
        ├── workspaces.json
        └── admin.json
```

## Key Architectural Decisions

1. **Content-driven via JSON** — Page copy and menu data live in `src/page-contents/`. Components import JSON directly; no API layer yet.
2. **Shadcn-first UI** — New UI elements should be installed via `npx shadcn@latest add <component>` before hand-rolling. Existing: button, dropdown-menu.
3. **Custom design tokens** — All ServiceNow-specific colors, sizes, and spacing are defined as `nowservice-*` tokens in `tailwind.config.js`. Components use tokens, never raw hex.
4. **Catch-all routing** — `[...slug]/page.tsx` handles any route not explicitly defined, showing a Coming Soon page. Menu leaf labels are slugified to produce link targets.

## Design Token Cheat-Sheet

### Colors (`nowservice-*`)
- **Brand:** `green`, `green-dark`, `green-light`, `star`
- **Header:** `header`, `header-bar`, `nav-hover`, `nav-border`, `pill`, `search-bg`, `search-dropdown-bg`
- **Hero:** `hero`, `hero-title`, `hero-subtitle`, `hero-desc`
- **Body/Cards:** `body`, `section-label`, `section-title`, `card-desc`, `selected-bg`
- **Navy palette:** `navy`, `navy-dark`, `navy-light`

### Sizing
- Header height: `h-nowservice-header` (52px)
- Hero height: `h-nowservice-hero` (430px)
- Logo width: `w-nowservice-logo` (138px)
- Card width: `w-nowservice-card` (648px)
- Content max-width: `max-w-nowservice-content` (1336px)

## Current State (Feb 2026)

### Implemented
- Fixed header with full navigation (All mega-menu, Favorites, History, Workspaces, Admin)
- Search with scope dropdown
- Profile dropdown
- Home page: hero banner with CTA + 4 "Go Further" feature cards
- Catch-all slug routing with Coming Soon fallback
- Custom theme matching ServiceNow's color palette

### Not Yet Implemented
- Actual ITAM pages (asset lists, asset detail, dashboards)
- Backend / API integration
- Authentication
- Zustand stores (dependency installed but unused)
- React Query data fetching (dependency installed but unused)

## How to Run

```bash
npm install
npm run dev          # http://localhost:3000
npm run build        # Production build
npm run test         # Playwright tests
npm run lint         # ESLint
```

## Agent Reference

See [[AGENTS]] (`AGENTS.md` in repo root) for component-level details and conventions when adding new UI.