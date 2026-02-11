# Feature Summary (Excel-Ready)

## Per Category

### IKEA Furniture Assembly
- 0% complete
- No files exist, route defined in constants only

### Moving
- 5% complete
- ComingSoon placeholder only

### Truck Assisted Help Moving
- 5% complete
- ComingSoon placeholder only

### Furniture Assembly
- 60% complete (on branch)
- Branch: `fb-furniture-assembly`, PR #9
- Has dedicated form (1066 lines), payment & recommendations pages
- "UI matches TaskRabbit with some quirks"
- Copy-pasted from TV mounting — leftover variable names and wrong route (`/trash-removal/recommendations`)
- Furniture type onChange handler is empty, "Your Items" section non-functional
- Created duplicate constants file at `/src/constants/index.ts`
- Conflicts with `fb-cleaning` on 3 shared files

### General Mounting
- 5% complete
- ComingSoon placeholder only

### Cleaning
- 70% complete (on branch)
- Branch: `fb-cleaning`, PR #11
- 4 pages: form, recommendations, schedule, confirm-details
- Frequency discounts: Weekly 15%, Bi-weekly 10%, Monthly 5%
- Bug: `CLEANING_FREQUENCY_SELECTION` undefined at runtime on "Select & Continue"
- Bug: `getTaskLabel()` missing cleaning case — falls through to generic "tasks" label
- Progress bar header duplicated (~80-100 lines) across all 4 pages — should be shared component
- No cleaning-specific tasker data — reuses generic dataset aliased as `cleaningTaskersData`
- Mixed sessionStorage/localStorage strategy — can cause stale data
- No form validation, extensive `any` types, `console.log` and `alert()` left in
- Hardcoded date (29) and time ("5:30pm") defaults
- "Unit or apt #" required but shouldn't be
- Taskers are generic, need category-specific ones
- Calendar shows immediately, should have loader
- Calendar styling doesn't match original site

### TV Mounting
- 100% complete, merged to main
- Unique 3-step flow (all others use 4)
- Custom form: TV count, mount types, wall material, add-ons
- Self-contained single page

### Heavy Lifting & Loading
- 100% complete, merged to main
- Standard 4-step flow
- Form: location + task size + details
- Self-contained single page

### Electrical Work
- 100% complete, merged to main (PR #10)
- Standard 4-step flow
- Form identical to Heavy Lifting, could be generalized

### Plumbing Help
- 70% complete
- Sub-pages exist (details, recommendations, schedule, confirm)
- Main entry page still ComingSoon — needs redirect
- Uses shared DescribeTaskSection

### Yard Work
- 70% complete
- Same pattern as Plumbing — ComingSoon entry, sub-pages built
- Has dedicated tasker data file (223KB)
- Uses shared DescribeTaskSection

### Trash & Furniture Removal
- 75% complete
- Same pattern as Plumbing/Yard — ComingSoon entry, sub-pages built
- Largest details page (508 lines), unique vehicle selector
- Has dedicated tasker data file (218KB)
- Uses shared DescribeTaskSection

### Indoor Painting
- 5% complete
- ComingSoon placeholder only

### Furniture Repair
- 5% complete
- ComingSoon placeholder only

### Run Errands
- 5% complete
- ComingSoon placeholder only

### Landscaping
- 5% complete
- ComingSoon placeholder only

### Flooring
- 0% complete
- No files, no route

### Wall Repair
- 5% complete
- ComingSoon placeholder only

---

## Shared Components
- `DescribeTaskSection` — Cleaning, Plumbing, Yard Work, Trash Removal
- `BrowseTaskersSection` — Cleaning, Trash Removal, Yard Work, Electrical Work, Heavy Lifting
- `ConfirmDetailsSection` — all standard 4-step categories
- `ConfrimFrequencyDetailsSection` — Cleaning only
- `ChooseDateTimeSection` — all standard 4-step categories
- `TaskerProfileModal` — all categories with tasker browsing
- 39 shared UI components (Header, Footer, ProgressBar, PaymentForm, Calendar, etc.)

---

## Cross-Branch Conflicts
- `booktask/page.tsx` — modified by both `fb-cleaning` and `fb-furniture-assembly`
- `BrowseTaskersSection.tsx` — modified by both branches
- `constants.ts` — modified by both branches
- Recommendation: merge `fb-cleaning` first, then rebase `fb-furniture-assembly`

---

## Open PRs (4)
- PR #11: Fb cleaning — full 4-step flow, has bugs and code quality issues noted above
- PR #9: Fb furniture assembly — full flow but copy-pasted from TV mounting with leftover artifacts
- PR #4: Reusable Details Page — competing architecture using Radix/Tailwind/lucide instead of MUI, uses "TaskBee" branding, SF locations (vs NY/NJ in main)
- PR #3: Header with Progress Bar — subset of PR #4, adds HeaderWithProgressBar and Logo components

---

## Global Issues
- Generic taskers shared across categories; only Yard Work & Trash Removal have dedicated data
- `CLEANING_FREQUENCY_SELECTION` runtime error (undefined, missing import/export)
- Typo: `ConfrimFrequencyDetailsSection` should be `ConfirmFrequencyDetailsSection`
- Two competing form architectures: `DescribeTaskSection` (MUI) vs `details_page_component` branch (Radix/Tailwind) — PRs #3/#4
- PRs #3/#4 add Radix UI + tailwind-merge + lucide-react dependencies that conflict with MUI direction
- Duplicate constants file: `/src/constants/index.ts` created by PR #9 alongside existing `/src/config/constants.ts`
- MUI in 75 files, Tailwind in only 4 — should migrate those 4 and drop Tailwind
- Hardcoded date (29) and time ("5:30pm") defaults in both Cleaning and Furniture Assembly PRs
- Orphaned branch: `feature/landscape` (no merge base with main)
- IKEA Furniture Assembly and Flooring have zero files
