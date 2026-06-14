---
title: "Manual Review: fix-customer-creation"
tags:
  - ledgerpilot
  - testing
  - manual-review
  - customers
created: 2026-05-28T23:13:00+03:00
branch: fix-customer-creation
pr: 252
---

# Manual Review — fix-customer-creation (#252)

Branch: `fix-customer-creation`
Base URL: `http://localhost:3001`

---

## Before you start

1. Make sure the dev server is running.
2. Open `http://localhost:3001/state-diff` in a separate tab.
3. Click **Reset All** to start from a clean baseline.

---

## Test 1: Create New Customer (all fields)

**Navigation:** Sidebar → **Sales & Get Paid** → **Customer Hub** → **Customers & Leads** (or `/app/customers?jobId=customers&tab=customers`)

**What to test:**

1. Verify the summary cards at the top show **dynamic values** (Estimates, Unpaid income, Overdue, Open invoices & credits, Recently paid) — not all zeros or all identical.
2. Click the green **New customer** button. A panel slides in from the right.

**Fill in all fields:**

3. **Name and contact section:**
   - **Title**: `Mr`
   - **First name**: `John`
   - **Middle name**: `Q`
   - **Last name**: `Public`
   - **Suffix**: `Jr`
   - **Company name**: `Acme Corp`
   - **Customer display name ***: `John Q Public Jr` — verify this is a **plain text input** with no dropdown chevron
   - **Email**: `john@acme.com`
   - **Phone number**: `555-0100`
   - **Cc**: `cc@acme.com`
   - **Bcc**: `bcc@acme.com`
   - **Mobile number**: `555-0101`
   - **Fax**: `555-0102`
   - **Other**: `555-0103`
   - **Website**: `https://acme.com`
   - **Name to print on checks**: `J Q Public`

4. **Addresses section** (scroll down or click the Addresses tab icon):
   - **Billing address — Street address**: `123 Main St`
   - **City**: `Springfield`
   - **State**: `IL`
   - **ZIP code**: `62701`
   - **Country**: `US`

5. Click **Save**.
6. A toast should appear saying **"Customer is added"**.
7. The customer should appear at the top of the table (sorted newest-first).

**State-diff check:** Go to `/state-diff`.
- Only **"Qb Gym Sales Customers Store"** should be changed, showing +1 new record.
- Expand the diff and verify **all fields** are present:
  - `displayName`: `"John Q Public Jr"`
  - `givenName`: `"John"`, `middleName`: `"Q"`, `familyName`: `"Public"`
  - `title`: `"Mr"`, `suffix`: `"Jr"`
  - `companyName`: `"Acme Corp"`
  - `primaryEmailAddr`: `"john@acme.com"`
  - `ccEmailAddr`: `"cc@acme.com"`, `bccEmailAddr`: `"bcc@acme.com"`
  - `primaryPhoneFreeFormNumber`: `"555-0100"`
  - `mobileFreeFormNumber`: `"555-0101"`, `faxFreeFormNumber`: `"555-0102"`, `alternatePhoneFreeFormNumber`: `"555-0103"`
  - `webAddr`: `"https://acme.com"`
  - `printOnCheckName`: `"J Q Public"`
  - `notes`: `""` (empty string, not missing)
  - `billAddr.line1`: `"123 Main St"`, `billAddr.city`: `"Springfield"`, `billAddr.country_sub_division_code`: `"IL"`, `billAddr.postal_code`: `"62701"`, `billAddr.country`: `"US"`
  - Address keys must be **snake_case** (`line1`, `city`, `country_sub_division_code`, `postal_code`, `country`), NOT PascalCase

- [ ] All 22+ form fields persist correctly
- [ ] Address uses snake_case keys
- [ ] No dropdown chevron on display name
- [ ] State-diff shows only customers store changed
- [ ] Toast appears on save

---

## Test 2: Edit Existing Customer (address roundtrip)

**Navigation:** Open any invoice from `/app/invoices`, click the customer name/info area to open the edit panel.

**What to test:**

1. The **billing address fields** should be pre-filled with the customer's saved address — not blank.
2. Edit the street address (e.g., change to `456 Oak Ave`).
3. Click **Save**.
4. Reopen the same customer — the updated address should persist.

