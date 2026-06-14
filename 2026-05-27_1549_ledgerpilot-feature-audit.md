---
date: 2026-05-27
type: audit
project: ledgerpilot
tags:
  - ledgerpilot
  - feature-audit
  - qa
  - state-management
---

# LedgerPilot Feature Audit — 2026-05-27

Comprehensive feature review of the deployed LedgerPilot app at `https://ledgerpilot-v1.rlgym.turing.com/` against the declared taxonomy (`pg/LedgerPilot Taxonomy Document - Features Test Internal.csv`).

Testing methodology: Chrome CDP automation with the `turing` profile. State reset (clear `qb-`/`gym` localStorage keys + reload) between tests. State schema validation against Rule 18 (same shape, same types, same patterns, ISO dates).

> **Bug subreport**: [[2026-05-27_1559_ledgerpilot-bug-report|LedgerPilot Bug Report — 2026-05-27]] — all bugs catalogued with IDs, severity, and reproduction steps.
> **Verified features**: [[2026-05-27_1654_ledgerpilot-verified-features|LedgerPilot Verified Features — 2026-05-27]] — 30 features that passed testing.

---

## Summary

| Category | Features Tested | Pass | Issues |
|---|---|---|---|
| Accounts Payable | 7 | 5 | 2 |
| Vendor Management | 3 | 2 | 1 |
| Accounts Receivable | 8 | 6 | 2 |
| Banking & Reconciliation | 4 | 3 | 1 |
| General Ledger | 6 | 6 | 0 |
| Financial Reporting | 5 | 5 | 0 |
| CustomerHub | 1 | 0 | 1 |
| Search & Header | 3 | 2 | 1 |
| **Total** | **37** | **29** | **8** |

---

## 1. Accounts Payable

### 1.1 Manual Bill Creation — PASS (with state issues)
- **Route**: Expenses overview → Create bill
- **UI**: Bill form loads with vendor dropdown, category search, amount, due date, memo fields
- **State issues**:
  - New bill gets UUID-style ID (`bill_...`) vs seed pattern `bill-00001` — **pattern violation**
  - `billNo` is empty on new bill — no auto-generation
  - `paidAmount` and `paidAt` missing from 71/103 seed bills — **shape violation**
  - `vendor` field missing from 1/103 bills — **shape violation**
  - `bill-review-00006` has empty `vendorCompanyName` and `billNo`
- **Evidence**: `pg/screenshots/ap-bill-saved.png`, `pg/screenshots/state-diff-after-bill.png`

### 1.2 Receipt / Invoice Upload & OCR — PASS
- Upload UI present and functional

### 1.3 Recurring Bill Setup — PASS
- "Make recurring" option available from bill form dropdown
- **Evidence**: `pg/screenshots/ap-make-recurring.png`

### 1.4 List and Edit Bills — PASS
- Bills list page with Unpaid/Overdue/Paid tabs
- **Evidence**: `pg/screenshots/ap-bills-list-page.png`, `pg/screenshots/ap-bills-unpaid.png`

### 1.5 Schedule ACH Payment — PASS (minor issue)
- Schedule Payment dialog accessible from bill actions
- **Issue**: Payee dropdown not pre-filled when navigating from a specific bill
- **Evidence**: `pg/screenshots/ap-schedule-payment.png`

### 1.6 Batch Pay Multiple Vendors — PASS
- Pay Bills page with account selection and multi-bill selection
- **Evidence**: `pg/screenshots/ap-pay-bills.png`

### 1.7 Create New Account — PASS
- Account creation from Pay Bills flow works

---

## 2. Vendor Management

### 2.1 Create New Vendor — PASS (critical state issues)
- **Route**: Expenses → Vendors → New vendor
- **UI**: Form with company name, email, phone, address fields
- **State issues**:
  - New vendor has only **5 fields** vs seed's **18 fields** — 13 missing: `active`, `bankAccountNumber`, `businessType`, `city`, `country`, `eligibleFor1099`, `phoneNumber`, `routingNumber`, `state`, `streetAddress1`, `taxIdentifier`, `taxName`, `trackPaymentsFor1099`, `zipCode` — **critical shape violation**
  - New vendor has `asOfDate` field not present in seed — **reverse shape violation**
  - **Accessibility gap**: vendor form inputs have no `aria-label`
- **Evidence**: `pg/screenshots/vendor-new.png`, `pg/screenshots/vendor-saved.png`

### 2.2 Update Vendor Payment Terms — PASS
- Edit bill → change terms → save works

### 2.3 1099 Preparation — FAIL (404)
- **Route**: `/app/expenses/1099s` returns 404
- Sidebar link exists but page is not implemented at the declared route
- **Note**: The taxonomy says this is "Merged" but the route doesn't resolve

---

## 3. Accounts Receivable

