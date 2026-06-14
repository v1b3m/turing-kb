# Paymatrix (Workforce DP) — Implemented Feature Report

**Generated:** 2026-05-21  
**Live app:** http://localhost:5174  
**Reference tracker:** `pg/ADP Tracker - Frontend dev tracking.csv`  
**Source:** `/home/v1b3m/Dev/Turing/paymatrix/frontend/src`

> **Note on tracker vs. code:** The tracker is a PR-level planning doc and is slightly behind the actual codebase. Where the code has moved ahead of tracker status (e.g., pay profile drawers, hire form steps, worksheet grid), the actual implemented state is noted.

---

## Authentication

Passwordless email / OTP login — no passwords ever set.

**Flow:** `/login` → enter email → backend sends OTP → `/sessionid` exchanges token → `/workforce`

### Seeded Accounts

**Super Admins**

| Email | Name |
|---|---|
| `john.doe@example.com` | John Doe |
| `jane.smith@example.com` | Jane Smith |
| `bob.wilson@example.com` | Bob Wilson |
| `carol.jones@example.com` | Carol Jones |

**Standard Users (active, 44 total — sample)**

| Email | Name |
|---|---|
| `sarah.johnson@example.com` | Sarah Johnson |
| `james.chen@example.com` | James Chen |
| `maria.garcia@example.com` | Maria Garcia |
| `david.kim@example.com` | David Kim |
| *(through `tyler.mitchell@example.com`)* | … |

**Inactive — cannot log in**
- `aaron.brooks@example.com`
- `rebecca.gray@example.com`
- `scott.bennett@example.com`

---

## Tracker Status Summary

