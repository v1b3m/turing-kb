---
title: ReturnMax Test Batch — Prompts
batch_id: 2a9adc7f-da16-41f1-a9b7-18480d35fcfc
batch_url: https://staging-rl-gym-harness.turing.com/verification/batches/2a9adc7f-da16-41f1-a9b7-18480d35fcfc
prompt_count: 45
source: staging-rl-gym-harness
scraped: 2026-06-15 (latest versions; batch is actively re-pinned by others)
tags: [turing, rlgym, returnmax, verification-batch]
---

# ReturnMax Test Batch — Prompts

> [!info] Batch
> - **Name:** ReturnMax Test Batch
> - **Batch ID:** `2a9adc7f-da16-41f1-a9b7-18480d35fcfc`
> - **Batch page:** https://staging-rl-gym-harness.turing.com/verification/batches/2a9adc7f-da16-41f1-a9b7-18480d35fcfc
> - **Prompts:** 45 (testing **latest** versions)
> - **Default gym:** Returnmax V1 — https://returnmax-v1.rlgym.turing.com
> - **Data URL:** https://github.com/turing-rlgym/returnmax/blob/ots_v1/data/seed/

> [!warning] Batch is a moving target
> Versions are actively re-pinned by other reviewers. Prompt text/metadata here is a snapshot from 2026-06-15. Re-fetch `/api/v1/verification/prompts/{uuid}` before a run if in doubt.

## Testing

Per-prompt test notes live in `returnmax-test-batch/` — frontmatter is the source of truth for multi-agent runs. Full procedure + reset recipe in [[_AGENT-PROTOCOL]].

**Target:** `http://localhost:3001` (the `returnmax-1` repo). **Clean state per prompt:** replicate the app's `resetAll` (fetch `/api/v1/env/defaultState` → rewrite `rm_*` localStorage keys → reload); the `/state-diff` "Reset All" button is Shadow-DOM and not `eval`-reachable. After running, use the state-diff logic as the verification oracle.

### Kickoff prompt (paste into a fresh session)

> Run the ReturnMax test batch loop. Read your memory (`MEMORY.md` → returnmax-test-batch-workflow) and `turing-kb/returnmax-test-batch/_AGENT-PROTOCOL.md` first, then follow it exactly. Loop, lowest task_id first:
> 1. In `turing-kb/returnmax-test-batch/`, pick the lowest-numbered note with `status: untested` and empty `owner`. Claim it (`owner: <your-id>`, `status: in-progress`, `claimed_at`, `updated_at` in UTC+3), then re-read to confirm you won the race.
> 2. Reset `http://localhost:3001` to clean state with the reset recipe in the protocol; verify clean (`rm_tax_return.state.w2s[0].wages === 92500.75`).
> 3. Drive the app via the chrome-cdp `turing` profile (`CDP_PROFILE=turing`, absolute cdp.mjs path) to perform the note's **Prompt** exactly. Trust prompt text + actual seed over `prep_work` when they conflict, and note any discrepancy.
> 4. Verify the result against the seed (state-diff logic): do the changed slices/fields match intent? any unintended mutations? Record under `## Findings`.
> 5. No bug → `status: passed`. Bug → log under `## Bugs found` (bump `bug_count`), `status: bug-found`, fix in `returnmax-1` (micro-commits), open a PR, set `fix_pr` + `status: fix-pushed`, re-test clean → `verified`.
> 6. Update `updated_at` on every change. After ~5 notes, checkpoint and tell me to `/clear` so a fresh session resumes from the `.base` "Available to claim" view.
> Start with `returnmax-medium-001`.

![[returnmax-test-batch.base]]

## Index

| # | Task ID | Ver | L3 | Complexity | Link |
|---|---------|-----|----|-----------|------|
| 1 | `returnmax-medium-001` | v3 | Report W-2 Wages from One or More Employers | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/4c19e586-7877-439a-8c0e-fceb39b180fd/task) |
| 2 | `returnmax-medium-002` | v2 | Upload, Tag, and Manage Tax Documents in the Documents Hub | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/eaf88916-d448-453a-a430-58e8705d29d1/task) |
| 3 | `returnmax-medium-003` | v2 | Enter Personal Profile, SSN, and Contact Details | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/8af51809-5b65-448f-ae33-51efb52287ac/task) |
| 4 | `returnmax-medium-004` | v1 | Compare Standard and Itemized Deductions and Select the Best Option | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/618aa522-ae74-4d66-a4f9-4e33ececa7a4/task) |
| 5 | `returnmax-medium-005` | v1 | Select Returns, Sign Electronically, Submit, and Track Status | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/a0eec452-6f53-44f0-91b5-b6909081f958/task) |
| 6 | `returnmax-medium-006` | v2 | Add and Qualify Dependents Through the Dependent Wizard | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/6ab42127-f1ec-4350-875a-7a1d82c17626/task) |
| 7 | `returnmax-medium-007` | v1 | Enter Investment Income from 1099-INT, 1099-DIV, and 1099-B | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/ea2c1d6c-1d5a-4a93-977a-1a2d7205f7e4/task) |
| 8 | `returnmax-medium-008` | v1 | Run the Final Error Check and Resolve Issues in the Fix Hub | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/ed5b1f0c-8da4-4b9c-b8e4-ed4814aed5e8/task) |
| 9 | `returnmax-medium-009` | v1 | Browse Product Tiers and Start a New Return | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/0292ca63-336c-4452-89e7-02d23dc98391/task) |
| 10 | `returnmax-medium-010` | v1 | Claim Child Tax Credit, EITC, Child Care, and Education Credits | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/dcfca7d3-f92d-4f37-9df4-3b51b5bb9e75/task) |
| 11 | `returnmax-medium-011` | v1 | Configure Notification Preferences and App Settings | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/795de7d2-462a-43bf-8384-f50e182ace46/task) |
| 12 | `returnmax-medium-012` | v3 | Enter Mortgage Interest, Property Taxes, and SALT Deductions | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/850c4b06-e9a1-4c72-9e58-47b7cb72586f/task) |
| 13 | `returnmax-medium-013` | v1 | Start a Live Chat or Screen Share Session with a Tax Expert | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/8c43f17b-0b13-45ba-9756-655e08508694/task) |
| 14 | `returnmax-medium-014` | v1 | Prepare State Return with Auto-Filled Federal Data | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/427d5181-74a5-4405-9b7d-19440ebf26f8/task) |
| 15 | `returnmax-medium-015` | v1 | Select Tax Year, Resume or Start a Return, and Review Tax Law Updates | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/4b3b42b2-7ba1-4729-ae58-071877eaad2c/task) |
| 16 | `returnmax-medium-016` | v1 | Navigate the AMT Wizard and Calculate Alternative Minimum Tax | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/a4b9520b-eb02-4e02-9d8d-333a1fe850d6/task) |
| 17 | `returnmax-medium-017` | v1 | Verify Personal Info and Complete the Federal Review Checklist | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/d982bd68-04cb-4e3a-bcef-413e57f28ed6/task) |
| 18 | `returnmax-medium-018` | v1 | Enter Schedule C Business Income and Self-Employment Expenses | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/1c6fbebc-0326-4b22-a58e-a2b9d2b9ca54/task) |
| 19 | `returnmax-medium-019` | v1 | Review Order, Pay for ReturnMax, and Apply a Service Code | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/1c6bea5e-ded5-46ea-b5d3-c4cd311bb7b7/task) |
| 20 | `returnmax-medium-020` | v1 | Add Charitable Donations, Medical Expenses, and Other Itemized Items | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/95448dad-3c0f-4729-9f6b-5db25a630550/task) |
| 21 | `returnmax-medium-021` | v2 | Import Documents from Financial Institutions via Linked Accounts | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/0b0251c7-bf1e-4db8-8eae-6946b491cf64/task) |
| 22 | `returnmax-medium-022` | v1 | Handle Extension Filing, Underpayment Penalty, and Estimated Tax Planning | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/58fccc5f-fac4-4af1-a1d8-f7153bf2f1ea/task) |
| 23 | `returnmax-medium-023` | v1 | Set Filing Status and Compare MFJ vs. MFS | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/e4107462-7d21-4641-8d8e-4771187ab745/task) |
| 24 | `returnmax-medium-024` | v1 | Request Expert Review, MAX Protection, or Full-Service Tax Filing | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/9413998f-0e39-4b09-a5cd-a6ba98c35497/task) |
| 25 | `returnmax-medium-025` | v2 | Search Help, Bookmark Sections, Generate Tax Vouchers, Print Documents, and Review Fees | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/5fe4dc70-f2ac-47b9-bb5f-ecf8bea8b836/task) |
| 26 | `returnmax-medium-026` | v1 | Register an Account or Sign In | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/01b555b7-3847-4b38-ac38-e6e34313bb8c/task) |
| 27 | `returnmax-medium-027` | v1 | Report Retirement Distributions, Unemployment, and Other 1099 Income | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/465f7806-466c-4f3a-9e52-37a3767bd785/task) |
| 28 | `returnmax-medium-028` | v1 | Claim Student Loan Interest, IRA, HSA, and Educator Deductions | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/1bd7a6fd-f6aa-4693-b827-35e26ba8d514/task) |
| 29 | `returnmax-medium-029` | v1 | Review State Return and Advance to Final Review | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/0a974e19-d218-480b-a946-f86387815683/task) |
| 30 | `returnmax-medium-030` | v1 | Set Up Direct Debit, Split Refund, or Request a Refund Advance | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/220ac651-ca03-4d6c-ab8e-7bd7278cc555/task) |
| 31 | `returnmax-medium-031` | v1 | Manage Account Settings, Order Summary, and Download Tax Return | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/eed1723e-109b-43ee-93b8-f402405573a2/task) |
| 32 | `returnmax-medium-032` | v1 | File Amended Return, Change of Address, and Other IRS Forms | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/6c38ad5f-57ca-4eef-8ea7-0c8f410d48a9/task) |
| 33 | `returnmax-medium-033` | v1 | Enter State Deductions, Credits, and Allocate Multi-State Income | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/0c2a4c23-d0c1-43e7-98e2-0a901bd92d57/task) |
| 34 | `returnmax-medium-034` | v1 | Complete Guided Onboarding with Situation Check | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/f7e55a11-e8dd-413f-bd9c-ecca2cf04b77/task) |
| 35 | `returnmax-medium-035` | v1 | Declare Uncommon Situations and Military Status | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/adb19765-16cf-4f00-b9b9-54b5ffb4e92b/task) |
| 36 | `returnmax-medium-036` | v1 | Maximize Self-Employment Deductions Including Home Office and Vehicle | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/611b25f4-10ed-41ff-ad05-6a831e44ef8f/task) |
| 37 | `returnmax-medium-037` | v1 | Report Rental, Royalty, and Other Investments Income | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/483b4f6b-3ca4-4e40-a04e-138570a62df8/task) |
| 38 | `returnmax-medium-038` | v1 | Claim Energy, EV, Adoption, Foreign Tax, and Other Credits | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/2fdd7bf9-c7b6-4afb-befd-e0156b1afcf7/task) |
| 39 | `returnmax-medium-039` | v1 | Review Accuracy Guarantee and Add Audit Defense | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/7343d34c-be02-4f7b-a741-3662bf4f9159/task) |
| 40 | `returnmax-medium-040` | v1 | Manage Identity Protection PIN and Identity Theft Claims | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/30860c43-0b15-41fc-9005-42fef215f1c6/task) |
| 41 | `returnmax-medium-041` | v1 | Submit by Mail, File an Extension, or Amend a Filed Return | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/d963d18c-2c9d-4306-b04f-eb8eb9d41da1/task) |
| 42 | `returnmax-medium-042` | v1 | Full Simple W-2 Return: Onboarding Through E-File Confirmation | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/8b20c82e-cc37-40d2-9ce6-f7eed506abd6/task) |
| 43 | `returnmax-medium-043` | v1 | Full Self-Employed Return: Schedule C, Deductions, Expert Assist, and Payment | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/e459232f-915e-4092-8ad2-b1c50092ea97/task) |
| 44 | `returnmax-medium-044` | v1 | Full Investor Return: Linked Account Import, Capital Gains, and Itemized Deductions | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/f3b8de54-3e08-42a3-bd3a-88bb705abb62/task) |
| 45 | `returnmax-medium-045` | v1 | Complete Multi-State and Prior-Year Import Return: From Data Import to State Filing | Medium | [task](https://staging-rl-gym-harness.turing.com/verification/prompts/551654a3-6246-4cbd-93fd-174ebde17a0f/task) |

---

## Prompts

### 1. `returnmax-medium-001` (v3)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/4c19e586-7877-439a-8c0e-fceb39b180fd/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-001]]
- **Taxonomy:** Federal Income › Wages & Compensation › Report W-2 Wages from One or More Employers
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 3/45/4

**Prompt:**

> On your current Documents Hub screen, locate the file row for '2025-luma-labs-w2.pdf', click its action icon, and rename the file to 'Luma Labs W-2 2025 Final'. On your current Your income and expenses page, locate the Wages and Salaries (W-2) row item for Luma Labs LLC and click its corresponding blue Add/Edit button. On the W-2 summary table, click Edit next to the Luma Labs entry to open the form field panel. Execute two specific modifications: change your Box 1 wages from $92,500.75 to $95,000.00, and update your Box 2 federal tax withheld from $13,450.25 to $14,000.00 and select yes for the question "Did Luma Labs LLC provide on-site child care?".

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) exists for tax year 2025 with no W-2 income currently entered. Employer 'Luma Labs LLC' is the source employer with EIN '74-1234567'. Valid Box 1 wages: $92,500.75; Box 2 federal tax withheld: $13,450.25; Box 16 CA state wages: $92,500.75; Box 17 CA state tax withheld: $6,125.40. Document '2025-luma-labs-w2.pdf' exists in the Documents Hub with status 'reviewed'. The refund estimate pill in the sticky header updates in real-time after each W-2 save. The renamed target filename 'Luma Labs W-2 2025 Final' does not yet exist in the Documents Hub.

