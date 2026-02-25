# Feature Analysis: Task Categories

_Last updated: 2026-01-31_

## Summary Table

| # | Category | Route | Status | Branch | Completeness |
|---|----------|-------|--------|--------|-------------|
| 1 | IKEA Furniture Assembly | `/ikea-assembly` | No files | — | 0% |
| 2 | Moving | `/moving` | ComingSoon | — | 5% |
| 3 | Truck Assisted Help Moving | `/truck-moving` | ComingSoon | — | 5% |
| 4 | Furniture Assembly | `/furniture-assembly` | ComingSoon (main), Active (branch) | `fb-furniture-assembly` (9 ahead) | 60% (branch) |
| 5 | General Mounting | `/mounting` | ComingSoon | — | 5% |
| 6 | Cleaning | `/cleaning` | Full flow (branch) | `fb-cleaning` (2 ahead) | 70% |
| 7 | TV Mounting | `/tv-mounting` | **Complete** (main) | Merged | 100% |
| 8 | Heavy Lifting & Loading | `/heavy-lifting` | **Complete** (main) | Merged | 100% |
| 9 | Electrical Work | `/electrical-work` | **Complete** (main) | Merged | 100% |
| 10 | Plumbing Help | `/plumbing-help` | ComingSoon entry, sub-pages exist | — | 70% |
| 11 | Yard Work | `/yard-work` | ComingSoon entry, sub-pages exist | — | 70% |
| 12 | Trash & Furniture Removal | `/trash-removal` | ComingSoon entry, sub-pages exist | — | 75% |
| 13 | Indoor Painting | `/indoor-painting` | ComingSoon | — | 5% |
| 14 | Furniture Repair | `/furniture-repair` | ComingSoon | — | 5% |
| 15 | Run Errands | `/run-errands` | ComingSoon | — | 5% |
| 16 | Landscaping | `/landscaping` | ComingSoon | — | 5% |
| 17 | Flooring | `/flooring` | No files | — | 0% |
| 18 | Wall Repair | `/wall-repair` | ComingSoon | — | 5% |

**Totals:** 3 complete on main, 1 in-progress on branch (Cleaning), 1 on feature branch (Furniture Assembly), 3 partially built (Plumbing/Yard/Trash), 8 ComingSoon placeholders, 2 with no files at all.

---

## Detailed Per-Category Analysis

### 1. IKEA Furniture Assembly
- **Route:** `/ikea-assembly` (defined in constants but no directory exists)
- **Files:** None
- **Notes:** Route is in `CATEGORY_ROUTES` but no `src/app/ikea-assembly/` directory was created.

### 2. Moving
- **Route:** `/moving`
- **Files:** `src/app/moving/page.tsx` (~7 lines, ComingSoon)
- **Notes:** Placeholder only.

### 3. Truck Assisted Help Moving
- **Route:** `/truck-moving`
- **Files:** `src/app/truck-moving/page.tsx` (~7 lines, ComingSoon)

