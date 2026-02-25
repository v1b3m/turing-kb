# NowService - IT Asset Management Admin Console

## Overview

A ServiceNow ITAM admin console clone built with modern web technologies. It replicates the look, feel, and interaction patterns of ServiceNow's ITAM dashboard as a frontend interface.

**Package name:** `nowservice-itam` | **Version:** 1.0.0 | **Private:** yes

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14.2.33 (App Router) |
| Language | TypeScript 5 |
| Styling | Tailwind CSS 3.3.0 |
| UI Components | Shadcn UI (New York style) + Radix UI primitives |
| Icons | Lucide React |
| State Management | Zustand 5 (installed, not heavily used yet) |
| Data Fetching | TanStack React Query 5 (installed, not active yet) |
| Testing | Playwright 1.58.2 |
| Mock Data | Faker.js |
| Git Hooks | Husky + lint-staged (ESLint on pre-commit) |

**Fonts:** Inter (default), Lato, Open Sans

---

## Project Structure

```
src/
├── app/                         # Next.js App Router pages
│   ├── page.tsx                 # Home page (hero + "Go Further" cards)
│   ├── layout.tsx               # Root layout with Header
│   ├── globals.css              # Global styles + Tailwind layers
│   ├── [...slug]/page.tsx       # Catch-all → "Coming Soon" pages
│   └── coming-soon/
├── components/
│   ├── ui/                      # Shadcn UI components (button, dropdown-menu)
│   ├── layout/                  # Layout components
│   │   ├── Header.tsx           # Fixed header with full navigation
│   │   ├── AllMenuDropdown.tsx  # Hierarchical tree menu with filter
│   │   ├── ListMenuDropdown.tsx # Generic list menu (Favorites, History, etc.)
│   │   ├── ProfileDropdown.tsx  # User profile menu
│   │   └── SearchWithScopeDropdown.tsx  # Search bar with scope selector
│   └── icons/                   # Custom SVG icons (HeroBanner, CardIllustration)
├── lib/
│   └── utils.ts                 # cn() helper (clsx + tailwind-merge)
└── page-contents/               # Static JSON content
    ├── home-page.json           # Hero section + "Go Further" cards
    └── menus/
        ├── all.json             # Hierarchical tree menu data
        ├── favorites.json
        ├── history.json
        ├── workspaces.json
        └── admin.json
```

**Config files:** `next.config.js` (standalone output, remote images from `dev218807.service-now.com`), `tailwind.config.js`, `tsconfig.json`, `components.json` (Shadcn), `.eslintrc.json`

---

## Architecture

### Layout Hierarchy

```
RootLayout (layout.tsx)
├── Header (fixed, z-50, height: 52px, dark navy)
│   ├── Logo + Navigation
│   ├── AllMenuDropdown (tree menu with filter/search)
│   ├── ListMenuDropdown x4 (Favorites, History, Workspaces, Admin)
│   ├── SearchWithScopeDropdown (Global, CSM/FSM, Service Ops, Knowledge)
│   ├── Utility icons (Globe, Messages, Menu, Help, Notifications)
│   └── ProfileDropdown (profile, preferences, impersonate, logout)
├── Spacer (h-nowservice-header)
└── Page Content
```

### Routing

- `/` - Home page with hero banner + 6 feature cards
- `/*` - Catch-all route displays "Coming Soon" with dynamic title from slug
- Menu items generate slugs from labels (e.g. "Activity Groups" -> `/activity-groups`)

### Key Components

| Component | Description |
|-----------|-------------|
| **AllMenuDropdown** | Recursive tree menu from `all.json`. Supports expand/collapse, regex filtering, hover actions (edit/star), slug-based navigation |
| **ListMenuDropdown** | Renders flat lists from JSON. Supports icons, sub-labels, timestamps, filtering |
| **SearchWithScopeDropdown** | Search input with dropdown scope selector (4 scopes) |
| **ProfileDropdown** | User profile section, preferences links, admin actions, logout |

### Home Page Sections

1. **Hero** - Title ("NowService Studio"), subtitle, description, CTA button, hero SVG illustration. Background: `#002239`
2. **Go Further** - 6 cards in 2-column responsive grid. Cards: IDE, Creator Studio, App Engine, Workflow, UI Builder, Mobile App Builder. Each with gradient background, illustration, and link button.

---

## Theme System

Three-layer theming:

1. **Tailwind base colors** - Semantic (primary, secondary, success, error, info, warning) + generic scales
2. **NowService tokens** (`nowservice-*` namespace in tailwind.config.js):
   - Brand: `#62d84e` (green), `#76F59A` (star)
   - Header: `#022d42`, `#0d3849` (dark navy)
   - Hero: `#002239`
   - Custom sizing: header 52px, hero 430px, logo 138px, card 648px, content max 1336px
3. **CSS variables** in `globals.css` - `--background`, `--foreground`, etc. with dark mode support

---

## Data Structures

**Menu tree (all.json):**
```json
{
  "items": [
    {
      "label": "Section Name",
      "id": "section-id",
      "hasActions": true,
      "defaultExpanded": true,
      "children": [
        { "label": "Leaf Item", "highlight": true }
      ]
    }
  ]
}
```

**List menus (favorites/history/workspaces/admin .json):**
```json
{
  "items": [
    "Simple string item",
    { "label": "Label", "subLabel": "...", "timestamp": "2 hrs ago", "icon": "calendar" }
  ]
}
```

---

## Development

```bash
npm run dev      # Dev server at localhost:3000
npm run build    # Production build
npm run start    # Production server
npm run lint     # ESLint
npm run test     # Playwright tests (tsx tests/run.ts)
```

Pre-commit: Husky triggers lint-staged which runs `eslint --fix` on staged `.js/.jsx/.ts/.tsx` files.

---

## Current State

**Implemented:**
- Static home page with hero + feature cards
- Full navigation system with tree/list dropdowns
- Search with scope selection
- User profile menu
- Catch-all "Coming Soon" pages
- Responsive design, accessible (Radix primitives)

**Ready for extension:**
- Database integration (TanStack Query installed)
- State management (Zustand installed)
- Additional Shadcn components (dialog, tabs, select, etc. via Radix)
- Real API connections, auth, admin dashboard pages, ITAM interfaces

---

## Documentation

- **README.md** - Quick start, stack overview, commands
- **AGENTS.md** - Guide for AI agents: Shadcn usage, key paths, data structures, theme tokens, component patterns, best practices for extending