---

### 2. `returnmax-medium-002` (v2)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/eaf88916-d448-453a-a430-58e8705d29d1/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-002]]
- **Taxonomy:** Document Management › Document Upload & Tagging › Upload, Tag, and Manage Tax Documents in the Documents Hub
- **Persona:** Returning User | **Complexity:** Medium | **Est. hops/steps/actions:** 3/38/3

**Prompt:**

> Three tax documents came in over the last few weeks and none of them are tagged properly in the hub. Upload the file 'san-francisco-marin-food-bank-2025-receipt.pdf' which is sitting untagged in the Documents tab, tag it as a cypto transactions, then rename the 1099-B from Fidelity Investments ('2025-fidelity-1099-b.pdf') to 'Fidelity 1099-B 2025 Reviewed', and delete the Coinbase crypto statement '2025-coinbase-gain-loss.csv' since that income was entered manually.

**Prep work:**

> Document 'san-francisco-marin-food-bank-2025-receipt.pdf' exists in the Documents Hub with no tag assigned. Document '2025-fidelity-1099-b.pdf' exists with form_type 'form_1099_b' and status 'reviewed'. Document '2025-coinbase-gain-loss.csv' exists with form_type 'crypto_statement' and status 'parsed'. All three documents belong to user Maya Srinivasan (tax year 2025). The Documents Hub shows uploaded/imported tax documents in a sortable table with kebab menus for edit and delete actions. Tag options include charitable donation receipt among the 21 available tax form tag options.

---

### 3. `returnmax-medium-003` (v2)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/8af51809-5b65-448f-ae33-51efb52287ac/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-003]]
- **Taxonomy:** Personal Information › Filer Identity & Contact › Enter Personal Profile, SSN, and Contact Details
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/42/4

**Prompt:**

> Getting the personal info section locked down before the deadline. On the Profile page, update the occupation from whatever is currently listed to 'Software Engineer' using the searchable combobox. Then on the Contact page, confirm the mailing address is set to '2148 Valencia Street, Apt 3, San Francisco, CA 94110' and the phone is '415-210-4837', update either if they differ. Finally, open the SSN page and make sure the entry for '321-54-9876' is saved correctly by toggling the visibility to verify the format, then save.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress for tax year 2025. Profile page (/my-info/profile) exists with occupation field accepting a searchable combobox; 'Tax Consultant' is a valid occupation option. Contact page (/my-info/contact) exists for entering mailing address, phone, and email; valid target address is '2148 Valencia Street, Apt 3, San Francisco, CA 94110' and valid phone is '415-210-4837'. SSN page (/my-info/ssn) exists with a masked input and visibility toggle; SSN '321-54-9876' is the filer's SSN. All fields persist to the personalInfo object in the tax return store.

---

### 4. `returnmax-medium-004` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/618aa522-ae74-4d66-a4f9-4e33ececa7a4/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-004]]
- **Taxonomy:** Deductions & Credits › Standard vs. Itemized Decision › Compare Standard and Itemized Deductions and Select the Best Option
- **Persona:** Investor / Advanced Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 3/35/3

**Prompt:**

> Before locking in the deduction strategy, need to compare both paths. On the Standard vs. Itemized overview page, review the auto-calculated Standard Deduction for Head of Household filing status and note the dollar amount. Then enter the mortgage interest from 'Rocket Mortgage' ($11,840.29) and property tax ($6,240.18) to build the itemized side. Once both amounts are visible, check which option ReturnMax recommends and select it, confirming the refund pill in the header updates after the choice is saved.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress; filing status is 'head_of_household'. Standard Deduction overview page (/deductions) exists and auto-calculates the standard deduction based on filing status. Itemized deduction entry pages exist for mortgage interest and property tax. Deduction item for 'Rocket Mortgage' exists with mortgage interest $11,840.29 and property tax $6,240.18 available as valid entry values. ReturnMax highlights the recommended choice with a dollar difference. The refund pill in the sticky header updates immediately after the deduction decision is saved.

---

### 5. `returnmax-medium-005` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/a0eec452-6f53-44f0-91b5-b6909081f958/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-005]]
- **Taxonomy:** Filing & Payment › E-File Submission & Confirmation › Select Returns, Sign Electronically, Submit, and Track Status
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/48/4

**Prompt:**

> The return is ready to go out. On the Ready to File page, check both the Federal and State (California) return checkboxes. Move to the Verify page and enter the prior year AGI of $96,720.52 and set a new 5-digit signing PIN. Then on the Submit page click 'Transmit returns now'. Once the Confirmation page loads, note the federal submission ID and check that a California refund summary card is showing, then navigate to the Status page to confirm the timeline has advanced past 'Submitted'.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is fully prepared and has passed Final Review. The Ready to File page (/file/ready-to-file) exists with Federal and State (California) return checkboxes; Continue is disabled until at least one is selected. The Verify page (/file/verify) collects prior year AGI; valid prior year AGI is $96,720.52. A new 5-digit signing PIN must be set; prior year AGI field and signing PIN field both have validation. The Submit page (/file/submit) has a 'Transmit returns now' button. The Confirmation page (/file/confirmation) shows federal and state submission IDs, submission date, and refund summary cards. The Status page (/file/status) shows a 5-step timeline: Submitted, Accepted, Processing, Approved, Refund Sent.

---

### 6. `returnmax-medium-006` (v2)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/6ab42127-f1ec-4350-875a-7a1d82c17626/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-006]]
- **Taxonomy:** Personal Information › Dependents › Add and Qualify Dependents Through the Dependent Wizard
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/55/5

**Prompt:**

