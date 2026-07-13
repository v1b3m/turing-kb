---
tags:
  - returnmax
  - stores
  - zustand
---

# Stores

All 13 Zustand stores follow the pattern `{ state: X; actions: X }`. All persisted via `safeLocalStorage`.

## Primary Stores

### useTaxReturnStore (`store/tax-return-store.ts`, ~4500 lines)

The central store. Holds the entire `TaxReturnState` object — the single aggregate root for all tax return data.

**State shape (key sections):**
- `personalInfo`, `spouseInfo`, `dependents[]`
- `income.w2s[]`, `income.form1099INT[]`, `income.form1099DIV[]`, `income.form1099B[]`, ... (15+ income types)
- `deductions` (25+ deduction categories nested)
- `credits` (20+ credit types nested)
- `stateReturns[]` (per-state return data)
- `otherTaxSituations` (17+ IRS form buckets)
- `filing`, `paymentInfo`, `expertReview`, `errorCheck`, `onboarding`
- `filingSummary` (computed), `filingProgress`, `managedClients[]`

**Key patterns:**
- Deterministic IDs via `generateRecordId(entityType, ...parts)` for most entities
- Random UUIDs via `generateId()` for runtime-created form/OTS records
- Cross-store writes to `useFiledReturnsStore` and `useOrderReceiptStore` on file/pay

### useAuthStore (`store/auth-store.ts`)

Authentication + per-user data isolation.
- **State:** `{ currentUser: User | null }`
- **Actions:** `login`, `logout`, `register`, `signInWithEmail`, `markOnboarded`, `resetPassword`
- Implements per-user store backup/restore on user switch

### useUsersStore (`store/users-store.ts`)

Flat user list. Seed has 4 users (Maya onboarded, Marcus/Elena/Sophia not).
- Migration resets to seed on every version bump ("the seeded filer list is canonical")

## UI Stores

### useTaxUIStore (`store/tax-ui-store.ts`)

UI-only state: current section/page, sidebar state, bookmarks, toasts, help/expert panels, document upload, income questions completed flag. Toasts stripped on persist (session-only).

### useSettingsStore (`store/settings-store.ts`)

Currently just `showProgressBar: boolean`.

### useModalStore (`store/modal-store.ts`)

Modal dialog: title, body, isOpen, callback. Always starts closed on rehydrate.

### useSearchStore (`store/search-store.ts`)

Topic search overlay: query, results, recent searches.

## Data Stores

### useDocumentsStore (`store/documents-store.ts`)

Uploaded tax documents organized by tax year. `thumbnailDataUrl` stripped on persist.

### useLinkedAccountsStore (`store/linked-accounts-store.ts`)

Connected financial institution accounts (mock data).

### useFiledReturnsStore (`store/filed-returns-store.ts`)

Previously filed returns with `FiledReturnSnapshot` (granular income, deduction, credit, dependent data). Written to by `useTaxReturnStore` on e-file submission.

### useOrderReceiptStore (`store/order-receipt-store.ts`)

Order receipts by tax year. Written to by `useTaxReturnStore` on payment confirmation.

### useNotificationsStore (`store/notifications-store.ts`)

In-app notifications. Seeded with 3 samples. Cross-talks with `useTaxUIStore` to close panels on open.

### useArticlesStore (`store/articles-store.ts`)

Help article feedback: `{ [articleId]: { isHelpful: boolean | null } }`.
