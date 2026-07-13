---
template: Harness Task Review
description: Review note template for Harness prompt submissions
tags: [harness, review, task]
---

**Sheet Link:** https://docs.google.com/spreadsheets/d/1ZdeP_aaCOrrbBMRPZp4SvY1dtI_5yx_FnKeBUq07iKI/edit?gid=0#gid=0&range=3692:3692
**Task Link:** https://rl-gym-harness.turing.com/v2/tasks/fb55e9bd-80d1-42a1-aef2-bb18bd877cf0

### Prompt

Maya's My Info needs correcting after some life changes. Update her occupation to Enrolled Agent and check the uncommon-situations box for being a full-time student. Set her marital status to Divorced, indicate she has two dependents, and change her contact details to 1455 Market Street, Suite 600, Zip 94103 with phone 415-694-2231. Leave the City, State and SSN on file unchanged. Then add the dependent she now supports: an "Another person" dependent named Rohan Velu, born 08/22/2007, SSN 412-55-8830, a U.S. citizen, whose relationship to her is Nephew. Answer the support questions this way: he lived with her the whole year and all of that time was in the U.S., he did not live with another relative for more than six months, his gross income was under $5,200.00, and she paid more than half of his living expenses. Leave his uncommon-situation boxes unchecked, and on the home page select that she paid more than half the cost of keeping up her home. With the divorced status and this qualifying dependent, the filing status resolves to Head of Household, which she keeps.

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


---

## Notes

#### Reported Gym Issue

```
[R1]: 06/21/2026 - Aditya Bhaople  
[GYM Issue]:  
- The trainer added a new dependent, Rohan Velu, to Maya’s profile with the relation set as “Nephew.” This action is saved in the final JSON under ReturnMax (v1) → rm_tax_return → state → dependents. However, the newly created record has inconsistent relation values: the `relationship` field is set to “son,” while the `personRelationship` field is set to “Nephew.”
```