> Adding Stacy to the return before credits get finalized. From My Info, start a new dependent for Stacy Rivera (DOB: September 12, 2018, SSN: 234-56-7891, relationship: daughter). Work through the wizard: select 'my_child' as relation type, confirm she is a US citizen who lived with me all year, that the other parent is not claiming her, and that she spent most nights at my home. Complete the qualified-child determination and confirm she appears on the dependents landing page as a qualifying child.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress for tax year 2025; filing status is 'head_of_household'. No dependent named 'Ella Rivera' currently exists on the return. The dependent wizard begins from My Info and Dependents with an 'Add a dependent' button. Wizard steps include: relationship selection, child SSN input (valid SSN: 234-56-7890), child info (first name: Ella, last name: Rivera, DOB: 2017-09-12), child situation (US citizen, lived in home all year), and additional info pages. Valid relationship: 'my_child' (daughter). 'Most nights' answer: 'my_home'. 'Other parent claiming': false. The dependents landing page surfaces the dependent after wizard completion.

---

### 7. `returnmax-medium-007` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/ea2c1d6c-1d5a-4a93-977a-1a2d7205f7e4/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-007]]
- **Taxonomy:** Federal Income › Investment & Retirement Income › Enter Investment Income from 1099-INT, 1099-DIV, and 1099-B
- **Persona:** Investor / Advanced Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/50/4

**Prompt:**

> The investment income section has three sources that need to be entered before the return can go to review. Under Federal income, enter interest income from Chase Bank ($148.36, no federal tax withheld) from the 1099-INT. Then add the long-term capital gain from Fidelity Investments: VTI ETF shares, proceeds $8,450.64, cost basis $7,900.10, sold August 20 2025, acquired May 10 2023, basis reported to IRS. Finally record the BTC crypto sale from Coinbase: proceeds $3,120.00, cost basis $2,440.00, long-term, sold June 14 2025, basis not reported to IRS. Check that the net investment gain updates in the refund summary.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress for tax year 2025. No investment income has been entered yet. 1099-INT income: payer 'Chase Bank', amount $148.36, no federal tax withheld. 1099-B sale: 'VTI ETF shares' from Fidelity Investments, proceeds $8,450.64, cost basis $7,900.10, date acquired 2023-05-10, date sold 2025-08-20, long-term gain $550.54, basis reported to IRS: true. Crypto sale: 'BTC sale' from Coinbase, proceeds $3,120.00, cost basis $2,440.00, date acquired 2024-02-10, date sold 2025-06-14, long-term gain $680.00, basis reported to IRS: false. All three income types are accessible under Federal, Income.

---

### 8. `returnmax-medium-008` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/ed5b1f0c-8da4-4b9c-b8e4-ed4814aed5e8/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-008]]
- **Taxonomy:** Review & Error Correction › Final Error Check & Issue Resolution › Run the Final Error Check and Resolve Issues in the Fix Hub
- **Persona:** Expert-Assisted Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 3/40/3

**Prompt:**

> The final error check just finished and it flagged two items that need to be resolved before submitting. On the Results page, open the Fix Hub and address the first flagged item by navigating to its dedicated fix page and completing the required fields. Once that item shows a 'fixed' badge, go back to the Fix Hub and resolve the second item the same way. After both items are marked fixed, advance past the 'Almost Done' confirmation to reach the 'Review Results' state so the return is cleared to file.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) has completed the error check progress bar and is on the Results page (/final-review/results). The Results page shows 2 issues flagged. The Fix Hub (/final-review/fix-hub) lists the 2 actionable items with 'needs-fix' status badges and Fix buttons. At least one item routes to a dedicated fix page (e.g., /fix/medical-expenses or /fix/foreign-accounts). Each fix page has required fields that must be completed for the item to show a 'fixed' badge. The 'Almost Done' and 'Review Results' states are shown after all items are resolved.

---

### 9. `returnmax-medium-009` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/0292ca63-336c-4452-89e7-02d23dc98391/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-009]]
- **Taxonomy:** Onboarding & Product Selection › Product Discovery & Sign-Up › Browse Product Tiers and Start a New Return
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 3/35/3

**Prompt:**

> First time filing with ReturnMax and need to pick the right product. From the landing page, view the three filing options and navigate to the product comparison page to see all four tiers side by side. Review the feature checkmarks for the Deluxe tier ($69 federal) and the Premier tier ($109 federal), then select Deluxe to enter the filing experience. Once inside the Tax Home dashboard, confirm the 'Up next' card is showing and that the continue button is available.

**Prep work:**

> The ReturnMax landing page (/signin) is accessible. The product comparison page (/personal-taxes) shows four tiers: Free Edition ($0), Deluxe ($69 federal), Premier ($109 federal), and Self-Employed ($129 federal), each with feature checkmarks and a CTA button. Selecting a tier routes to the filing experience at /index/tto. The landing page and auth pages are only reachable after explicitly signing out; the default app state is logged in. The Tax Home dashboard shows an 'Up next' card and a Continue filing button after entering the TTO experience.

---

### 10. `returnmax-medium-010` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/dcfca7d3-f92d-4f37-9df4-3b51b5bb9e75/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-010]]
- **Taxonomy:** Deductions & Credits › Tax Credits › Claim Child Tax Credit, EITC, Child Care, and Education Credits
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/48/4

**Prompt:**

> Credits need to be finalized for Ella before the return goes to review. On the Child Tax Credit page, confirm Ella Rivera (DOB: 2017-09-12) is listed as a qualifying child and that the $2,000 credit amount is assigned to her. Then on the Child and Dependent Care Credit page (Form 2441), add the childcare provider 'Mission Kids After School Club' with TIN '81-2345678' and enter the amount paid of $3,600.00. Finally navigate to the Earned Income Credit wizard and complete the eligibility flow for a Head of Household filer with one qualifying child, confirming the EIC estimate populates in the header.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress; filing status 'head_of_household'. Dependent 'Ella Rivera' (dependent_id: b65b7cd2) exists as a qualifying child with DOB 2017-09-12. Child Tax Credit page exists under Deductions & Credits; $2,000 credit is valid for one qualifying child. Child & Dependent Care Credit page (Form 2441) exists; valid provider name is 'Mission Kids After School Club', provider TIN '81-2345678', amount paid $3,600.00. Earned Income Credit wizard exists under Deductions & Credits with eligibility gating based on income, filing status, and qualifying children. The header refund pills update after each credit is saved.

---

### 11. `returnmax-medium-011` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/795de7d2-462a-43bf-8384-f50e182ace46/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-011]]
- **Taxonomy:** Account & Settings › User Preferences & Notifications › Configure Notification Preferences and App Settings
- **Persona:** Returning User | **Complexity:** Medium | **Est. hops/steps/actions:** 4/42/4

**Prompt:**

> Notification settings are a mess after switching devices. In Notifications settings (/my-info/notifications), turn on SMS notifications (currently off) and turn off marketing emails (currently on). Then in the main Settings store, toggle the progress bar visibility off, set the auto-save interval to 60 seconds (from the current 30-second default), and enable high contrast mode. Once all five changes are saved, open the Notification Bell in the header to mark all unread notifications as read.

**Prep work:**

> User Maya Srinivasan (settings_id: b5635a79) has current settings: sms_notifications: false, marketing_emails: true (to be toggled to false), show_progress_bar: true (to be toggled to false), auto_save_interval_ms: 30000 (to be changed to 60000), accessibility_high_contrast: false (to be toggled to true). The Notifications page (/my-info/notifications) manages email and SMS delivery preferences. The Settings store is accessible from My Info. The Notification Bell in the header shows an unread badge; three notifications exist for Maya: 'Tax documents imported' (unread), 'Review mortgage interest' (unread), and 'Refund estimate updated' (read). A 'Mark all as read' button exists in the slide-in notification panel.

---

### 12. `returnmax-medium-012` (v3)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/850c4b06-e9a1-4c72-9e58-47b7cb72586f/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-012]]
- **Taxonomy:** Deductions & Credits › Itemized Deductions › Enter Mortgage Interest, Property Taxes, and SALT Deductions
- **Persona:** Investor / Advanced Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/52/4

**Prompt:**

> Review the active Rocket Mortgage card on your 1098 summary page to verify that Interest Paid reads $11,840.29, then click Done. Once you reach the Tax Breaks Breakdown screen, click Edit next to Property and real estate taxes and update the amount from $6,240.18 to $6,500.00. Then, click Edit next to Donations to charity and increase the amount from $1,250.00 to $1,500.00. Finally, change the deduction type from Itemized to Standard.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress with itemized deductions selected. The Form 1098 mortgage interest flow exists under Deductions & Credits. Valid lender: 'Rocket Mortgage', interest paid: $11,840.29, loan amount: $388,000, points paid: $0, PMI paid: $0, property tax: $6,240.18. The SALT page (/deductions/salt) collects state income tax, sales tax, and property tax with a $10,000 cap warning that displays when the combined entry would exceed $10,000. California state income tax withheld is $6,125.40 (from the CA state return). The property taxes page (/deductions/property-taxes) exists as a separate entry point.

---

### 13. `returnmax-medium-013` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/8c43f17b-0b13-45ba-9756-655e08508694/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-013]]
- **Taxonomy:** Expert Assistance › Live Expert Help › Start a Live Chat or Screen Share Session with a Tax Expert
- **Persona:** Expert-Assisted Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 3/38/3

**Prompt:**

> Running into a question about whether the VTI ETF gain changes my bracket before submitting. Open the Expert Help FAB and start a Live Chat session. Once connected to a tax expert, ask whether the $550.54 long-term capital gain from Fidelity Investments affects the effective tax rate at the current income level. Then close that chat and open a Screen Share session instead, entering the session code when prompted so the expert can see the capital gains entry on the 1099-B page.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress; expert_help_open is currently false. The Expert Help FAB (96 px, bottom-right, always visible) opens expert assistance options. The Live Chat page (/expert/chat) connects to a tax expert with a chat interface including message display, text input, file attachment, and end-chat button. The Screen Share page (/expert/screenshare) initiates a cobrowse session with a session code input and three security benefit cards. Capital gains from Fidelity Investments exist: VTI ETF long-term gain $550.54 (income_item_id: cdb16bd4). Current effective tax rate is 5.1% and marginal bracket is 22%.