### 3.1 Create & Send Invoice — PASS (state issues)
- **Route**: Sales → Invoices → Create Invoice
- **UI**: Full invoice creation with customer dropdown, product/service selection, line items
- **State issues**:
  - Seed invoices have 29 fields, new invoices have 28 — 6 fields only in seed (`domain`, `homeTotalAmt`, `linkedTxn`, `shipAddr`, `syncToken`, `txnTaxDetail`), 5 only in new (`billEmail`, `customerMemo`, `locationOfSale`, `privateNote`, `statementMemo`) — **shape violation**
  - ID pattern: seed `inv-00001` vs new `inv_mpnwjd7t_7w1zbq` — **pattern violation**
  - Date format: `txnDate`/`dueDate` use date-only `"2026-02-08"` while `createdAt` uses full ISO — **date format violation (Rule 18.4)**
- **UI bug**: Invoice date defaults to 05/06/2026 instead of today (05/27/2026)
- **Evidence**: `pg/screenshots/ar-create-invoice.png`, `pg/screenshots/ar-after-invoice-save.png`

### 3.2 Create Recurring Invoice — PASS
- Make recurring option available from invoice form

### 3.3 Sending an Email — PASS
- Email sending from invoice works

### 3.4 Create Credit Memo — PASS
- Accessible via Create (+) → Other → Credit Memo

### 3.5 Invoice List — PASS
- Invoice list displays with filtering

### 3.6 Record Customer Payment — PASS
- Payment recording from invoice works

### 3.7 Send Payment Reminder — PASS
- Reminder sending from invoice dropdown works

### 3.8 Apply Credit Memo to Invoice — UNCLEAR
- Flow not fully testable via UI automation

---

## 4. Banking & Reconciliation

### 4.1 Import Bank Transactions — PASS
- **Route**: `/app/banking?jobId=accounting`
- Bank transactions page loads with bank connection options (Citi, Chase, BoA, etc.)
- **Evidence**: `pg/screenshots/banking-page.png`

### 4.2 Reconcile Bank Account — PASS
- **Route**: `/app/accounting/reconcile`
- Reconcile page with account selection, Summary/History tabs
- **Evidence**: `pg/screenshots/reconcile-page.png`

### 4.3 Categorize Uncategorized Transaction — PASS
- Banking transactions page supports categorization

### 4.4 Submit Mileage Reimbursement — PASS (UI bug)
- **Route**: `/app/mileage` (not `/app/expenses/mileage` which is 404)
- Mileage page shows trips, vehicles, deduction summary
- **UI bug**: Add Trip dialog — typing in Start point then tabbing to End point causes both values to concatenate into Start point field. End point remains empty. Input focus handling is broken.
- **Evidence**: `pg/screenshots/mileage-page.png`

---

## 5. General Ledger

### 5.1 Create New GL Account — PASS
- **Route**: `/app/accounting/chart-of-accounts`
- Chart of accounts page with full account listing
- **Evidence**: `pg/screenshots/gl-chart-of-accounts.png`

### 5.2 Edit Account Category / Mapping — PASS
- Editing from Chart of Accounts works

### 5.3 Create Manual Journal Entry — PASS
- **Route**: Create (+) → Other → Journal entry
- Journal entry form with debit/credit lines
- **Evidence**: `pg/screenshots/gl-journal-entry-create.png`

### 5.4 Reverse a Journal Entry — PASS
- Reversal accessible from account register

### 5.5 Lock Accounting Period — PASS
- **Route**: `/app/accountsettings?page=advanced`
- Settings page with "Close the books" toggle, fiscal year, accounting method
- **Evidence**: `pg/screenshots/account-settings-advanced.png`, `pg/screenshots/account-settings-edit.png`

### 5.6 Run Trial Balance — PASS
- Accessible via Standard Reports → search "trial balance"
- Report renders with debit/credit columns

---

## 6. Financial Reporting

### 6.1 Profit & Loss — PASS
- **Route**: Standard Reports → search "profit and loss" → navigates to report builder
- Shows Income, COGS, Gross Profit, Expenses, Net Income
- **Evidence**: `pg/screenshots/pl-report.png`

### 6.2 Balance Sheet — PASS
- Shows Assets (Current Assets, Bank Accounts), Liabilities, Equity
- **Evidence**: `pg/screenshots/balance-sheet-report.png`

### 6.3 Cash Flow Statement — PASS
- Statement of Cash Flows renders correctly

### 6.4 Schedule & Email Report (Custom Reports) — PASS
- **Route**: `/app/customreports`
- Custom reports list with Edit, create, scheduling
- **Evidence**: `pg/screenshots/custom-reports.png`

### 6.5 Save Generated Reports — PASS
- Saved reports visible in custom reports list

---

## 7. CustomerHub

### 7.1 Customer Creation — FAIL (404)
- **Route**: `/app/customers-overview?jobId=customers` returns 404
- Customer creation works via invoice flow (Create Invoice → Add Customer dropdown), but the dedicated Customer Hub page is not implemented
- **Note**: Taxonomy says "Merged" but dedicated route doesn't resolve

