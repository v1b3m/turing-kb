---
title: LedgerPilot Full App Review
date: 2026-05-29T16:34:00+03:00
tags:
  - ledgerpilot
  - review
  - qa
  - turing
type: review
branch: chore/fix-1099s
---

# LedgerPilot Full App Review

**Date:** 2026-05-29
**Branch:** `chore/fix-1099s`
**App URL:** `http://localhost:3001`
**Review method:** Chrome CDP visual review + code-level playbook audit
**Reference:** `pg/module-review-playbook.md` and `pg/LedgerPilot Taxonomy Document`

---

## Executive Summary

The app is a functional accounting SaaS frontend with 20 Zustand stores, 42 routes, and 243+ components. Most modules render correctly and navigation works. However, the review uncovered **1 critical issue** (report builder blank pages), **3 high-severity code bugs**, **5 medium-severity code bugs**, and **1 state corruption issue** during zero-change save testing.

| Severity | Count | Category |
|----------|-------|----------|
| Critical | 1 | Report builder renders blank |
| High | 3 | Address key mutation, date format inconsistency |
| Medium | 5 | Float corruption, line ID regen, txnTaxDetail wipe, `\|\|` vs `??` |
| Low | 1 | 28 files still use FeatureUnavailableDialog |
| State noise | 1 | Invoice save corrupts unrelated stores |

---

## Module-by-Module Review

### 1. Homepage & Navigation

**Status:** Functional with minor issues

**Working:**
- Homepage renders with greeting, Business Feed, create actions, Business at a glance widgets (Bank Accounts, P&L, Invoices)
- Sidebar navigation: Create, Bookmarks, Home, Feed, Reports, All apps, Pinned (Accounting, Expenses, Sales)
- Coming-soon page for unimplemented features (Bookmarks, Feed, Payroll, Team, Time, Projects) — properly branded with green icon
- "Create invoice" action works and navigates to invoice editor

**Issues:**
- **Sidebar Create button** shows plain `FeatureUnavailableDialog` instead of `ComingSoonDialog`
- **Homepage create action buttons** (Get paid online, Record payment, Create estimate, Record expense, Show all) all show plain `FeatureUnavailableDialog`
- **"All apps" sidebar link** navigates to `/app/banking?jobId=accounting` — should probably show an all-apps view

---

### 2. Sales Module (Invoices, Credit Memos, Payments)

**Status:** Functional, multiple code-level bugs from playbook

**Working:**
- Sales & Get Paid overview with funnel (Not Paid / Paid / Deposited), income chart
- Invoice list with filters, status display, View/Edit and Receive payment actions
- Invoice editor: customer selection, line items table, tabs (Edit/Email/Payor/PDF), totals
- Credit Memo editor: customer, email, billing address, line items, discount/tax/shipping
- Receive Payment page: customer selection, record/charge options, payment details
- Sales Transactions list with summary metrics and filters

**Code bugs (from playbook audit):**

| # | Bug | File:Line | Severity |
|---|-----|-----------|----------|
| 1 | `\|\|` on billEmail in credit memo hydration | `credit-memo-editor-content.tsx:1012` | Medium |
| 2 | `\|\|` on billEmail in recurring invoice hydration | `invoice-editor-content.tsx:725` | Medium |
| 3 | `addressRecordFromText` missing original param in credit memo | `credit-memo-editor-content.tsx:171, 1339` | High |
| 4 | `addressRecordFromText` missing original param in recurring invoice | `invoice-editor-content.tsx:1423` | Medium |
| 5 | Line ID regeneration in credit memo | `credit-memo-editor-content.tsx:234, 1345` | Medium |
| 6 | Floating-point corruption in `updateCreditMemo` | `sales-store.ts:2523` | Medium |
| 7 | `txnTaxDetail` no fallback in `updateCreditMemo` | `sales-store.ts:2560` | Medium |
| 8 | Date format inconsistency: invoice uses `YYYY-MM-DDT00:00:00.000Z`, credit memo uses `YYYY-MM-DD` | `invoice-editor-content.tsx:1278` vs `credit-memo-editor-content.tsx:1210` | High |