---

### 14. `returnmax-medium-014` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/427d5181-74a5-4405-9b7d-19440ebf26f8/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-014]]
- **Taxonomy:** State Tax Filing › State Return Preparation › Prepare State Return with Auto-Filled Federal Data
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/45/4

**Prompt:**

> The California return pulled in the federal data automatically but a few adjustments are still needed. On the State Adjustments page, enter U.S. bond interest as a subtraction of $0 (confirming nothing applies) and add the Social Security exemption subtraction for the correct amount. Then on the State Deductions and Credits page, enter the state disability insurance amount from Box 14 of the W-2 (CASDI: $720.00) as an additional state deduction. Review the state refund amount in the header pill and confirm it shows the California refund before clicking 'Continue to Final Review'.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress; California (CA) state return (state_return_id: 02b3ccc5) exists with status 'completed' and return type 'resident'. The State intro page (/index/tto/state) explains federal data has transferred. State Adjustments page exists with subtraction options including U.S. bond interest and Social Security exemption. State Deductions and Credits page (/state/credits) exists; CASDI amount from W-2 Box 14 is $720.00 and is a valid state deduction entry. The state refund amount for California is $815.50 and should be visible in the header pill. The 'Continue to Final Review' button appears after reviewing state sections.

---

### 15. `returnmax-medium-015` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/4b3b42b2-7ba1-4729-ae58-071877eaad2c/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-015]]
- **Taxonomy:** Onboarding & Product Selection › Return Setup & Resume › Select Tax Year, Resume or Start a Return, and Review Tax Law Updates
- **Persona:** Returning User | **Complexity:** Medium | **Est. hops/steps/actions:** 3/36/3

**Prompt:**

> Coming back to finish a return that was started last tax season. On the Your Tax Returns dashboard (/myreturnmax), select tax year 2025 and click Continue filing to get back to the Tax Home dashboard. From the Explore section, open the Tax Law Updates page and review the Standard Deduction and Child Tax Credit update entries, note the directional indicators for both. Then close Tax Law Updates and use the Continue filing button on the Tax Home dashboard to resume in the current section, which should be 'review'.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) exists for tax year 2025 with status 'In progress' and current_section 'review'. The Your Tax Returns dashboard (/myreturnmax) shows year-selector buttons (2025, 2024, 2023) and return cards with per-return actions. The Tax Home dashboard's Explore section links to the Tax Law Updates page (/prior-year/tax-law-updates), which lists 6 categorized updates including Standard Deduction and Child Tax Credit with directional indicators (up/down/same). The Continue filing button on the Tax Home dashboard routes to the current_page '/index/tto/review'.

---

### 16. `returnmax-medium-016` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/a4b9520b-eb02-4e02-9d8d-333a1fe850d6/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-016]]
- **Taxonomy:** Other Tax Situations › AMT & Complex Tax Calculations › Navigate the AMT Wizard and Calculate Alternative Minimum Tax
- **Persona:** Investor / Advanced Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/58/5

**Prompt:**

> The AMT section has been sitting incomplete since the ISO exercise questions came up last month. From Other Tax Situations, search 'AMT' to open the Alternative Minimum Tax wizard. On Step 1, answer that common AMT deductions do apply. On Step 2, select 'ISO exercise' as the applicable situation. On Steps 3 and 4, enter ISO exercise amount of $0 and ISO short-term disposition of $0 since the shares were not sold this year. Complete Steps 5 and 6 by confirming all uncommon adjustment fields at $0. On Step 7, review the calculated AMT using the 2025 exemption amounts and confirm the result is $0. Advance through Step 8 to save the AMT determination to the return.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress; other_tax_situations_json shows amt.gateAnswer is null and amt.applies is false, AMT wizard is not yet completed. The Other Tax Situations hub search bar accepts 'AMT' as a valid query and routes to the AMT wizard (Surface 90). Step 1: gate question about common AMT deductions; Step 2: situation checkboxes including 'ISO exercise'; Steps 3-4: ISO exercise and ISO short-term/long-term disposition amount fields; Steps 5-6: 12 total uncommon adjustment currency fields (all valid at $0); Step 7: review page showing calculated AMT using 2025 exemption amounts with an expandable details table; Step 8: confirm or edit. visitedOtherTaxTopics already includes 'identity_protection_pin' and 'estimated_tax_payments'.

---

### 17. `returnmax-medium-017` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/d982bd68-04cb-4e3a-bcef-413e57f28ed6/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-017]]
- **Taxonomy:** Review & Error Correction › Federal Review › Verify Personal Info and Complete the Federal Review Checklist
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/44/4

**Prompt:**

> Federal Review needs to be completed so the return can advance to state. Open Federal Review and go through Step 1: confirm that the name 'Maya Srinivasan', SSN '321-54-9876', and date of birth 'March 14, 1988' are all correct, update any field that does not match. In Step 2, check the 5-item review checklist. The 'Credits' item shows 'in-progress' at 80%, click its Review button and navigate into the credits section to complete the outstanding credit item. Return to Federal Review and confirm all 5 checklist items show 'completed' status before advancing.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress; current_section is 'review'. Federal Review page (/index/tto/federal-review) exists. Step 1 collects name, SSN, and date of birth; valid values are first name 'Maya', last name 'Srinivasan', SSN '321-54-9876', DOB '1988-03-14'. Step 2 shows a 5-item checklist: Personal information, Filing status, Income, Deductions & credits, and Other tax situations. Progress by section shows credits at 80% (in-progress). The Review button for each checklist item drills back into that section. All other sections (my_info 100%, income 100%, deductions 100%, state 100%, payment 100%) are complete.

---

### 18. `returnmax-medium-018` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/1c6fbebc-0326-4b22-a58e-a2b9d2b9ca54/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-018]]
- **Taxonomy:** Federal Income › Self-Employment & Business Income › Enter Schedule C Business Income and Self-Employment Expenses
- **Persona:** Self-Employed Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/55/5

**Prompt:**

> Got the business numbers together for the Schedule C. Under Federal, Income, Self-Employment, create a new Schedule C entry for 'Blue Cedar Studio' with business code for consulting. Enter gross receipts of $95,000 and itemize the following business expenses: advertising $1,200, office supplies $650, and travel $2,100. Save the Schedule C. Then open the Deduction Maximizer (/index/tto/deductions/maximizer) and review the 6 deduction categories, click Review on any category that shows unclaimed deductions available so they are surfaced for the filer.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress for tax year 2025. No Schedule C entry currently exists (self_employment_businesses table is empty). The Schedule C entry flow exists under Federal, Income, Self-Employment. Valid business name: 'Blue Cedar Studio' (user email: hello@bluecedarstudio.co). Gross receipts field, business code combobox, and expense category fields (advertising, office, travel among others) all exist on the Schedule C form. Valid expense amounts: advertising $1,200, office supplies $650, travel $2,100. The Deduction Maximizer (/index/tto/deductions/maximizer) scans 350+ deductions across 6 categories: Home & property, Medical & health, Education, Retirement & savings, Charitable giving, Family & dependents. Each category shows claimed vs. available items with Review buttons.

---

### 19. `returnmax-medium-019` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/1c6bea5e-ded5-46ea-b5d3-c4cd311bb7b7/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-019]]
- **Taxonomy:** Filing & Payment › Payment & Order Management › Review Order, Pay for ReturnMax, and Apply a Service Code
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 3/38/3

**Prompt:**

> The order needs a second look before payment goes through. Navigate to the Order Summary (/index/tto/order-summary) and check the line items, confirm that the ReturnMax edition, California state return, and audit support add-on are all listed. Add the MAX Defend & Restore add-on using its toggle, then verify the subtotal updates. On the Pay ReturnMax page, enter card number '4242424242424242', expiration '12/27', CVV '123', cardholder name 'Maya Srinivasan', and check the box to reuse the tax return mailing address. Apply service code 'SAVE20' on the service code page and confirm the discount status before finalizing.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in the filing flow. Order Summary page (/index/tto/order-summary) exists showing product line items with a MAX Defend & Restore add-on toggle (Add/Remove state). The 2024 order receipt shows prior line items: Federal return preparation, California state return preparation, and Audit support. The Pay ReturnMax page (/pay-returnmax) accepts card number, expiration, CVV, cardholder name, and billing address with a checkbox to reuse the tax return address. Tax return address is '2148 Valencia Street, Apt 3, San Francisco, CA 94110'. Service code page (/file/pay/service-code) accepts a service code and shows status confirmation at /file/pay/service-code/status.

---

### 20. `returnmax-medium-020` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/95448dad-3c0f-4729-9f6b-5db25a630550/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-020]]
- **Taxonomy:** Deductions & Credits › Itemized Deductions › Add Charitable Donations, Medical Expenses, and Other Itemized Items
- **Persona:** Investor / Advanced Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/48/4

**Prompt:**

> The charitable and medical expenses need to go into the itemized section before the deduction comparison is meaningful. Under Deductions and Credits, add a cash charitable donation to 'San Francisco Marin Food Bank' for $1,250 dated November 10, 2025. Then add a non-cash charitable donation entry for clothing donated to Goodwill, use the value calculator to estimate fair market value and enter the result. Finally, open the medical and dental expenses page and enter total qualifying medical expenses; confirm the 7.5% AGI threshold warning appears given the current adjusted gross income of $90,240.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress with itemized deductions selected. Adjusted gross income is $90,240; the 7.5% AGI threshold for medical expenses is $6,768. The charitable donations flow exists under Deductions & Credits with separate paths for cash donations and non-cash donations. Valid cash donation: organization 'San Francisco Marin Food Bank', amount $1,250.00, date '2025-11-10'. Non-cash donation path includes a Goodwill value calculator and fair market value entry. The medical and dental expenses page exists; the 7.5% AGI threshold enforcement displays a warning when entered medical expenses fall below the threshold. Itemized running total is shown on the overview page.

