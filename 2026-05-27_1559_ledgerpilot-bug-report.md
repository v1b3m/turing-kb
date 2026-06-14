---
date: 2026-05-27
type: bug-report
project: ledgerpilot
tags:
  - ledgerpilot
  - bugs
  - qa
  - state-management
---

# LedgerPilot Bug Report — 2026-05-27

Subreport of [[2026-05-27_1549_ledgerpilot-feature-audit|LedgerPilot Feature Audit — 2026-05-27]]. Contains all bugs found during the feature audit, organized by category.

**App URL**: https://ledgerpilot-v1.rlgym.turing.com/

---

## State Schema Violations (Rule 18)

State can be inspected at `/state-diff`. Open it in a second tab — it compares seed data against current localStorage. To reset state: open browser DevTools → Console → run:

```js
Object.keys(localStorage).filter(k => k.includes('qb-') || k.includes('gym')).forEach(k => localStorage.removeItem(k)); location.reload();
```

### BUG-S01: Vendor store — critical shape violation

- **Functional Module**: Accounts Payable
- **Feature Area**: Vendor Management
- **Sub-Feature**: Create New Vendor
- **Store**: `qb-gym-vendor-store`
- **Severity**: Critical
- **Description**: New vendors created via the UI have only **5 fields** while seed vendors have **18 fields**. 13 fields are missing from new records: `active`, `bankAccountNumber`, `businessType`, `city`, `country`, `eligibleFor1099`, `phoneNumber`, `routingNumber`, `state`, `streetAddress1`, `taxIdentifier`, `taxName`, `trackPaymentsFor1099`, `zipCode`. Additionally, new vendors have an `asOfDate` field not present in seed vendors (reverse shape violation).
- **Impact**: Any code reading vendor records cannot rely on field presence. Breaks filtering, reporting, and 1099 workflows.
- **Steps to reproduce**:
  1. Reset state (see above)
  2. Open `/state-diff` in a second tab and note the vendor store shape
  3. In the main tab, go to the homepage (`/app/homepage`)
  4. In the left sidebar, hover over **Expenses** to expand the submenu
  5. Click **Vendors**
  6. Click **New vendor** (top-right)
  7. Fill in any company name (e.g. "Test Vendor Co") and an email
  8. Click **Save**
  9. Go back to `/state-diff` and reload — compare the new vendor record against any seed vendor
  10. The new record will have ~5 fields; seed records have ~18
- **Evidence**: `pg/screenshots/vendor-saved.png`

### BUG-S02: Bill store — shape violations

- **Functional Module**: Accounts Payable
- **Feature Area**: Vendor Bill Entry / List and Edit Bills
- **Sub-Feature**: Manual Bill Creation / List and Edit
- **Store**: `qb-bill-store`
- **Severity**: High
- **Description**: `paidAmount` and `paidAt` missing from 71/103 bills. `vendor` field missing from 1/103 bills. `bill-review-00006` has empty `vendorCompanyName` and `billNo`.
- **Impact**: Payment tracking and vendor reporting unreliable for bills missing payment fields.
- **Steps to reproduce**:
  1. Reset state
  2. Open `/state-diff` in a second tab
  3. In the main tab, go to `/app/homepage`
  4. In the left sidebar, hover over **Expenses** to expand the submenu
  5. Click **Bills**
  6. The bill list loads — no action needed, this is a seed data issue
  7. On `/state-diff`, inspect `qb-bill-store` — compare any bill with `id: "bill-00001"` against `id: "bill-review-00006"`
  8. Note that `paidAmount` and `paidAt` are present on some bills but missing from others
- **Evidence**: `pg/screenshots/state-diff-after-bill.png`

### BUG-S03: Invoice store — shape, pattern, and date violations