**Already fixed (from invoice module review):**
- Invoice `billEmail` hydration uses `??` correctly
- Invoice `addressRecordFromText` passes original address
- Invoice line IDs preserved via `lineId` field
- Invoice `txnTaxDetail` falls back to existing value
- `showLineInputs` correctly checks data presence

---

### 3. Expenses Module (Bills, Bill Payments, Vendors, Mileage)

**Status:** Functional, well-implemented

**Working:**
- Expense overview with bill pipeline, spending chart, spending insights donut
- Bills list with tabs (For review / Unpaid / Paid / Recurring), filters, export
- Bill editor with autofill panel (BETA), vendor selection, line items, memo, attachments
- Vendors list with summary metrics, search, contact details, 1099 tracking
- Mileage tracking with tax year selector, vehicle info, trip management
- Pay Bills page with payment account selection, date, bill selection table
- Bill area already migrated to `ComingSoonDialog`

**No playbook bugs found in the expenses module.**

---

### 4. Journal Entry Module

**Status:** Functional

**Working:**
- Journal Entry editor with date, journal number, lines table (Account / Debits / Credits / Description / Name)
- Totals row, add/clear lines, memo, attachments
- Save / Save and new / Make recurring / Cancel buttons

**No issues found.**

---

### 5. Accounting Module (Chart of Accounts, Reconcile)

**Status:** Functional

**Working:**
- Chart of Accounts: account list with Number, Name, Account Type, Detail Type, QuickBooks Balance, Bank Balance
- Actions: View register, Run report, New account, Batch actions
- Reconcile: account selector, beginning/ending balance, service charge/interest fields, Start reconciling button
- Breadcrumb navigation (Chart of accounts > Asset register > Reconcile)

**No issues found.**

---

### 6. Reports Module

**Status:** Critical issue — report builder renders blank

**Working:**
- Standard Reports list page: favorites, business overview, shortcuts, report search
- Custom Reports list page: saved reports with name, creator, date range, access, actions
- Reports sidebar navigation: Standard, Custom, Management (coming soon), KPI, Dashboards, etc.

**Issues:**
- **CRITICAL: Report builder page renders completely blank** for ALL report types (P&L, Trial Balance, Balance Sheet, Cash Flow, AR Aging). URL `/app/report/builder?report=PANDL` shows only the shell (header + sidebar) with no report content. React Suspense boundaries (`<!--$--><!--/$-->`) never resolve.
- **Report items in the standard reports list do not navigate** to the report builder when clicked. This was also noted in the taxonomy document for Balance Sheet.
- 13 files in reports module still use `FeatureUnavailableDialog`

---

### 7. 1099 Filing & Prepare Module

**Status:** Functional

**Working:**
- Prepare 1099 wizard: 3-step flow (Quick setup / Review / Pay and file)
- Company info verification with edit capability
- 1099 Filing Details: tabs (E-file / Recipients & W-9s / Completed Forms)
- Review panel showing vendor forms with statuses
- Autofilled forms workflow explanation

**No issues found.**

---

### 8. Banking Module

**Status:** Functional

**Working:**
- Bank Transactions page with bank connection wizard (Citi, Chase, BoA, Wells Fargo, etc.)
- Search by bank name
- Upload transactions option for CSV/XLS/OFX

**Notes:**
- Seeded bank transactions don't appear until user uploads — noted in taxonomy as expected behavior but potentially confusing

---

### 9. Settings & Search

**Status:** Functional

**Working:**
- Account Settings with sidebar nav (Company / Usage / Payments / Accounting / Sales / Expenses / Time / Advanced)
- Editable settings cards (Accounting method, Customer label, Communications, Chart of accounts, Categories, Automation)
- Full Search with tabs (All / Transactions / Help), showing 723 results
- Jump-to sidebar (Live Experts, Bank Transactions, Reports, Customers, Vendors)
- Customers & Leads page with customer list, summary metrics, actions

**No issues found.**

---

### 10. Customer Hub

**Status:** Functional

**Working:**
- Customers & Leads page with tabs (Customers / Leads)
- Customer list with name, company, phone, open balance, actions
- Summary metrics bar
- Inline sidebar with Customer Hub sub-navigation (Overview, Customers & leads, Opportunities, Estimates, Contacts, Appointments, Reviews)
- New customer creation

---

## State-Diff Verification

### Zero-change save test (INV-00001)