---

### 21. `returnmax-medium-021` (v2)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/0b0251c7-bf1e-4db8-8eae-6946b491cf64/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-021]]
- **Taxonomy:** Document Management › Document Import & Linked Accounts › Import Documents from Financial Institutions via Linked Accounts
- **Persona:** Investor / Advanced Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/46/4

**Prompt:**

> Go to the Linked Accounts tab under Documents and click Add an account. Select Wells Fargo from the financial institution list, review the provider connection screen, and click Connect. Next, return to the account list, click Add an account again, select Edward Jones, and complete its link. Once both show as connected, click the Documents tab, press the blue Add tax docs button, upload the file '2026-fidelity-1099-b.pdf', select the W-2 tax form, and save it to the document inventory manager.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress. The Linked Accounts tab on the Documents Hub already shows Fidelity Investments (connected) and Coinbase (connected). 'Chase Bank' is not yet linked; it is a valid financial institution that can be searched and connected. 'Vanguard' is also not yet linked and is a valid provider. The Document Import page (/index/tto/document-autofill) supports W-2 and 1099 PDF upload with OCR auto-fill simulation. Document '2025-chase-1099-int.pdf' exists in the Documents Hub (document_id: b4f8a876) with form_type 'form_1099_int', amount $148.36, and status 'reviewed'. After upload on the import page, the user reviews parsed data before it is written to the corresponding income form.

---

### 22. `returnmax-medium-022` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/58fccc5f-fac4-4af1-a1d8-f7153bf2f1ea/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-022]]
- **Taxonomy:** Other Tax Situations › Additional Filing Actions › Handle Extension Filing, Underpayment Penalty, and Estimated Tax Planning
- **Persona:** Self-Employed Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/50/4

**Prompt:**

> The extension deadline is coming up and the estimated payments need to be reconciled first. Open the Extension Filing page from Other Tax Situations and enter estimated total tax liability of $4,748, total payments made of $15,150 (including withholding), and $0 as the amount paying with extension, confirm the balance due auto-calculates and the orange payment-deadline warning appears. Then open the Underpayment Penalty page (/other/underpayment) and review the Form 2210 details given the prior year tax liability of $8,116 and prior year AGI of $96,720.52. Finally go to Apply Refund (/other/apply-refund) and direct $200 of the expected refund to next year's estimated taxes.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress; visitedOtherTaxTopics includes 'estimated_tax_payments'. Extension Filing page (/file/extension or Other Tax Situations) exists; fields: estimated total tax liability, total payments made, amount paying with extension; balance due auto-calculates; orange warning distinguishes filing deadline from payment deadline. Valid values: total tax $4,748, total payments $15,150, payment with extension $0. Underpayment page (/other/underpayment) shows Form 2210 details; underpayment object shows prior year tax liability $8,116.00 and prior year AGI $96,720.52. Apply Refund page (/other/apply-refund) allows directing a portion of the refund to next year's estimated taxes; apply_refund.wantsToApply is currently null.

---

### 23. `returnmax-medium-023` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/e4107462-7d21-4641-8d8e-4771187ab745/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-023]]
- **Taxonomy:** Personal Information › Filer Identity & Contact › Set Filing Status and Compare MFJ vs. MFS
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 3/36/3

**Prompt:**

> Need to review the filing status options before committing. On the Filing Status page (/my-info/filing-status), the current status is 'head_of_household', open the page to review all five filing status options and their descriptions. Then switch temporarily to 'Married Filing Jointly' to trigger the Filing Status Optimizer, which should show a side-by-side MFJ vs. MFS refund comparison with a 'Recommended' highlight. Note which option the optimizer recommends, then switch back to 'Head of Household' and save that as the confirmed filing status for the return.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress; current filing status is 'head_of_household'. The Filing Status page (/my-info/filing-status) shows five options: Single, Married Filing Jointly, Married Filing Separately, Head of Household, and Qualifying Surviving Spouse. The Filing Status Optimizer (/my-info/filing-status-optimizer) is triggered when a married status is selected and displays a side-by-side MFJ vs. MFS estimated refund comparison with a 'Recommended' highlight and a 'Use recommended status' button. The selected status is stored in filingStatus on the tax return store.

---

### 24. `returnmax-medium-024` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/9413998f-0e39-4b09-a5cd-a6ba98c35497/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-024]]
- **Taxonomy:** Expert Assistance › Expert Review & Full Service › Request Expert Review, MAX Protection, or Full-Service Tax Filing
- **Persona:** Expert-Assisted Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 5/55/5

**Prompt:**

> The Expert Review flow needs to go all the way through before submitting. Start the Expert Review from the post-Final Review flow. Complete consent Step 1 (multi-use consent): expand the benefit cards, sign with full name 'Maya Srinivasan' and today's date. Complete consent Step 2 (data sharing consent): sign the same way. On the Check Summary page review the issue count. On the Issues list, filter to 'NEEDS REVIEW' items and click Fix on the first listed issue to address it. Return to Connect Expert and select the Chat option to initiate contact with the available expert. On the MAX Upsell page, review the product comparison table and click 'Get MAX' to add MAX protection at $79.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) has completed Final Review and is at the Expert Review entry point (/index/tto/expert-review/). Two-step consent management exists: Step 1 multi-use consent and Step 2 data sharing consent, each with expandable benefit cards, a signature drawer for full name/spouse/date, and an FAQ drawer. Valid signature name: 'Maya Srinivasan'. Check Summary page shows an issue count. Issues list has at least one 'NEEDS REVIEW' badge item with a Fix button. Connect Expert page offers Call, Chat, Schedule, or Skip options with availability shown. MAX Upsell page shows a product comparison table at $79 with Get MAX or Skip buttons.

---

### 25. `returnmax-medium-025` (v2)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/5fe4dc70-f2ac-47b9-bb5f-ecf8bea8b836/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-025]]
- **Taxonomy:** Tax Tools & Resources › Tax Tools › Search Help, Bookmark Sections, Generate Tax Vouchers, Print Documents, and Review Fees
- **Persona:** Self-Employed Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/46/4

**Prompt:**

> Review the active Rocket Mortgage card on your 1098 summary page to verify that Interest Paid reads $11,840.29, then click Done. Once on the tax breaks breakdown screen, execute these modifications: first, click Edit next to Property and real estate taxes and update the value from $6,240.18 to $6,500.00; second, click Edit next to Energy efficient home improvements and select yes for ever questions asked and write $200 in all the amount fields; third, click Edit next to Donations to charity and increase the value from $1,250.00 to $1,500.00; fourth, change the deduction type from Itemized to Standard. And finally, go to the print centre and download the Federal Return (Form 1040) pdf.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress. The global Search panel is triggered from the header search icon; 'mortgage interest' is one of the 24 searchable help topic categories. Bookmarks page (/index/tto/bookmarks) exists; bookmarked_pages_json includes '/index/tto/deductions/mortgage-interest' and '/index/tto/review'. Each bookmark has a 'Go to section' navigation button. Tax tools hub (/index/tto/tools) links to Print Center. Print Center (/tools/print-center) lists five document types: Federal Return, State Return, Supporting Worksheets, Tax Summary, and Payment Voucher, each with a Save/Print button. My Fees (/tools/my-fees) shows a product breakdown table.

---

### 26. `returnmax-medium-026` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/01b555b7-3847-4b38-ac38-e6e34313bb8c/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-026]]
- **Taxonomy:** Onboarding & Product Selection › Product Discovery & Sign-Up › Register an Account or Sign In
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 3/38/3

**Prompt:**

> Marcus needs to get his account set up before starting the 2025 return. Sign out of the current session to reach the auth pages. On the registration page (/auth/register), fill in first name 'Marcus', last name 'Reed', date of birth '04/17/1986', email 'marcus.reed@gmail.com', password 'Password123!', and password confirmation 'Password123!'. Submit the registration and confirm it passes email-uniqueness and Zod schema validation. If the system flags a conflict, navigate to the login page (/auth/login) instead, enter 'marcus.reed@gmail.com' and 'Password123!', and confirm routing to the Tax Home dashboard on success.

**Prep work:**

> User Marcus Reed (user_id: 01e42041) exists with email 'marcus.reed@gmail.com', password 'Password123!', date of birth '1986-04-17', and has_onboarded: false. The registration page (/auth/register) requires first name, last name, date of birth (MM/DD/YYYY), email, password, and password confirmation with Zod schema validation and email-uniqueness checks. The login page (/auth/login) requires email and password and routes to the Tax Home dashboard on success. Auth pages are only reachable after explicitly signing out. The 'Forgot password?' link leads to /auth/forgot-password.

---

### 27. `returnmax-medium-027` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/465f7806-466c-4f3a-9e52-37a3767bd785/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-027]]
- **Taxonomy:** Federal Income › Investment & Retirement Income › Report Retirement Distributions, Unemployment, and Other 1099 Income
- **Persona:** Investor / Advanced Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/50/4

**Prompt:**

> Several 1099 income types came in and all of them need to be recorded before Federal Review. Under Federal, Income, enter retirement distribution income (1099-R) from a pension source with the applicable taxable amount. Then add unemployment compensation (1099-G) from California EDD with the state unemployment amount. Finally, enter a Social Security benefit from SSA-1099 with the gross benefit amount. After all three are saved, open the income overview to confirm all 17 income categories show add/edit capability and that the three newly entered forms appear in the list.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress; no 1099-R, 1099-G, or SSA-1099 entries currently exist. Federal income overview shows 17 income categories with add/edit capability. 1099-R page exists under Federal, Income for retirement distributions (pension, IRA, 401(k), rollovers). 1099-G page exists for unemployment compensation; California EDD is a valid unemployment payer. SSA-1099 page exists for Social Security benefits. Each form page persists to the corresponding income type in the tax return store. The income overview shows all categories after each form is saved.