- [ ] Address fields pre-populate on edit (not blank)
- [ ] Edited address persists after save and reopen

---

## Test 3: Select All / Deselect All

**Navigation:** Same Customers & Leads page.

**What to test:**

1. Click the **header checkbox** (top-left of the table, above all rows).
2. All visible row checkboxes should become checked.
3. Click the header checkbox again.
4. All row checkboxes should become unchecked.
5. Check a few individual rows, then click the header checkbox — all should check. Click again — all should uncheck.

- [x] Header checkbox selects all visible rows
- [x] Header checkbox deselects all visible rows
- [x] Individual row checkboxes work independently

---

## Test 4: Summary Cards (dynamic values)

**Navigation:** Same Customers & Leads page.

**What to test:**

1. The 5 summary cards at the top should show values computed from store data:
   - **Estimates** — count and total from estimates store
   - **Unpaid income** — sum of invoice balances > 0
   - **Overdue** — sum where `dueDate < today` and `balance > 0`
   - **Open invoices & credits** — count of open invoices + credit memos
   - **Recently paid** — sum of payments in last 30 days
2. The values should not be hardcoded zeros or placeholder amounts.

> [!note]
> With the current seed data, Unpaid income / Overdue / Open invoices & credits may show the same value ($475.86) because all unpaid invoices are past due and there are no open credits. This is a seed data limitation, not a bug.

- [ ] Cards show non-zero values from actual store data
- [ ] Labels include counts (e.g. "54 unpaid income")

---

## Test 5: ComingSoonDialog on Customers page

**Navigation:** Same Customers & Leads page.

**What to test:**

1. Click **Customer types** button (top-right, next to New customer):
   - Green "IN THE WORKS" label
   - Title: **"Customer types is on the way"**
   - Click **Got it** to close.
2. Click **Check out the new view with filters** link:
   - Title: **"New view with filters is on the way"**
3. Click the **Leads** tab:
   - Title: **"Leads is on the way"**
4. Click the **print icon** (top-right toolbar):
   - Title: **"Print is on the way"**
5. Click the **export icon** (top-right toolbar):
   - Title: **"Export is on the way"**
6. Click the **settings/gear icon** (top-right toolbar):
   - Title: **"Settings is on the way"**

- [ ] All 6 buttons show ComingSoonDialog with illustrated card and "IN THE WORKS" label
- [ ] Each dialog shows the correct feature-specific title
- [ ] Not the old plain FeatureUnavailableDialog

---

## Test 6: Invoice Editor — Add Customer toast

**Navigation:** Go to `/app/invoice` (create a new invoice), click **+ Add customer**, fill the customer panel, and save.

**What to test:**

1. After saving, a single toast should appear: **"Customer saved"**.
2. There should NOT be a duplicate/flickering toast ("Customer is added" followed by "Customer saved").

- [ ] Single toast appears
- [ ] No duplicate or flickering toast

---

## After all tests

1. Go to `/state-diff`.
2. Click **Reset All** to restore the clean baseline.
3. Verify **0 changed** stores.

## What changed in this branch

| What | Why |
|------|-----|
| Customer addresses normalized to snake_case | `normalizeCustomerAddress()` output PascalCase but panel reads snake_case — blank address fields on edit |
| All 22 customer form fields now persist | `onSave` mapped only 8 of 22 fields; address, cc/bcc, mobile, fax, website, notes were dropped |
| Customer summary cards are dynamic | Were hardcoded constants; now computed from invoice, payment, estimate, credit memo stores |
| Display name dropdown removed | `select` prop showed a non-functional chevron |
| Dead toast removed from invoice editor | `showToast("Customer is added")` immediately overwritten by `showToast("Customer saved")` |
| `FeatureUnavailableDialog` → `ComingSoonDialog` | 12 triggers on Customers page now use the illustrated card dialog |
| Header checkbox wired to select/deselect all | Was a static uncontrolled checkbox with no handler |
| `defaultState.json` updated | Customer address keys aligned to snake_case to eliminate 96 false-positive diffs |
