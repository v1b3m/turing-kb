---
title: Harness Prompt Review Flow
description: Reproducible workflow for reviewing prompt submissions on Harness
date: 2026-06-21
tags: [harness, review, prompt-review, msr, trainer-issue, gym-issue, ar-issue]
---

# Harness Prompt Review Flow

> **Source:** `pg/MSR-Production-Review-checklist.md`  
> **Goal:** Establish a reproducible, iterative review process for prompt submissions by developers on Harness.

---

## 1. Review Goal

Identify errors in trainer-created tasks. Categorize every issue into one of three buckets:

- **Trainer Issue** — problem with the prompt or trainer execution
- **GYM Issue** — problem with the simulated UI/app environment
- **AR Issue** — problem with the automated review / AI summary

> Always check **all three categories** for every task. Do not rely only on the examples in the checklist.

---

## 2. Pre-Review Setup

1. Open the task in Harness.
2. **Navigate to the trajectory:** The task page shows a "Human Run" card under *Singular Runs*. **You must click the card to select the run first** — the Trajectory tab will not contain data if accessed without selecting a specific run.
3. Watch the trainer video (if available).
4. Note the metadata: gym(s), task ID, prompt text, initial/final JSON.
5. Reset UI-GYM state if needed: run `localStorage.clear()` in the browser console.
6. Download seed data + JSON schema for the relevant gym(s) when grounding checks are required.

---

## 3. Review Lifecycle

Review happens entirely in the Google Sheet tracker — no status changes are made in Harness.

### Sheet Status Rules

| Review outcome | Sheet status | Action |
|---|---|---|
| Task is clean (no issues) | **Ready for Calibration** | None |
| Trainer issues only (no gym issues) | **Ready for Calibration** | Fix the task yourself, then update status |
| Gym issues (with or without trainer issues) | **Rejected - Gym Issues** | Add a clear comment in the **Gym Issues** column |

### Gym Issues Comment

When setting **Rejected - Gym Issues**, the comment must include:

- What the gym did or did not do (the observed behavior).
- What the correct behavior should be.
- References to the relevant state keys or seed data where applicable.
- If a Trainer Issue is also present, note it separately so it's clear which issues are train-side vs gym-side.

---

## 4. Harness Trajectory Layout

Once on the **Trajectory** tab, the page is organized as follows:

- **Recording / replay panel** — video replay of the human run.
- **Run transcript** — step-by-step list of actions taken.
- **Action toolbar** (next to the replay) with icon buttons:
  - **Open file browser** — opens a modal file explorer listing all submitted artifacts:
    - `screenshots/` folder
    - `action_timeline.json`
    - `local_storage_initial.json`
    - `local_storage_final.json`
    - `raw_events.json`
    - `recording.webm`
    - `task_data.json`
    - Also allows **Download all files as ZIP**.
  - **Open localStorage diff** — opens a modal diff viewer with three tabs:
    - **AI Summary** — natural-language requirement-by-requirement verdict (met / partially met / not met).
    - **JSON Diff V2** — structured diff showing added, removed, and modified localStorage paths (Flat Changes / Tree views, plus path/value filter).
    - **JSON Diff V1 (deprecated)** — older diff format.
  - **Compare** — compare against other runs/sources.

Use these tools during JSON checks (Step 6) and AI Summary checks (Step 10).

---

## 5. Reproducible Review Steps

### Step 0 — Prompt Practicality & Grounding
- Is the prompt practical and meaningful for a real employee/user (not QA automation)?
- No contradictions.
- Not purely DB filtering.
- **Grounding:** every referenced entity, value, and state must be verifiable from seed JSON data. If UI conflicts with seed data, trust the seed data.

### Step 1 — Prompt Realism, Semantic Sense & Complexity
- Logically realistic within the product context.
- Complexity matches assigned category (task may score higher, never lower).
- No synthetic/fragmented feel.

### Step 2 — Prompt Feasibility
- Fully end-to-end feasible in the gym.
- All features, filters, actions, and data exist.
- No unsupported workflows.