---

### 28. `returnmax-medium-028` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/1bd7a6fd-f6aa-4693-b827-35e26ba8d514/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-028]]
- **Taxonomy:** Deductions & Credits › Above-the-Line Deductions › Claim Student Loan Interest, IRA, HSA, and Educator Deductions
- **Persona:** Self-Employed Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/48/4

**Prompt:**

> Three above-the-line deductions are ready to enter. Under Deductions and Credits, open the Student Loan Interest page and enter lender 'Nelnet' with interest paid of $640.42 (Form 1098-E). Then go to the HSA page (Form 8889) and enter coverage type 'family', self-contributions $3,000, employer contributions $1,200, total distributions $850.25, and qualified expenses $850.25. Finally open the IRA contributions page and enter a traditional IRA deduction amount; confirm the income eligibility check runs and the deduction appears in the running total. Check the header refund pill after all three are saved.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress. Deduction item for student loan interest exists with lender 'Nelnet' and interest paid $640.42. Student Loan Interest page exists under Deductions & Credits (up to $2,500 limit). HSA page (Form 8889) exists; valid values: coverage type 'family', self-contributions $3,000.00, employer contributions $1,200.00, distributions $850.25, qualified expenses $850.25. Deduction item for HSA (deduction_item_id: 5869d504) exists with provider 'Lively HSA' and total amount $4,200.00. IRA contributions page exists; income eligibility checks run for traditional IRA deduction. The header refund pill updates after each deduction is saved.

---

### 29. `returnmax-medium-029` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/0a974e19-d218-480b-a946-f86387815683/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-029]]
- **Taxonomy:** State Tax Filing › State Review & Completion › Review State Return and Advance to Final Review
- **Persona:** Multi-State Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 3/38/3

**Prompt:**

> The California state return is nearly done but a few checklist items need to clear before it can advance. On the State Review page (/state/review), check the 3-item checklist, State income, State deductions & adjustments, and State credits, and identify any items showing incomplete status. Click the Review button for each incomplete item, navigate to that section, and complete the outstanding field or confirmation. Return to State Review after each one. Once all three items show complete, click 'Continue to Final Review'. On the State Complete page, confirm the message indicating California preparation is finished.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) has a California state return (state_return_id: 02b3ccc5) in progress. State Review page (/state/review) shows a California refund summary alongside a 3-item checklist: State income, State deductions & adjustments, and State credits. Per-item completion status and Review buttons are shown; conditional rendering highlights incomplete sections. Valid California state refund: $815.50. State Complete page (/state/complete) exists and confirms state preparation is finished. The 'Continue to Final Review' button appears only when all items are complete.

---

### 30. `returnmax-medium-030` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/220ac651-ca03-4d6c-ab8e-7bd7278cc555/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-030]]
- **Taxonomy:** Filing & Payment › Tax Payments & Refund Options › Set Up Direct Debit, Split Refund, or Request a Refund Advance
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/46/4

**Prompt:**

> The refund is coming in and it needs to be split properly. On the Federal Tax Payment page (/file/tax-payment), confirm the Direct debit tab is selected and enter routing number '111000025', account number '000123456789', account type 'checking', and a payment date. Then navigate to the Refund Split page (Form 8888) and add a second bank account for splitting: routing number '021000021', account number '9876543210', type 'savings', deposit amount $2,000. Confirm an 'add' button is available for a third account. Finally, open the Refund Advance page and review the $4,000 at 0% APR eligibility details and the Refund Transfer alternative at $39.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in the filing flow. Payment election (payment_election_id: 2b2ad08f) exists with method 'direct_deposit'; bank account routing number '111000025', account number '000123456789', account type 'checking'. consentToEFile is false. The Federal Tax Payment page (/file/tax-payment) has three tabs: Direct debit, Pay by check, Pay by card on IRS.gov. Refund Split page (Form 8888) supports up to 3 bank accounts with routing number, account number, account type, and deposit amount per account, plus add/remove buttons. Refund Advance page shows $4,000 at 0% APR eligibility with a Refund Transfer alternative at $39 fee. Total federal refund is $10,402.

---

### 31. `returnmax-medium-031` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/eed1723e-109b-43ee-93b8-f402405573a2/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-031]]
- **Taxonomy:** Account & Settings › Account Management › Manage Account Settings, Order Summary, and Download Tax Return
- **Persona:** Returning User | **Complexity:** Medium | **Est. hops/steps/actions:** 3/44/4

**Prompt:**

> Need to pull everything together for records after the 2024 return was filed. From the My Account section, open the Download Tax Return page (/index/tto/download) and download the federal PDF and the worksheets PDF using their individual download buttons, note the IRS retention notice shown. Then navigate to the Receipt page (/index/tto/receipt) and confirm the 2024 order receipt is in 'paid' state showing the correct line items: Federal return preparation ($89.00), California state return preparation ($59.00), and Audit support ($39.00) with a total of $187.00 and payment method 'Visa **** 4242'. Finally open the Clear & Start Over page and read the data-deletion warning list without confirming, then navigate away without clicking Delete.

**Prep work:**

> User Maya Srinivasan (user_id: aecd2ba0) has a filed 2024 return. Download Tax Return page (/index/tto/download) offers 5 download formats: .tax file, federal PDF, state PDF, worksheets PDF, and summary PDF, each with a download button and an IRS retention notice. Receipt page (/index/tto/receipt) shows order receipt (order_receipt_id: ee960861) in 'paid' state with line items: 'Federal return preparation' $89.00, 'California state return preparation' $59.00, 'Audit support' $39.00; total $187.00; payment method 'Visa **** 4242'; paid at 2025-03-17. Clear & Start Over page (/index/tto/clear-start-over) presents a red-styled warning with a data-deletion list and checkbox confirmation before the Delete button.

---

### 32. `returnmax-medium-032` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/6c38ad5f-57ca-4eef-8ea7-0c8f410d48a9/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-032]]
- **Taxonomy:** Other Tax Situations › Additional Filing Actions › File Amended Return, Change of Address, and Other IRS Forms
- **Persona:** Expert-Assisted Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/48/4

**Prompt:**

> An error was discovered after the 2024 return was filed and it needs a formal amendment. From Other Tax Situations, open the Form 1040-X amendment flow. Select 'Income' and 'Deductions/Credits' as the change categories. In the explanation textarea, enter a clear reason for the amendment referencing the income change. Click 'Continue to amendment'. After the amendment is initiated, also complete the Change of Address section: set wantsChangeOfAddress to yes, indicate an individual return, enter the previous address as '2148 Valencia Street, Apt 3, San Francisco, CA 94110' and the new address as '550 Post Street, Apt 12, San Francisco, CA 94102'. Save both changes.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress. The amended return page exists under Other Tax Situations and at /file/amended. Form 1040-X presents change-category checkboxes: filing status, income, deductions/credits, dependents, tax payments, other. An explanation textarea and 'Continue to amendment' button exist. Change of Address section (Form 8822) exists under Other Tax Situations; fields include individualReturn toggle, previous address, and new address. Valid previous address: '2148 Valencia Street, Apt 3, San Francisco, CA 94110'. Valid new address: '550 Post Street, Apt 12, San Francisco, CA 94102'. visitedOtherTaxTopics currently includes 'estimated_tax_payments' and 'identity_protection_pin'.

---

### 33. `returnmax-medium-033` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/0c2a4c23-d0c1-43e7-98e2-0a901bd92d57/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-033]]
- **Taxonomy:** State Tax Filing › State Credits & Multi-State Allocation › Enter State Deductions, Credits, and Allocate Multi-State Income
- **Persona:** Multi-State Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/48/4

**Prompt:**

> Moving back to California full-time next year, but this return covers a part-year situation. On the State Returns list, confirm California is showing as a resident return with status 'completed'. Then open the State Deductions and Credits page (/state/credits) and add a state disability insurance deduction (CASDI $720.00) and a state child tax credit for Ella Rivera. Navigate to the Multi-State Filing page (/state/multi-state) and add a Texas entry: residency status 'part-year' and income $0 (confirming no Texas filing is required). Then add California again as 'full' year resident with income $93,879.65 to confirm the allocation is correct.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) has California state return (state_return_id: 02b3ccc5) with status 'completed' and return_type 'resident'. State Deductions and Credits page (/state/credits) exists with fields for state disability insurance and state child tax credit. Valid CASDI amount: $720.00. Dependent Ella Rivera (dependent_id: b65b7cd2) is a qualifying child eligible for state child tax credit. Multi-State Filing page (/state/multi-state) provides a dynamic add/remove form with a 50-state dropdown, residency status (full-year, part-year, nonresident), and income amount. Multi-state data (multi_state_json) currently shows California full-year at $93,879.65. Texas has no income tax; a part-year Texas entry with $0 income should generate a no-filing-required confirmation.

---

### 34. `returnmax-medium-034` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/f7e55a11-e8dd-413f-bd9c-ecca2cf04b77/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-034]]
- **Taxonomy:** Onboarding & Product Selection › Return Setup & Resume › Complete Guided Onboarding with Situation Check
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 3/45/4

**Prompt:**

> First time through the TTO onboarding flow and it needs to be completed cleanly. From the welcome screen on the UnifiedOnboardingPage, acknowledge the welcome and advance to loading. On the situations form, check 'Has W-2 job', 'Has children or dependents', 'Owns home', and 'Sold stock or crypto', confirm no conflict validation fires. Leave 'Is self-employed' unchecked. Advance to the verify step and enter date of birth '03/14/1988' and SSN last four '9876' using the SSN method. Complete the moreInfo step by entering country 'United States' and address query '2148 Valencia Street, San Francisco, CA 94110'. On the priorYearFiling step, select method 'returnmax'. Confirm the Tax Home dashboard loads showing the 'Up next' card.

