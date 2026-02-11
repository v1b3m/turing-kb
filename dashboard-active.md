# `/dashboard/active` Data Flow

The `/dashboard/active` page data comes from a **Zustand store persisted to localStorage**.

## Data Flow

1. **Store:** `src/stores/useTasksStore.ts` — a Zustand store with `persist` middleware, stored under localStorage key `"tasks-storage"`
2. **Initial data:** 3 hardcoded mock tasks (Furniture Assembly/pending, Heavy Lifting/completed, Heavy Lifting/canceled)
3. **Page:** `src/app/(pages)/dashboard/active/page.tsx` — reads `useTasksStore((s) => s.tasks)`, filters to `status !== 'completed'`, sorts by `createdAt` descending
4. **Rendering:** passes filtered tasks to `TaskList` → `TaskCard` components

```
┌─────────────────────────────────────────┐
│  Browser localStorage                   │
│  Key: "tasks-storage"                   │
│  Contains: { tasks: [...] }             │
└──────────────┬──────────────────────────┘
               │
               ↓ (hydration on app load)
┌─────────────────────────────────────────┐
│  Zustand Store: useTasksStore           │
│  - tasks: ConfirmedTask[]               │
│  - updateTaskStatus(id, status)         │
│  - addTask, removeTask, etc.            │
└──────────────┬──────────────────────────┘
               │
               ↓ (via hook)
┌─────────────────────────────────────────┐
│  /dashboard/active/page.tsx             │
│  - Filters: status !== 'completed'      │
│  - Sorts by createdAt (descending)      │
└──────────────┬──────────────────────────┘
               │
               ↓ (props)
┌─────────────────────────────────────────┐
│  TaskList → TaskCard                    │
│  Renders: title, tasker, date, location │
└─────────────────────────────────────────┘
```

## Key Characteristics

- **No API calls:** Data is entirely client-side, stored in localStorage
- **Mock data:** The store ships with 3 hardcoded test tasks
- **Mutations:** Tasks can be modified via store methods (`updateTaskStatus`, `addTask`, `removeTask`)
- **Persistence:** Changes aßutomatically persist to localStorage via Zustand's middleware
