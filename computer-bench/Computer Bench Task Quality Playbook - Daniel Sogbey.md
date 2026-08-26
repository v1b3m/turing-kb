# Computer Bench Task Quality Playbook

**By Daniel Sogbey**

*A practical field guide for building fair, reproducible, reviewer-ready tasks.*

*Transcribed to Markdown from the original 13-page PDF on 2026-08-26.*

> **Core principle**
>
> Do not start by trying to make the model fail. Start by understanding the task well enough to know what a fair failure looks like.
>
> Only 1/5 or 2/5 full GLM-5.2 passes are acceptable for our target standard.

---

## 0. Unofficial Companion Note

This playbook is an unofficial practical companion. It is meant to help trainers think clearly and work consistently, but it does not replace lead instructions, the QC platform, onboarding material, or the project quality standards.

**Use it together with the official resources:**

- New QC platform
- Meeting recording (QC platform demo)
- Meeting notes
- Updated Onboarding Documentation
- Onboarding Meeting Notes
- Onboarding Meeting Recording
- QC Task Quality Standard
- Shannon Task QC North Star

---

## 1. The North Star

Computer Bench quality is not mainly about running commands or passing a QC script. Those matter, but they are mechanics.

**The real skill is task judgment:**

- Can I understand the user request without reading the answer first?
- Can I explain the inputs and rules in plain English?
- Can I solve or trace one representative case myself?
- Can I predict what a fair verifier should check?
- Can I tell the difference between a real model failure and a broken task?
- Can another reviewer reproduce my result from the files I upload?

**The target is a task that is:**

- realistic,
- solvable,
- deterministic,
- reviewer-worthy,
- reproducible,
- difficult because of visible reasoning, not hidden traps.

**Preferred difficulty:**

- Ideal: 1/5 or 2/5 full GLM-5.2 passes.
- Not acceptable for final submission: 3/5, 4/5, or 5/5; continue fair hardening.
- Risky: 0/5; do not submit unless a lead explicitly approves and solvability is strongly proven.

---

## 2. The Whole Flow In One View

**Use this loop for every task, regardless of domain:**

1. Preserve the fresh task.
2. Read `instruction.md` first.
3. Read the input files and policy files.
4. Translate the task into plain English.
5. Solve or trace one representative case yourself.
6. Ask pressure-test questions.
7. Build your own expected verifier checklist before trusting the real verifier.
8. Compare your expected verifier with the actual verifier.
9. Identify real gaps.
10. Harden the prompt, inputs, solution, and verifier together.
11. Run Oracle and require 1.000.
12. Run GLM-5.2 and target 1/5 or 2/5 full passes.
13. Inspect failures and confirm they are model-owned.
14. Run QC and use human judgment on findings.
15. Package evidence so the reviewer can reproduce the result.

**Short version:**

Read, understand, trace one case, pressure-test, build your own verifier, compare with the real verifier, harden from visible gaps, then prove with Oracle, GLM, QC, and packaging.

**Workflow map:**

```
Fresh task
  -> instruction.md cold read
  -> input and policy review
  -> plain-English understanding
  -> one representative case
  -> pressure-test questions
  -> expected verifier checklist
  -> real verifier comparison
  -> fair hardening
  -> Oracle 1.000
  -> GLM 1/5 or 2/5
  -> QC and stability
  -> reproducible upload package
```

**Gold standard:**

The task should feel clear to a careful human, but difficult for a model that skims, shortcuts, or fails to reconcile files.

---

## 3. Start With The Prompt, Not The Answer

Open only `instruction.md` first.

**Ask:**

- What is the user actually asking for?
- What files or evidence does the prompt mention?
- What outputs are required?
- What constraints are real user needs?
- What wording sounds like a harness or benchmark checklist?
- Is anything ambiguous before I see the solution?

Do not open the solution first. The answer can make a weak prompt feel clearer than it really is.

**Never:**

Do not start by editing, adding edge cases, or trusting the answer key. Start by understanding the user's request.

**Good prompt cleanup:**

