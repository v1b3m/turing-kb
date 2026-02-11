# Branch: `wajid/confirm-page-flow`

**Base:** `main`
**Commits:** 8 commits (oldest → newest)

## Overview

This branch overhauls the **booking confirmation flow** and introduces a **dashboard for task management**. The key themes are:

1. Replace mock/hardcoded confirm-page data with real data from the booking store and sessionStorage.
2. Add a Zustand-persisted `useTasksStore` so confirmed bookings become trackable tasks.
3. Replace the old `/mytasks` page with a new `/dashboard` section (active + completed tabs).
4. Unify the progress bar across all booking steps to a consistent 4-step model.
5. UI polish: pricing breakdown, donation toggle, rounded progress dots, filter dropdowns via `CountrySelect`.

---

## File-by-file Changes

### 1. `src/app/(pages)/book/[taskId]/confirm/page.tsx` (confirm page)

**Before:** Pulled data from `getConfirmBookingMock()` using `serviceId` + `taskerId` from query params. The tasker, dateTime, durationLabel, and taskTypeName all came from mock data. On confirm, it showed an `alert()` and redirected back to recommendations.

**After:**
- **Tasker** is read from `sessionStorage.getItem('selectedTasker')` (set during the recommendations/schedule step).
- **Schedule** is read from `sessionStorage.getItem('selectedSchedule')`, parsed as `{ date, time }` JSON, and formatted into a readable label via `formatScheduleDateTime()` (e.g. `"Tue, Feb 10 at 9:00am"`).
- **Task type name** comes from `storedBooking.details.serviceTitle` (booking store) or falls back to `service.title`.
- **Pricing** is computed inline: `hourlyRate = 24`, `trustSupportFeeRate = 0.368`, `totalRate = hourlyRate + hourlyRate * trustSupportFeeRate`.
- **Donation section** is conditionally shown only when `serviceId` has an entry in `bookingConfig`.
- **On confirm:**
  - Calls `useTasksStore.addTask(...)` to persist the booking as a pending task (with tasker info, pricing, location, dateTime, etc.).
  - Clears sessionStorage keys (`selectedTasker`, `selectedSchedule`).
  - Redirects to `/dashboard/active` instead of back to recommendations.
- No longer uses `taskerId` query param or `getConfirmBookingMock`.

### 2. `src/app/(pages)/book/[taskId]/recommendations/page.tsx` (recommendations page)

**Before:** Had an inline custom header with a hand-coded progress bar (130+ lines of JSX), an inline sort dropdown with open/close state, and native `<select>` elements for filters.

**After:**
- **Header** replaced with the shared `<Header>` component, passing `showProgressBar`, `totalSteps={4}`, `currentStep` (2 or 3 depending on `scheduleModalOpen`), and `stepLabels`.
- **Sort dropdown** replaced with `<CountrySelect>` component (reused from elsewhere), eliminating the manual `sortOpen` state.
- **Specific time filter** also replaced with `<CountrySelect>`.
- Unused imports removed (`ChevronDown`, `Link`, `getAllTimeOfDay`, `getAllSpecificTimes`).
- Adds `roundedDropdownClass` constant for consistent `!rounded-full px-4 text-lg` styling.
- Defines `BOOKING_STEP_LABELS` array: `["Describe your task", "Browse Taskers & prices", "Choose date & time", "Confirm details"]`.
- Spacing and layout adjustments (increased gap/padding, changed elite tasker ring from green to purple).
- Filter sidebar restructured: the trust badge is pulled out of the bordered filter card into its own separate card.

### 3. `src/components/confirm/ConfirmBookingSummary.tsx`

**Before:** Showed a simple hourly rate with a "No hidden fees" link and basic cancellation text.

**After:**
- Displays a **"Price Details"** section with three line items:
  - Hourly Rate: `$24.00/hr`
  - Trust and Support Fee: `$8.83/hr` (with a `?` info icon)
  - Total Rate: `$32.83/hr` (with a top border separator)
- Shows **minimum hours** (from `getMinimumHours(tasker)`) below the date/time.
- Updated cancellation policy text: mentions 1-hr deposit, 24-hr cancellation window, non-refundable if <24 hrs.
- Adds "See cancellation policy here." link.

