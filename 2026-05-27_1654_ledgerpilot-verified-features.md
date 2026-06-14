---
date: 2026-05-27
type: audit
project: ledgerpilot
tags:
  - ledgerpilot
  - feature-audit
  - qa
  - verified
---

# LedgerPilot Verified Features — 2026-05-27

Features that passed testing during the [[2026-05-27_1549_ledgerpilot-feature-audit|LedgerPilot Feature Audit — 2026-05-27]]. See [[2026-05-27_1559_ledgerpilot-bug-report|LedgerPilot Bug Report — 2026-05-27]] for failures.

**App URL**: https://ledgerpilot-v1.rlgym.turing.com/

---

## Accounts Payable

### Receipt / Invoice Upload & OCR

- **Functional Module**: Accounts Payable
- **Feature Area**: Vendor Bill Entry
- **Sub-Feature**: Receipt / Invoice Upload & OCR
- **Status**: Pass
- **Description**: Upload UI allows attaching receipt/invoice files to bills. OCR extraction maps line items and GL codes from uploaded documents.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. In the left sidebar, hover over **Expenses** to expand the submenu
  3. Click **Vendors** or **Bills**
  4. Open an existing bill or create a new one
  5. Look for the upload/attach area in the bill form
  6. Upload a PDF or image file
  7. Confirm the file attaches and line items are extracted

### Recurring Bill Setup

- **Functional Module**: Accounts Payable
- **Feature Area**: Vendor Bill Entry
- **Sub-Feature**: Recurring Bill Setup
- **Status**: Pass
- **Description**: Bills can be set up as recurring via the "Make recurring" option in the bill form dropdown. Allows scheduling monthly/weekly/custom repeat bills.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. In the left sidebar, hover over **Expenses** → click **Bills**
  3. Open an existing bill or create a new one
  4. In the bill form, look for the dropdown menu (top-right area or actions menu)
  5. Click **Make recurring**
  6. Confirm the recurring schedule template opens

### List and Edit Bills

- **Functional Module**: Accounts Payable
- **Feature Area**: List and Edit Bills
- **Sub-Feature**: List and Edit
- **Status**: Pass
- **Description**: Bills list page displays all bills with Unpaid/Overdue/Paid tab filtering. Individual bills can be opened for viewing and editing, including changing payment terms.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. In the left sidebar, hover over **Expenses** → click **Bills**
  3. The bills list loads with tabs: **Unpaid**, **Overdue**, **Paid**
  4. Click each tab to verify filtering works
  5. Click on any bill row to open it for editing
  6. Change a field (e.g. payment terms) and click **Save**
  7. Confirm the change persists after navigating away and back

### Batch Pay Multiple Vendors

- **Functional Module**: Accounts Payable
- **Feature Area**: Payment Processing
- **Sub-Feature**: Batch Pay Multiple Vendors
- **Status**: Pass
- **Description**: Pay Bills page allows selecting a payment account and then selecting multiple bills from different vendors to pay in a single batch. Supports scheduling the batch payment.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. In the left sidebar, hover over **Expenses** → click **Bills**
  3. Click **Pay Bills** (or equivalent batch pay action)
  4. Select a payment account from the dropdown
  5. Select multiple bills by checking their checkboxes
  6. Click **Schedule payment**
  7. Confirm the payment is scheduled for all selected bills

### Create New Account

- **Functional Module**: Accounts Payable
- **Feature Area**: Create New Account
- **Sub-Feature**: Create New account
- **Status**: Pass
- **Description**: New payment accounts can be created from within the Pay Bills flow. Account type selection limits parent account options appropriately.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. In the left sidebar, hover over **Expenses** → click **Bills**
  3. Click **Pay Bills**
  4. In the account selection dropdown, click **Add new**
  5. Fill in account details (name, type)
  6. Confirm the account is created and selectable in the dropdown

### Update Vendor Payment Terms

- **Functional Module**: Accounts Payable
- **Feature Area**: Vendor Management
- **Sub-Feature**: Update Vendor Payment Terms
- **Status**: Pass
- **Description**: Vendor payment terms can be updated by editing a bill associated with the vendor. Changes to terms (e.g. Net-30 to Net-45) persist after save.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. In the left sidebar, hover over **Expenses** → click **Bills**
  3. Click on any bill to open it for editing
  4. Change the **Payment terms** dropdown to a different value
  5. Click **Save**
  6. Reopen the bill and confirm the new terms are saved

---

## Accounts Receivable

### Create Recurring Invoice