- **Functional Module**: Accounts Receivable
- **Feature Area**: Invoice Creation
- **Sub-Feature**: Create & Send Invoice
- **Store**: `qb-gym-sales-invoices-store`
- **Severity**: High
- **Description**: Three distinct violations:
  1. **Shape**: Seed invoices have 29 fields, new invoices have 28. 6 fields exist only in seed (`domain`, `homeTotalAmt`, `linkedTxn`, `shipAddr`, `syncToken`, `txnTaxDetail`), 5 exist only in new (`billEmail`, `customerMemo`, `locationOfSale`, `privateNote`, `statementMemo`).
  2. **Pattern**: Seed IDs use `inv-00001`, new IDs use `inv_mpnwjd7t_7w1zbq`.
  3. **Date (Rule 18.4)**: `txnDate`/`dueDate` use date-only format `"2026-02-08"` while `createdAt` uses full ISO `"2026-05-27T..."`. All dates must be ISO 8601.
- **Impact**: ID pattern mismatch breaks lookups that assume a format. Date inconsistency breaks sorting and comparison logic.
- **Steps to reproduce**:
  1. Reset state
  2. Open `/state-diff` in a second tab
  3. In the main tab, go to `/app/homepage`
  4. Click the **Create (+)** button in the left sidebar (top icon, circle with plus)
  5. In the popup menu, click **Invoice**
  6. In the invoice form, select a customer from the **Customer** dropdown
  7. Add a line item — click the **Product/Service** dropdown and pick any product
  8. Click **Save** (bottom of form)
  9. On `/state-diff`, reload and inspect `qb-gym-sales-invoices-store`
  10. Compare the new invoice's fields, ID format, and date formats against any seed invoice
- **Evidence**: `pg/screenshots/ar-after-invoice-save.png`

### BUG-S04: Customer store — shape violation

- **Functional Module**: CustomerHub
- **Feature Area**: Customer creation
- **Sub-Feature**: Customer creation
- **Store**: `qb-gym-sales-customers-store`
- **Severity**: Medium
- **Description**: `companyName` missing from 49/97 customer records.
- **Impact**: Customer display and search may show blanks or fail for half the records.
- **Steps to reproduce**:
  1. Reset state
  2. Open `/state-diff`
  3. Inspect `qb-gym-sales-customers-store` — compare multiple customer records
  4. Some will have `companyName`, others won't — no UI action needed, this is a seed data issue

### BUG-S05: Items store — shape violation

- **Functional Module**: Accounts Receivable
- **Feature Area**: Invoice Creation
- **Sub-Feature**: Create & Send Invoice
- **Store**: `qb-gym-sales-items-store`
- **Severity**: Medium
- **Description**: 6 fields inconsistently present across items: `syncToken`, `domain`, `qtyOnHand`, `invStartDate`, `expenseAccountRefValue`, `assetAccountRefValue`.
- **Impact**: Inventory and product listing may break on records missing expected fields.
- **Steps to reproduce**:
  1. Reset state
  2. Open `/state-diff`
  3. Inspect `qb-gym-sales-items-store` — compare multiple item records
  4. Note that some items have `qtyOnHand`, `invStartDate`, etc. while others don't — seed data issue

### BUG-S06: Email logs store — shape violation

- **Functional Module**: Accounts Receivable
- **Feature Area**: Invoice Creation
- **Sub-Feature**: Sending an email
- **Store**: `qb-gym-sales-email-logs-store`
- **Severity**: Medium
- **Description**: `invoiceId` missing from 60/84 email log records.
- **Impact**: Email history cannot be linked back to invoices for 71% of records.
- **Steps to reproduce**:
  1. Reset state
  2. Open `/state-diff`
  3. Inspect `qb-gym-sales-email-logs-store` — compare records
  4. 60 out of 84 will be missing `invoiceId` — seed data issue

### BUG-S07: Report workflows store — shape violation

- **Functional Module**: Financial Reporting
- **Feature Area**: Custom Reports
- **Sub-Feature**: Schedule & Email Report
- **Store**: `qb-gym-report-workflows-store`
- **Severity**: Low
- **Description**: `emailCc` missing from 10/20 workflow records.
- **Impact**: Scheduled report emails may skip CC recipients.
- **Steps to reproduce**:
  1. Reset state
  2. Open `/state-diff`
  3. Inspect `qb-gym-report-workflows-store` — compare records
  4. 10 out of 20 will be missing `emailCc` — seed data issue