### 4. `src/components/confirm/ConfirmPaymentForm.tsx`

**Before:** Always showed the donation section. Billing notice was an icon+text saying "You won't be billed until your task is complete."

**After:**
- Accepts `showDonation` (boolean) and `totalRate` (number) props.
- **Donation section** is conditionally rendered based on `showDonation`.
- **Billing notice** replaced with a text paragraph: "Once you confirm, your payment method will be charged `$totalRate` for 1 hour of work. Rest assured — this deposit is fully refundable if you cancel at least 24 hrs before the appointment. All tasks have a 1 hour minimum."

### 5. `src/stores/useTasksStore.ts` (NEW)

A new **Zustand store** with `persist` middleware (localStorage key: `"tasks-storage"`).

**Types:**
- `TaskStatus`: `'pending' | 'completed' | 'canceled'`
- `TaskerInfo`: `{ id, firstname, lastname, avatarUrl }`
- `ConfirmedTask`: `{ id, serviceTitle, taskDetails, location, dateTime, tasker: TaskerInfo, hourlyRate, totalRate, status, createdAt }`

**State & Actions:**
- `tasks: ConfirmedTask[]`
- `isHydrated: boolean` (for SSR-safe rendering)
- `addTask(data)` — generates a unique ID (`task-{timestamp}-{random}`), appends to tasks array, returns ID.
- `updateTaskStatus(taskId, status)` — updates a task's status.
- `getTask(taskId)`, `getPendingTasks()`, `getCompletedTasks()` — accessors.
- `removeTask(taskId)` — removes from array.
- Persistence: uses `createJSONStorage(() => localStorage)`, partializes to only persist `tasks`. Skips hydration on server. Sets `isHydrated = true` on rehydrate.

### 6. Dashboard pages (ALL NEW)

**`src/app/(pages)/dashboard/layout.tsx`:**
- Tabbed layout with "Current" (`/dashboard/active`) and "Completed" (`/dashboard/completed`) tabs.
- Uses `usePathname()` for active tab highlighting.
- Styled with `border-b-2 border-primary` for active tab.

**`src/app/(pages)/dashboard/active/page.tsx`:**
- Reads all tasks from `useTasksStore`, filters to non-completed (`status !== 'completed'`), renders via `<TaskList>`.
- Provides `onCancelTask` handler that calls `updateTaskStatus(taskId, 'canceled')`.
- Waits for hydration before rendering.

**`src/app/(pages)/dashboard/completed/page.tsx`:**
- Filters to `status === 'completed'` tasks, renders via `<TaskList>`.
- No cancel action (read-only view).

### 7. Dashboard components (ALL NEW)

**`src/app/(pages)/dashboard/_components/TaskCard.tsx`:**
- Renders a single confirmed task as a card with:
  - Header: service title + ellipsis menu (Radix `Popover`) with "Cancel task" option (only for pending tasks).
  - Notice banner: "Taskers usually contact you to chat about your task in less than 10 hours." (hidden for canceled tasks).
  - Body: tasker avatar (or initials fallback), tasker name, date/time, location, "View More Details" link, status badge.
  - Confirmation modal (`ModalWithActions`): "Cancel task?" with "Keep task" / "Cancel task" buttons.
- Status badge variants: `pending` → primary, `completed` → success, `canceled` → error.

**`src/app/(pages)/dashboard/_components/TaskList.tsx`:**
- Renders a list of `TaskCard`s, or an empty state component if no tasks.

**`src/app/(pages)/dashboard/_components/DashboardEmptyState.tsx`:**
- Default empty state: "Have something else on your to-do list?" with a "Check It Off Your List" button linking to `/`.

### 8. `src/app/(pages)/mytasks/page.tsx` (DELETED)

The old `/mytasks` page is completely removed. It was a stub with client-side tab switching and a loading spinner, but no real task data. Replaced by the new `/dashboard` routes.

### 9. `src/app/(pages)/layout.tsx`

**Before:** Only profile pages got `paddingTop: "0px"`.

**After:** Both `/book/` and `/dashboard/` paths get `paddingTop: "0px"` (since they use their own header/layout).