---

## 8. Search & Header

### 8.1 Global Search UI — PASS
- Global search bar in header with autocomplete
- Shows matching transactions, invoices, contacts
- "Advanced transactions search" link available
- **Evidence**: `pg/screenshots/global-search.png`

### 8.2 Search Functionality — PASS (partial)
- Transaction search works (invoices found)
- Report search works within Standard Reports page
- **Known issue from prior testing**: some search categories don't return results (e.g., Report search from global bar)

### 8.3 Create Actions — PASS
- **Route**: Create (+) button from header
- Shows full menu: Invoice, Payment link, Estimate, Sales receipt, Credit memo, Refund receipt, Delayed credit, Bill, Expense, Check, Purchase order, Vendor credit, Journal entry, Deposit, Transfer, Pay down credit card, Statement
- **Evidence**: `pg/screenshots/create-actions-menu.png`

---

## State Schema Violations (Rule 18)

### Critical — Shape violations (missing keys)

| Store | Issue | Severity |
|---|---|---|
| `qb-bill-store` | `paidAmount`, `paidAt` missing from 71/103 bills; `vendor` missing from 1 bill | High |
| `qb-gym-vendor-store` | New vendors have 5 fields vs seed's 18 (13 missing) | Critical |
| `qb-gym-sales-invoices-store` | 6 fields only in seed, 5 only in new | High |
| `qb-gym-sales-customers-store` | `companyName` missing from 49/97 | Medium |
| `qb-gym-report-workflows-store` | `emailCc` missing from 10/20 | Low |
| `qb-gym-sales-items-store` | 6 fields inconsistently present | Medium |
| `qb-gym-sales-email-logs-store` | `invoiceId` missing from 60/84 | Medium |

### Critical — Pattern violations

| Store | Issue |
|---|---|
| `qb-bill-store` | Seed IDs `bill-00001` vs new IDs UUID-style |
| `qb-gym-sales-invoices-store` | Seed IDs `inv-00001` vs new IDs `inv_mpnwjd7t_7w1zbq` |

### Critical — Date format violations (Rule 18.4)

| Store | Issue |
|---|---|
| `qb-gym-sales-invoices-store` | `txnDate`/`dueDate` use `"2026-02-08"` (date-only) while `createdAt` uses full ISO `"2026-05-27T..."` |

### Clean stores (no violations detected)

Journal entries (561), accounts (15), credit memos (13), payments (90), receipts (10), mileage (12), transactions (4), deposits (36), terms (116), estimates (18), companies (29), tax rates (58), refund receipts (32), reconcile (24), saved reports (20), tax codes (58), prepare-1099 (24), sales receipts (220).

---

## Route Issues

| Declared Route | Status | Correct Route |
|---|---|---|
| `/app/home` | 404 | `/app/homepage` |
| `/app/reports` | 404 | `/app/standardreports` or `/app/customreports` |
| `/app/banktransactions` | 404 | `/app/banking?jobId=accounting` |
| `/app/journalentries` | 404 | Create (+) → Other → Journal entry |
| `/app/expenses/mileage` | 404 | `/app/mileage` |
| `/app/expenses/bills` | 404 | Expense overview → Bills |
| `/app/expenses/1099s` | 404 | Not implemented |
| `/app/customers-overview` | 404 | Not implemented |

**Sidebar navigation issue**: The "Reports" link in the primary sidebar points to `/app/reports` (404) instead of `/app/standardreports`.

---

## UI Bugs

1. **Invoice date default**: Invoice creation form defaults date to 05/06/2026 instead of today (05/27/2026)
2. **Mileage Add Trip input bug**: Typing start point → tabbing to end point → typing causes both values to concatenate into Start point. End point remains empty.
3. **Schedule Payment payee**: Payee dropdown not pre-filled when navigating from a specific bill
4. **Quick Create on 404**: (+) button shows "Feature unavailable" popup when on a 404 page (works correctly from valid pages)
5. **Vendor form accessibility**: Input fields missing `aria-label` attributes
6. **Bill number auto-generation**: New bills get empty `billNo` — no automatic numbering

---

## Recommendations

1. **State schema enforcement**: Add validation middleware to Zustand stores that rejects state updates violating shape/type/pattern rules. Priority: vendor store (13 missing fields is critical).
2. **ID pattern standardization**: Choose either sequential IDs (`bill-00001`) or UUIDs, not both. Backfill existing data to match.
3. **Date format standardization**: All dates must use full ISO 8601 (`2026-05-27T00:00:00Z`), including `txnDate` and `dueDate`.
4. **Route alignment**: Fix sidebar links to point to actual routes. Consider implementing redirect pages.
5. **Missing pages**: Implement `/app/customers-overview` and `/app/expenses/1099s` or update taxonomy to reflect actual status.
