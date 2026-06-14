# Manual Review Guide — Banking, GL, Mileage, Settings

Branch: `fix/expenses-module-review`
Base URL: `http://localhost:3001`

---

## Before you start

1. Make sure the dev server is running (`npm run dev` or whatever start command you use).
2. Open `http://localhost:3001/state-diff` in a browser tab — you'll come back to this after each test.
3. On the state-diff page, click **Reset All** to start from a clean baseline. You should see **33 total, 0 changed, 33 clean**.

> **Note:** Reset All may not clear every store reliably (observed with mileage and journal entries). If a store still shows as changed after Reset All, refresh the page and try again. As a last resort, you can manually reset a store by pasting its seed value from `data/defaultState.json` into localStorage via the browser console.

---

## ✓ Feature 1: Import Bank Transactions

**Navigation:** Sidebar → **Accounting** → **Bank Transactions** (or go to `/app/banking`)

**What to test:**

1. You'll see a "Bank transactions" page with bank logos (Citi, Chase, etc.) and an "Upload transactions" button.
2. Click any bank logo (e.g. **Citi**). A dialog should appear with:
   - A card illustration at the top
   - Green "IN THE WORKS" label
   - Title: **"This feature is on the way"**
   - A "Got it" button
3. Click **Got it** to close.
4. Click **Upload transactions** on the right side. This opens a full-page file upload screen.
5. Upload a CSV file with columns like `Date, Description, Amount, Type`. Example:
   ```
   Date,Description,Amount,Type
   05/01/2026,Office Supplies Purchase,-45.99,debit
   05/05/2026,Client Payment Received,1500.00,credit
   05/10/2026,Utility Bill,-120.50,debit
   ```
6. After upload, the system processes the file (uploading → classifying → extracting). You'll see a review screen saying something like "3 need closer look".
7. Click **"I'll review myself"** to see transaction details (amounts, descriptions, account info).
8. Click **Continue** → **Import** to complete the import.
9. You should return to the banking page with your imported transactions listed as "Pending" in the table.

**State-diff check:** Go to `/state-diff`. Only **"Qb Gym Transactions Store"** should be changed, showing your imported transactions as new records.

**Pass criteria:** Full import flow works end-to-end. Bank logos show the "In the works" card dialog. State-diff shows only transactions store changed.

---

## ✓ Feature 2: Reconcile Bank Account

**Navigation:** Sidebar → **Accounting** → then go to `/app/reconcile` directly.

**What to test:**