### 4. Furniture Assembly
- **Route:** `/furniture-assembly`
- **Branch:** `fb-furniture-assembly` (9 commits ahead of main, last activity 2025-11-14)
- **PR:** [#9](https://github.com/turing-rlgym/task-rabbit/pull/9)
- **Main branch:** ComingSoon placeholder
- **Branch has:**
  - `FurnitureAssemblyForm.tsx` (1066 lines) — dedicated form component (exports as `DescribeTaskSection`, confusing naming)
  - 3 new pages: `confirm-details/`, `recommendations/`, `schedule/`
  - Added `FURNITURE_ASSEMBLY_OPTIONS` constant (IKEA only, non-IKEA, both)
  - Created duplicate constants file at `/src/constants/index.ts`
  - Modifies shared files: `booktask/page.tsx`, `BrowseTaskersSection.tsx`, `constants.ts`
- **User notes:** "UI matches TaskRabbit with some quirks"
- **PR review issues:**
  - Copy-paste from TV mounting — variable names and comments from TV mounting remain
  - Wrong route: `router.push('/trash-removal/recommendations')` — should be furniture-assembly
  - Furniture type selection onChange is empty (`console.log("onChange")`)
  - "Your Items" section UI present but non-functional
  - Hardcoded date (`29`) and time (`"5:30pm"`) defaults
- **Conflict risk:** 3 shared files also modified by `fb-cleaning` (see Cross-Branch Conflicts below)

### 5. General Mounting
- **Route:** `/mounting`
- **Files:** ComingSoon placeholder
- **Notes:** No implementation yet per user's manual analysis.

### 6. Cleaning
- **Route:** `/cleaning`
- **Branch:** `fb-cleaning` (2 commits ahead, last activity 2025-11-17)
- **PR:** [#11](https://github.com/turing-rlgym/task-rabbit/pull/11)
- **Pages:**
  - `src/app/cleaning/page.tsx` (400 lines) — main form with location, job size, details
  - `src/app/cleaning/recommendations/page.tsx` (284 lines) — browse taskers
  - `src/app/cleaning/schedule/page.tsx` (376 lines) — date/time selection
  - `src/app/cleaning/confirm-details/page.tsx` (260 lines) — confirmation with frequency discounts
- **Special features:**
  - Frequency discount selection: One-time (0%), Weekly (15%), Bi-weekly (10% — "Most Popular"), Monthly (5%)
  - Uses `ConfrimFrequencyDetailsSection.tsx` (63.6KB) for frequency logic
- **Known bugs:**
  - `TypeError: Cannot read properties of undefined (reading 'map')` at `ConfrimFrequencyDetailsSection.tsx:297` — `CLEANING_FREQUENCY_SELECTION` is undefined when `.map()` is called on "Select & Continue"
  - `getTaskLabel()` in `BrowseTaskersSection` has no `"cleaning"` case — falls through to default `"tasks"` label
  - `savedFrequency` loaded from localStorage but never assigned to state
- **Code quality issues (from PR #11 review):**
  - Progress bar header duplicated verbatim (~80-100 lines) across all 4 pages — should be extracted to shared component
  - Large portion of diff is formatting (single→double quotes); actual feature code is smaller than 4300-line diff suggests
  - No cleaning-specific tasker data file — reuses generic `taskersData` aliased as `cleaningTaskersData`
  - Mixed storage: `sessionStorage` for cross-page wizard state, `localStorage` for persistence — inconsistent, can cause stale data
  - No form validation beyond `trim()` checks; no error messages shown to user
  - `ConfrimFrequencyDetailsSection` is 1637 lines — needs decomposition
  - Booking confirmation uses `alert()` placeholder
  - Extensive `any` types (`selectedTasker`, `cardData`, `handleSelectTasker`)
  - `console.log` statements left in
  - Hardcoded date (`29`) and time (`"5:30pm"`) defaults
- **User notes:**
  - Original TaskRabbit does NOT require "Unit or apt #" before continuing; clone does
  - Taskers are generic/shared across all categories — need category-specific tasker data
  - Calendar appears immediately; should show a loader first
  - Calendar font sizes, background colors, font colors don't match original site

### 7. TV Mounting ✓
- **Route:** `/tv-mounting`
- **Status:** Complete on main (branch merged)
- **Pages:** `src/app/tv-mounting/page.tsx` — all-in-one page with internal step management
- **Step flow:** 3 steps (unique among all categories; others use 4)
  - Step 1: Tell us about your task
  - Step 2: Schedule your task
  - Step 3: Confirm details → Booking confirmed
- **Components:**
  - `TVMountingForm.tsx` (13KB) — TV count (1-5), help lifting options, mount types (fixed/tilting/articulating), wall material, add-on services (hide wires, install speakers, device setup)
  - `TVMountingScheduling.tsx` (7.9KB)
  - `TVMountingSummary.tsx` (2.6KB)
- **Form data type:** `TVMountingFormData` with specialized fields

### 8. Heavy Lifting & Loading ✓
- **Route:** `/heavy-lifting`
- **Status:** Complete on main (branch merged)
- **Pages:** `src/app/heavy-lifting/page.tsx` — all-in-one page, standard 4-step flow
- **Components:** `HeavyLiftingForm.tsx` (9.7KB), `TaskerList.tsx`, `DateTimeSelectionModal`
- **Form:** Location + task size (small/medium/large) + task details
- **User notes:** User initially described it as "very minimal" but full 4-step flow is implemented.

### 9. Electrical Work ✓
- **Route:** `/electrical-work`
- **Status:** Complete on main (branch merged via PR #10)
- **Pages:** `src/app/electrical-work/page.tsx` — all-in-one page, standard 4-step flow
- **Components:** `ElectricalWorkForm.tsx` (9.8KB) — structurally identical to Heavy Lifting form
- **Notes:** Same form pattern as Heavy Lifting (location + size + details). Could potentially be generalized.

### 10. Plumbing Help
- **Route:** `/plumbing-help`
- **Main page:** ComingSoon placeholder
- **Sub-pages exist:**
  - `details/page.tsx` — task description form
  - `recommendations/page.tsx` — tasker browsing
  - `schedule/page.tsx` — date/time selection
  - `confirm-details/page.tsx` — confirmation
- **Uses:** `DescribeTaskSection` shared component with localStorage keys `plumbing_task_*`
- **Blocker:** Main entry page still shows ComingSoon — needs to redirect to `/plumbing-help/details`

### 11. Yard Work
- **Route:** `/yard-work`
- **Pattern:** Same as Plumbing Help — ComingSoon entry, full sub-page flow exists
- **Sub-pages:** `details/`, `recommendations/`, `schedule/`, `confirm-details/`
- **Data:** Has dedicated `yardWorkTaskersData.ts` (223KB) — category-specific tasker data
- **Uses:** `DescribeTaskSection` shared component

### 12. Trash & Furniture Removal
- **Route:** `/trash-removal`
- **Pattern:** Same as Plumbing/Yard — ComingSoon entry, full sub-page flow
- **Notable:** Largest details page (508 lines) — includes unique vehicle requirement selector (None/Car/Truck)
- **Data:** Has dedicated `trashRemovalTaskersData.ts` (218KB)
- **Uses:** `DescribeTaskSection` shared component

### 13–18. Not Started Categories

All of these have only a ComingSoon placeholder page (~7 lines):

| Category | Route | Files |
|----------|-------|-------|
| Indoor Painting | `/indoor-painting` | `page.tsx` (ComingSoon) |
| Furniture Repair | `/furniture-repair` | `page.tsx` (ComingSoon) |
| Run Errands | `/run-errands` | `page.tsx` (ComingSoon) |
| Landscaping | `/landscaping` | `page.tsx` (ComingSoon) |
| Wall Repair | `/wall-repair` | `page.tsx` (ComingSoon) |
| Flooring | — | No route, no files |

---

## Shared Components

### Task Flow Components (`src/components/task/`)

| Component | Size | Used By |
|-----------|------|---------|
| `DescribeTaskSection.tsx` | 30.7KB | Cleaning, Plumbing, Yard Work, Trash Removal |
| `BrowseTaskersSection.tsx` | 43.8KB | Cleaning, Trash Removal, Yard Work, Electrical Work, Heavy Lifting |
| `ConfirmDetailsSection.tsx` | 53.9KB | Standard 4-step categories |
| `ConfrimFrequencyDetailsSection.tsx` | 63.6KB | Cleaning only |
| `ChooseDateTimeSection.tsx` | 10.3KB | Standard 4-step categories |
| `TaskerProfileModal.tsx` | 11KB | All categories with tasker browsing |

### UI Components (`src/components/ui/`)

39 shared UI components. Key ones used across categories:

- **Navigation:** `Header.tsx`, `Footer.tsx`, `ProgressBar.tsx`, `CategoryInfo.tsx`
- **Forms:** `Input.tsx`, `Checkbox.tsx`, `RadioButton.tsx`, `LocationSelector.tsx`, `FormSection.tsx`
- **Payment:** `PaymentForm.tsx` (14KB), `BookingSummary.tsx`, `PaymentConfirmationPage.tsx`
- **Tasker display:** `TaskerCard.tsx`, `TaskerDetails.tsx`, `FilterSidebar.tsx`
- **Date/Time:** `Calendar.tsx`, `TimeSelector.tsx`, `DateTimeSelectionModal.tsx` (9.5KB)
- **Placeholder:** `ComingSoon.tsx` — used by 11 categories

### Two Development Patterns

1. **Self-contained page** — TV Mounting, Heavy Lifting, Electrical Work
   - All steps managed in a single `page.tsx` with internal state
   - Each has its own form component (e.g., `TVMountingForm.tsx`)

2. **Multi-page with shared DescribeTaskSection** — Cleaning, Plumbing, Yard Work, Trash Removal
   - Separate route for each step (`/details`, `/recommendations`, `/schedule`, `/confirm-details`)
   - State persisted via localStorage with category-specific key prefixes
   - Shared `DescribeTaskSection.tsx` handles the first form step

### Alternative Architecture (unmerged — PRs #3 and #4)

Branch `feature/details_page_component` (4 commits ahead) introduced a competing pattern:
- **PR #3** added `HeaderWithProgressBar` and `Logo` components (reusable progress indicator)
- **PR #4** added full reusable details page at route `/book/[id]/details` (dynamic ID pattern, different from category-specific routes)
- `src/components/details_page/` with its own `useTaskForm` hook (291 lines) including step-by-step validation
- 5 new UI components: `AutoCompleteInput`, `Collapsible`, `InputField`, `OptionWrapper`, `TextArea`
- **Different tech stack:** Uses Radix UI + Tailwind + lucide-react icons (vs MUI used everywhere else)
- **Added dependencies:** `@radix-ui/react-collapsible`, `clsx`, `tailwind-merge`, `lucide-react`
- **Brand name:** Uses "TaskBee" instead of "TaskRabbit" throughout
- 22 hardcoded San Francisco locations (vs main branch using NY/NJ addresses)
- Step 3 in progress bar has empty icon and description
- This is a parallel/alternative architecture to the existing `DescribeTaskSection` pattern — needs a decision on which to keep. The Tailwind/Radix stack conflicts with the MUI direction of the rest of the app.

---

## Cross-Branch Conflict Zones

Three files are modified by **both** `fb-cleaning` and `fb-furniture-assembly`:

| File | fb-cleaning | fb-furniture-assembly | Conflict Risk |
|------|-------------|----------------------|---------------|
| `src/app/(pages)/booktask/page.tsx` | 110 lines | 114 lines | **High** — both modify category routing |
| `src/components/task/BrowseTaskersSection.tsx` | 987 lines | 987 lines | **High** — core shared component |
| `src/config/constants.ts` | 244 lines | 234 lines | **High** — both add category config |

**Merge strategy recommendation:** Merge `fb-cleaning` first (smaller diff, 2 commits ahead), then rebase `fb-furniture-assembly` on top and resolve conflicts.

---

## Global Issues

1. **Generic taskers** — All categories share the same tasker pool. Only Yard Work and Trash Removal have dedicated tasker data files. Other categories need category-specific tasker data.
2. **`CLEANING_FREQUENCY_SELECTION` runtime error** — The constant is undefined at runtime in `ConfrimFrequencyDetailsSection.tsx:297`, likely a missing import or export from `constants.ts`.
3. **Typo in component name** — `ConfrimFrequencyDetailsSection` should be `ConfirmFrequencyDetailsSection`.
4. **Two competing form architectures** — `DescribeTaskSection` vs `details_page_component` branch. Need to pick one.
5. **MUI vs Tailwind** — MUI used in 75 files (62.5%), Tailwind in only 4 files (`Header.tsx`, `Footer.tsx`, `SearchBar.tsx`, `booktask/page.tsx`). Recommendation: migrate remaining 4 files to MUI and drop Tailwind.
6. **Orphaned branch** — `feature/landscape` has no merge base with main, likely abandoned.
7. **IKEA Furniture Assembly and Flooring** — Have no files at all, not even ComingSoon placeholders.