- Make the request sound like a real person delegated the work.
- Remove harness language such as working directory boilerplate, `/app` framing, and final-action ceremony.
- Keep exact deliverable names when the task genuinely needs structured outputs.
- Keep policy constraints that affect the answer.
- Do not leak exact final values.
- Do not make the task vague just to sound natural.

**Prompt realism test:**

Would a real manager, analyst, engineer, nurse, accountant, reviewer, or operator plausibly ask for this work in this way?

---

## 4. Understand The Inputs Like A Human

Read the policy or rules file before the data table when one exists.

**Examples:**

- Finance: read the accounting policy before the CSVs.
- Code/security: read the security rule or coding standard before the source files.
- Health: read the procedure before the patient/site rows.
- Travel: read the connection-time policy before the itinerary rows.
- Legal/general: read the rubric or contract standard before the clause matrix.

Translate the domain into plain English.

**Use this table pattern:**

| Term or Rule | Plain English | Model May Misunderstand | Fair Hardening |
|---|---|---|---|
| A policy term | What it means simply | The likely shortcut | A visible case that tests it |
| A threshold | When it applies | Off-by-one or wrong comparison | A boundary case |
| An exception | When the normal rule changes | Apply the main rule blindly | A clean exception case |
| A required output | What the user needs | Correct reasoning saved wrong | Verifier checks file and content |

**Important principle:**

Do not invent hard cases out of nowhere. Root each hardening idea in a policy rule, missing input coverage, realistic edge case, verifier gap, or observed model shortcut.

**Gold standard:**

Every hardening idea should have a sentence that starts with: "This comes from..." and points to a visible rule, row, verifier gap, or model shortcut.

---

## 5. Solve Or Trace One Case Yourself

Before asking a model to harden the task, prove you understand one representative case.

**Depending on the domain, this means:**

- Finance: calculate one row or account.
- Code: trace one function, route, migration, or dataflow.
- Health: classify one patient, site, culture, or allocation.
- Travel: calculate one connection or itinerary.
- Legal/general: audit one clause, item, or document row.

**For that one case, identify:**

- the input facts,
- the governing rule,
- the decision or calculation,
- the expected output,
- the shortcut a model might take.

If you cannot do one case by hand, you are not ready to harden the task.

---

## 6. Ask Pressure-Test Questions

Pressure-test questions are how you find fair difficulty.

**Ask:**

- What if the normal rule has an exception?
- What if two rows look similar but require different treatment?
- What if a threshold boundary is exactly hit?
- What if a date, timestamp, or version changes which rule applies?
- What if one file says the general rule and another file gives the override?
- What if a value is real but should not count toward the main metric?
- What if an item is partially approved, partially excluded, or partially expired?
- What if the model uses one shortcut formula everywhere?
- What if the memo sounds right but the numbers are wrong?
- What if the numbers are right but the required file or JSON summary is wrong?
- What if the verifier checks file existence but not the actual hard decision?

**Turn each good question into one of:**

- a data case,
- a verifier check,
- a prompt clarification,
- a solution correction.

---

## 7. Where Real Gaps Usually Come From

Look for gaps in five places:

| Gap Source | What To Look For | Good Hardening |
|---|---|---|
| Policy rules | Exceptions, thresholds, precedence, time windows | Add a visible case that forces the rule |
| Input coverage | Only easy examples, no near misses, no negative cases | Add realistic contrasting rows |
| Verifier coverage | Checks too few files, weak totals, no negative controls | Add deterministic checks |
| Prompt realism | Harness language, answer leakage, machine-shaped instructions | Humanize without losing requirements |
| GLM trajectory | Model shortcut, missed rule, wrong file, bad aggregation | Harden the exact visible failure mode |

**Bad hardening:**

- hidden requirements,
- ambiguous wording,
- missing files,
- brittle prose-only checks,
- random extra rows,
- unverifiable subjective requirements,
- changing the task into a different task.

**Good hardening:**