- **Functional Module**: Accounts Receivable
- **Feature Area**: Invoice Creation
- **Sub-Feature**: Create Recurring Invoice
- **Status**: Pass
- **Description**: Invoices can be set up as recurring via the "Make recurring" option from the invoice form. Allows scheduling repeating invoices on a custom cadence.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. Click the **Create (+)** button in the left sidebar
  3. Click **Invoice**
  4. Fill in customer, line items, and amounts
  5. In the invoice form actions/dropdown, click **Make recurring**
  6. Confirm the recurring template form opens

### Sending an Email

- **Functional Module**: Accounts Receivable
- **Feature Area**: Invoice Creation
- **Sub-Feature**: Sending an email
- **Status**: Pass
- **Description**: Invoices can be sent via email directly from the invoice creation/edit form. Email is dispatched and logged.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. Click the **Create (+)** button in the left sidebar → click **Invoice**
  3. Fill in customer, line items, and amounts
  4. Click **Save and send** (or use the send email action)
  5. Confirm the email send dialog appears and email is dispatched

### Create Credit Memo

- **Functional Module**: Accounts Receivable
- **Feature Area**: Invoice Creation
- **Sub-Feature**: Create Credit Memo
- **Status**: Pass
- **Description**: Credit memos can be created to issue credits against customer accounts. Accessible via the Create (+) menu under Other.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. Click the **Create (+)** button in the left sidebar
  3. In the popup menu, scroll to **Other** section
  4. Click **Credit memo**
  5. Fill in customer, amount, and line item details
  6. Click **Save**
  7. Confirm the credit memo appears in the sales transactions list

### Displaying the created and initial invoice

- **Functional Module**: Accounts Receivable
- **Feature Area**: Invoice List
- **Sub-Feature**: Displaying the created and initial invoice
- **Status**: Pass
- **Description**: The invoice list page displays both seed invoices and newly created invoices with filtering capabilities.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. In the left sidebar, hover over **Sales** to expand the submenu
  3. Click **Invoices**
  4. Confirm the invoice list displays with all invoices (seed + any new)
  5. Use the filter/tab controls to verify filtering works

### Record Customer Payment

- **Functional Module**: Accounts Receivable
- **Feature Area**: Payment Collection
- **Sub-Feature**: Record Customer Payment
- **Status**: Pass
- **Description**: Customer payments can be recorded against invoices from the invoice view. Payment is applied and the invoice balance updates.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. In the left sidebar, hover over **Sales** → click **Invoices**
  3. Click on any unpaid invoice
  4. Click **Receive payment** (from the actions/dropdown)
  5. Fill in payment amount, method, and date
  6. Click **Save**
  7. Confirm the invoice status updates to reflect the payment

### Send Payment Reminder

- **Functional Module**: Accounts Receivable
- **Feature Area**: Payment Collection
- **Sub-Feature**: Send Payment Reminder
- **Status**: Pass
- **Description**: Payment reminders can be sent to customers for overdue or outstanding invoices directly from the invoice dropdown menu.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. In the left sidebar, hover over **Sales** → click **Invoices**
  3. Click on any invoice to open it
  4. Click the dropdown/actions menu
  5. Click **Send reminder**
  6. Confirm the reminder email dialog appears and sends

### Run AR Aging Report

- **Functional Module**: Accounts Receivable
- **Feature Area**: Aging & Collections
- **Sub-Feature**: Run AR Aging Report
- **Status**: Pass
- **Description**: AR Aging report renders with aging buckets (0-30, 31-60, 61-90, 90+ days) grouped by customer. Accessible via Standard Reports search.
- **Steps to verify**:
  1. Navigate to `/app/standardreports`
  2. In the report search bar, type "Accounts receivable aging summary"
  3. Select the report from the dropdown
  4. Confirm the report renders with aging buckets by customer
  5. Verify totals across buckets

---

## Banking & Reconciliation

### Import Bank Transactions

- **Functional Module**: Banking & Reconciliation
- **Feature Area**: Bank Feeds
- **Sub-Feature**: Import Bank Transactions
- **Status**: Pass
- **Description**: Bank transactions page loads with bank connection options (Citi, Chase, BoA, Wells Fargo, Capital One, etc.). Supports importing transaction data.
- **Steps to verify**:
  1. Navigate to `/app/banking?jobId=accounting`
  2. Confirm the banking page loads with bank connection options
  3. Verify bank logos and connection buttons are displayed
  4. Confirm transaction import UI is accessible

### Reconcile Bank Account

- **Functional Module**: Banking & Reconciliation
- **Feature Area**: Bank Reconciliation
- **Sub-Feature**: Reconcile Bank Account
- **Status**: Pass
- **Description**: Reconciliation page provides account selection with Summary and History tabs. Allows matching bank transactions against book entries.
- **Steps to verify**:
  1. Navigate to `/app/accounting/reconcile`
  2. Select an account from the account dropdown
  3. Confirm **Summary** and **History** tabs are present
  4. Click through each tab to verify content loads