### Step 3 — Prompt Quality / Ambiguity
- Entities uniquely identifiable.
- No duplicate-name ambiguity.
- No missing business logic or undefined outcomes.

### Step 4 — Noisy UI Actions
- JSON should not contain unnecessary/noisy UI actions.

### Step 5 — Cross-Gym Task Validation
- At least 2 state-changing operations across different gyms (per metadata).
- Read-only in one gym is insufficient.

### Step 6 — JSON Checks
- **Datatype validation:** correct formats.
- **Value hallucination:** JSON reflects only actual UI changes.
- **Prompt ↔ Final State Mismatch:** final state exactly matches prompt intent.
- **Wrong Field Mapping:** updates written to correct fields.

### Step 7 — Exact Prompt Text
- Relevant fields must contain exact text and formatting from the prompt.

### Step 8 — Initial vs Final JSON Comparison
- No hallucinated values.
- No unwanted fields updated.
- Additions/modifications at the right place.

### Step 9 — AR Issues
- Check AR output in **both** versions:
  - [ ] Run version
  - [ ] Trainer/reviewer version
- If the task **failed AR review**, treat the AR feedback as evidence, not a verdict:
  1. Read the AR failure reason(s).
  2. Decide whether the flagged issue is a real task defect. If yes, log it as a **Trainer Issue** or **GYM Issue** (AR correctly caught it).
  3. If AR rejected a correct task or flagged a non-existent problem, log it as an **AR Issue**.
- Report AR issues in two sub-categories only:
  - **Missed actual issue** — the task is wrong, but AR did not flag it.
  - **Repeated hallucination / false positive** — AR flags something that is not wrong. Do **not** report one-off hallucinations; report only when the same false pattern shows up across multiple tasks.

### Step 10 — AI Summary Check
- Verify state changes documented in final JSON.
- AI summary hallucinations are scrutinized more rigorously than AR hallucinations.

---

## 6. Issue Classification Reference

### Trainer Issue
| Sub-category | When to flag |
|---|---|
| Prompt Infeasibility | Gym lacks functional capacity to fulfill request |
| Prompt Ambiguity | Imprecise language leading to multiple outcomes |
| Incorrect Work | Feasible & clear prompt executed incorrectly (complete or partial) |
| AR Feedback Ignorance | Trainer did not fix an issue previously flagged by AR |

### GYM Issue
| Sub-category | When to flag |
|---|---|
| UI Issues | Incorrect mapping or broken functionality in the UI |
| JSON Structure | New entries don't match structure/datatype of existing entries |
| JSON Values Hallucination | Values don't match UI enum/options |

### AR Issue
| Sub-category | When to flag |
|---|---|
| Missed actual issue | AR failed to point out a real problem |
| Hallucinations | Repeated false positives; flag only repetitive issues |

---

## 7. Common Mistakes / Client Feedback Points

1. **Prompt ↔ Final State Mismatch**
2. **Schema / Field Mapping Errors**
3. **Environment / Gym Capability Mismatch**
4. **Ambiguous or Underspecified Prompts**
5. **Business Logic / Workflow Inconsistency**
6. **Data Consistency & Referential Integrity Issues**
7. **Artifact / Source Delivery Errors**
8. **Non-Actionable / Low-Value Tasks**
9. **Prompt Quality & Formatting Issues**
10. **Practicality / Operational Plausibility Issues**

---

## 8. Tools & Prompts

### JSON Diffing (localStorage snapshots)

Raw Harness localStorage dumps store every value as an escaped JSON string, so a plain `diff` is mostly noise. Two helpers live in `pg/` (they are intentionally kept untracked because `pg/` is not versioned in this repo):

- **`pg/jsonfmt.py`** — parse nested JSON strings, sort keys, and pretty-print. Works as a normal CLI tool or as a Git textconv driver.
- **`pg/diff-json.sh`** — normalize two JSON files and run `git diff --no-index` on them, giving you semantic diffs without staging the files.

#### One-off comparison of initial vs final snapshots

```bash
./pg/diff-json.sh pg/local_storage_initial.json pg/local_storage_final.json
```

#### Generate pretty-printed files for manual inspection