- visible in the files,
- rooted in the policy,
- realistic for the domain,
- deterministic to grade,
- solvable by a careful human or model,
- aligned across prompt, input, solution, verifier, and evidence.

**Mini example from `fin-f30-percentage-of-completion`:**

| Fresh Observation | Fair Question | Hardened Case | Verifier Pressure |
|---|---|---|---|
| Fresh data had an unagreed claim only | What if a claim is fully agreed or partly agreed? | Add agreed and split-claim contracts | Check that agreed amounts are included and unagreed amounts are excluded |
| Fresh data had unstabilized materials | What if other real cost should not show progress? | Add abnormal wasted cost | Check progress excludes waste while cost of sales includes it |
| Policy says onerous provision cannot go below zero | What if the formula produces a negative provision? | Add a zero-floor near miss | Check provision is 0.00, not negative |
| Only a few basic contracts existed | What if similar rows need different treatment? | Add near-identical clean and dirty cases | Check the model did not use one shortcut formula |

**Why this works:**

The hardening is not random. Each new case is rooted in a visible policy rule or missing counterpart in the fresh data.

---

## 8. Build Your Own Verifier Before Trusting The Real One

The prompt defines the task. The verifier is only the marking scheme.

Before writing `tests/verifier.json`, write what a fair verifier should check.

**Group by deliverable:**

- required files,
- headers or schema,
- row-level answers,
- totals and summaries,
- memo or explanation requirements,
- JSON fields,
- negative controls,
- formatting only when the prompt asks for it.

**Then compare your checklist with the actual verifier:**

| Result | Meaning | Action |
|---|---|---|
| Covered | The verifier checks what the prompt requires | Keep |
| Partially covered | It checks the easy part but misses the hard part | Strengthen |
| Missing | A required output or rule is not checked | Add a deterministic check |
| Extra | Verifier checks something not asked | Remove or align prompt/verifier |
| Brittle | Equivalent correct answers may fail | Make less wording-sensitive |

**Say this out loud when training others:**

> I do not start by trusting the verifier. I first build my own expected verifier from the prompt, then compare it with the actual verifier. If the verifier is missing important checks, I harden it. If it checks something the prompt did not ask for, I remove or adjust it. The verifier should grade the real task, not hidden preferences.

---

## 9. How To Harden Verifiers Fairly

A stronger task needs a stronger verifier.

Good verifier checks are deterministic and mapped to the prompt.

**They can check:**

- output files exist,
- CSV headers and row count,
- exact numeric rows when the output is deterministic,
- totals match row values,
- JSON keys and totals,
- required classifications,
- policy-specific decisions,
- memo coverage for important judgments,
- negative controls,
- no obvious extra or missing records.

**Use canary checks.**

A canary check is a smoke alarm for a common shortcut. It checks one case that catches a likely model mistake.

**Examples:**

- A boundary row catches `>=` vs `>`.
- A partial-approval row catches all-or-nothing reasoning.
- A negative credit catches always-add logic.
- A duplicate organism catches double-counting.
- A safe/unsafe code pair catches shallow pattern matching.
- A timezone offset catches local-time assumptions.

**Avoid:**

- exact prose wording unless the wording itself is required,
- hidden answer phrases,
- date-dependent checks,
- live web checks,
- unordered output assumptions,
- checks that reward only formatting while ignoring reasoning.

**Never:**

Do not use the verifier to hide the real task. A correct answer should pass because it solved the visible request, not because it guessed a private checklist.

---

## 10. Use An LLM As A Reviewer, Not A Replacement

AI assistance is useful, but human judgment stays responsible.

You can use any capable LLM for this review loop, for example Codex, Claude, ChatGPT, or another project-approved assistant.

**Good use of an LLM:**

- summarize the prompt,
- translate policy rules,
- inspect verifier coverage,
- suggest fair hardening ideas,
- compare expected and actual verifiers,
- identify brittle checks,
- help edit files consistently,
- review GLM trajectories.

**Bad use of an LLM:**