### Categorize Uncategorized Transaction

- **Functional Module**: Banking & Reconciliation
- **Feature Area**: Bank Reconciliation
- **Sub-Feature**: Categorize Uncategorized Transaction
- **Status**: Pass
- **Description**: Banking transactions page supports categorizing uncategorized transactions. Upload flow triggers display of seeded transactions for categorization.
- **Steps to verify**:
  1. Navigate to `/app/banking?jobId=accounting`
  2. Look for uncategorized transactions in the transaction list
  3. Click on a transaction to open the categorization view
  4. Select a category from the dropdown
  5. Confirm the transaction is recategorized

---

## General Ledger

### Create New GL Account

- **Functional Module**: General Ledger
- **Feature Area**: Chart of Accounts
- **Sub-Feature**: Create New GL Account
- **Status**: Pass
- **Description**: Chart of Accounts page displays full account listing. New accounts can be created with name, type, and number. Account type selection limits parent account options.
- **Steps to verify**:
  1. Navigate to `/app/accounting/chart-of-accounts`
  2. Confirm the chart of accounts table loads with all accounts
  3. Click **New** (or equivalent create button)
  4. Fill in account name, type, and number
  5. Click **Save**
  6. Confirm the new account appears in the chart of accounts list

### Edit Account Category / Mapping

- **Functional Module**: General Ledger
- **Feature Area**: Chart of Accounts
- **Sub-Feature**: Edit Account Category / Mapping
- **Status**: Pass
- **Description**: Existing accounts can be edited from the Chart of Accounts page. Category and mapping changes persist after save.
- **Steps to verify**:
  1. Navigate to `/app/accounting/chart-of-accounts`
  2. Click on any account row to open it for editing
  3. Change the account category or mapping
  4. Click **Save**
  5. Confirm the change persists after navigating away and back

### Create Manual Journal Entry

- **Functional Module**: General Ledger
- **Feature Area**: Journal Entries
- **Sub-Feature**: Create Manual Journal Entry
- **Status**: Pass
- **Description**: Journal entries can be created with debit/credit lines from the Create (+) menu. Supports multiple line items with account selection.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. Click the **Create (+)** button in the left sidebar
  3. In the popup menu, scroll to **Other** section
  4. Click **Journal entry**
  5. Add debit and credit lines with account selection and amounts
  6. Ensure debits and credits balance
  7. Click **Save**
  8. Confirm the journal entry is created

### Reverse a Journal Entry

- **Functional Module**: General Ledger
- **Feature Area**: Journal Entries
- **Sub-Feature**: Reverse a Journal Entry
- **Status**: Pass
- **Description**: Journal entries can be reversed from the account register view. The reversal creates a new offsetting entry.
- **Steps to verify**:
  1. Navigate to `/app/accounting/chart-of-accounts`
  2. Click on a specific account that has journal entries
  3. Click **View Register**
  4. Search for a journal entry in the register
  5. Click the **Reverse** action on the journal entry
  6. Confirm a reversing entry is created

### Lock Accounting Period

- **Functional Module**: General Ledger
- **Feature Area**: Period Close
- **Sub-Feature**: Lock Accounting Period
- **Status**: Pass
- **Description**: Settings page provides a "Close the books" toggle to lock accounting periods. Also includes fiscal year and accounting method configuration with Cancel/Save controls.
- **Steps to verify**:
  1. Navigate to `/app/accountsettings?page=advanced`
  2. Look for the **Close the books** toggle
  3. Toggle it on
  4. Set the closing date
  5. Click **Save**
  6. Confirm the setting persists after reload

### Run Trial Balance

- **Functional Module**: General Ledger
- **Feature Area**: Period Close
- **Sub-Feature**: Run Trial Balance
- **Status**: Pass
- **Description**: Trial Balance report renders with debit and credit columns for all accounts. Accessible via Standard Reports search.
- **Steps to verify**:
  1. Navigate to `/app/standardreports`
  2. In the report search bar, type "trial balance"
  3. Select the report from the dropdown
  4. Confirm the report renders with debit/credit columns
  5. Verify totals balance (debits = credits)

---

## Financial Reporting

### Run Profit & Loss Statement

- **Functional Module**: Financial Reporting
- **Feature Area**: Standard Reports
- **Sub-Feature**: Run Profit & Loss Statement
- **Status**: Pass
- **Description**: P&L report renders with Income, COGS, Gross Profit, Expenses, and Net Income sections. Navigates to a report builder view after selection.
- **Steps to verify**:
  1. Navigate to `/app/standardreports`
  2. In the report search bar, type "profit and loss"
  3. Select the report from the dropdown
  4. Confirm the report renders with Income, COGS, Gross Profit, Expenses, and Net Income
  5. Verify amounts are populated

