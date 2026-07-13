---
tags:
  - returnmax
  - data-layer
---

# Data Layer

All in `/data/`. Static data that seeds the application.

## Directory Structure

```
data/
├── state/defaultState.json    # Full initial localStorage state (all 13 keys)
├── initializers/              # TypeScript default state objects per store
├── seed/                      # Per-table JSON files matching RL Gym schema
├── content/                   # Static content definitions
└── articles-metadata.json     # Help article catalog (~180 articles)
```

## defaultState.json

`data/state/defaultState.json` — ~35KB JSON. Contains pre-serialized initial state for all 13 localStorage keys. Each value is JSON-stringified Zustand persisted state (including version number). Read by `StoreInitializer.tsx` to seed first-time visitors.

Keys: `auth`, `users`, `uiModal`, `rm_tax_return`, `rm_tax_ui`, `rm_settings`, `rm_documents`, `rm_linked_accounts`, `rm_articles`, `rm_filed_returns`, `rm_order_receipt`, `rm_notifications`, `rm_search`.

## Initializers (`data/initializers/`)

TypeScript modules that export the default state object for each store. Used by stores as fallback and by `defaultState.json` generation.

| File | Export | Notes |
|------|--------|-------|
| `initialize-tax-return.ts` | `initialTaxReturnState` | Largest initializer. Also exports defaults for dependents, spouse, expert review, filing, onboarding, AMT, underpayment, identity theft, W-4, change of address, IP PIN, state adjustments/credits, and every OTS sub-form |
| `initialize-users.ts` | `initialUsersState` | 4 seed users |
| `initialize-auth.ts` | `initialAuthState` | Maya logged in by default |
| `initialize-tax-ui.ts` | `initialTaxUIState` | currentSection: "my_info", sidebar: "full" |
| `initialize-settings.ts` | `initialSettingsState` | showProgressBar: true |
| `initialize-search.ts` | `initialSearchState` | Empty |
| `initialize-modal.ts` | `initialModalState` | Closed |
| `initialize-articles.ts` | `initialArticlesState` | One entry per article |

## Seed Data (`data/seed/`)

17 JSON files matching the RL Gym database schema. Used by the gym environment for state seeding:
`users.json`, `auth_sessions.json`, `tax_returns.json`, `tax_return_people.json`, `dependents.json`, `documents.json`, `income_items.json`, `asset_sales.json`, `self_employment_businesses.json`, `rental_properties.json`, `deduction_items.json`, `credit_items.json`, `estimated_tax_payments.json`, `state_returns.json`, `payment_elections.json`, `filed_returns.json`, `order_receipts.json`, `order_receipt_line_items.json`, `notifications.json`, `help_articles.json`, `help_article_feedback.json`, `managed_clients.json`, `user_settings.json`, `workflow_state.json`.

## Content (`data/content/`)

Static definitions used by UI components:
- `homepage.ts` — homepage content
- `income-topics.ts` — income section topic definitions
- `file-topics.ts` — filing section topics
- `final-review-topics.ts` — final review checklist
- `federal-review-checklist.ts` — federal review items
- `other-tax-topics.ts` — other tax situations definitions
- `help-articles.ts` + `help-article-content.tsx` — help article data + JSX content
- `investment-types.ts` — investment type catalog
- `linked-account-providers.ts` — mock financial institution configs
- `occupations.ts` — occupation list for autocomplete
- `scope.ts` — app scope configuration
- `tax-tools.ts` — tax tools content

## Helper Files

- `pg/` — debug/playground: state diffs, localStorage snapshots, walkthrough HTML, review checklists
- `rl_gym_schema/` — RL Gym database schema: DBML diagram, per-table JSON schemas, enum definitions
- `verifiers/data/assertions.json` — RL Gym verifier assertion data