### 10. `src/components/ui/Header.tsx`

- **Progress steps unified to 4:** Both `BOOK_CONFIRM_PROGRESS` and `BOOK_DESCRIBE_TASK_PROGRESS` now use the same 4-step labels: `["Describe your task", "Browse Taskers & prices", "Choose date & time", "Confirm details"]`.
  - Confirm page: step 4 of 4.
  - Details page: step 1 of 4.
- **Nav links updated:**
  - "Dashboard" → "My Tasks" (links to `/dashboard/active`)
  - Added "Book a Task" link (links to `/`)
  - Styling changed from `text-gray-500` to `text-secondary-800 font-semibold`.

### 11. `src/components/ui/ProgressBar.tsx`

- All step indicators changed from `rounded-md` to `rounded-full` (circles instead of rounded squares).
- Progress bar track and fill also changed to `rounded-full`.

### 12. `src/components/ui/CountrySelect.tsx`

- Added `ariaLabel` optional prop to allow custom `aria-label` on the listbox (falls back to `label` then `'Select'`).

### 13. `src/data/confirmBookingMock.ts`

- **Task types** no longer come from `taskTypes.json`. Instead sourced from `feature-set.json`, using `bookServiceId` as key and converting `serviceSlug` to title case via `slugToTitle()`.
- Removed `taskTypes` key from the `ConfirmBookingMockJson` type.

### 14. `src/data/taskTypes.json` (DELETED)

Hardcoded task type mapping removed. Data now derived from `feature-set.json`.

---

## Data Flow Summary

```
[Details Page] → writes to useBookingStore (location, task details, serviceTitle)
       ↓
[Recommendations Page] → user selects tasker → sessionStorage.setItem('selectedTasker', JSON.stringify(tasker))
       ↓
[Schedule Modal] → user picks date/time → sessionStorage.setItem('selectedSchedule', JSON.stringify({date, time}))
       ↓
[Confirm Page] → reads from:
   • useBookingStore (location, taskDetails, serviceTitle)
   • sessionStorage (selectedTasker, selectedSchedule)
   • bookingConfig (whether to show donation)
   • Inline pricing (hourlyRate=24, trustSupportFeeRate=0.368)
       ↓
[On Confirm] → useTasksStore.addTask(...) → persisted to localStorage
             → clears sessionStorage
             → redirects to /dashboard/active
       ↓
[Dashboard] → reads from useTasksStore → renders TaskCards
            → can cancel pending tasks (status → 'canceled')
```

---

## Config-Driven Form Templating (`booking.js` + `BookingDescribeTaskForm`)

The details page (`/book/[taskId]/details`) uses a **declarative, config-driven wizard** pattern. The config file defines *what* to show; a single generic React component interprets it.

### Config schema (`src/config/booking.js`)

A plain JS `module.exports` object keyed by **service ID** (number). Each entry declares:

| Key | Type | Purpose |
|---|---|---|
| `intro` | `string \| null` | Intro screen ID shown before the wizard steps. Currently only `'wall-mounting-options'` is implemented (renders a category picker). `null` = skip intro. |
| `steps` | `string[]` | Ordered array of step IDs — each maps to a form section in the renderer. |
| `rightContent` | `boolean?` | If `true`, use a two-column layout: form on the left, a sticky summary card on the right. |
| `hideTopBand` | `boolean?` | If `true`, suppress the info banner ("Tell us about your task...") above the form. |
| `vehicleRequirements` | `boolean?` | If `true`, show vehicle requirement radios inside the `task-options` step. |
| `multipleQuestionList` | `MultipleQuestionItem[]?` | Declarative question definitions for the `details-multiple-question` step. |

**`MultipleQuestionItem` shape:**
```ts
{
  question: string;          // The question text
  options: string[];         // Radio/checkbox options. Empty array = free-text textarea
  type?: 'radio' | 'checkbox'; // Default: radio (when options present)
  helperText?: string;       // Shown below the question
  maxLength?: number;        // For textareas — shows "X/Y" character counter
}
```

### Step ID → rendered section mapping

The renderer (`BookingDescribeTaskForm`) iterates `steps.map((stepId, i) => ...)` and pattern-matches:

