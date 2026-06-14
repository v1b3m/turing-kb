---
title: "1099 Filter Changes — Manual Testing"
date: 2026-05-29
tags:
  - ledgerpilot
  - testing
  - 1099
---

# 1099 Filter Changes — Manual Testing

Manual verification for two changes on `feature/journal-entry-account-creation`:

1. **Filters persist on change** — `completedFormsFilters` updates in the store whenever the user changes year, status, or form type (not only on download).
2. **State-diff displays filters** — the `/state-diff` page shows `completedFormsFilters` field-level diffs instead of silently swallowing the change.

---

## Prerequisites

- [ ] Clear localStorage (`/state-diff` → Reset All, or DevTools)
- [ ] Reload the app so seed data hydrates fresh
- [ ] Confirm `/state-diff` shows **0 changed** for `qb-gym-prepare-1099-store`

---

## Test 1 — Zero-change baseline

1. Navigate to **Expenses → Filing 1099 → Completed forms** tab
2. Do **not** touch any filter
3. Open `/state-diff`

> [!success] Expected
> `completedFormsFilters` shows as **unchanged** — seed default `{ year: "2025", status: "", formType: "" }` matches the persisted value written on mount.

---

## Test 2 — Year filter change appears in state-diff

1. On the Completed forms tab, change the **Year** dropdown to `2024`
2. Open `/state-diff`

> [!success] Expected
> `completedFormsFilters` shows a **modified** diff:
> - `year`: `"2025"` → `"2024"`
> - `status` and `formType` unchanged

---

## Test 3 — Status filter change appears in state-diff

1. Reset state (`/state-diff` → Reset All, reload)
2. On the Completed forms tab, change **Status** to `Accepted`
3. Open `/state-diff`

> [!success] Expected
> `completedFormsFilters` shows a **modified** diff:
> - `status`: `""` → `"accepted"`
> - `year` and `formType` unchanged

---

## Test 4 — Form type filter change appears in state-diff

1. Reset state, reload
2. On the Completed forms tab, change **Form Type** to `1099-NEC`
3. Open `/state-diff`

> [!success] Expected
> `completedFormsFilters` shows a **modified** diff:
> - `formType`: `""` → `"1099-nec"`
> - `year` and `status` unchanged

---

## Test 5 — Multiple filters combined

1. Reset state, reload
2. Set **Year** to `2024`, **Status** to `Pending`, **Form Type** to `1099-MISC`
3. Open `/state-diff`

> [!success] Expected
> `completedFormsFilters` shows all three fields as **modified**:
> - `year`: `"2025"` → `"2024"`
> - `status`: `""` → `"pending"`
> - `formType`: `""` → `"1099-misc"`

---

## Test 6 — Filters via popover

1. Reset state, reload
2. Click the **Filters** toolbar button (popover)
3. Set **Status** to `Sent`, **Form Type** to `1099-NEC`
4. Click **Apply**
5. Open `/state-diff`

> [!success] Expected
> Same field-level diff as direct select changes — popover commits flow through the same state path.

---

## Test 7 — Filters survive tab switch

1. Set any non-default filter (e.g. year `2024`)
2. Switch to a different tab or navigate away
3. Return to **Completed forms**
4. Verify the filter controls show the previously selected values

> [!success] Expected
> Filters are restored from the persisted store. No data loss on tab switch.

---

## Test 8 — Download still works

1. Set filters so the table has results (e.g. year `2025`, no status/form filter)
2. Click **Download**
3. Verify a CSV file downloads with the filtered rows

> [!success] Expected
> CSV downloads correctly. Download no longer has a side-effect of saving filters (the effect already did it).

---

## Test 9 — Reset restores default filters

1. Set non-default filters
2. Go to `/state-diff` → reset the `qb-gym-prepare-1099-store` slice
3. Reload and open Completed forms

> [!success] Expected
> Filters reset to defaults: year `2025`, status empty, form type empty. State-diff shows **0 changed**.
