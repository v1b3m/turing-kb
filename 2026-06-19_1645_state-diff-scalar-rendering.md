# State-Diff Rendering Failures for Scalar Values

## Symptoms

The `StateDiffViewer` shows a change badge in the header (e.g., "1 modified") but renders **no detail rows** in the body. The viewer appears broken: a heading, a "Reset to Seed" button, and empty space below.

This happens for **scalar** (non-object, non-array) sub-slices like `digitalAssetAnswer: "yes"`. It does NOT happen for arrays or objects — those render correctly.

## Root Cause

`computeKeyDiff` in `@turing-rlgym/cua-gym-utils/dist/frontend/diff-utils.js` has four comparison paths:

| Path | Trigger | `records` |
|---|---|---|
| `diffArrays` | Both seed & current are arrays | Field-level diffs per entity |
| `diffObjectOfArrays` | Objects whose values are arrays/primitives | Record per bucket |
| `diffObject` | Both are plain objects | Record with `fields[]` |
| **Fallback** (line 57–68) | Primitives, `null`, `undefined` | `records: []` |

The fallback path:

```js
const isAdded = seedData === null || seedData === undefined;
const isRemoved = currentData === null || currentData === undefined;
return {
  changeType: isAdded ? 'added' : isRemoved ? 'removed' : 'modified',
  totalRecords: 1,
  addedCount: isAdded ? 1 : 0,
  modifiedCount: !isAdded && !isRemoved ? 1 : 0,
  removedCount: isRemoved ? 1 : 0,
  unchangedCount: 0,
  records: [],  // always empty for scalars
};
```

`StateDiffViewer` at line 13 then maps over `records`:

```jsx
diff.records.map(record => <RecordDiffCard ... />)
```

Empty `records` → empty body. The header correctly shows the change count (from `modifiedCount`), but the map produces nothing.

### Two flavors

1. **Seed `null` → current non-null**: `isAdded = true` → "1 added" with no detail rows.
2. **Seed `""` → current `"yes"`**: both non-null → "1 modified" with no detail rows.

## Example

### Before (broken)

```typescript
// In OtherTaxSituations:
digitalAssetAnswer: "yes" | "no" | "";
```

Seed: `digitalAssetAnswer: ""`
Current: `digitalAssetAnswer: "yes"`

`computeKeyDiff("", "yes")` → `{ changeType: 'modified', records: [] }`

The state-diff renders:

```
Digital Asset Answer
Rm Tax Return · Other Tax Situations
~1 modified                         [Reset to Seed]
                                    ← nothing below this line
```

### After (fixed)

```typescript
// In OtherTaxSituations:
digitalAssets: { answer: "yes" | "no" | "" };
```

Seed: `digitalAssets: { answer: "" }`
Current: `digitalAssets: { answer: "yes" }`

`computeKeyDiff({ answer: "" }, { answer: "yes" })` → uses `diffObject` → produces:

```json
{
  "changeType": "modified",
  "records": [{
    "id": "root",
    "changeType": "modified",
    "fields": [{
      "field": "answer",
      "changeType": "modified",
      "seedValue": "",
      "currentValue": "yes"
    }]
  }]
}
```

The state-diff now renders a proper table row:

```
▶ root                                              modified
  answer    ""    "yes"    modified
```

## Solution

**Never store scalar values at keys that become their own state-diff sub-slices.** Wrap them in an object so `diffObject` can produce field-level records.

### Pattern

```typescript
// Bad: Scalar — diff has empty records
foo: "yes" | "no" | "";

// Good: Wrapped in object — diff has renderable records
foo: { value: "yes" | "no" | "" };
```

### Additional rule

**Never use `null` as a seed value** for a field that will hold user data. The fallback path treats seed `null` as "data doesn't exist" → `changeType: 'added'`, even when `null` is a legitimate initial state. Use `""` or another non-null sentinel.

## Files referenced

- `node_modules/@turing-rlgym/cua-gym-utils/dist/frontend/diff-utils.js` — `computeKeyDiff`
- `node_modules/@turing-rlgym/cua-gym-utils/dist/frontend/components/StateDiffViewer.js` — renders `diff.records.map()`
- `app/state-diff/_client.tsx` — state-diff page
- `app/state-diff-2/_components/StateDiff.tsx` — how slices are built via `buildKeyStates`