```bash
./pg/jsonfmt.py pg/local_storage_initial.json > pg/local_storage_initial.pretty.json
./pg/jsonfmt.py pg/local_storage_final.json   > pg/local_storage_final.pretty.json
```

#### Wire into `git diff` for tracked JSON files

Already configured in this repo:

```bash
git config diff.json.textconv "pg/jsonfmt.py"
git config diff.json.binary false
```

And `.gitattributes` contains:

```gitattributes
pg/*.json diff=json
```

Now `git diff` on tracked `pg/*.json` files shows key-sorted, unescaped changes instead of escaped-string churn. (For untracked files, use `pg/diff-json.sh`.)

### Grounding Check (LLM template)
Attach seed data + schema, then verify:
- Does each referenced entity exist in seed data?
- Are exact names/titles/casing correct?
- Are requested actions supported by schema/reducer/state model?
- Are there ambiguous duplicates?
- Are current-state claims correct?

### Prompt Realism Check
Use the "Fragmented Semantic Realism Detection" prompt to score 0–10 and flag synthetic stitching.

---

## 9. Task Learnings

> Extracted from reviewing `returnmax-hard-005`. These apply across tasks.

### "From scratch" / new-user state isolation is a GYM check

When the prompt says "file from scratch" or switches to a new user, the gym should **isolate state** per-user — prior users' income forms, deductions, credits, documents, linked accounts, and prior-year metadata should not carry over. If they do, this is a **GYM Issue** (no state isolation), not a trainer execution error. The diff should show a clean slate for the new user; verify by checking whether old entities (W-2s, 1099s, deductions, credits) from the prior user still appear in the final state.

### Read-only UI steps leave no localStorage trail

Actions like "verify the Profile review shows…" or "check whether he qualifies as Head of Household" are read-only/verification steps. They typically write nothing to localStorage. The trajectory **video** is the ground truth for these — not the diff, and not the AI Summary.

### AR false positives are common — validate before classifying

The AR review for `returnmax-hard-005` flagged seven items; five were false positives. Always cross-check AR claims against the actual diff and trajectory video before treating them as issues. Common categories of AR false positives:

- **PII in prompt** — if the information (SSN, email, phone) is operationally required by the task, the AR flag is noise.
- **Incomplete diff interpretation** — AR sees only one field change and assumes the other fields were missed, when they were already correct in the initial state.
- **Missing evidence for read-only steps** — AR expects localStorage evidence for verification actions that are purely UI/navigational.

### DOB propagation is a known GYM blind spot

Onboarding in the ReturnMax gym updates `rm_tax_return.personalInfo.dateOfBirth` but may not propagate the value to `auth.currentUser.dateOfBirth` or `users.state.users[<id>].dateOfBirth`. When checking DOB consistency, verify **all three locations**. Seed data DOB may also contradict the prompt — this is a GYM/seed issue, not a trainer error.

### Diff silence ≠ missed action

If a field was already correct in the initial state (e.g., two of three text notifications already `true`), the diff won't show a change for it. That doesn't mean the step was skipped. Always confirm by reading the **initial state**, not just the diff patch.

### Classify issues as you go

Each finding should be bucketed (Trainer → GYM → AR) as soon as it's identified. The classification for `returnmax-hard-005` evolved over discussion: four initial issues narrowed to one real Trainer Issue and one GYM Issue, with the AR flags mostly collapsing to false positives. Early classification prevents bias and keeps the review focused on what matters.

---

## 10. Iteration Log

- **2026-06-21** — Created initial review flow from `pg/MSR-Production-Review-checklist.md`.
- **2026-06-21** — Added Harness UI navigation tip: Trajectory tab reached via Human Run card click; must select a specific run first.
- **2026-06-21** — Added Harness Trajectory Layout section: recording/transcript, file browser contents, and localStorage diff modal tabs (AI Summary, JSON Diff V2, JSON Diff V1).
- **2026-06-21** — Added Sheet Status Rules to Section 3: Not Started → Ready for Calibration / Rejected - Gym Issues, with Gym Issues comment requirements.