**Procedure:** Clear localStorage, reload, open INV-00001, save without changes, check state-diff.

**Invoice store result:** Only `updatedAt` changed — **correct**.

**State noise detected:**
- **Accounts Store** showed changes (should not be affected by invoice save)
- **CRITICAL: Scheduled Bill Payments Store lost all 64 records** (dropped to 0) during an unrelated invoice save — **state corruption**

This violates the playbook's "zero noise in state diff" rule and CLAUDE.md Rule 18.5: "a user action must only touch the stores and records it logically affects."

---

## FeatureUnavailableDialog Audit

28 files still use the plain `FeatureUnavailableDialog` instead of the branded `ComingSoonDialog`:

| Area | Files |
|------|-------|
| Sales/Invoices | `invoice-editor-content.tsx`, `customer-panels.tsx`, `sales-invoices-content.tsx`, `sales-transactions-content.tsx`, `receive-payment-content.tsx`, `credit-memo-editor-content.tsx`, `sales-create-actions.tsx`, `sales-get-paid-content.tsx` |
| Reports | `report-card-toolbar.tsx`, `report-header-buttons.tsx`, `compare-select.tsx`, `trial-balance-report-filters.tsx`, `report-document-footer.tsx`, `report-builder-top-bar.tsx`, `standard-reports-item-row.tsx`, `standard-reports-toolbar.tsx`, `profit-loss-report-filter.tsx`, `balance-sheet-report-filter.tsx`, `cash-flow/statement-of-cash-flows-report-filter.tsx`, `ar-aging/accounts-receivable-aging-report-page.tsx`, `custom-reports/custom-report-action.tsx` |
| Shared/Layout | `app-layout.tsx`, `app-header.tsx`, `import-bank-transactions-dialog.tsx` |
| Search | `full-search-help-content.tsx`, `full-search-transactions-content.tsx` |
| Other | `receipts-content.tsx`, `recurring-content.tsx` |

The bill/expenses area has already been fully migrated to `ComingSoonDialog`.

---

## Priority Action Items

### P0 — Must fix
1. **Report builder blank pages** — All 5 report types render empty. React Suspense never resolves.
2. **State corruption on invoice save** — Scheduled Bill Payments Store loses all 64 records during unrelated invoice save.

### P1 — Should fix (playbook bugs)
3. **Credit memo `addressRecordFromText`** — Pass original address to preserve `customer` vs `line1` key.
4. **Date format inconsistency** — Normalize invoice and credit memo date formats.
5. **Credit memo `updateCreditMemo` float corruption** — Wrap `remainingCredit` arithmetic with `roundMoney()`.
6. **Credit memo line ID regeneration** — Preserve existing line IDs through form round-trip.
7. **Credit memo `txnTaxDetail` fallback** — Fall back to existing value in `updateCreditMemo`.
8. **Credit memo `billEmail` hydration** — Change `||` to `??` to preserve empty strings.

### P2 — Nice to have
9. **Migrate 28 files from `FeatureUnavailableDialog` to `ComingSoonDialog`** for UI consistency.
10. **Standard reports list navigation** — Report items should navigate to the report builder.
11. **Recurring invoice `addressRecordFromText`** — Pass original address in recurring creation path.

---

## Appendix: Playbook Bug Pattern Summary

| # | Pattern | Invoice | Credit Memo | Bill | Journal |
|---|---------|---------|-------------|------|---------|
| 1 | `\|\|` vs `??` on strings | Fixed | **Bug** | N/A | N/A |
| 2 | Address key mutation | Fixed | **Bug** | N/A | N/A |
| 3 | Line ID regeneration | Fixed | **Bug** | N/A | Low risk |
| 4 | Float balance corruption | Fixed | **Bug** | Low risk | N/A |
| 5 | `txnTaxDetail` wipe | Fixed | **Bug** | N/A | N/A |
| 6 | Date format inconsistency | **Bug** (format differs from credit memo) | **Bug** | N/A | N/A |
| 7 | Unpersisted fields | Needs manual test | Needs manual test | Needs manual test | N/A |
| 8 | `showLineInputs` | Fixed | N/A | N/A | N/A |
| 9 | FeatureUnavailableDialog | **28 files** | Included | Migrated | N/A |
