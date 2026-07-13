---
template: Harness Task Review
description: Review note template for Harness prompt submissions
tags: [harness, review, task]
---

**Sheet Link:** {{sheet_row_link}}
**Task Link:** {{task_url}}

### Prompt

{{paste_prompt_here}}

#### Breakdown

{{break_prompt_into_verifiable_actions_as_checklist}}

---

## State Verification Report

Compared `pg/local_storage_initial.json` → `pg/local_storage_final.json` with `./pg/diff-json.sh`.

### Verdict

{{one-line_summary:_trainer_execution_correct_or_not._issues_found.}}

### Action-by-action verification

| # | Prompt action | Expected diff | Removed/updated? | Notes |
|---|---|---|---|---|
| 1 | {{action}} | {{what_the_diff_should_show}} | ✅/❌ | {{detail}} |

### Issues that matter for the review

1. **{{issue_title}} ({{Trainer/GYM}} Issue).**  
   {{description_of_observed_vs_expected.}}

### Summary

{{concise_summary_of_what_was_correct_and_what_was_not.}}

---

## AR Review Findings

AR review output saved at `pg/ar-review.md`. Assessed against the actual diff and the trajectory video.

### AR flags assessment

| AR flag | Valid? | Classification | Reason |
|---|---|---|---|
| {{ar_flagged_item}} | ✅/⚠️/❌ | {{Trainer/GYM/AR_Issue_—_subcategory}} | {{why_the_flag_is_right_or_wrong.}} |

### Net AR-derived issues

{{what_AR_got_right_and_wrong.}}

### Combined verdict

**Sheet status: {{Ready_for_Calibration_or_Rejected_-_Gym_Issues}}.** {{final_rationale.}}