**Prep work:**

> User Maya Srinivasan (user_id: aecd2ba0) has has_onboarded: true but the onboarding flow is accessible for review. The UnifiedOnboardingPage phases are: welcome, loading, situations, verify, moreInfo, priorYearFiling. Situations checkboxes include: hasW2Job, isFreelancerGigWorker, isSelfEmployed, ownsSCorpPartnershipLlc, hasUnemploymentIncome, hasChildrenDependents, ownsHome, paidRent, ownsRentalProperty, soldStockOrCrypto, donatedToCharity, maximizeDeductionsCredits. Valid situations to select: hasW2Job, hasChildrenDependents, ownsHome, soldStockOrCrypto. Conflict validation fires for incompatible combinations. Verify step accepts dateOfBirth '1988-03-14' and ssnLastFour '9876' via SSN method. moreInfo address: '2148 Valencia Street, San Francisco, CA 94110'. priorYearFiling method: 'returnmax'.

---

### 35. `returnmax-medium-035` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/adb19765-16cf-4f00-b9b9-54b5ffb4e92b/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-035]]
- **Taxonomy:** Personal Information › Filer Identity & Contact › Declare Uncommon Situations and Military Status
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 3/36/3

**Prompt:**

> The uncommon situations section was skipped earlier and it needs to be completed. On the Armed Forces page (/my-info/military), set the answer to 'No' for US Armed Forces membership in 2025. On the Uncommon Situations page (/my-info/household), review the seven checkbox options, select 'None of these apply' to deselect all others and confirm it saves as the exclusive choice. Then open the inline help tooltip (i) next to 'Claimed as dependent' and verify the 380 px help panel opens on the right side with an article and Spanish translation option. Close the panel and save the page.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress; uncommon_situations_json shows noneApply: true and all individual situations as false. Armed Forces page (/my-info/military) has a Yes/No radio group for US Armed Forces membership in 2025. Uncommon Situations page (/my-info/household) has multi-select checkboxes: full-time student, claimed as dependent, legally blind, preparing for a deceased filer, nonresident alien spouse, incarcerated, IRS language preference, and exclusive 'None of these apply' option that deselects all others. Help tooltips (i) open a 380 px right-side panel with article search and Spanish translation. Selections persist to the tax return store.

---

### 36. `returnmax-medium-036` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/611b25f4-10ed-41ff-ad05-6a831e44ef8f/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-036]]
- **Taxonomy:** Deductions & Credits › Self-Employment Deductions › Maximize Self-Employment Deductions Including Home Office and Vehicle
- **Persona:** Self-Employed Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/58/5

**Prompt:**

> The self-employment deductions are incomplete and the Deduction Maximizer has not been run yet. Under Deductions and Credits, Self-Employment Deductions, enter the home office deduction: set office square footage ratio to 20% against annual rent. Then enter the self-employment health insurance deduction amount. Under vehicle expenses, select the mileage method and enter total business miles. After all three deduction entries are saved, open the Deduction Maximizer (/deductions/maximizer). Review the 6 scanned categories and click Review on 'Home & property' and 'Medical & health' to surface unclaimed items in those two categories. Note how many unclaimed deductions appear in each.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress. Self-Employment Deductions section exists under Deductions & Credits. Home office deduction page accepts a square footage ratio input; 20% is a valid ratio. Self-employment health insurance deduction page exists as a separate entry. Vehicle expenses page offers mileage or actual cost method; mileage method requires total business miles entry. Deduction Maximizer (/deductions/maximizer) scans 350+ deductions across 6 categories: Home & property, Medical & health, Education, Retirement & savings, Charitable giving, Family & dependents. Each category shows claimed vs. available items with Review buttons for unclaimed deductions.

---

### 37. `returnmax-medium-037` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/483b4f6b-3ca4-4e40-a04e-138570a62df8/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-037]]
- **Taxonomy:** Federal Income › Self-Employment & Business Income › Report Rental, Royalty, and Other Investments Income
- **Persona:** Investor / Advanced Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/50/4

**Prompt:**

> There are other investment income types that still need to be entered for the return. From Federal, Income, open the Other Investments page (/income/other-investments) and add an ESPP sale: enter broker name 'Fidelity Investments', investment type 'ESPP', acquisition details, date sold, proceeds, and cost basis. Then go to the Other Income page and add gambling winnings (W-2G) with the payer name and amount. Finally open the Rental Income page (/income/rental) and enter a rental property: address, annual rental income amount, and at least one expense category. Confirm all three new income types appear when the income overview is refreshed.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress; rental_properties table is empty and no ESPP or gambling winnings entries currently exist. Other Investments page (/income/other-investments) handles ESPP, NQSO, and other investment sales via a multi-step flow: broker name, investment type, acquisition details, date sold, proceeds, cost basis. Valid broker: 'Fidelity Investments'; valid investment type: 'ESPP'. Other Income page covers gambling winnings (W-2G) with payer name and amount fields. Rental Income page (/income/rental) accepts property address, rental income, and expenses by category. Income overview shows all 17 categories with add/edit capability.

---

### 38. `returnmax-medium-038` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/2fdd7bf9-c7b6-4afb-befd-e0156b1afcf7/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-038]]
- **Taxonomy:** Deductions & Credits › Tax Credits › Claim Energy, EV, Adoption, Foreign Tax, and Other Credits
- **Persona:** Investor / Advanced Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/48/4

**Prompt:**

> Several energy and family credits are available but not yet entered. Under Deductions and Credits, open the energy credits page and add the residential energy efficient home improvement credit for the high-efficiency heat pump installed on September 15, 2025, cost $5,200, improvement type HVAC, and confirm the 30% credit amount auto-calculates. Then open the Child Tax Credit page and confirm Ella Rivera ($2,000) is already listed. Finally navigate to the Adoption Credit page and enter zero qualified adoption expenses to confirm no credit applies, so the section is marked complete rather than skipped.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress. Credit item for 'energy' (credit_item_id: c96d4ec3) exists: improvementType 'hvac', cost $5,200.00, dateInstalled '2025-09-15', credit amount $1,200.00. Energy credits page exists under Deductions & Credits; 30% credit calculation applies to residential energy efficient home improvement. Child Tax Credit page shows Ella Rivera (related_dependent_id: b65b7cd2) with $2,000 already assigned. Adoption Credit page exists under Deductions & Credits (Form 8936 area) for qualified adoption expenses; entering $0 completes the section.

---

### 39. `returnmax-medium-039` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/7343d34c-be02-4f7b-a741-3662bf4f9159/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-039]]
- **Taxonomy:** Tax Tools & Resources › Guarantees & Audit Protection › Review Accuracy Guarantee and Add Audit Defense
- **Persona:** Expert-Assisted Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 3/38/3

**Prompt:**

> Considering whether Audit Defense is worth adding before filing. From Federal Review or Final Review, find and open the Accuracy Guarantee informational box and read the penalty/interest coverage details. Then navigate to the Audit Support page (/guarantees/audit-support) and review the five included items and three contact methods. Click 'Upgrade to Audit Defense' to open the Audit Defense page (/guarantees/audit-defense) and review the 6-row comparison table between Audit Support and Audit Defense. Add Audit Defense at $49.99 using the Add button, then confirm the order summary has been updated to reflect the new add-on.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) has reached the review stage. The 100% Accuracy Guarantee informational box is displayed on Federal Review, Final Review, and Expert Review pages. Audit Support page (/guarantees/audit-support) shows 'You are covered' messaging with 5 included items: Q&A guidance, IRS notices help, document prep, 7-year validity, and 3 contact methods (Phone, Email, Live chat). 'Upgrade to Audit Defense' button leads to Audit Defense page (/guarantees/audit-defense), which shows a comparison table (6 feature rows, Audit Support vs. Audit Defense columns) at $49.99 with Add and Decline buttons and a legal disclaimer. Adding Audit Defense updates the order summary.

---

### 40. `returnmax-medium-040` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/30860c43-0b15-41fc-9005-42fef215f1c6/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-040]]
- **Taxonomy:** Other Tax Situations › Identity, Security & IRS Correspondence › Manage Identity Protection PIN and Identity Theft Claims
- **Persona:** Expert-Assisted Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 4/46/4

**Prompt:**

> The IP PIN section has been sitting at an unknown state since onboarding. From the Other Tax Situations hub, open the IP PIN wizard. On Step 1, answer 'Yes', an IP PIN has been issued. On Step 2, the existing PIN table appears; add a new PIN entry by entering a 6-digit PIN and saving it. Confirm the new PIN row appears in the PIN table with an edit option. Then navigate to the Identity Theft section and on the gate question answer 'No' to indicate no identity theft has occurred, so that section is marked as visited and complete. Both IP PIN and identity theft should now appear in visitedOtherTaxTopics.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress. other_tax_situations_json shows ipPin.hasIpPin as null and ipPin.pin as empty, not yet completed. The IP PIN wizard in Other Tax Situations hub: Step 1 asks whether an IP PIN has been issued (Yes/No); Step 2 shows an existing PIN table with edit, delete (modal confirmation), and add options; Step 3 is a 6-digit input with validation. Identity theft section: gate question asks whether identity theft occurred (Yes/No). visitedOtherTaxTopics currently contains 'estimated_tax_payments' and 'identity_protection_pin'. The hub tracks visited topics and shows Start/Revisit/Edit status.

---

### 41. `returnmax-medium-041` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/d963d18c-2c9d-4306-b04f-eb8eb9d41da1/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-041]]
- **Taxonomy:** Filing & Payment › Print, Mail, and Return Amendments › Submit by Mail, File an Extension, or Amend a Filed Return
- **Persona:** Returning User | **Complexity:** Medium | **Est. hops/steps/actions:** 4/48/4

