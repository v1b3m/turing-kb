# 001 — Services Without Taskers

**Date:** 2026-02-24
**Branch:** `chore/update-taskers`

## Summary

Out of **146 unique bookingIds** in `src/data/service-list.json`, **2** have no tasker coverage across the 4 tasker chunk files (`taskers-1.json` through `taskers-4.json`, totalling 9,294 taskers).

| bookingId | Title                   | serviceSlug       | categorySlug | Status              |
|-----------|-------------------------|-------------------|--------------|---------------------|
| 2036      | Heavy Lifting & Loading | heavy-lifting     | featured     | Aliased to 3887 at API level |
| 2270      | Contactless Delivery    | pick-up-delivery  | contactless  | **No coverage**     |

### Details

- **2036 (Heavy Lifting & Loading):** Not a true gap. The API route (`src/app/api/taskers/route.ts:24`) remaps requests for `serviceId === 2036` to `3887` (Heavy Lifting). This means 2036 piggybacks off 3887's tasker pool at runtime.
- **2270 (Contactless Delivery):** Genuinely has zero taskers. No tasker in any chunk file has `2270` in their `serviceIds` or `subServiceIds` arrays.

## Verification Method

### How taskers map to services

Each tasker object carries two arrays of numeric bookingIds:

- `serviceIds` — primary service bookingIds the tasker serves
- `subServiceIds` — sub-service bookingIds the tasker serves

The search function `filterTaskersByServiceId` in `src/lib/server/taskers/all-taskers.ts:172` checks **both** arrays:

```ts
if (!(tasker.serviceIds?.includes(serviceId) || tasker.subServiceIds?.includes(serviceId))) continue;
```

### Verification script

Run from the project root (`/home/v1b3m/Dev/Turing/task-rabbit`):

```js
node -e "
const fs = require('fs');

// 1. Extract all bookingIds from service-list.json
const services = JSON.parse(fs.readFileSync('src/data/service-list.json', 'utf8'));
const allBookingIds = [...new Set(services.map(s => Number(s.bookingId)))].sort((a,b) => a-b);

// 2. Collect all serviceIds and subServiceIds from tasker chunks
const coveredByServiceIds = new Set();
const coveredBySubServiceIds = new Set();
let totalTaskers = 0;

for (let i = 1; i <= 4; i++) {
  const chunk = JSON.parse(fs.readFileSync('src/data/taskers/taskers-' + i + '.json', 'utf8'));
  totalTaskers += chunk.taskers.length;
  for (const tasker of chunk.taskers) {
    (tasker.serviceIds || []).forEach(id => coveredByServiceIds.add(id));
    (tasker.subServiceIds || []).forEach(id => coveredBySubServiceIds.add(id));
  }
}

const allCovered = new Set([...coveredByServiceIds, ...coveredBySubServiceIds]);

// 3. Find missing
const missing = allBookingIds.filter(id => !allCovered.has(id));

console.log('Total unique bookingIds in service-list.json:', allBookingIds.length);
console.log('Total taskers across all chunks:', totalTaskers);
console.log('Unique IDs in tasker serviceIds:', coveredByServiceIds.size);
console.log('Unique IDs in tasker subServiceIds:', coveredBySubServiceIds.size);
console.log('Combined unique coverage:', allCovered.size);
console.log('');
console.log('BookingIds WITHOUT any tasker coverage (' + missing.length + '):');
missing.forEach(id => {
  const svc = services.find(s => Number(s.bookingId) === id);
  console.log('  ' + id + ' - ' + (svc?.title || 'unknown'));
});
"
```

### Expected output

```
Total unique bookingIds in service-list.json: 146
Total taskers across all chunks: 9294
Unique IDs in tasker serviceIds: 81
Unique IDs in tasker subServiceIds: 63
Combined unique coverage: 144

BookingIds WITHOUT any tasker coverage (2):
  2036 - Heavy Lifting & Loading
  2270 - Contactless Delivery
```

## Action Required

- **2036:** No action needed — already handled by API alias to 3887.
- **2270 (Contactless Delivery):** Needs tasker generation. New taskers should go into a new chunk file (e.g., `taskers-5.json`) to avoid touching existing data.