### Run Balance Sheet

- **Functional Module**: Financial Reporting
- **Feature Area**: Standard Reports
- **Sub-Feature**: Run Balance Sheet
- **Status**: Pass
- **Description**: Balance Sheet report renders with Assets (Current Assets, Bank Accounts), Liabilities, and Equity sections.
- **Steps to verify**:
  1. Navigate to `/app/standardreports`
  2. In the report search bar, type "balance sheet"
  3. Select the report from the dropdown
  4. Confirm the report renders with Assets, Liabilities, and Equity sections
  5. Verify the accounting equation holds (Assets = Liabilities + Equity)

### Run Cash Flow Statement

- **Functional Module**: Financial Reporting
- **Feature Area**: Standard Reports
- **Sub-Feature**: Run Cash Flow Statement
- **Status**: Pass
- **Description**: Statement of Cash Flows renders with Operating, Investing, and Financing sections using the indirect method.
- **Steps to verify**:
  1. Navigate to `/app/standardreports`
  2. In the report search bar, type "statement of cash flow"
  3. Select the report from the dropdown
  4. Confirm the report renders with Operating, Investing, and Financing sections

### Schedule & Email Report

- **Functional Module**: Financial Reporting
- **Feature Area**: Custom Reports
- **Sub-Feature**: Schedule & Email Report
- **Status**: Pass
- **Description**: Custom reports page displays a list of reports with name, created by, last modified, date range, access, and email columns. Reports can be edited, created, and scheduled for automatic email delivery.
- **Steps to verify**:
  1. Navigate to `/app/customreports`
  2. Confirm the custom reports table loads with existing reports
  3. Click **Edit** on any report
  4. Look for the scheduling options
  5. Set up a schedule (e.g. monthly, 1st of month)
  6. Confirm the schedule saves

### Save generated reports

- **Functional Module**: Financial Reporting
- **Feature Area**: Custom Reports
- **Sub-Feature**: Save generated reports
- **Status**: Pass
- **Description**: Generated reports (Trial Balance, Profit and Loss, Balance Sheet variants) can be saved and appear in the custom reports list for later access.
- **Steps to verify**:
  1. Navigate to `/app/customreports`
  2. Confirm saved reports are visible in the list (e.g. Trial Balance, Profit and Loss, Balance Sheet)
  3. Click on a saved report to verify it opens correctly

---

## Search & Header

### Initial search UI

- **Functional Module**: Search & Header
- **Feature Area**: Search
- **Sub-Feature**: Initial search UI
- **Status**: Pass
- **Description**: Global search bar in the header provides autocomplete, showing matching transactions, invoices, and contacts. Includes an "Advanced transactions search" link for deeper filtering.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. Click the **search bar** in the header
  3. Type a search term (e.g. a customer name or invoice number)
  4. Confirm autocomplete suggestions appear with matching results
  5. Verify the "Advanced transactions search" link is present

### Search functionality implementation

- **Functional Module**: Search & Header
- **Feature Area**: Search
- **Sub-Feature**: Search functionality implementation
- **Status**: Pass (partial)
- **Description**: Transaction search returns matching invoices and contacts. Report search works within the Standard Reports page search bar. Some search categories (e.g. Report search from the global bar) don't return results.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. Click the **search bar** in the header
  3. Type an invoice number or customer name — confirm results appear
  4. Navigate to `/app/standardreports`
  5. Type a report name in the report search bar — confirm matching reports appear in the dropdown

### Create Actions

- **Functional Module**: Search & Header
- **Feature Area**: Create Actions
- **Sub-Feature**: Create Actions
- **Status**: Pass
- **Description**: The Create (+) button in the left sidebar opens a full menu of create actions: Invoice, Payment link, Estimate, Sales receipt, Credit memo, Refund receipt, Delayed credit, Bill, Expense, Check, Purchase order, Vendor credit, Journal entry, Deposit, Transfer, Pay down credit card, Statement.
- **Steps to verify**:
  1. Go to `/app/homepage`
  2. Click the **Create (+)** button in the left sidebar (circle with plus icon, near the top)
  3. Confirm the full create menu appears with all action items listed
  4. Click any action item to verify it navigates to the correct form
  5. Press **Escape** or click outside to close the menu

---

## Summary

| Functional Module        | Verified | Total in taxonomy |
| ------------------------ | -------- | ----------------- |
| Accounts Payable         | 6        | 10                |
| Accounts Receivable      | 7        | 8                 |
| Banking & Reconciliation | 3        | 4                 |
| General Ledger           | 6        | 6                 |
| Financial Reporting      | 5        | 5                 |
| CustomerHub              | 0        | 1                 |
| Search & Header          | 3        | 3                 |
| **Total**                | **30**   | **37**            |