1. You'll see a "Reconcile" page with an account dropdown, beginning balance, ending balance, and ending date fields.
2. Select an account from the dropdown (e.g. **Accounts Receivable**).
3. Enter an **Ending balance** (e.g. `100.00`).
4. Set an **Ending date** (e.g. today's date).
5. Click **Start reconciling**. You should see the reconciliation active page with:
   - Payments / Deposits / All tabs
   - A "Difference" amount showing the gap between your ending balance and cleared items
   - Transaction rows from journal entries
6. Try clicking **Save for later** → should show ComingSoonDialog: **"Save for later is on the way"**
7. In the top-right corner, click **Summary**. A dialog should appear:
   - Green "IN THE WORKS" label
   - Title: **"Summary is on the way"**
8. Click **Got it**, then click **History by account**:
   - Title should be: **"History by account is on the way"**
9. Click **Got it** to close.

**State-diff check:** Only **"Qb Gym Reconcile Store"** should be changed.

**Pass criteria:** Reconciliation flow works end-to-end. Both Summary and History by account buttons show the "In the works" card dialog with their specific feature name.

---

## ✓ Feature 3: Categorize Uncategorized Transaction

**Navigation:** Same banking page (`/app/banking`)

**What to test:**

1. You need imported transactions first (complete Feature 1 first, or import a CSV).
2. On the banking page, you should see your imported transactions listed in a table.
3. Find a transaction row and look for a **"Select category"** dropdown or button in the category column.
4. Click it and select a category from the dropdown (e.g. **"Purchases"**).
5. The category should update in the UI immediately.

**Pass criteria:** Category can be selected and updates in the UI. Unimplemented actions show "In the works" dialog.

---

## ✓ Feature 4: Submit Mileage Reimbursement

**Navigation:** Sidebar → **Expenses** → **Mileage** (or go to `/app/mileage`)

**What to test:**

1. You'll see a "Mileage" page with tax year, potential deductions, trips logged chart, and vehicle info.
2. Verify seed data is present: 16.30 total business miles, 16 total miles, 4 vehicles, $0.725 per mile.
3. Click **Add trip** (green button, top-right). A panel slides in from the right with:
   - **Trip date**: defaults to today
   - **Distance (mi)**: number input
   - **Start point** / **End point**: address text fields
   - **Business / Personal**: radio buttons (Business selected by default)
   - **Business purpose**: dropdown with options like Client meeting, Site visit, Bank errand, etc.
   - **Vehicle**: dropdown (defaults to 2022 Toyota Camry)
   - **Round Trip**: toggle
4. Fill in:
   - **Distance**: e.g. `15.5`
   - **Start point**: e.g. `123 Main St, New York, NY`
   - **End point**: e.g. `456 Oak Ave, Brooklyn, NY`
   - **Business purpose**: select e.g. "Client meeting"
5. Click **Save**.
6. The trip should appear in the **Unreviewed** tab table with your date, location (start → end), distance, potential deductions, vehicle, and "Business" type badge.
7. The summary card should update (e.g. total business miles increased, potential deduction increased).

**State-diff check:** Only **"Qb Mileage Store"** should be changed, showing +1 new record with:
- `date` in ISO format (e.g. `"2026-05-28T12:00:00.000Z"`)
- `createdAt` in ISO format
- All fields properly typed

**Pass criteria:** Trip saves and appears in the list. State-diff shows only mileage store changed with ISO dates.

---

## ✓ Feature 5: Create New GL Account

**Navigation:** Sidebar → **Accounting** → **Chart of Accounts** (or go to `/app/chartofaccounts`)

**What to test:**

1. You'll see a table listing accounts (Accounts Receivable, Undistributed Funds, Checking, etc.) with columns: Number, Name, Account Type, Detail Type, QuickBooks Balance, Bank Balance, Action.
2. Note the current count (e.g. "1 - 17" in the pagination).
3. Click the green **New account** button (top-right). A drawer slides in from the right with fields:
   - **Account name** (required)
   - **Account number**
   - **Account type** dropdown (Bank, Accounts Receivable, Fixed Assets, Expenses, etc.)
   - **Detail type** dropdown (options change based on account type)
   - **Make this a subaccount** checkbox
   - **Description** textarea
   - **Lock account** toggle (Unlocked / Locked)
4. Fill in:
   - **Account name**: e.g. `Office Equipment`
   - **Account number**: e.g. `1500`
   - **Account type**: select e.g. "Fixed Assets"
   - **Detail type**: select e.g. "Furniture & Fixtures"
   - **Description**: e.g. `Office furniture and equipment`
5. Click **Save**.
6. The drawer closes and the new account appears in the table. Pagination should update (e.g. "1 - 18").

**State-diff check:** Only **"Qb Gym Accounts Store"** should be changed, showing +1 new record with:
- `createdAt` and `updatedAt` in ISO format
- `accountNumber` and `description` preserved as empty strings (not `undefined`) if left blank

7. **Also test the Batch actions dialog:**
   - Click **Batch actions** dropdown (top-left, next to the filter).
   - Click **Make inactive**. A dialog should appear:
     - Green "IN THE WORKS" label
     - Title: **"Batch actions is on the way"**
   - Click **Got it** to close.

8. **Also test Give feedback and Run report:**
   - Click **Give feedback** (top area) → should show **"This feature is on the way"**
   - Click **Run report** on any account row → should show **"This feature is on the way"**

**Pass criteria:** Account creation works end-to-end. "Batch actions", "Give feedback", and "Run report" show "In the works" dialog. State-diff shows only accounts store changed.

---

## ✓ Feature 6: Edit Account Category / Mapping

**Navigation:** Same Chart of Accounts page (`/app/chartofaccounts`)

**What to test:**

1. Find any account row in the table and click **View register** in the Action column.
2. You'll navigate to the account register page (e.g. `/app/register?accountId=acct-00121`) showing:
   - Account name and dropdown at the top
   - Ending balance
   - Table with columns: Date, Ref No./Type, Payee/Account, Memo, Due Date, Billed, Paid, Open Balance
   - Journal entry rows with JE numbers
3. Click **< Back to Chart of Accounts** to return.

**State-diff check:** No stores should change from just viewing the register.

**Pass criteria:** View register navigates correctly to the register page with transaction data. No state-diff noise from viewing.

---

## ✓ Feature 7: Create Manual Journal Entry

**Navigation:** Click the **Create (+)** button in the sidebar (top of sidebar) → **Other** → **Journal Entries** → **New Journal Entry**. Or go to `/app/journal`.

**What to test:**

1. You'll see the journal entry form with fields: Journal date, Journal no., and a table with columns (#, Account, Debits, Credits, Description, Name).
2. Fill in:
   - **Journal date**: Pick any date from the calendar
   - **Row 1**: Click the Account dropdown → select an account (e.g. "Checking") → type a debit amount (e.g. `100`)
   - **Row 2**: Click the Account dropdown → select a different account (e.g. "Sales") → the credit amount should auto-fill to `100`
3. Click **Save and close** (green button, bottom-right).
4. Now go to `/state-diff` and check:
   - Only **1 store** should be changed: "Qb Gym Journal Entries Store"
   - Click on it to expand — you should see +1 new record
   - The `txnDate` field should be in full ISO format like `"2026-05-28T00:00:00.000Z"` (not `"05/28/2026"`)
5. **Also test the dialogs on this page:**
   - Click the clock icon (History) in the header → should show **"History is on the way"**
   - Click **Feedback** button (top-right) → should show **"Feedback is on the way"**
   - Click **Make recurring** at the bottom → should show **"Make recurring is on the way"**
   - Click **Add attachment** or the paperclip area → should show **"Attachments is on the way"**

**Pass criteria:** Journal entry saves with ISO date. State-diff shows only journal entries store changed. All unimplemented buttons show "In the works" dialog with feature-specific titles.

---

## ✓ Feature 8: Reverse a Journal Entry

**Navigation:** From Chart of Accounts (`/app/chartofaccounts`), click **View register** on an account that has journal entries (e.g. Accounts Receivable). Click on a journal entry row (e.g. JE-00349) to open it in the journal form.

**What to test:**

1. You'll see the journal entry form pre-filled with the existing entry's data (date, accounts, debits, credits, memo).
2. At the bottom center, click **Reverse**.
3. A new journal entry form appears with:
   - Journal number with "R" suffix (e.g. **JE-00349R**)
   - Journal date set to the **first day of the next month** from the original entry (e.g. original is 03/23/2026 → reversed is **04/01/2026**)
   - Debits and credits **swapped** from the original
   - Memo: "Reverse of JE [original number] - [original memo]"
4. Click **Save** to save the reversed entry.

**State-diff check:**
- Only **"Qb Gym Journal Entries Store"** should change, showing +1 new record.
- The reversed entry's `txnDate` should be in ISO format (`2026-04-01T00:00:00.000Z`), not `MM/DD/YYYY`.
- `createdAt` and `updatedAt` should also be ISO format.

**Pass criteria:** Reversed entry has ISO-format date, correct first-of-next-month date, swapped debits/credits. Clean state-diff (only journal entries store).

---

## ✓ Feature 9: Lock Accounting Period

**Navigation:** Click the **Settings gear icon** (top-right of the app header) → you'll land on the Settings page. Click **Advanced** in the left sidebar (it may already be selected). Or go to `/app/accountsettings`.

**What to test:**

1. Scroll down to the **Accounting** section (4th section down, after Accounting method, Customer label, Communications with Intuit). You'll see: First month of fiscal year, First month of tax year, Accounting method, Close the books.
2. Click the **Edit** button on the right side of the **Accounting** section header. (There are multiple Edit buttons — it's the 4th one down.)
3. The section expands into an edit form with dropdowns and a **Close the books** toggle switch.
4. Toggle **Close the books** ON (click the switch — it turns green).
5. New fields appear: **Closing date** (a date picker) and **Allow changes after viewing a warning** (dropdown).
6. Click the **calendar icon** next to Closing date → pick any date (e.g. today).
7. Click **Save** (green button).
8. The section should collapse back to read view showing "Closing date: 05/28/2026" (or whatever you picked).
9. Go to `/state-diff`:
   - Only **1 store** should be changed: "Qb Gym Companies Store"
   - The diff should show only the `closeTheBooks`, `closingDate`, and `allowChangesType` fields changed on one company record.
10. **Also test the dialogs on this page:**
    - Click **Give feedback** (top-right of Settings header) → should show **"Feedback is on the way"**
    - Click the **? help icon** (top-right) → should show **"Help is on the way"**
    - Click **Edit** on any other section (e.g. "Accounting method", "Customer label") → should show **"This feature is on the way"**
11. **To undo:** Click Edit on Accounting section again, toggle Close the books OFF, click Save.

**Pass criteria:** Close the books toggle works end-to-end. State-diff shows only companies store changed. Settings dialogs show "In the works" with feature-specific titles.

---

## After all tests

1. Go to `/state-diff`.
2. If you made changes during testing (journal entries, trips, accounts, lock period), you can click **Reset All** to restore the clean baseline.
3. If Reset All doesn't fully clear a store, manually remove the added record via browser console or refresh and retry.
4. Verify **33 total, 0 changed, 33 clean**.

## Quick reference — what changed in this branch

| What | Why |
|------|-----|
| Journal dates now save as ISO (`2026-05-28T00:00:00.000Z`) | Was saving as `05/28/2026`, causing format noise in state-diff |
| Reversed journal entry dates also save as ISO | `getFirstDayOfNextMonth()` was returning `MM/DD/YYYY` |
| Accounts store uses `??` instead of `\|\|` | Was converting empty strings to `undefined` on save |
| 10 files swapped from `FeatureUnavailableDialog` to `ComingSoonDialog` | Old dialog was plain text with OK button; new one has card illustration and "In the works" branding |
| `defaultState.json` regenerated | Seed dates normalized so baseline is clean |

## Test results summary

| Feature | Functional Flow | ComingSoonDialogs | State-Diff |
|---------|----------------|-------------------|------------|
| 1. Import Bank Transactions | CSV upload → review → import | Bank logos: "This feature is on the way" | Only transactions store |
| 2. Reconcile Bank Account | Account → balance → reconcile | Summary, History by account, Save for later | Only reconcile store |
| 3. Categorize Transaction | Select category on transaction | Same as Feature 1 | N/A (shared store) |
| 4. Mileage Add Trip | Fill form → save → appears in list | N/A (page is fully implemented) | Only mileage store, ISO dates |
| 5. Create New GL Account | Fill form → save → appears in table | Batch actions, Give feedback, Run report | Only accounts store, ISO dates |
| 6. Edit Account / View Register | Navigate to register, view entries | Give feedback, Run report | No changes (read-only) |
| 7. Create Journal Entry | Fill accounts/amounts → save | History, Feedback, Make recurring, Attachments | Only journal entries store, ISO dates |
| 8. Reverse Journal Entry | Click Reverse → new entry with "R" suffix | Same as Feature 7 | Only journal entries store, ISO dates |
| 9. Lock Accounting Period | Toggle Close the Books → save | Feedback, Help, Edit on other sections | Only companies store |