**Prompt:**

> The return cannot be e-filed this year and needs to go out by mail instead. Open the Print & Mail page (/file/print) and download the federal return PDF and the California state return PDF using their individual buttons. Read through all five mailing instructions and note the April 15, 2026 deadline. Then navigate to the Extension Filing page (/file/extension) as a backup: enter estimated total tax liability $4,748, total payments made $15,150, and $0 as amount paying with extension, confirm balance due auto-calculates to $0 and the orange warning note about the payment deadline displays. Save the extension form.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress. Print & Mail page (/file/print) offers federal and state return PDF downloads with 5 mailing instructions and an April 15, 2026 deadline note. California state return (state_return_id: 02b3ccc5) PDF is available for download. Extension Filing page (/file/extension) exists; fields: estimated total tax liability, total payments already made, amount paying with extension; balance due auto-calculates; orange warning distinguishes filing deadline from payment deadline. Valid values: total tax $4,748, total payments $15,150 (including $13,450.25 federal withheld plus $1,800 estimated payments), amount with extension $0. Balance due should calculate to $0.

---

### 42. `returnmax-medium-042` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/8b20c82e-cc37-40d2-9ce6-f7eed506abd6/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-042]]
- **Taxonomy:** End-to-End Workflows › Complete Filing Journeys › Full Simple W-2 Return: Onboarding Through E-File Confirmation
- **Persona:** First-Time Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 5/65/5

**Prompt:**

> A first-time W-2 filer needs to go from the Tax Home dashboard all the way to the Confirmation page in one session. Start at the Tax Home dashboard. Complete My Info: enter profile details for Maya Srinivasan (occupation 'Tax Consultant'), confirm military status 'No', leave uncommon situations as 'None of these apply', set filing status to 'Head of Household', enter contact address '2148 Valencia Street, Apt 3, San Francisco, CA 94110' and phone '415-210-4837', and save SSN '321-54-9876'. Under Federal income, enter the W-2 for Luma Labs LLC ($92,500.75 wages, $13,450.25 federal withheld). Accept the Standard Deduction recommendation. Pass Federal Review with all 5 checklist items complete. Proceed through California state (accept auto-filled data). Run the Final Error Check to 0 issues. Skip Expert and MAX. Pay $0 (Free Edition). On Ready to File select Federal only. Enter prior year AGI $96,720.52 and a 5-digit signing PIN on Verify. Click 'Transmit returns now'. Confirm the green checkmark and submission ID on the Confirmation page.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress for tax year 2025. All seed data is in place: user Maya Srinivasan (DOB: 1988-03-14, SSN: 321-54-9876), W-2 from Luma Labs LLC ($92,500.75 wages), California state return, prior year AGI $96,720.52. Standard Deduction for Head of Household is auto-calculated. Federal Review 5-item checklist covers Personal information, Filing status, Income, Deductions & credits, and Other tax situations. Final Error Check progress bar runs 0-100%; 0 issues expected. Free Edition costs $0. Ready to File allows Federal-only selection. Verify page accepts prior year AGI and 5-digit signing PIN. Confirmation page shows green checkmark, federal and state submission IDs, and refund summary cards.

---

### 43. `returnmax-medium-043` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/e459232f-915e-4092-8ad2-b1c50092ea97/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-043]]
- **Taxonomy:** End-to-End Workflows › Complete Filing Journeys › Full Self-Employed Return: Schedule C, Deductions, Expert Assist, and Payment
- **Persona:** Self-Employed Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 5/65/5

**Prompt:**

> A self-employed filer with consulting income needs to file a complete return. Start at Tax Home and complete the onboarding situations form selecting 'Is self-employed' and 'Has W-2 job'. Under Federal income, add a Schedule C for 'Blue Cedar Studio' with gross receipts $95,000 and business expenses: advertising $1,200, office supplies $650, travel $2,100. Also enter the W-2 from Luma Labs LLC ($92,500.75). Under Deductions and Credits, enter the home office deduction at 20% ratio, self-employment health insurance, and all four quarterly estimated tax payments (Q1-Q4, $450 each, 2025 dates). Run the Deduction Maximizer. Navigate through Federal Review and California state. Run the Final Error Check. Open Expert Help via the FAB and ask a live-chat question about the home office calculation. Complete Expert Review (Connect Expert: Chat). On MAX Upsell click Skip. Pay for Self-Employed tier ($129 federal) via the card form. Select both Federal and State on Ready to File. Enter prior year AGI and signing PIN. Submit and confirm.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress for tax year 2025. No Schedule C currently exists; 'Blue Cedar Studio' is the valid business name. W-2 from Luma Labs LLC exists ($92,500.75). Estimated tax payments exist: Q1 $450 (2025-04-15), Q2 $450 (2025-06-16), Q3 $450 (2025-09-15), Q4 $450 (2026-01-15). Home office deduction page exists; 20% ratio is valid. Deduction Maximizer (/deductions/maximizer) scans 6 categories. Expert Help FAB opens at bottom-right. Expert Review flow includes two consent steps and Connect Expert options (Call, Chat, Schedule, Skip). Self-Employed tier costs $129 federal. California state return (state_return_id: 02b3ccc5) exists. Prior year AGI: $96,720.52.

---

### 44. `returnmax-medium-044` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/f3b8de54-3e08-42a3-bd3a-88bb705abb62/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-044]]
- **Taxonomy:** End-to-End Workflows › Complete Filing Journeys › Full Investor Return: Linked Account Import, Capital Gains, and Itemized Deductions
- **Persona:** Investor / Advanced Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 5/65/5

**Prompt:**

> An investor with multiple imported accounts needs to file a complete return. In the Documents Hub, open the Linked Accounts tab and confirm Fidelity Investments and Coinbase are already connected. Navigate to Document Import (/index/tto/document-autofill) and import '2025-fidelity-1099-b.pdf', review the OCR-parsed data and confirm it writes to the 1099-B income form (VTI ETF long-term gain $550.54). Under Federal income, verify the imported 1099-B data, then manually add the 1099-INT from Chase Bank ($148.36) and the crypto sale from Coinbase (BTC, proceeds $3,120, cost basis $2,440, long-term). Under Deductions, the Standard Deduction exceeds itemized, accept the recommendation. Navigate through Federal Review (all 5 checklist items) and California state. Run the Final Error Check (0 issues). Complete Expert Review with MAX protection added ($79). Pay Premier tier ($109 federal) via the card form. Select both Federal and State on Ready to File. Enter prior year AGI $96,720.52 and signing PIN. Submit and track the 5-step refund timeline on the Status page.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress for tax year 2025. Fidelity Investments (linked_account_id: 610a833c) and Coinbase (linked_account_id: 52499a3a) are connected in the Linked Accounts tab. Document '2025-fidelity-1099-b.pdf' (document_id: 9986617f) exists with form_type 'form_1099_b' and status 'reviewed'; VTI ETF long-term gain $550.54. Chase Bank 1099-INT: payer 'Chase Bank', amount $148.36. Coinbase crypto sale: BTC, proceeds $3,120, cost basis $2,440, long-term. Standard Deduction recommended over itemized for Head of Household. Federal Review 5-item checklist complete. Final Error Check: 0 issues. Expert Review MAX add-on costs $79. Premier tier costs $109 federal. California state return (state_return_id: 02b3ccc5) exists. Prior year AGI: $96,720.52. Status page shows a 5-step timeline.

---

### 45. `returnmax-medium-045` (v1)

- **Task page:** https://staging-rl-gym-harness.turing.com/verification/prompts/551654a3-6246-4cbd-93fd-174ebde17a0f/task
- **Test note:** [[2026-06-14_1939_returnmax-medium-045]]
- **Taxonomy:** End-to-End Workflows › Complete Filing Journeys › Complete Multi-State and Prior-Year Import Return: From Data Import to State Filing
- **Persona:** Multi-State Filer | **Complexity:** Medium | **Est. hops/steps/actions:** 5/65/5

**Prompt:**

> A returning filer who moved states mid-year needs to complete a full multi-state return. Start at Tax Home and click the 'Double-check last year' Explore card to review the Double-Check Last Year expert service. Then navigate to Prior Year Import: select 'Import from ReturnMax', which pre-fills personal info (Maya Srinivasan, DOB 1988-03-14), the Luma Labs LLC W-2, bank routing info, dependent Ella Rivera, and prior year AGI $96,720.52. Review the pre-filled data and update the W-2 wages if needed. Under Federal income, confirm income totals. Complete Federal Review with all 5 items. Proceed to State: on the Multi-State Filing page, add California as full-year resident with income $93,879.65, then add Texas as part-year with income $0. Enter California-specific adjustments and credits (CASDI $720). On State Review, advance all 3 checklist items to complete and click 'Continue to Final Review'. Run the Final Error Check. Complete the filing flow through submission and confirm the California refund card ($815.50) appears on the Confirmation page.

**Prep work:**

> Tax return for Maya Srinivasan (tax_return_id: 6aafcfc5) is in progress for tax year 2025. Filed return for 2024 (filed_return_id: eeff9127) exists with status 'accepted', AGI $96,720.52, taxpayer name 'Maya Srinivasan'. Prior Year Import page (/prior-year/import) offers 'Import from ReturnMax' method, which pre-fills personal info, W-2s, bank info, dependents (Ella Rivera), and prior year AGI. Multi-State Filing page (/state/multi-state) exists; multi_state_json currently shows California full-year at $93,879.65. Texas is a valid state with no income tax; part-year $0 income entry generates a no-filing-required confirmation. California CASDI deduction: $720.00. State Review 3-item checklist must all reach complete. California state refund: $815.50. Confirmation page shows federal and state refund summary cards.

---

