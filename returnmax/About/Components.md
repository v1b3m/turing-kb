---
tags:
  - returnmax
  - components
---

# Components

All in `/components/`. Organized by domain: app shell, landing, unified, UI primitives, review, notifications, search.

## App Shell (`components/app/`)

| Component | Purpose |
|-----------|---------|
| `ClientLayout.tsx` | Top-level client wrapper (providers, context) |
| `StoreInitializer.tsx` | Seeds `defaultState.json` into empty stores on first load, starts RL Gym env state sync |
| `IndexAppLayout.tsx` | Layout wrapper for `/index/*` routes |
| `AppHeader.tsx` | Main app header |
| `AppFooter.tsx` | Main app footer |
| `AppContentArea.tsx` | Content area wrapper |
| `HelpPanel.tsx` | Help article slide-out panel |
| `ExpertHelpPanel.tsx` | Expert chat/AI help panel |
| `ToastTray.tsx` | Toast notification container |
| `ProgressBar.tsx` | Filing progress indicator |
| `RouteStatusSync.tsx` | Syncs current route to `useTaxUIStore` section/page |
| `ResetListener.tsx` | Listens for gym reset events |
| `PageTransition.tsx` | Page transition animation wrapper |

## Landing Page (`components/landing/`)

Marketing/sign-in page components: `HomeRouter.tsx`, `SignInCard.tsx`, `Header.tsx`, `Footer.tsx`, `Content.tsx`, `TopBanner.tsx`, `ReviewsCarousel.tsx`, `PageWrapper.tsx`, `LegalDisclosures.tsx`.

## Unified Experience (`components/unified/`)

Modern UI variant used by `/unified/[year]/index/*` routes:
- `UnifiedAppShell.tsx` — main app shell (sidebar, header, footer, content)
- `UnifiedSidebar.tsx` — sidebar navigation with section progress rings
- `UnifiedHeader.tsx`, `UnifiedFooter.tsx`
- `UnifiedPageContainer.tsx` — page wrapper
- `SectionProgressRing.tsx` — circular progress indicator per section
- `SidebarFlyout.tsx` — flyout menu
- `PersonalInfoSpot.tsx` — personal info summary badge
- `SavedBadgeIcon.tsx` — "saved" status indicator

### Onboarding (`components/unified/onboarding/`)
`UnifiedOnboardingPage.tsx` + step components: situations, prior year filing, verification, more info.

### Document Vault (`components/unified/vault/`)
Full document management UI: `VaultPageContent.tsx`, `UploadDropzone.tsx`, `DocumentsList.tsx`, `DocumentRow.tsx`, `LinkedAccountsView.tsx`, `FindAccountSearch.tsx`, `AddDocsChooser.tsx`, upload progress/success/failure rows, tagging, edit/delete modals.

## TTO Components (`components/index/tto/`)

`TaxHomeResourceAccordions.tsx` — TTO home page resource accordions.

## Review Components (`components/review/`)

| Component | Purpose |
|-----------|---------|
| `ReviewTodoList.tsx` | To-do list for review items |
| `ReviewFoundScreen.tsx` | Issues found screen |
| `CheckingReturnScreen.tsx` | "Checking your return" loading animation |
| `ExplainThisAmountButton.tsx` | "Explain this amount" help trigger |

## UI Primitives (`components/ui/`)

| Component | Purpose |
|-----------|---------|
| `Button.tsx` | Reusable button |
| `Card.tsx` | Card container |
| `Modal.tsx` | Modal dialog |
| `Accordion.tsx` | Accordion |
| `Tabs.tsx` | Tabs |
| `Checkbox.tsx` | Checkbox with label |
| `CurrencyInput.tsx` | Currency input field |
| `DateInput.tsx` | Date input field |
| `OccupationInput.tsx` | Occupation autocomplete |
| `inputs/TextInput.tsx` | Text input |

## Search & Notifications (`components/search/`, `components/notifications/`)

`SearchOverlay.tsx` — full-screen topic search. `NotificationTray.tsx`, `NotificationBadge.tsx`.