- accepting edits blindly,
- letting it invent hidden rules,
- using it to add random traps,
- trusting it when it cannot cite file evidence,
- letting it remove required outputs for the sake of natural wording.

**Rule:**

Every LLM suggestion must be checked against the prompt, input files, solution, verifier, and reviewer expectations.

**Gold standard:**

The LLM should help you see more clearly. It should not become the reason you stop thinking.

---

## 11. Evidence: Prove The Task Is Solid

A task is not ready because it "looks hard." It is ready when the evidence supports it.

**Minimum evidence:**

- Oracle reward is exactly 1.000.
- GLM-5.2 has zero infrastructure exceptions.
- GLM full-pass count is 1/5 or 2/5.
- Failures are model-owned, not task-owned.
- Unified QC returns Keep or has clearly documented advisory false positives.
- Final QC has no blocking critical/high findings.
- Stability regrades pass for the frozen answer.
- The final package contains all required evidence.

**Model-owned failures:**

- missed a visible rule,
- applied the wrong precedence,
- used a shortcut formula,
- missed a special case,
- wrote the wrong file or incomplete output,
- failed to reconcile totals.

**Task-owned failures:**

- missing file,
- ambiguous instruction,
- hidden requirement,
- broken environment,
- verifier checks unasked behavior,
- solution and verifier disagree,
- nondeterministic output.

Do not count task-owned failures as difficulty.

**Before upload:**

If GLM fails, read the trajectory. If the task caused the failure, fix the task. If the model missed a visible rule with all needed files present, that is valid difficulty.

---

## 12. QC Is Advisory, Not A Substitute For Judgment

QC is a guide. It is not the whole review.

Use either local QC or the QC platform when the tracker allows it. If the task is being reworked through the new platform flow, treat the platform output as the final reviewer-facing evidence and include it in the upload package.

**Fix real issues:**

- missing files,
- broken Docker or environment,
- prompt leaks,
- verifier mismatch,
- missing trajectories,
- unsupported claims,
- packaging mistakes,
- blocking Final QC findings.

**Document false positives clearly:**

- explain why the check does not fit the task,
- cite the files that prove the task is still valid,
- rerun QC after task-affecting changes.

Do not get stuck chasing false positives that do not apply.

---

## 13. Packaging Checklist

Use the current tracker column as the source of truth.

**For the newer Upload_reworked-QC flow, prepare one self-contained zip:**

`<task-id>-upload-reworked-qc.zip`

The zip should open directly to the task package contents, not to a pile of unrelated old zips.

```text
task.toml
instruction.md
README.md
environment/
tests/
solution/
  golden_trajectory.json
evaluations/
  glm-5-2/
    r1/
    r2/
    r3/
    r4/
    r5/
  stability/
    r1/
    r2/
    r3/
    r4/
    r5/
    golden_trajectory.json
qc/                        # Unified QC output: summary.md, results.json, results.csv
verifications/             # Final QC / QC platform evidence
REWORKED_QC_README.json    # trainer note
```

Each GLM run folder should contain enough evidence for a reviewer to inspect the attempt:

```text
evaluations/glm-5-2/r1/
  agent/trajectory.json
  result.json
  verifier/
```

Each stability folder should contain the repeated verifier result for the frozen answer:

```text
evaluations/stability/r1/
  result.json
  verifier/
```

The `qc/` folder is for Unified QC output:

```text
qc/
  summary.md
  results.json
  results.csv
```

The `verifications/` folder is for Final QC or QC platform evidence:

```text
verifications/
  report.html
  summary.json
  report.json
  full_report.json
  issues.csv
  unified-qc/
```