### BUG-S08: Bill store — pattern violation

- **Functional Module**: Accounts Payable
- **Feature Area**: Vendor Bill Entry
- **Sub-Feature**: Manual Bill Creation
- **Store**: `qb-bill-store`
- **Severity**: High
- **Description**: Seed bill IDs use sequential format `bill-00001`, new bills created via UI get UUID-style IDs. Mixed patterns in the same collection.
- **Impact**: Any code that parses or sorts by ID format will behave inconsistently.
- **Steps to reproduce**:
  1. Reset state
  2. In the main tab, go to `/app/homepage`
  3. In the left sidebar, hover over **Expenses** → click **Bills**
  4. Click **Create bill** (top-right area, or from expense overview)
  5. Fill in vendor, amount, category, and click **Save**
  6. Open `/state-diff` → inspect `qb-bill-store`
  7. The new bill's `id` will be a UUID-style string; scroll to see seed bills with IDs like `bill-00001`

### BUG-S09: Bill number not auto-generated

- **Functional Module**: Accounts Payable
- **Feature Area**: Vendor Bill Entry
- **Sub-Feature**: Manual Bill Creation
- **Store**: `qb-bill-store`
- **Severity**: Medium
- **Description**: New bills created via the UI get an empty `billNo` field. No auto-generation of bill numbers.
- **Impact**: Bills cannot be referenced by number in search, reports, or payment flows.
- **Steps to reproduce**:
  1. Follow the same steps as BUG-S08 to create a new bill
  2. On `/state-diff`, inspect the new bill record
  3. The `billNo` field will be an empty string `""`

---

## UI Bugs

### BUG-U01: Invoice date defaults to wrong date

- **Functional Module**: Accounts Receivable
- **Feature Area**: Invoice Creation
- **Sub-Feature**: Create & Send Invoice
- **Severity**: Medium
- **Description**: Invoice creation form defaults the date to 05/06/2026 instead of today's date (05/27/2026). The date is ~3 weeks in the past.
- **Steps to reproduce**:
  1. Go to `/app/homepage`
  2. Click the **Create (+)** button in the left sidebar (top icon)
  3. Click **Invoice** in the popup menu
  4. Look at the **Invoice date** field near the top of the form
  5. It shows `05/06/2026` instead of today's date
- **Expected**: Date should default to today
- **Evidence**: `pg/screenshots/ar-create-invoice.png`

### BUG-U02: Mileage — Add Trip input concatenation

- **Functional Module**: Banking & Reconciliation
- **Feature Area**: Expense Report Submission
- **Sub-Feature**: Submit Mileage Reimbursement
- **Severity**: High
- **Description**: In the Add Trip dialog, typing in the Start point field, then moving focus to the End point field and typing, causes both values to concatenate into the Start point field. The End point remains empty. The input focus handling is broken — keystrokes continue going to the Start point regardless of which field is focused.
- **Steps to reproduce**:
  1. Go to `/app/mileage` (note: **not** `/app/expenses/mileage`, which is 404)
  2. Click the green **Add trip** button (top-right)
  3. A dialog opens with **Start point** and **End point** text fields
  4. Click into the **Start point** field and type an address (e.g. "123 Main St")
  5. Click or tab into the **End point** field
  6. Type a second address (e.g. "456 Oak Ave")
  7. Look at the **Start point** field — it now contains both addresses concatenated ("123 Main St456 Oak Ave")
  8. The **End point** field is empty
- **Expected**: Each field should receive its own input independently
- **Evidence**: `pg/screenshots/mileage-page.png`

### BUG-U03: Schedule Payment — payee not pre-filled

- **Functional Module**: Accounts Payable
- **Feature Area**: Payment Processing
- **Sub-Feature**: Schedule ACH Payment
- **Severity**: Low
- **Description**: When navigating to Schedule Payment from a specific bill's action menu, the Payee dropdown is not pre-filled with the bill's vendor. User must manually select the vendor.
- **Steps to reproduce**:
  1. Go to `/app/homepage`
  2. In the left sidebar, hover over **Expenses** → click **Bills**
  3. In the bills list, find any bill row and click the **Schedule payment** button/link in its row (or use the dropdown arrow → Schedule payment)
  4. The Schedule Payment form opens
  5. Look at the **Payee** dropdown — it is empty, even though you navigated from a specific bill with a known vendor