| Component | Dev | Tracker Status |
|---|---|---|
| Header | Lucas Mauricio | ✅ Merged |
| Footer | Erons Gberaese | ✅ Merged |
| Landing page sections | Lucas Mauricio | ✅ Merged |
| Dashboard UI kit | Francisco Ponce | ✅ Merged |
| Dashboard foundation improvements | Francisco Ponce | ✅ Merged |
| Payroll module | Francisco Ponce | ✅ Merged |
| HR module | Francisco Ponce | ✅ Merged |
| People module | Francisco Ponce | ✅ Merged |
| Support / Customer Service pages | Erons Gberaese | ✅ Merged |
| Small Business Resources page | Erons Gberaese | ✅ Merged |
| ADP Workforce Home + Employment profile + tabs | Francisco Ponce | ✅ Merged |
| Edit Drawer | Lucas Mauricio | ✅ Merged |
| People menu (improved UI) | Erons Gberaese | ✅ Merged |
| Landing page fidelity improvements | Erons Gberaese | ✅ Merged |
| Payroll Dashboard page | Lucas Mauricio | ✅ Merged |
| Manage Payroll page | Lucas Mauricio | ✅ Merged |
| Pay Profile landing page | Francisco Ponce | ✅ Merged |
| HR Landing page | Erons Gberaese | ⏳ In Progress |
| HR Dashboard page | Erons Gberaese | ✅ Merged |
| Hire — location selector (US / INT) | Erons Gberaese | ✅ Merged |
| Benefits landing page | Francisco Ponce | ✅ Merged |
| Full payroll flow (Dashboard + Manage + Schedule + Worksheet + Pay Profile drawers) | Francisco Ponce | ✅ Merged (PR #43) |
| Take Action dropdown on Profiles page | Bhushan Suresh Bisen | ✅ Merged |
| Payroll dashboard backend integration | Francisco Ponce | ⏳ In Progress |
| Pay Profile sidebars/drawers (video reference) | — | ✅ Implemented in code (not tracked separately) |
| Worksheet grid | — | ✅ Implemented in code (not tracked separately) |
| Hire Form — Personal step (Step 1) | Erons Gberaese | ✅ Implemented in code |
| Hire Form — Employment step (Step 2) | Erons Gberaese | ✅ Implemented in code |
| Reports table | — | ❌ Not started |
| Hire / HR dashboard widgets | — | ❌ Not started |
| Benefits sidebars/drawers | — | ❌ Not started |
| Payroll full-year schedule w/ backend | — | ❌ Not started |
| Payroll start new payroll w/ backend | — | ❌ Not started |

---

## Feature Details by Area

### Public / Marketing Site

These pages exist outside the authenticated workforce shell.

#### Header (Lucas Mauricio — Merged PR #2)
- ADP-branded top navigation bar
- Logo, primary nav links, search, user controls

#### Footer (Erons Gberaese — Merged PR #3)
- Five-column nav grid: Contact Us, What We Offer, Who We Serve, Resources, About ADP
- Bottom bar: country selector (Radix Select), social icons (Facebook, X, YouTube, LinkedIn — custom SVG), legal links, disclaimer
- Inter variable font (replaces proprietary BentonSans/TaubSans)
- Shared `AdpLogo` and `SocialIcons` primitives reused in Header and Footer

#### Landing Page Sections (Lucas Mauricio — Merged)
- Hero section
- Stats section
- Product tour section
- Business size cards
- Contact form section

#### Landing Page Fidelity Improvements (Erons Gberaese — Merged PR #30)
- Hover backgrounds on Full Benefit Information and Helpful Links list rows
- Chart.js Doughnut in the My Pay widget
- Blue underlined labels on Dashboard icons
- "Things to Do" column renamed with DashBadge notifications pill
- Footer: added "Product Feedback" link, ExternalIcon for Privacy/Legal/Requirements, corrected dead routes to app-level 404

#### Support / Customer Service Pages (Erons Gberaese — Merged PR #23)

**`/contact-us/customer-service.aspx`**
- ADP Customer Service & Support page
- Two interactive Radix UI accordions: one for individual employees, one for company administrators
- Five wired admin links (employee password reset, W-2 & 1099, tax guides, administrator support, login & support) pointing to correct anchors on the Client Admin Support page
- Mobile app section with App Store / Google Play badges and a live QR code

**`/contact-us/support-for-client-administrators.aspx`**
- Nine collapsible accordion sections: Forms W-2, Overtime, Agency Notices, Password Reset, Payroll Schedule, Small Business Support, Product Support, Online Support, Security Alerts
- Anchor-based deep-linking: URL hash (e.g. `#resetpassword`) auto-opens the matching section and smooth-scrolls to it
- Side-by-side QR codes and state selector for regional support

#### Small Business Resources Page (Erons Gberaese — Merged PR #24)

---

### Dashboard UI Kit (Francisco Ponce — Merged PR #22)

Reusable primitives in `frontend/src/shared/ui/dashboard/`:

| Primitive | Type |
|---|---|
| Button | base component |
| Card | base component |
| Badge | base component |
| Table | base component |
| Input | base component |
| Avatar | base component |
| Dropdown Menu | Radix wrapper |
| Tooltip | Radix wrapper |
| Dialog | Radix wrapper |
| Popover | Radix wrapper |
| Select | Radix wrapper |
| Tabs | Radix wrapper |
| Toast | Radix wrapper |
| Sheet (slide-over) | Radix wrapper |
| List Item | layout primitive |
| Icon Button | composite |
| Calendar | date picker |

Dashboard color/theme tokens defined in `frontend/src/index.css`.

---

### Workforce App Shell

All routes under `/workforce` are protected — unauthenticated users are redirected to `/login`.

**Top bar (ADP Workforce Now style):**
- Brand logo / home link
- Global search input (stub)
- What's New, Things to Do, Calendar, Learn, Bridge, Support, Marketplace buttons (stubs)
- User profile menu showing initials

**Primary navigation:**

| Item | Behaviour |
|---|---|
| Home | → `/workforce` |
| Resources | Stub |
| Myself | Stub |
| People | Opens mega-menu → Profiles/Directory |
| Process | Opens mega-menu → Payroll Dashboard, Manage Payroll, Schedule, HR Dashboard |
| Reports & Analytics | Stub |
| Setup | Stub |
| Favorites | Stub |

---

### Home Dashboard — `/workforce`

*(Francisco Ponce — Merged; ADP Workforce Home PR #25)*

- Two large navigation cards: **HR** → `/workforce/process/hr/dashboard`, **Payroll** → `/workforce/process/payroll/dashboard`
- Chart.js Doughnut showing employee status distribution
- Quick-stat badges
- Links to relevant deep pages

---

### People / Profiles — `/workforce/people/profiles/directory`

*(Francisco Ponce — Merged; People module + Bhushan Suresh Bisen — Take Action PR #33; Erons Gberaese — People menu PR #26)*

**Employee list:**
- All employees fetched live from the backend API (up to 250, paginated on the backend)
- Each row card: Avatar (initials + color), Full Name, Job Code + Job Title, Status badge (Active / On Leave / Terminated), Position ID, Hire Date, masked Tax ID
- **Expand row** → reveals: Department, Rate Type, Associate ID, Company Code, Worked-In Country, ACA Worker Category, ACA Benefit Status, ACA Stability Period
- **Click employee name** → navigates to their profile

**Toolbar:**
- **Directory Filter dropdown** — filter by employment status (Active / Inactive / Terminated / All)
- **Prev / Next** — step through the filtered list; shows "X of Y" count
- **Search for People dialog** — modal search against the backend (name, job title, etc.)
- **Take Action dropdown** — context-sensitive actions for the highlighted employee (requires a row to be active)
- **Field Selector** — stub button

**People mega-menu** (Erons Gberaese — PR #26):
- Improved UI with quick-jump links to profile sub-sections

---

### Employee Profile — `/workforce/people/profiles/:profileId`

*(Francisco Ponce — Merged PR #25 + Lucas Mauricio — Edit Drawer)*

**Profile Header:**
- Avatar, Full Name, Job Title, Status badge, Hire Date, Position ID
- **Reveal SSN** eye button — API call to un-mask the SSN; toggles across header and Personal tab Tax ID card simultaneously

5 content tabs:

---

#### Personal Tab

| Section | What's displayed | Edit |
|---|---|---|
| **Name** | Legal Name (first/middle/last), Preferred/Chosen Name, Pronouns, Payroll Name, Professional Suffix, Other Last Names Used | ✅ Drawer |
| **Contact Information** | Personal Email (clickable mailto), Work Email (clickable mailto), Personal Mobile, Work Mobile, Work Phone; expandable for: Home Phone, Work Phone Extension, Mail Stop, Notification Email Preference, Personal/Work Fax, Personal/Work Pager + extensions | ✅ Drawer |
| **Demographics** | Birth Date (year hidden by default, eye toggle to reveal), computed Age, Preferred Language, Marital Status, Gender/Sex Self-ID, Sex; expandable for: Gender, Ethnicity/Race ID Method, Ethnicity, Race, Tobacco User, Education Level, Medicare, Medicaid, Number of Dependents | ✅ Drawer |
| **Addresses** | Legal Address, Alternate Address, Work Address, Works from Home | ✅ Drawer |
| **Tax ID** | Tax ID Type, masked Tax ID + eye-reveal button (API call to fetch full SSN) | ✅ Two-step drawer |
| **Profile Picture** | Placeholder with spec (PNG/JPG/JPEG up to 200 KB) | ❌ Stub |
| **Emergency Contacts** | Primary contact name, relationship, personal mobile; CA-specific notice; "Add a contact" / "Add a doctor" links | ✅ Drawer (form) |

**Edit Drawers:**

- **Edit Name** — Legal First/Middle/Last, Preferred Name, Pronouns, Payroll Name (Last, First Middle), Salutation, Generation Suffix, Professional Suffix, Other Last Names Used
- **Edit Contact Information** — Personal/Work email, Personal/Work/Home mobile, Work phone + extension, Mail Stop, Notification email preference, Fax fields, Pager fields. Validates email format and required fields before calling the API.
- **Edit Demographics** — Birth date, Sex, Marital Status, Gender/Sex Self-ID, Gender, Ethnicity/Race ID Method, Ethnicity, Race, Tobacco User, Education Level, Medicare, Medicaid, Number of Dependents
- **Edit Addresses** — Full form for Legal / Alternate / Work addresses: country (US/CA/MX/GB/AU), Address Lines 1-3, City, State (code extracted from "MI - Michigan" format), Zip, County. Works from Home toggle. Validates required fields (Line 1, City, State, Zip, Country) before saving.
- **Edit Tax ID** — Two-step: first shows masked SSN with an inline "Edit" button. Clicking Edit fetches the real unmasked SSN from the API and shows an edit form. Validation: must be exactly 9 digits; re-entry must match (unless "Applied for" checked). Save calls PATCH person endpoint.
- **Edit Emergency Contacts** — Contact name, relationship, personal mobile

---

#### Employment Tab

*(Part of Francisco Ponce — Merged PR #25)*

"Show as of" date stub for historical view. 3-column grid of section cards, all editable via slide-out drawers.

| Section | Key Fields |
|---|---|
| **Position** | Job Title, Reports To (with inline edit), Position Start Date, Management Position, Job Change Reason, Job Function, Worker Category, Pay Grade, Worked-In Country, Job Class, Position ID, FLSA, Company Code, NAICS Workers' Comp, File #, EEOC Job Classification, EEO Establishment, Officer/Owner |
| **Status** | Status, Hire Date (inline edit icon), Hire Reason, Termination Date/Reason, Voluntary/Involuntary, Severance Pay Start/End dates, Eligible for Rehire, Optional Rehire Status, Last Day Worked, Comments; On Leave block (Leave Return Date/Reason); Rehire block (Rehire Date/Reason) |
| **Regular Pay** | Monthly salary, Pay Frequency, Annual Salary link, Rate 2, Premium Rate Factors, Standard Hours, Use FLSA Overtime, Cancel Automatic Pay, Change Reason, Tipped Employee, Wage Entity; plus Incentive Pay link (stub), Salary step assignment (stub), Cancel automatic pay (stub) |
| **Corporate Groups** | Business Unit, Location, Benefits Eligibility Class, Union Code, Home Department, Union Local, Home Cost Number |
| **Employment** | Seniority Date, Associate ID, Credited Service Date, Hire Source, Adjusted Service Date, Normal/Early Retirement Dates, ACA Return Date |
| **Work Schedule** | FTE, Scheduled Hours, Assigned Shift, Hours Period; Time Off sub-section: Restricted Period Calendar, Default Start Time, Accrual Date, Default Request Hours |
| **Custom Fields** | HiBob ID, Guideline ID |
| **Additional Earnings** | Empty-state display; View/Add stub buttons |
| **Time & Attendance** | Edit drawer available; note: "This position doesn't use T&A" |

**Edit Drawers:**

- **Edit Position** — job title, worker category, management position, FLSA, Officer/Owner, pay grade, EEO establishment, job change reason, job description, job function, job class, NAICS, position start date, **position change effective date** (date picker). Shows pending change requests for this employee. Submitted as a `position` change request.
- **Status Details Drawer** (wide — 60vw) — shows current status history, lists scheduled change requests; "Change Status" button opens an elevated overlay drawer; "Change Hire Date" inline button opens a separate drawer
- **Change Status** (elevated drawer within StatusDetails) — new status selector (Active / Inactive / On Leave / Terminated / Deceased / Retired), required fields by status (Termination Date required for Terminated); submitted as an `employment_status` change request
- **Change Hire Date** — date picker for new hire date, validated before save
- **Regular Pay Details Drawer** (wide — 60vw) — shows compensation from the pay-profile API, allows salary/rate edits with an effective date; supports future-dated changes via `regular_pay` change request
- **Edit Corporate Groups** — Business Unit, Location, Benefits Eligibility Class, Union Code/Local, Home Cost Number
- **Edit Employment** — seniority/service/retirement dates, hire source
- **Edit Custom Fields** — HiBob ID, Guideline ID
- **Edit Time & Attendance** — T&A configuration form

**Change Request System:** Position, status, and regular-pay changes go through `/change-requests`. If `effective_date` ≤ today the backend applies the change immediately; if future-dated the record is stored as scheduled and surfaced in the drawer's pending-changes list.

---

#### Benefits Tab

*(Francisco Ponce — Merged PR #34)*

- Benefit plan enrollment table (plan name, type, coverage, employee cost, effective date, beneficiaries)
- "Show as of" date picker (stub — coming soon)
- Data fetched live via `useEmployeeBenefitsQuery`

> **Tracker note:** "Benefits sidebars/drawers" listed as not started — no edit drawers are implemented yet; the tab is read-only.

---

#### Talent Tab

8 sections, each with an **Add (+)** button that opens a `TalentAddDrawer`. All entries save to the backend API. Supports "Save & Add Another" flow.

| Section | Required fields | Optional fields |
|---|---|---|
| **Licenses & Certifications** | License/Cert type | Issued By, Effective Date, Expiration Date, License/Cert ID, Company-Paid Amount + Currency, Renewal Requirement, User Fields 1 & 2, Comments |
| **Education** | Degree | Learning Institution, Major, Second Major, Minor, Honorary Recognition, Credit Type, Credits Completed, GPA, Company-Paid Amount + Currency, User Fields, Comments |
| **Training** | Course Name | Subject, Start Date, Completion Date, Grade, Hours, Course Costs + Currency, Other Costs + Currency, Description |
| **Skills** | Skill | Proficiency Level, Date Acquired, Experience Years/Months, Last Used, Certification Required?, Test Required? |
| **Awards** | Name | Awarding Body, Date Awarded, Award Type |
| **Memberships** | Membership type | Issued By, Effective/Expiration dates, Membership ID, Chapter, Title, Company-Paid Amount + Currency, User Fields, Comments |
| **Previous Employers** | Company Name | Start/End dates, Website, Phone, Last Salary + Currency + Frequency, Reason for Leaving, Contact name/title/phone/email, User Fields, Comments; nested "Add Job Details" sub-form (Job Title, Job Class, Responsibilities) |
| **Languages** | Language | Spoken/Written Proficiency, Date Acquired, Native Speaker? |

---

#### Statutory Compliance Tab

| Section | What's shown | Edit |
|---|---|---|
| **I-9 / Citizenship** | US Work Authorization Status, I-9 Review Date, List A/B/C document types | ✅ Wide drawer with full document fields (issuing authority, document numbers, receipt/expiration dates) |
| **EEO** | Sex, Ethnicity, Race, EEOC Job Classification; links to EEO poster and EEO-1 report | ❌ Read-only |
| **Protected Veteran** | Status + discharge date | ✅ Drawer |
| **Section 503 Disability** | Disability Status | ❌ Edit button is a stub |
| **OSHA Records** | List of incidents (type, date, case #, description); per-record delete button | ❌ Add is a stub; Delete ✅ |
| **FMLA Records** | List of leaves (reason, start/end dates, status); per-record delete button | ❌ Add is a stub; Delete ✅ |
| **ADA Disability** | ADA status | ✅ Drawer |

---

### Pay Profile — `/workforce/pay/profile`

*(Francisco Ponce — Merged PR #32; drawers described in PR #43)*

**Employee selection:**
- Defaults to first active employee
- URL param `?employeeId=` for direct linking
- Prev / Next arrows step through the active employee list
- Mock mode via `?mock=1`

**Panels:**

| Panel | Description |
|---|---|
| **Pay Summary** | Collapsible strip — gross pay, net pay, deductions totals; "View all" opens Pay Summaries Drawer |
| **Regular Pay** | Base rate, pay type, frequency, standard hours; "Edit" opens Regular Pay Drawer |
| **Deductions** | List of deductions (read-only panel) |
| **Direct Deposits** | Bank accounts + split percentages; "Edit" opens Direct Deposits Drawer |
| **Federal Tax Withholding** | W-4 filing status, allowances; "Edit" opens Federal Tax Withholding Drawer |
| **State Withholding** | Worked-in state, marital status, exemptions; "Edit" opens State Withholding Drawer |
| **Local Tax** | Local tax jurisdictions (read-only panel) |
| **Accumulators** | YTD earnings accumulators + time-off accumulators |
| **Time Off** | Accrual balances and time-off settings |
| **Other Pay Settings** | Additional pay configuration |
| **Pay & Tax Statements Delivery** | Paper/electronic delivery preference |
| **WFN Panel** | Workforce Now system fields |

**Five slide-over drawers (all functional):**
1. **Regular Pay Drawer** — edit rate, pay type, standard hours
2. **Pay Summaries Drawer** — detailed pay statement history
3. **Direct Deposits Drawer** — bank accounts and split percentages
4. **Federal Tax Withholding Drawer** — W-4 filing status, allowances, additional withholding
5. **State Withholding Drawer** — state, marital status, exemptions, additional amounts

> **Tracker note:** "Pay Profile sidebars/drawers" listed without a status — they are fully implemented as part of PR #43.

---

### Payroll Dashboard — `/workforce/process/payroll/dashboard`

*(Lucas Mauricio — Merged; Francisco Ponce — backend integration In Progress)*

**Run cards** — infinite scroll, live data from API:

Each card shows: organization name, run number, run type, period, pay date, status badge, total gross pay, total hours, number of pays, pay group period dates.

**Actions by status:**

| Status | Actions |
|---|---|
| **Draft** | "Manage payroll" link (→ Manage page with this run) + "Calculate payroll" button → calls calculate API, transitions run to Calculated |
| **Calculated** | "Make changes" button → calls revert API, transitions back to Draft; "Approve" button → opens confirmation dialog |
| **Approved** | "Manage payroll" link + "View reports" stub + overflow "Mark as Complete" → calls complete API |
| **Submitted / Completed** | "View report" stub + "Start new payroll" stub + "Track your package" stub |

**Approve confirmation dialog:** Shows total gross pay (large display), pay date, and pay period. Requires explicit confirm before calling approve API.

**Right sidebar:**
- **Payroll Actions:** Start Input for Concurrent Payroll (stub), Start an off-cycle payroll (opens Off-Cycle dialog), Manage All Time Cycles (stub)
- **Smart Links:** Company Tax Documents, Statement of Deposit, Total Hours Worked, Tax Notices, Payroll Schedule (live link → `/workforce/process/payroll/schedule`), Quarter-End Dashboard, Tax Forms, Pay Profile (live link → `/workforce/pay/profile`), Validation Tables

**Off-Cycle Payroll Dialog:** Company Code dropdown, pay date picker, off-cycle type selector. Cancel / Create buttons.

**Toolbar:** Filters (stub), Multi-Company (stub), View history (stub).

---

### Manage Payroll — `/workforce/process/payroll/manage`

*(Lucas Mauricio — Merged; referenced as from video at 41:20)*

5 URL-backed tabs (`?tab=`):

| Tab | Content |
|---|---|
| **Things to Do** | Task checklist items for the active payroll run |
| **Worksheets** | List of worksheets for the run; each row links to `/workforce/process/payroll/manage/worksheet/:id` |
| **Other Payments** | Other payment entry list |
| **People Insights** | People-level payroll insights view |
| **Additional Insights** | Additional analytics view |

No drawers or dialogs in this area — navigation is via links and tab switching.

---

### Payroll Worksheet — `/workforce/process/payroll/manage/worksheet/:worksheetId`

*(Part of Francisco Ponce — PR #43 / Payroll flow)*

**Header:** Worksheet code, company name, pay period, in-balance / out-of-balance indicator, total employee count, back link to Manage Payroll.

**Toolbar:** Search/filter controls.

**Editable grid** (react-data-grid):

| Column | Editable |
|---|---|
| File No | Display |
| Name | Display |
| Regular Hours | ✅ |
| OT Hours | ✅ |
| Temp Rate | ✅ |
| Temp Desc | ✅ |
| Vacation | ✅ |
| Other Hours Code + Amount | ✅ |
| Reg Earnings | ✅ |
| Bonus | ✅ |
| Commission | ✅ |
| Other Earnings Code + Amount | ✅ |
| Adjust Deduction Code + Amount | ✅ |
| Rate Code | ✅ |
| Shift | ✅ |

**Summary rows** at the bottom sum all numeric columns.

**Footer action bar:** CANCEL / SAVE (calls `useWorksheetSaveMutation`) / AUTO BALANCE (disabled) / DONE.

No drawers or dialogs.

> **Tracker note:** "Worksheet grid landing page" listed without a status — fully implemented.

---

### Payroll Schedule — `/workforce/process/payroll/schedule`

*(Part of Francisco Ponce — PR #43 / Payroll flow)*

All state is URL-backed (`companyCode`, `selectedId`, `view`, `year`, `page`, `scheduleIdx`).

| Component | Description |
|---|---|
| **Schedule Header** | Company context, year selector, schedule index |
| **Schedule List** | Paginated table of pay periods — period number, dates, pay date, lock indicator, holiday indicator; first/prev/next/last pagination; List / Calendar view toggle |
| **Schedule Calendar** | Monthly calendar showing pay dates and period markers (shown when Calendar view is active) |
| **Schedule Details** | Right-panel detail view for the selected schedule entry |

No drawers or dialogs.

> **Tracker note:** "Payroll full-year schedule with backend integration" listed as not started — the page renders but schedule data integration is incomplete.

---

### HR Dashboard — `/workforce/process/hr/dashboard`

*(Erons Gberaese — Merged PR #36)*

- "Start Employee Change" controls
- **Things to Do table** — sortable, paginated task list
- Three right-column cards:
  - Create Surveys & Broadcasts
  - Hire & Onboard Employees (links to hire flow)
  - Your HR Setup
- Design tokens consistent with home page (`bg-white`, `border-[1.5px] border-[#ebebe7]`, `text-[#495686]` / `text-[#3950a1]` link colors)
- Chart.js Doughnut showing employee status breakdown
- Navigation card links to HR and Payroll areas

---

### Hire Landing Page — `/workforce/hr/hire`

*(Erons Gberaese — Merged PR #38)*

- **US / International toggle buttons** with corner triangle badge indicators (filled blue + white checkmark for active; neutral gray for inactive) — pixel-accurate match to ADP WFN reference
- Links to the hire form

---

### New Hire Form — `/workforce/hr/hire/new`

*(Erons Gberaese — implemented in code; Steps 1 & 2 listed in tracker as not tracked separately)*

A **6-step accordion wizard**. Steps are sequential (must complete previous to advance). Top stepper shows progress with check / number / eye icons per step.

---

**Step 1 — Personal** *(required)*

| Field Group | Fields | Required |
|---|---|---|
| Name | First, Middle, Last | First ✅ |
| Payroll Name | Last, First Middle | — |
| Associate ID | Pre-filled, editable | — |
| Contact | Personal Mobile (country code picker + number), Personal Email, "Use for notification" checkbox | — |
| Hire Details | Hire Date (date picker), Reason for Hire (New / Rehire / Transfer, clearable + Add), Company Code (from organizations API, clearable) | Hire Date ✅, Company Code ✅ |
| "Ask the New Hire" banner | Button to delegate personal info entry | Stub |
| Tax Identity | Tax ID Type (SSN / ITIN), Sex, Tax ID, Re-enter Tax ID, "Applied for" checkbox, Birth Date (date picker) | Tax ID ✅ (unless Applied for), Birth Date ✅ |
| Legal Address | Country, Address Line 1-3, City, State/Territory, Zip Code, County | Line 1 ✅, City ✅, State ✅ |
| Demographic | Section header; "More Fields" sidebar stubs (Ethnicity/Race ID method, Ethnicity, Race, Gender) | — |

Validation on "Next": First Name, Hire Date, Company Code, Address Line 1, City, State, Birth Date required. Tax ID must be 9 digits and match re-entry. Error banner lists all missing required fields.

Salary rounding: if annual salary is not evenly divisible by the number of pay periods, it is auto-adjusted and a warning banner is shown.

---

**Step 2 — Custom Fields**
- HiBob ID (optional text input)

---

**Step 3 — Employment** *(required)*

| Field | Required | Notes |
|---|---|---|
| File # | Read-only | System-generated |
| Job Title | ✅ | Dropdown with Add button (stub) |
| Worker Category | ✅ | CAS-Casual / Full-Time / Part-Time / Contractor, with Add button |
| Business Unit | — | Dropdown with Add button |
| Location | — | Dropdown with Add button |
| Home Department | — | Dropdown with Add button |
| Benefits Eligibility Class | ✅ | Dropdown with Add button |
| ACA Benefit Status | ✅ | Radio: Measurement / Full-Time |
| NAICS Workers' Comp | — | Dropdown with Add button |
| Work Email | ✅ | Validated format |
| "Use for notification" | — | Checkbox |
| I-9 Method | — | Radio: Yes electronically / Yes on paper |
| E-Verify Work Location | Shown if electronic I-9 | Select with explanation text |

Static read-only fields: Position ID (`—`), Employee Status (`Active`), Reports To (`No one` with inline edit stub).

---

**Step 4 — Payroll** *(required)*

| Field | Required | Notes |
|---|---|---|
| Pay Frequency | ✅ | Biweekly / Weekly / Semi-Monthly / Monthly |
| Compensation Type | — | Salary / Hourly |
| Annual Salary | ✅ | Auto-rounds to divisible by pay periods; shows warning + computed Salary per Pay |
| Data Control | — | Select with Add |
| Clock | — | Select with Add |
| Treasury Tipped Occupation Code | — | Select |
| Rate 2 Amount | ✅ | Pre-filled at 4.8078 |
| Basis of Pay | — | Salary / Hourly |
| Rate Multiplier | Display | 1.5 × 1.0 |
| Standard Hours | ✅ | Pre-filled at 80.00 |
| Pay Group | ✅ | Select (Use Period End Date 1 / 2) |

---

**Step 5 — Tax** *(required)*

| Field | Required |
|---|---|
| Federal Filing Status | ✅ |
| Worked In State | ✅ |
| SUI/SDI Tax Code | ✅ |
| State Marital Status | — |
| State Exemptions | — |
| Lived-In State / Locality | — |
| Worked-In Locality | — |
| Local Exemptions | — |
| SOC Code | — |
| Job Title Compliance | — |

---

**Step 6 — Review**
- Renders all collected data for confirmation before submitting
- Calls `createHireEmployee` API on final submit

---

## UI Flows

Step-by-step instructions for every major action in the app. All routes assume the user is already authenticated. If not, start with **Flow 1**.

---

### Flow 1 — Log In

1. Open `http://localhost:5174/login`
2. Enter a valid email address (e.g. `john.doe@example.com`) and click **Send OTP**
3. Check the email for a one-time code; enter it in the OTP field and click **Verify**
4. The app redirects to `/sessionid` — click **Continue**
5. You land on `/workforce` (Home Dashboard)

> **Shortcut for local dev:** The `/sessionid` page shows the current session and a **Copy** button. You are already logged in if the page shows "You're logged in".

---

### Flow 2 — Navigate the App Shell

From any page inside `/workforce`:

- **Home** — click the **WorkForce** logo (top-left) or the **Home** link in the primary nav
- **People → Directory** — click **People** in the primary nav → the mega-menu appears → click **Profiles** or **Directory**
- **Payroll Dashboard** — click **Process** in the primary nav → click **Payroll Dashboard**
- **Manage Payroll** — click **Process** → **Manage Payroll**
- **Payroll Schedule** — click **Process** → **Payroll Schedule**
- **HR Dashboard** — click **Process** → **HR Dashboard**
- **Pay Profile** — click **Process** → **Pay Profile**, or use the **Pay Profile** Smart Link in the Payroll Dashboard sidebar

---

### Flow 3 — Browse the Employee Directory

1. Navigate to **People → Profiles** (URL: `/workforce/people/profiles/directory`)
2. The list loads all active employees (default filter: Active)
3. **Change status filter** — click the **People with active status** dropdown → choose Active / Inactive / Terminated / All
4. **Page through** — use **Previous** / **Next** buttons; the counter shows "1 of 45"
5. **Expand a row** — click the **chevron/arrow** on any employee card to reveal Department, Rate Type, Associate ID, Company Code, ACA fields
6. **Open an employee profile** — click the employee's **name button**
7. **Search** — click **Search for people** → type name or job title → select a result to navigate to that profile

---

### Flow 4 — View and Edit an Employee Profile

1. From the Directory, click an employee name → lands on their **Employment** tab by default
2. Switch tabs by clicking **Personal** / **Employment** / **Benefits** / **Talent** / **Statutory Compliance** links under the profile header
3. **Reveal SSN** — click the eye icon **Show Tax ID** in the profile header; the SSN field in the header and on the Personal tab both unmask simultaneously
4. **Navigate between employees** — click **Previous profile** / **Next profile** arrows in the top-right of the profile header
5. **View business card** — click **View business card** in the header toolbar (shows contact info popover)

---

### Flow 5 — Edit Personal Information

From an employee's **Personal** tab:

#### Edit Name
1. Click **Edit Name** on the Name card
2. A slide-over drawer opens — update Legal First / Middle / Last, Preferred Name, Pronouns, Payroll Name, Salutation, Generation Suffix, Professional Suffix, Other Last Names
3. Click **Save** — drawer closes; card refreshes with new data

#### Edit Contact Information
1. Click **Edit Contact Information**
2. Update Personal / Work Email, Personal / Work / Home Mobile, Work Phone + Extension, Mail Stop, Notification Email Preference, Fax, Pager fields
3. Click **Save** (validates email format and required fields before calling the API)

#### Edit Demographics
1. Click **Edit Demographics**
2. Update Birth Date (date picker), Sex, Marital Status, Gender/Sex Self-ID, Gender, Ethnicity/Race ID Method, Ethnicity, Race, Tobacco User, Education Level, Medicare, Medicaid, Number of Dependents
3. Click **Save**

#### Edit Addresses
1. Click **Edit Addresses**
2. Select address type (Legal / Alternate / Work); choose Country (US / CA / MX / GB / AU)
3. Fill Address Line 1 (required), City (required), State (required), Zip (required), County
4. Toggle **Works from Home** if applicable
5. Click **Save** (validates required fields)

#### Edit Tax ID (two-step)
1. Click **Edit Tax ID** — drawer opens showing masked SSN
2. Click the inline **Edit** button — fetches the real SSN from the API
3. Edit the Tax ID field (must be 9 digits); re-enter to confirm (or check "Applied for" to skip)
4. Click **Save**

#### Edit Emergency Contacts
1. Click **Edit Emergency Contacts**
2. Fill Contact Name, Relationship, Personal Mobile
3. Click **Save**

---

### Flow 6 — Edit Employment Information

From an employee's **Employment** tab:

#### Change Position
1. Click **Edit Position** on the Position card
2. Drawer opens — update Job Title, Worker Category, Management Position, FLSA, Pay Grade, EEO Establishment, Job Change Reason, Position Start Date
3. Set **Position Change Effective Date** (date picker) — if future-dated, stored as a scheduled change request; if today or past, applied immediately
4. Click **Save** — any pending change requests for this employee are listed in the drawer header

#### Change Employment Status
1. Click **Edit Status** on the Status card → **Status Details Drawer** opens (60vw wide)
2. Review current status history and any pending scheduled changes
3. Click **Change Status** → an elevated overlay drawer opens
4. Select new status: Active / Inactive / On Leave / Terminated / Deceased / Retired
5. For **Terminated**: fill Termination Date (required), Voluntary/Involuntary, Reason, Last Day Worked
6. For **On Leave**: fill Leave Return Date, Leave Reason
7. Click **Save** — submitted as an `employment_status` change request

#### Change Hire Date
1. In the Status Details Drawer, click **Change Hire Date**
2. Pick a new hire date from the date picker
3. Click **Save**

#### Edit Regular Pay
1. Click the salary amount link or **Regular Pay information** button on the Regular Pay card → **Regular Pay Details Drawer** opens (60vw wide)
2. Edit Annual Salary / Rate; set the **Effective Date**
3. If effective date is in the future, the change is stored as a scheduled `regular_pay` change request
4. Click **Save**

#### Edit Corporate Groups
1. Click the edit icon on the Corporate Groups card
2. Update Business Unit, Location, Benefits Eligibility Class, Union Code, Union Local, Home Cost Number
3. Click **Save**

---

### Flow 7 — View Benefits

1. Navigate to an employee profile → click the **Benefits** tab
2. The enrollment table loads live data (plan name, type, coverage, employee cost, effective date, beneficiaries)
3. No edit actions yet — tab is read-only

---

### Flow 8 — Add a Talent Entry

From an employee's **Talent** tab:

1. Locate the section you want to add to (Licenses, Education, Training, Skills, Awards, Memberships, Previous Employers, Languages)
2. Click the **(+) Add** button on that section
3. A `TalentAddDrawer` slides open with the form fields for that entry type
4. Fill the required field (each section has exactly one required field — e.g., "License/Cert type" for Licenses, "Degree" for Education)
5. Fill optional fields as needed
6. Click **Save** — entry is added and the drawer closes
7. To add another entry immediately: click **Save & Add Another** — the form clears but the drawer stays open

---

### Flow 9 — Edit Statutory Compliance

From an employee's **Statutory Compliance** tab:

#### I-9 / Citizenship
1. Click **Edit** on the I-9 / Citizenship card
2. Wide drawer opens — update US Work Authorization Status, I-9 Review Date, List A / B / C document fields (issuing authority, document numbers, receipt and expiration dates)
3. Click **Save**

#### Protected Veteran
1. Click **Edit** on the Protected Veteran card
2. Update Veteran Status and Discharge Date
3. Click **Save**

#### ADA Disability
1. Click **Edit** on the ADA Disability card
2. Update ADA Status
3. Click **Save**

#### Delete an OSHA or FMLA Record
1. Locate the record in the OSHA Records or FMLA Records list
2. Click the **Delete** (trash) icon on that row
3. Confirm the deletion — record is removed from the list

---

### Flow 10 — View and Edit Pay Profile

1. Navigate to **Process → Pay Profile** (URL: `/workforce/pay/profile`)
2. The page defaults to the first active employee; their SSN is masked and status badge is shown
3. **Switch employees** — click **Previous employee** / **Next employee** arrows, or append `?employeeId=<uuid>` to the URL
4. **Expand/collapse panels** — click any panel header (Regular Pay, Deductions, Direct Deposits, Federal, State, Local, Accumulators, Time Off, Other Pay Settings, Delivery, WFN) to expand/collapse
5. **View pay summary history** — click **View all pay summaries** or **Full pay summary** → Pay Summaries Drawer opens (shows historical pay statements)

#### Edit Regular Pay
1. Click **Regular Pay** button/heading on the Regular Pay panel
2. Regular Pay Drawer slides open — edit Rate, Pay Type, Standard Hours
3. Click **Save**

#### Edit Direct Deposits
1. Click **Direct Deposits** button on the Direct Deposits panel
2. Drawer opens — add/edit bank accounts and split percentages
3. Click **Save**

#### Edit Federal Tax Withholding
1. Click **Federal** button on the Federal Withholding panel
2. Drawer opens — update W-4 Filing Status, Allowances, Additional Withholding amount
3. Click **Save**

#### Edit State Withholding
1. Click **State** button on the State Withholding panel
2. Drawer opens — update Worked-In State, Marital Status, Exemptions, Additional State Withholding
3. Click **Save**

---

### Flow 11 — Run a Payroll (Full Lifecycle)

Navigate to **Process → Payroll Dashboard** (URL: `/workforce/process/payroll/dashboard`).

The dashboard shows payroll run cards in reverse chronological order, loaded with infinite scroll.

#### Calculate a Draft payroll
1. Find a card with status **Draft** (status badge visible; "Due [date]" label)
2. Click **Calculate payroll** — API call is made; the card status transitions to **Calculated** and the "Last calculated at" timestamp updates

#### Revert a Calculated payroll back to Draft
1. Find a **Calculated** card (shows "Awaiting approval")
2. Click **Make changes** — the run reverts to Draft status

#### Approve a Calculated payroll
1. Find a **Calculated** card
2. Click **Approve** — a confirmation dialog opens showing Total Gross Pay, Pay Date, and Pay Period
3. Review the totals and click **Approve** in the dialog — status transitions to **Approved**

#### Complete an Approved payroll
1. Find an **Approved** card
2. Click **More options** (⋯) → click **Mark as Complete** — status transitions to **Completed**
3. The card shows "Payroll Complete — Congratulations!" message

#### Start an Off-Cycle Payroll
1. In the right sidebar, under **Payroll Actions**, click **Start an off-cycle payroll**
2. Dialog opens — select Company Code from the dropdown, pick Pay Date, select Off-Cycle Type
3. Click **Create off-cycle payroll**

---

### Flow 12 — Manage Payroll

Navigate to **Process → Manage Payroll** (URL: `/workforce/process/payroll/manage`), or click **Manage payroll** on a Draft payroll run card.

The header shows the payroll run context (company, pay date/period, total gross, total hours, employee count).

**Switch tabs** by clicking:
- **Things to Do** — task checklist (`?tab=things-to-do`)
- **Worksheets** — list of worksheets for this run (`?tab=worksheets`)
- **Other Payments** — additional payment entries (`?tab=other-payments`)
- **People Insights** — employee-level insights (`?tab=people-insights`)
- **Additional Insights** — analytics view (`?tab=additional-insights`)

**From the Worksheets tab:**
1. The list shows available worksheets (Bi-Weekly Hourly, Bi-Weekly Salary, etc.)
2. Click a worksheet row to open it at `/workforce/process/payroll/manage/worksheet/:worksheetId`

---

### Flow 13 — Edit a Payroll Worksheet

Navigate to a worksheet (URL: `/workforce/process/payroll/manage/worksheet/:worksheetId`).

1. The grid loads all employees for this worksheet. Columns: File No (read-only), Name (read-only), Regular Hours, OT Hours, Temp Rate, Temp Desc, Vacation, Other Hours Code, Other Hours Amount, Reg Earnings, Bonus, Commission, Other Earnings Code, Other Earnings Amount, Adjust Deduction Code, Adjust Deduction Amount, Rate Code, Shift
2. **Edit a cell** — click any editable cell; type the new value; press Tab or Enter to move to the next cell
3. **Summary rows** at the bottom of the grid auto-update as you edit numeric columns
4. **Search/filter employees** — use the **Search** toolbar above the grid
5. **Save** — click **SAVE** in the footer action bar; changes are sent to the API via `useWorksheetSaveMutation`
6. **Cancel** — click **CANCEL** to discard unsaved changes
7. **Return** — click **Back to Manage Regular Payroll** link in the header

---

### Flow 14 — View Payroll Schedule

Navigate to **Process → Payroll Schedule** (URL: `/workforce/process/payroll/schedule`).

1. **Select company / year** — use the Company and Year controls in the Schedule Header
2. **Browse pay periods** — the Schedule List shows all pay periods: period number, start/end dates, pay date, lock indicator, holiday indicator
3. **Paginate** — use First / Prev / Next / Last pagination controls
4. **Switch view** — click **List** or **Calendar** toggle; in Calendar view a monthly calendar shows pay dates and period markers
5. **Select a period** — click a row in the list (or a date in calendar view) → the Schedule Details panel on the right updates with full detail for that entry

---

### Flow 15 — HR Dashboard Actions

Navigate to **Process → HR Dashboard** (URL: `/workforce/process/hr/dashboard`).

#### Start an Employee Change
1. Under **Start Employee Change**, click one of:
   - **Job Change** — navigates to the Job Change flow
   - **Place on Leave** — navigates to the Leave flow
   - **Terminate Employment** — navigates to the Termination flow
   - **Add Another Position** — navigates to the Add Position flow

#### Work through Things to Do tasks
1. The **Things to Do** table shows pending tasks with a **Continue** link per row
2. Click **Continue** on a task to navigate to the relevant action
3. Use **Previous page** / **Next page** to paginate through tasks

#### Go to Hire & Onboard
1. Under the **Hire & Onboard Employees** card, click **Hire someone** → navigates to `/workforce/hr/hire`

---

### Flow 16 — Hire a New Employee (6-Step Wizard)

Navigate to **HR Dashboard → Hire someone** (URL: `/workforce/hr/hire`), then click the hire button → `/workforce/hr/hire/new`.

The accordion wizard shows all steps in a stepper at the top. Steps must be completed in order.

#### Step 1 — Personal
1. Enter **First Name** (required)
2. Optionally fill Middle Name, Last Name, Payroll Name, Preferred Name
3. Enter **Personal Mobile** (select country code with the flag picker, then enter number) and **Personal Email**
4. Set **Hire Date** (date picker, required) and **Reason for Hire** (New / Rehire / Transfer)
5. Select **Company Code** from the organizations dropdown (required)
6. Under **Tax Identity**: select Tax ID Type (SSN / ITIN), enter **Sex**, enter **Tax ID** (9 digits), re-enter to confirm, set **Birth Date** (required)
7. Under **Legal Address**: fill **Address Line 1** (required), **City** (required), **State** (required), Zip, County; select Country
8. Click **Next** — validation runs; any missing required fields are listed in a red error banner at the top

#### Step 2 — Custom Fields
1. Optionally enter **HiBob ID**
2. Click **Next**

#### Step 3 — Employment
1. Select **Job Title** (required, dropdown)
2. Select **Worker Category** (required): CAS-Casual / Full-Time / Part-Time / Contractor
3. Optionally set Business Unit, Location, Home Department
4. Select **Benefits Eligibility Class** (required)
5. Select **ACA Benefit Status** (required): Measurement or Full-Time
6. Optionally set NAICS Workers' Comp code
7. Enter **Work Email** (required, validated format)
8. Choose **I-9 Method** radio: "Yes electronically" or "Yes on paper"
9. If electronic I-9, select **E-Verify Work Location**
10. Click **Next**

#### Step 4 — Payroll
1. Select **Pay Frequency** (required): Biweekly / Weekly / Semi-Monthly / Monthly
2. Select **Compensation Type**: Salary or Hourly
3. Enter **Annual Salary** (required) — if the amount is not evenly divisible by the number of pay periods, a warning banner appears with the auto-rounded amount and the computed Salary per Pay
4. Set **Standard Hours** (required, pre-filled at 80.00)
5. Select **Pay Group** (required)
6. Optionally set Data Control, Clock, Rate 2 Amount, Basis of Pay
7. Click **Next**

#### Step 5 — Tax
1. Select **Federal Filing Status** (required)
2. Select **Worked In State** (required)
3. Select **SUI/SDI Tax Code** (required)
4. Optionally set State Marital Status, State Exemptions, Lived-In State, Worked-In Locality, Local Exemptions, SOC Code, Job Title Compliance
5. Click **Next**

#### Step 6 — Review
1. All collected data is rendered for review (Personal, Employment, Payroll, Tax sections)
2. Review each field for accuracy
3. Click **Submit** — calls `createHireEmployee` API; on success the new employee record is created and you are redirected to their profile

**At any time:**
- Click **Save and exit** to preserve progress and return later
- Click **Cancel** to discard the draft and exit the wizard

---

## What's Implemented But Not in the Tracker

These features exist in the live codebase but were not listed in the tracker CSV:

| Feature | Location |
|---|---|
| Passwordless auth (OTP / magic link) | `/login` → `/sessionid` |
| Session management / ProtectedRoute | `src/app/protected-route.tsx` |
| "Coming Soon" toast system | `use-coming-soon.tsx` hook |
| Search for People dialog | `search-for-people-dialog.tsx` |
| Business Card dialog (contact info popover) | `business-card-dialog.tsx` |
| Status Details Drawer (60vw, w/ change request list) | `status-details-drawer.tsx` |
| Regular Pay Details Drawer (60vw, w/ effective dating) | `regular-pay-details-drawer.tsx` |
| Employee Change Request system (effective dating) | `use-employee-query.ts` + backend |
| SSN reveal API integration (profile header + Tax ID card) | `revealEmployeeSsn` + outlet context |
| Full Talent tab — all 8 entry types with API persistence | `talent-page.tsx` + `use-employee-query.ts` |
| Statutory Compliance — I-9, Protected Veteran, ADA drawers | `statutory-compliance-page.tsx` |
| OSHA / FMLA record delete | `statutory-compliance-page.tsx` |

---

## Features Marked "Coming Soon" (Visible Stubs)

These UI elements exist and show a "Coming Soon" toast instead of performing an action:

- Global search, Calendar, What's New, Things to Do (header), Learn, Bridge, Support, Marketplace
- Resources / Myself / Reports & Analytics / Setup nav items
- Walk Me Through (payroll dashboard)
- History icons on Employment section cards (Position, Status, Corporate Groups, Employment, Custom Fields)
- Payroll run: Filter, Multi-Company toggle, View history
- Payroll run (completed): View report, Start new payroll, Track your package
- "Things to Do" count link on payroll cards
- Supporting documents attachment on Status card
- Print on Employment tab
- Regular Pay card: Incentive Pay, Salary step assignment, Cancel automatic pay
- Field Selector on Profiles page
- "Add a contact" / "Add a doctor" on Emergency Contacts card
- OSHA Add / FMLA Add buttons
- Section 503 Disability Status edit button
- EEO poster / EEO-1 report links
- Profile picture upload
- Show as of date picker (Employment tab, Benefits tab)
- "Ask the New Hire" on Hire Form
- Hire Form sidebar "More Fields" links