`REWORKED_QC_README.json` is a top-level trainer note alongside `qc/` and `verifications/` (the PDF's ASCII tree nests it oddly, but its prose — "include it in the upload package", "write a short file-grounded explanation in REWORKED_QC_README.json" — places it with the package evidence).

**Important rules:**

- Keep `solution/golden_trajectory.json` inside the task.
- Also include a root copy of `golden_trajectory.json` when the sample/package format expects it.
- Include `tests/test.sh`, `tests/test_outputs.py`, `tests/verifier.json`, and any verifier libraries used by the task.
- Do not include `.DS_Store`, scratch folders, old upload zips, cache folders, or unrelated attempts.
- If `verifications/report.html` says PASS, the package is clean from the Final QC gate.
- If it says CONDITIONAL, inspect the issue. Fix real critical or high issues. For advisory false positives, write a short file-grounded explanation in `REWORKED_QC_README.json`.

---

## 14. Trainer Note Template

Keep notes simple, technical, and human.

**Template:**

```
I hardened this from a basic task into a fair reasoning task. Final validation: Oracle 1.000
with 0 exceptions, GLM-5.2 X/5 full passes, mean Y, 0 exceptions, Unified QC Keep, and Final QC
PASS.

The difficulty comes from visible task facts, not hidden requirements. I strengthened the
prompt, inputs, solution, and verifier together so the model has to apply the policy across
realistic edge cases and reconcile the required outputs.

The final package includes the task, golden trajectory, five GLM runs, stability evidence, and
QC reports so a reviewer can reproduce the result.
```

**If QC has advisory findings:**

```
QC raised [finding]. I treated it as advisory because [file evidence]. The task still has [
Oracle/GLM QC evidence], and the verifier checks the required deliverables deterministically.
```

---

## 15. Prompt Bank

**Cold-read prompt:**

Read `instruction.md` only. Explain the task in plain English, list the required deliverables, identify any ambiguity or machine-like wording, and do not read the solution yet.

**Input-understanding prompt:**

Read the input and policy files. Explain the task rules in plain English, then identify what difficulty coverage is present and what important realistic edge cases are missing.

**Manual-case prompt:**

Using only the task inputs, trace one representative case step by step. Cite the input facts, apply the governing rule, show the expected output, and identify the model shortcut this case should catch.

**Expected-verifier prompt:**

Using `instruction.md` only, create an expected verifier checklist. Group it by deliverable. Mark which checks are essential, which are nice-to-have, and which would be unfair because the prompt did not ask for them. Do not read `tests/verifier.json` yet.

**Verifier-comparison prompt:**

Compare your expected verifier checklist from `instruction.md` with `tests/verifier.json`. Mark each expected item as Covered, Partially Covered, Missing, or Extra. Then explain what reviewer risk each gap creates.

**Fair-hardening prompt:**

Suggest changes to make this task harder without changing the core user request. Focus on visible policy rules, missing edge cases, near-miss rows, verifier gaps, and model shortcuts.

For each idea, explain what changes, why it is harder, what files need updating, and any fairness risk. Do not edit files yet.

**Verifier-hardening prompt:**

Review `tests/verifier.json` against the prompt, input files, and final expected outputs.

Identify missing checks, extra unfair checks, brittle wording checks, and places where the verifier gives credit too easily. Suggest deterministic verifier improvements only.

**GLM-failure review prompt:**

Review this GLM trajectory and classify the failure. Was it model-owned or task-owned? Cite the exact prompt, input, output, verifier, or trajectory evidence for your conclusion. If task-owned, suggest the smallest fair fix.

**QC-judgment prompt:**

Review the QC report findings. Separate real task issues from advisory or false-positive findings. For each real issue, propose a fix. For each advisory finding, write a short human-
judgment explanation grounded in the task files.

---

## 16. Final Reviewer-Worthy Checklist

**Before upload, confirm:**

- The prompt sounds human and natural.
- The task is not vague.
- The task has no hidden requirements.
- Input files contain enough realistic edge cases.
- The solution matches the final inputs.
- The verifier maps to the prompt.
- The verifier checks the hard cases.
- Oracle is 1.000.
- GLM difficulty is 1/5 or 2/5.
- GLM failures are genuine reasoning misses.
- QC outputs are present.
- Packaging matches the current standard.
- Trainer note explains what changed and why.

**Closing principle:**

The best tasks are not confusing. They are clear, fair, and just deep enough that a fast model shortcut fails while a careful solver can still get them right.
