---
template: Harness Task Review
description: Review note template for Harness prompt submissions
tags: [harness, review, task]
---

**Sheet Link:** https://docs.google.com/spreadsheets/d/1ZdeP_aaCOrrbBMRPZp4SvY1dtI_5yx_FnKeBUq07iKI/edit?gid=0#gid=0&range=3690:3690
**Task Link:** https://rl-gym-harness.turing.com/v2/tasks/516e98a2-cb17-4dda-ba1c-9658faf7faf0

### Prompt

Cleaning up my numbers before I file as Head of Household. I want to update three areas in Deductions & Credits and then lock in my deduction method for 2025. Open my existing San Francisco Marin Food Bank cash gift and raise it from $1,250.00 to $2,400.00. Then add a second cash donation of $900.00 to "Habitat for Humanity". Add a traditional IRA contribution I never entered. In the IRA section, choose traditional only, indicate I contributed for the 2025 tax year, and that it is not a repayment of a distribution and not a military reservist distribution. On the amounts screen, enter total 2025 traditional IRA contributions of $4,000.00, and enter $10.00 for the amount contributed between January 1, 2026 and April 15, 2026, leaving the military reservist repayment blank. My four 2025 federal quarterly estimated payments were actually $700.00 each, not $450.00. Open Federal estimated taxes for 2025 (Form 1040-ES). It opens on a read-only summary of the current $1,800.00 total, so step back into the quarterly payment detail and set every quarter to $700.00 (total $2,800.00), leaving the payment dates unchanged. On the Deductions & Credits overview, set the deduction method to standard.

#### Breakdown

- [ ] Open existing SF Marin Food Bank cash gift, change amount from $1,250 to $2,400
- [ ] Add second cash donation: $900 to "Habitat for Humanity"
- [ ] Add traditional IRA contribution:
	- [ ] Traditional only (no Roth)
	- [ ] Contributed for 2025 tax year
	- [ ] Not a repayment of a distribution
	- [ ] Not a military reservist distribution
	- [ ] Total 2025 traditional IRA contributions: $4,000.00
	- [ ] Contributions between Jan 1, 2026 and Apr 15, 2026: $10.00
	- [ ] Military reservist repayment: leave blank
- [ ] Update all four 2025 quarterly estimated tax payments from $450 → $700 each
- [ ] Leave payment dates unchanged
- [ ] Set deduction method to standard on Deductions & Credits overview

---

## State Verification Report

Compared `pg/local_storage_initial.json` → `pg/local_storage_final.json` with `./pg/diff-json.sh`.

### Verdict

**Trainer execution is correct.** All prompt actions are reflected in the diff. One pre-existing GYM issue confirmed: editing a charitable contribution amount clears the other fields on the same record.

### Action-by-action verification

| # | Prompt action | Expected diff | Removed/updated? | Notes |
|---|---|---|---|---|
| 1 | Update SF Marin Food Bank cash gift: $1,250 → $2,400 | `amount` = 2400 | ✅ Updated | Amount changed correctly. GYM issue: `description` ("Monthly donations") and `dateContributed` ("2025-11-10") were cleared to empty strings. |
| 2 | Add $900 cash donation to Habitat for Humanity | New entry in `charitableContributions` with `organizationName: Habitat for Humanity`, `amount: 900`, `type: cash` | ✅ Added | `dateContributed` and `description` are empty (prompt didn't specify these) |
| 3 | Add traditional IRA: traditional only, 2025, not repayment, not military | `ira.traditional` populated; `ira.roth` = null | ✅ Added | `contributedInTaxYear: true`, `isRepaymentOfDistribution: false`, `militaryReservistDistribution: false` |
| 4 | IRA amounts: $4,000 total, $10 Jan-Apr 2026, no reservist repayment | `amount: 4000`, `contributionsAfterYearStart: 10`, `militaryReservistRepayment: 0` | ✅ Updated | `deductibleAmount` auto-set to 4000 |
| 5 | Update all four quarterly estimated payments: $450 → $700 | Each quarter `amount` = 700 | ✅ Updated | Q1: 700, Q2: 700, Q3: 700, Q4: 700; dates unchanged |
| 6 | Leave payment dates unchanged | `datePaid` values unchanged | ✅ Unchanged | Q1: 2025-04-15, Q2: 2025-06-16, Q3: 2025-09-15, Q4: 2026-01-15 |
| 7 | Set deduction method to standard | `deductionMethod: "standard"` | ✅ Updated | Changed from "itemized" |
| 8 | IRA added to tax break selections | `selectedTaxBreakIds` includes "ira" | ✅ Added | |

### Issues that matter for the review

1. **Editing charitable contribution amount clears other fields (GYM Issue — pre-existing).**  
   Confirmed: when updating the SF Marin Food Bank cash gift amount from $1,250 to $2,400, the gym cleared `description` (from "Monthly donations" to "") and `dateContributed` (from "2025-11-10" to ""). The save path overwrites the full record instead of merging the partial edit. Record ID `17d49cf3-85d1-505b-b384-ff08c738cc78`.

### Summary

All prompt actions executed correctly: Food Bank amount updated, Habitat for Humanity added, IRA added with all specified parameters, quarterly estimated payments updated, deduction method switched to standard. One GYM issue (pre-existing): editing a charitable contribution amount overwrites other fields on the record.

---

## AR Review Findings

AR approved — no flags.

### Net AR-derived issues

None.

### Combined verdict

**Sheet status: Rejected - Gym Issues.** Trainer execution is correct. One GYM issue: editing an existing charitable contribution amount clears the `description` and `dateContributed` fields instead of preserving them.

---

## Notes
- AR for this task is approved

#### Reported Gym Issue:

```
[R1]: 6/21/2026  
When updating an existing cash charitable contribution in Deductions & Credits, editing only the amount clears other fields on the same record instead of preserving them.  
  
Location: ReturnMax (v1) → rm_tax_return → state → deductions → charitableContributions → record id: "17d49cf3-85d1-505b-b384-ff08c738cc78" (San Francisco Marin Food Bank)  
  
Steps: Open the existing Food Bank cash gift and change the amount from $1,250.00 to $2,400.00.  
  
Expected: Only amount updates; description and dateContributed stay as they were.  
  
Actual: amount updates correctly to 2400, but description goes from "Monthly donations" to "" and dateContributed goes from "2025-11-10" to "". The record ID is unchanged, so this looks like the save/update path is overwriting the full object and dropping unchanged metadata rather than merging partial edits.
```
