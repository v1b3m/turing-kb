---
title: ReturnMax Test Batch — Agent Protocol
tags: [returnmax-test-protocol]
---

# Agent Protocol — ReturnMax Test Batch

**The per-prompt note frontmatter is the single source of truth.** Multiple agents run in
parallel against the 45 notes in this folder. Coordinate only through frontmatter — do not
keep claim state anywhere else.

Dashboard: `![[returnmax-test-batch.base]]` · Index: [[2026-06-14_1939_returnmax-test-batch-prompts]]

## Coordination fields

| Field | Meaning |
|-------|---------|
| `status` | `untested` → `in-progress` → `passed` \| `bug-found` → `fix-pushed` → `verified` |
| `owner` | Agent id holding the claim. **Empty = available.** |
| `claimed_at` | ISO-8601 (UTC+3) when claimed. Used to detect stale leases. |
| `updated_at` | ISO-8601 (UTC+3) of the last status change. |
| `bug_count` | Number of bugs logged in the note body. |
| `fix_pr` | PR/commit URL once a fix is pushed. |

## Claim loop (each agent)

1. **Pick** a note where `status == untested` and `owner` is empty (lowest task_id first).
2. **Claim**: set `owner: <your-agent-id>`, `status: in-progress`, `claimed_at`, `updated_at`.
3. **Re-read** the note. If `owner` is not your id, another agent won the race — back off and pick another.
4. **Reset to clean state** (see Operational notes), then **run** the prompt against the
   **local app at `http://localhost:3001`** (= the `returnmax-1` repo we fix). Drive via the
   chrome-cdp `turing` profile.
5. **Record** observations under `## Findings`. Verify against the seed using the state-diff
   logic (which sections/fields changed; any unintended mutations). For each bug, add a bullet
   under `## Bugs found` and bump `bug_count`.
6. **Resolve status**:
   - No bug → `status: passed`.
   - Bug → `status: bug-found`. Push a fix, set `fix_pr`, then `status: fix-pushed`.
     After re-running clean, `status: verified`.
7. Always update `updated_at` on every change.

## Stale lease rule

If `status == in-progress` and `claimed_at` is older than **45 minutes**, the lease is stale:
any agent may re-claim by overwriting `owner` and `claimed_at`.

## Rules

- One agent per note at a time. Never edit a note whose `owner` is another active agent.
- Only the coordination + results fields and the body sections are mutable. Prompt metadata
  (`task_id`, `prompt_uuid`, taxonomy, etc.) is immutable.
- Keep `bug_count` in sync with the bullets under `## Bugs found`.
- Fixes follow the repo's micro-commit rule; link the PR in `fix_pr`.

## Operational notes (validated 2026-06-15)

**Target:** `http://localhost:3001` = `/Users/v1b3m/Dev/Turing/returnmax-1` (`chore/keep-hacking`),
the code we fix. (Port 3000 = a different bfloat-workbench / returnmax checkout — do **not** use.)
The deployed gym `returnmax-v1.rlgym.turing.com` is the official batch target but renders the app
in a frame top-level `eval` can't reach — only drive it by screenshot + `clickxy` if ever needed.

**Driving:** on `localhost:3001`, normal app pages are directly scriptable — chrome-cdp `eval`,
CSS selectors, `click`, `type` all work. The `/state-diff` page is the exception: it renders inside
a **Shadow DOM** (`ShadowWrapper`), so its buttons are invisible to `eval` (use the reset recipe
below instead of clicking its "Reset All").

**Clean-state reset (replicates the app's `resetAll`)** — run via `eval`, then reload the app page:
```js
const seed = await (await fetch("/api/v1/env/defaultState",{headers:{Accept:"application/json"},cache:"no-store"})).json();
const SKIP = new Set(["_dirty_entities","_all_seeded"]);
for (const [k,v] of Object.entries(seed)) { if (SKIP.has(k)) continue; localStorage.setItem(k, typeof v==="string"?v:JSON.stringify(v)); }
// then: reload the page so Zustand re-hydrates from the seed
```
Verify clean: `rm_tax_return.state.w2s[0].wages === 92500.75` and `2025-luma-labs-w2.pdf` present in
`rm_documents`. (State lives in localStorage keys `rm_*`; seed is `data/state/defaultState.json`,
served at `/api/v1/env/defaultState`.)

**Metadata caveat:** some prompts' `prep_work` is stale relative to the latest version (e.g.
medium-001 v3 says "no W-2 entered" but the seed has the Luma Labs W-2). Trust the **prompt text +
actual seed state**, not prep_work, when they conflict — and note the discrepancy in Findings.

**Always re-prefix CDP commands with `CDP_PROFILE=turing` and use the absolute cdp.mjs path.**