- **Expected**: Payee should be pre-filled with the bill's vendor
- **Evidence**: `pg/screenshots/ap-schedule-payment.png`

### BUG-U04: Quick Create shows "Feature unavailable" on 404 pages

- **Functional Module**: Search & Header
- **Feature Area**: Create Actions
- **Sub-Feature**: Create Actions
- **Severity**: Low
- **Description**: The (+) Quick Create button shows a "Feature unavailable" popup when clicked from a 404 page. It works correctly from valid pages.
- **Steps to reproduce**:
  1. Navigate to any 404 route — e.g. type `/app/reports` in the address bar and hit Enter
  2. The page shows "Error 404 — Page not found"
  3. Click the **(+)** Create button in the left sidebar
  4. Instead of showing the create menu, a popup appears saying "Feature unavailable"
  5. Now navigate to `/app/homepage` and click the same **(+)** button — the full create menu appears correctly
- **Expected**: Create menu should work regardless of current page context

### BUG-U05: Vendor form — missing accessibility labels (skip)

- **Functional Module**: Accounts Payable
- **Feature Area**: Vendor Management
- **Sub-Feature**: Create New Vendor
- **Severity**: Low
- **Description**: Input fields in the vendor creation form lack `aria-label` attributes. Screen readers cannot identify the purpose of each field.
- **Steps to reproduce**:
  1. Go to `/app/homepage`
  2. In the left sidebar, hover over **Expenses** → click **Vendors**
  3. Click **New vendor** (top-right)
  4. Open browser DevTools (F12) → Elements tab
  5. Inspect any text input in the form (company name, email, phone, etc.)
  6. None of the `<input>` elements have `aria-label` or `aria-labelledby` attributes
- **Expected**: All inputs should have appropriate `aria-label` or `aria-labelledby`
- **Evidence**: `pg/screenshots/vendor-new.png`

---

## Route / Navigation Bugs

### BUG-R01: Sidebar "Reports" link points to 404

- **Functional Module**: Financial Reporting
- **Feature Area**: Standard Reports
- **Sub-Feature**: Run Profit & Loss Statement / Run Balance Sheet / Run Cash Flow Statement
- **Severity**: High
- **Description**: The "Reports" link in the primary sidebar navigation points to `/app/reports`, which returns 404. The actual reports pages are at `/app/standardreports` and `/app/customreports`.
- **Steps to reproduce**:
  1. Go to `/app/homepage`
  2. In the left sidebar, look for the **Reports** link (has a line-chart icon, between Feed and All apps)
  3. Click **Reports**
  4. The page shows "Error 404 — Page not found"
  5. Manually navigate to `/app/standardreports` — the reports page loads correctly
- **Evidence**: `pg/screenshots/reports-404.png`

### BUG-R02: Multiple sidebar routes return 404 (skip)

- **Functional Module**: Cross-cutting (affects Accounts Payable, Banking & Reconciliation, Financial Reporting, CustomerHub)
- **Feature Area**: Search & Header (sidebar navigation)
- **Sub-Feature**: N/A — routing infrastructure, not a single taxonomy feature
- **Severity**: Medium
- **Description**: Several routes declared in the sidebar navigation do not resolve. The sidebar shows these links, but clicking them leads to a 404.
- **Steps to reproduce**: Click each of these sidebar links or navigate to the URL directly:

| Sidebar Route | Status | How to find the correct page instead |
|---|---|---|
| `/app/home` | 404 | Navigate to `/app/homepage` |
| `/app/reports` | 404 | Navigate to `/app/standardreports` |
| `/app/banking/transactions` | 404 | Hover **Accounting** in sidebar → click **Overview**, or go to `/app/banking?jobId=accounting` |
| `/app/expenses/mileage` | 404 | Navigate to `/app/mileage` directly |
| `/app/expenses/bills` | 404 | Hover **Expenses** in sidebar → click **Overview**, then use the **Create bill** button |
| `/app/expenses/1099s` | 404 | No working route exists — page not implemented |
| `/app/customers-overview?jobId=customers` | 404 | No working route exists — page not implemented |