| Step ID | Section | Inputs |
|---|---|---|
| `task-location` | "Your task location" | Street address text input + unit/apt text input |
| `task-options` | "Task options" | Task size radios (Small/Medium/Large) + conditional vehicle requirement radios |
| `view-items-furniture` | "Your Items" | Furniture type radios (IKEA only / Other / Both) |
| `details-multiple-question` | "Details" | Dynamically rendered from `multipleQuestionList`: radios when `options.length > 0`, checkboxes when `type: 'checkbox'`, textarea when `options` is empty |
| `details-of-your-task` | "Tell us the details" | Single free-form textarea |

### Step state machine

Each section has **three visual states** based on `currentStep` vs its position `i+1`:

- **Future** (`currentStep < stepNumber`): collapsed card, greyed-out heading, no inputs shown.
- **Active** (`currentStep === stepNumber`): expanded card, active heading, inputs rendered, "Continue" button.
- **Completed** (`currentStep > stepNumber`): collapsed card showing a summary of entered data. Clickable to re-expand (edit). Shows a checkmark icon, or a pencil icon on hover.

### Example service configurations

**Service 12** (General Cleaning) — simple 3-step:
```js
{ intro: null, steps: ['task-location', 'task-options', 'details-of-your-task'] }
```

**Service 15** (TV Mounting) — 2-step with dynamic questions and two-column layout:
```js
{
  intro: null,
  steps: ['task-location', 'details-multiple-question'],
  rightContent: true,
  hideTopBand: true,
  multipleQuestionList: [
    { question: 'How many TVs...?', options: ['1','2','3','4','5'] },
    { question: 'Will someone help lift...?', options: [...], helperText: '...' },
    { question: 'Mount type?', options: [...], type: 'checkbox' },
    { question: 'Wall material?', options: [...], type: 'checkbox' },
    { question: 'Add-on services?', options: [...], type: 'checkbox' },
    { question: 'Anything else?', options: [], maxLength: 500 },
  ],
}
```

**Service 36** (Wall Mounting — General) — has an intro screen:
```js
{
  intro: 'wall-mounting-options',
  steps: ['task-location', 'details-multiple-question'],
  rightContent: true, hideTopBand: true,
  multipleQuestionList: [...]
}
```
When `intro === 'wall-mounting-options'` and `introCompleted` is false, the component renders a full-screen category picker (Hang Art, Install Blinds, Mount Furniture, etc.) before entering the step wizard.

### Right column (summary card)

When `rightContent: true`, the form renders in a `grid-cols-[1fr,360px]` layout. The right column is a sticky sidebar showing:
- A placeholder image
- A "Details for your Tasker" card that live-updates as questions are answered (each answer is a clickable link that scrolls to its question)
- A "Taskrabbit Happiness Pledge" trust badge

### How the confirm page uses `bookingConfig`

The confirm page (`/book/[taskId]/confirm`) imports the same config but only uses it for a **feature gate**: `const showDonation = serviceId > 0 && serviceId in configRecord`. If the service has a config entry, the donation section is shown on the payment form. The config's `steps`/`intro` fields are not used on the confirm page.

### Adding a new service

To add a new service type, you only need to:
1. Add an entry to `src/config/booking.js` with the service ID as key
2. Define `steps`, and optionally `multipleQuestionList`, `rightContent`, `vehicleRequirements`, `intro`, `hideTopBand`
3. No new React components are needed — `BookingDescribeTaskForm` handles everything generically

---

## Key Architectural Decisions

1. **sessionStorage for cross-page transient data** (selected tasker, schedule) rather than putting it in the Zustand booking store — keeps it ephemeral and auto-cleared on tab close.
2. **Zustand + localStorage for confirmed tasks** — persistent across sessions, SSR-safe with hydration gating.
3. **Unified 4-step progress bar** across all booking pages instead of varying step counts per page.
4. **CountrySelect as generic dropdown** — repurposed from its original country-picker role to handle sort and filter dropdowns on the recommendations page.
5. **Pricing computed client-side** with hardcoded rates (`$24/hr` + 36.8% trust fee) rather than fetched from an API.