### BUG-R03: 1099s page not implemented

- **Functional Module**: Accounts Payable
- **Feature Area**: Vendor Management
- **Sub-Feature**: 1099 Preparation
- **Severity**: Medium
- **Description**: The taxonomy declares 1099 Preparation as "Merged", but `/app/expenses/1099s` returns 404. No alternative route found. Data exists in `qb-gym-prepare-1099-store` (24 records) but there is no UI to view it.
- **Steps to reproduce**:
  1. In the left sidebar, hover over **Expenses** to expand the submenu
  2. Click **1099s**
  3. The page shows "Error 404 — Page not found"
  4. Try navigating directly to `/app/expenses/1099s` — same result
  5. No other route serves this page

### BUG-R04: Customer Hub page not implemented (skip)

- **Functional Module**: CustomerHub
- **Feature Area**: Customer creation
- **Sub-Feature**: Customer creation
- **Severity**: Medium
- **Description**: The taxonomy declares Customer Creation as "Merged", but `/app/customers-overview?jobId=customers` returns 404. Customer creation is partially available through the invoice creation flow (Add Customer dropdown), but there is no dedicated Customer Hub page.
- **Steps to reproduce**:
  1. In the left sidebar, click **All apps** (grid icon near bottom of the top section)
  2. In the flyout, look for **Customer Hub** and click it
  3. The page shows "Error 404 — Page not found"
  4. **Workaround**: Customers can be added during invoice creation — Create (+) → Invoice → Customer dropdown → Add new

---

## Bug Priority Matrix

| Bug ID  | Priority | Functional Module        | Feature Area                            | Sub-Feature                                                               |
| ------- | -------- | ------------------------ | --------------------------------------- | ------------------------------------------------------------------------- |
| BUG-S01 | Critical | Accounts Payable         | Vendor Management                       | Create New Vendor                                                         |
| BUG-S02 | High     | Accounts Payable         | Vendor Bill Entry / List and Edit Bills | Manual Bill Creation / List and Edit                                      |
| BUG-S03 | High     | Accounts Receivable      | Invoice Creation                        | Create & Send Invoice                                                     |
| BUG-S08 | High     | Accounts Payable         | Vendor Bill Entry                       | Manual Bill Creation                                                      |
| BUG-U02 | High     | Banking & Reconciliation | Expense Report Submission               | Submit Mileage Reimbursement                                              |
| BUG-R01 | High     | Financial Reporting      | Standard Reports                        | Run Profit & Loss Statement / Run Balance Sheet / Run Cash Flow Statement |
| BUG-S04 | Medium   | CustomerHub              | Customer creation                       | Customer creation                                                         |
| BUG-S05 | Medium   | Accounts Receivable      | Invoice Creation                        | Create & Send Invoice                                                     |
| BUG-S06 | Medium   | Accounts Receivable      | Invoice Creation                        | Sending an email                                                          |
| BUG-S09 | Medium   | Accounts Payable         | Vendor Bill Entry                       | Manual Bill Creation                                                      |
| BUG-U01 | Medium   | Accounts Receivable      | Invoice Creation                        | Create & Send Invoice                                                     |
| BUG-R02 | Medium   | Cross-cutting            | Navigation / Routing                    | Sidebar link resolution                                                   |
| BUG-R03 | Medium   | Accounts Payable         | Vendor Management                       | 1099 Preparation                                                          |
| BUG-R04 | Medium   | CustomerHub              | Customer creation                       | Customer creation                                                         |
| BUG-S07 | Low      | Financial Reporting      | Custom Reports                          | Schedule & Email Report                                                   |
| BUG-U03 | Low      | Accounts Payable         | Payment Processing                      | Schedule ACH Payment                                                      |
| BUG-U04 | Low      | Search & Header          | Create Actions                          | Create Actions                                                            |
| BUG-U05 | Low      | Accounts Payable         | Vendor Management                       | Create New Vendor                                                         |
