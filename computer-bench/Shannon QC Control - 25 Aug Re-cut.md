---
title: "Shannon QC Control - 25 Aug Re-cut"
tags:
  - computer-bench
  - qc
  - spec
---
# Resources · Shannon QC Control

> [!info] What changed on 25 Aug 2026
>
> The trainer flow was re-cut in the all-hands. If you learned this job before that date, these six are the differences:
>
> - **Four GLM runs, not five.** A finished five-run task does not need re-running — see [the gates](https://qc-api-713053229214.us-central1.run.app/resources#the-job).
> - **The client accepts 1, 2 or 3 of 4 passing.** 4 of 4 is rejected as too easy. Stop hardening once you are inside the band.
> - **There is no score.** The bar is fourteen clean review areas, not 95.
> - **Stability is Turing's to run**, not yours — along with Oracle mode and the 0-of-4 re-run. See [Who checks what](https://qc-api-713053229214.us-central1.run.app/resources#division-of-labour).
> - **`review.csv` comes from the review form now**, not a hand-edited template. See [Step 6](https://qc-api-713053229214.us-central1.run.app/resources#review-csv).
> - **`qc_report.html` ships inside the bundle.** The client requires it at the task root — see [Step 5](https://qc-api-713053229214.us-central1.run.app/resources#step-5).

## What are the client's goals

The client's goals are to generate working (the tasks run fine) and accurate (the prompt is natural, the verifier is fair) tasks within their difficulty target, to train their model on their dataset (which we have mined to produce the base tasks). We have generated tasks to that spec. Your role is to help us ensure that these tasks are correct, accurate, functioning, and fair.

## What is a good task

The best and only way to do this well is to fully and properly learn what a good task is. The best analogy for this work is a teacher writing a test in school. The task is the test, the student is the AI.

Imagine you were in school, the teacher gave you a test (the task) and graded you based on the verifier rubric. Your job is to write a good test and a good rubric for the AI.

Your job is to take the draft tasks that come out of our system and use this framework to improve and correct them — like teachers taking a bad first draft of a test and turning it into a good, useful test that can actually honestly gauge a student's intelligence (the test = the tasks, the students = the AI).

A good task is a test given to a student (the AI) that is:

- functional, not broken, and hence can actually be learned (no software bugs)
- *appropriately difficult* for the student (the client accepts 1, 2 or 3 of the four GLM runs passing — never 4 of 4)
- honestly tests something the AI would be reasonably asked to do (prompt realism)
- fairly and objectively grades the outcome (rubric correctness)
- meets the formatting requirements required by the client (four GLM runs, one reward-1.0 run from a non-oracle model, the delivery QC report shipped in the bundle, file structure in the standard format)

## How we measure success

Our success is **not** dependent on hitting arbitrary criteria that we define. Our success, and the acceptance of a task, is *solely* dependent on the client. The client pays us per task, and that is why we can only pay on tasks accepted by the client, not by us or any QC system we have.

> [!warning] There is no score any more
>
> The client removed the score parameter on 25 Aug 2026. Chasing 95 or 85 is over: the bar is that
>
> **all fourteen review areas come back clean**
>
> and the client accepts the task. The QC platform still shows a score and still needs a Delivery Gate
>
> `PASS`
>
> before it will let you submit — treat that as our local gate, not the client's bar.

The QC system is a *reasonable approximation* of what the client wants — we have gone through a few rounds of changes with it *because* we have gotten further feedback from the client on their requirements. Make your tasks as high quality as you can, because that increases the chances of acceptance.

All of our systems are set up *to help* you get your tasks as good as possible. If they are good and meet the quality bar they will be accepted by the client as a paid task. This is also why we're not enforcing peer reviews at the moment — just the fact of tasks going through peer review doesn't ensure quality. What ensures quality is that everyone learns and works towards making tasks that **actually meet the bar of a "good task"**, not merely "completed".

> There is no alternative to you going through the task itself, checking step by step if the task is sound engineering and logic wise, working, and well designed, just like if you were a teacher designing an appropriate and reasonable test or curriculum for your student. This is also what will maximize the chance of task acceptance the most, NOT any number on a report.

## A lot of this work is debugging

- Is the task functional at all — is the underlying data, connector tools working or buggy? Is the logic in the task correct or broken? Does the task have one right answer that can be graded, or is it very ambiguous?
  - If it is buggy, wrong, broken, or ambiguous, can you fix it? If so, fix it; if not, drop it and move on. This is a lot of engineering work — a lot of bug fixing.
- Is the task well designed at all — is it a realistic task that an AI would actually do, or contrived? Are there obvious issues like the prompt is overly prescriptive (too much information), or the answer is leaked in the prompt?
  - If so, can you fix it? If not, drop it and move on.
- Are the verifiers accurate / non-buggy, correctly scoped, and fairly grade the task?
  - Does the verifier rubric actually grade the task accurately, or is it arbitrary, inaccurate, doesn't match up, and generally unfair? If the rubric has problems, can you fix it? If so, fix it; if not, drop it and move on. Are all the major items asked for in the prompt actually graded, and graded correctly? Or are the verifiers grading unfairly or grading items the model is not expected to do by the prompt?
- Do you have all the formatting and artifacts required by the client? Again: four GLM runs, one reward-1.0 run from a non-oracle model, `review.csv` and `qc_report.html` inside the bundle, file structure in the standard format.

## Do not game the system

Our QC script is just a helper for you to make good tasks. If you make a good task, it will probably pass the QC system. However, if you are just trying to pass the QC system, that doesn't mean the task is good, and we or the client *will* catch those cases. This is called **reward hacking**, and we are very sensitive to rejecting it.

If you game the system to just score high on the QC script — for example by making the verifiers arbitrarily easy or hard, or giving so much guidance in the prompt that it is not realistic — we or the client will find out that the prompt is actually not realistic, the verifier is actually broken, the task is actually unsound, and the task will not be accepted. It doesn't matter if the task got 95% on the delivery gate.

Your job is not to climb a score. Your job is to do everything you can to ensure, debug, and test that this task is actually correct, well-designed, and sound before handing it off. That is the best way to maximize your likelihood of task acceptance.

You *can* use AI to help you understand the task better and figure out what you need to fix. *You* need to make the decisions about what and how to fix those items — not just blindly telling AI to make modifications and blindly accepting them. One way to do this: ask the AI if the task is sound, if there are any correctness issues, structural or logic issues, or any other major issues of correctness that would block us from shipping this task to a client.

---

# Resources · Shannon QC Control

> [!info] What changed on 25 Aug 2026
>
> The trainer flow was re-cut in the all-hands. If you learned this job before that date, these six are the differences:
>
> - **Four GLM runs, not five.** A finished five-run task does not need re-running — see [the gates](https://qc-api-713053229214.us-central1.run.app/resources#the-job).
> - **The client accepts 1, 2 or 3 of 4 passing.** 4 of 4 is rejected as too easy. Stop hardening once you are inside the band.
> - **There is no score.** The bar is fourteen clean review areas, not 95.
> - **Stability is Turing's to run**, not yours — along with Oracle mode and the 0-of-4 re-run. See [Who checks what](https://qc-api-713053229214.us-central1.run.app/resources#division-of-labour).
> - **`review.csv` comes from the review form now**, not a hand-edited template. See [Step 6](https://qc-api-713053229214.us-central1.run.app/resources#review-csv).
> - **`qc_report.html` ships inside the bundle.** The client requires it at the task root — see [Step 5](https://qc-api-713053229214.us-central1.run.app/resources#step-5).

## What the job actually is

We build benchmark tasks: realistic, self-contained work packages that a frontier model (for us: **GLM-5.2**) tries to complete inside a Docker container. A grader ("verifier") then scores the model's output automatically.

The client doesn't want tasks the model can solve. They want tasks that are hard, fair, and honestly measured.

So your job on every task is:

1. Make sure the task is solvable and correctly graded (the gold answer scores a perfect **1.0**).
2. Make sure it's genuinely difficult for GLM-5.2 (**1, 2 or 3** of the four runs may fully pass — never all four).
3. Make sure the difficulty comes from real reasoning work, not from a confusing prompt, missing files, or a broken grader.

Most tasks arrive from the mining team already working — and too easy. Expect your first GLM battery to pass **4/4**. That is normal. Hardening it is the actual job, not a sign something went wrong.

### The two gates every task must clear

| Gate | Requirement |
| --- | --- |
| **Oracle** | Reward exactly **1.0**. Not 0.98, not 0.9167. Anything less is a defect in the task, never partial credit. It has to hold on repeated runs, not once. |
| **Difficulty** | Across **4** GLM-5.2 runs, the client accepts **1 of 4**, **2 of 4** or **3 of 4** fully passing. **4 of 4 is rejected** — the task is too easy. **0 of 4** can still be submitted, but Turing then has to re-run it on another frontier model to prove it is solvable at all, and only about a third of those are accepted. |

> [!warning] Four runs, not five
>
> (25 Aug 2026). If you already finished a task on five runs, you do
>
> **not**
>
> need to re-run it — drop one run and ship four. Mind the trap: a 5/5 task with one run removed is still 4/4, and 4/4 is rejected. Those need real hardening, not arithmetic. A 1/5 task should have a
>
> *failing*
>
> run dropped, so it reads 1/4.

"Fully pass" means reward exactly **1.0** on that run. A run that passes 8 of 10 verifiers is not a pass. Always report all four individual rewards, not just the average.

Those two gates are about the task. Separately, two files are about *you*: `README.md` and `review.csv`, both at the task root, both written by the reviewer. The QC platform blocks submission on `review.csv` specifically — see [§9](https://qc-api-713053229214.us-central1.run.app/resources#review-csv). Clearing both gates and forgetting the review record still cannot be handed over.

## Who checks what — you, and Turing

The review has fourteen areas (see [Step 6](https://qc-api-713053229214.us-central1.run.app/resources#review-csv)). **Twelve of them are yours.** The rest are ours, because they need access or models you do not have — and trainers are currently losing days to all of them:

| Area | Why it is ours |
| --- | --- |
| **Layer 3 · Oracle Mode** | We re-run the golden in modes you cannot reach. You still keep your own Oracle at 1.0 — that is what makes the task gradable — but the cross-mode replay is ours. |
| **Layer 2 Stability** | Re-grading the same frozen rollout to confirm the verifier answers the same way every time. We run it. You do not need to ship a `stability/` folder. |
| **Cross-trial · Calibration** | Reading the spread across all four trials — whether the failures have one understandable cause rather than noise — needs the aggregate we hold. We run it. |
| **The 0-of-4 re-run** | When none of your four GLM runs pass, we re-run the task on a frontier model you have no key for, to prove it is solvable. |

Everything else in the fourteen is yours, and the client reviews your answers on those.

> [!info] Two of these are rows in `review.csv`, and you may leave them empty
>
> *Layer 2 Stability*
>
> and
>
> *Cross-trial · Calibration*
>
> are ours to run, so the platform stopped requiring them on 26 Aug 2026 — blank, or missing altogether, and your package still submits. If you did look at either, write it down anyway: it is read, it just cannot block you.
>
> *Layer 3 Oracle Mode*
>
> is different — the cross-mode replay is ours, but your own oracle run at 1.0 is yours, so that row is still required.

## What a task package looks like

Every task is one folder. Zip **this folder only** when you submit:

```
task/                          # ← zip THIS folder only
├── task.toml
├── instruction.md
├── review.csv                 # your QC review — one row per review area
├── qc_report.html             # the Delivery Gate report — client requires it
├── README.md                  # your change summary
├── tests/                     # manifest.json, test_*.py, etc.
├── environment/               # Dockerfile, entrypoint, _app/ mirror, …
├── solution/
│   ├── golden_trajectory.json # required Oracle asset
│   ├── solve.sh / solve.py    # if present
│   └── …
└── evaluations/               # named folders only. Nothing loose in here.
    ├── solvability/           # can a model solve it? ONE reward-1.0 run
    │   └── r1/                # any model — GLM, Claude, GPT… — never the oracle
    │       ├── agent/trajectory.json
    │       ├── result.json
    │       └── verifier/reward.json
    ├── difficulty/            # how often? FOUR independent GLM rollouts
    │   ├── r1/
    │   │   ├── agent/trajectory.json
    │   │   ├── result.json
    │   │   ├── verifier/reward.json
    │   │   ├── verifier/verifier_summary.json
    │   │   └── config.json    # trial-level OK
    │   └── r2/ r3/ r4/
    ├── stability/             # Turing runs this — ship it only if you have it
    │   ├── repeat-01/result.json
    │   └── repeat-02/result.json
    └── platform/              # NOT YET — see the note below
        ├── e2b/oracle/        # required to be 1.0
        ├── e2b/run/           # + sanity_check.md
        ├── modal/oracle/      # required to be 1.0
        └── modal/run/         # + sanity_check.md
```

> [!warning] `evaluations/platform/` is agreed but not yet accepted
>
> It is part of the folder spec settled with the client on 25 Aug 2026, and it will be required. The Delivery Gate does not take it today:
>
> `R17`
>
> allows only
>
> `solvability/`
>
> ,
>
> `difficulty/`
>
> and
>
> `stability/`
>
> under
>
> `evaluations/`
>
> , and a directory the gate cannot name is scored as a
>
> *failed model run*
>
> — which can sink an otherwise clean bundle.
>
> **Do not add `platform/` until we tell you the gate has been updated.**
>
> It is drawn above so you know what is coming.

If any of the core pieces (`task.toml`, `instruction.md`, `tests/`, `environment/`, `solution/`) is missing, report it — don't repair it yourself.

Three files at the root are **not** part of the mined package — they are yours to add. `review.csv` is the record of your review, written through the [review form](https://qc-api-713053229214.us-central1.run.app/resources#review-csv); `README.md` is your change summary; and `qc_report.html` is the Delivery Gate's own report, which the client requires you to ship inside the bundle. The QC platform will not let you submit without `review.csv` — see [Step 6](https://qc-api-713053229214.us-central1.run.app/resources#review-csv), and [Step 5](https://qc-api-713053229214.us-central1.run.app/resources#step-5) for how the report gets in there.

### What you'll see when you open each one

**`instruction.md`** — a work request written as if a real colleague sent it. Example:

> "We are conducting a clinical equipment fit-eligibility audit across 30 resident prescriptions… evaluate each against the clinical profiles, recliner catalog, safety bulletins and Fit Policy PFR-2… save
>
> `recliner_fit_audit.csv`
>
> ,
>
> `recliner_fit_memo.md`
>
> ,
>
> `results.json`
>
> in your working directory."

One paragraph of context, the asks, and the exact deliverable filenames. Roughly 90–150 words of natural human voice. No `/app/input/` paths, no dumped column lists, no step-by-step recipe — real people don't write those, and recipes make the task trivially scriptable.

**`tests/manifest.json`** — a JSON list of verifiers. Each one has a name, a source (read this file / query this DB), and an assertion (equals this value / matches this JSON / a judged rubric). They look like this:

```
{
  "name": "audit_csv_exists",
  "source": { "type": "file", "file": { "command": "check_path_exists",
              "arguments": { "path": "recliner_fit_audit.csv" } } },
  "assertion": { "type": "deterministic", "expected": true,
                 "deterministic": { "path": "$.is_file", "comparison": "equals" } }
}
```

Each verifier is worth **1/N** of the run's score. That matters a lot — see [free points](https://qc-api-713053229214.us-central1.run.app/resources#free-points).

**`solution/`** — the known-correct answer, plus the script that produces it. This is what the Oracle run executes.

**`task.toml`** — the design doc. When the gold answer and a verifier disagree, this file settles which one is wrong.

## One-time setup

You need three things working before your first run: Docker, the harbor CLI, and your GLM API key.

```
# every single session — loads your keys
source ~/.config/harbor/env
```

That file sets:

- `OPENAI_API_KEY` / `GLM_API_KEY` — your personal LiteLLM proxy key
- `OPENAI_BASE_URL=http://34.41.10.8:4000/v1` — the team proxy
- `JUDGE_MODEL=openai/glm-5.2` — must be set, or every judged rubric silently scores 0

GLM-5.2 is the only model available to us. One personal key, one team proxy.

Set up a `glm-harbor-config.json` — that's the config you'll point every GLM run at. You edit only two fields per task: `tasks[0].path` and `job_name`.

Finally, sign in to the Shannon QC platform: [https://qc-api-713053229214.us-central1.run.app/](https://qc-api-713053229214.us-central1.run.app/) with your `@turing-gpt-git.com` Google account, paste your GLM key top-right → "Check Key" → wait for *glm-5.2 ready*. Don't touch the prefilled Base URL.

## 4. Read it like a picky reviewer

Before running anything, read `instruction.md` cold and — this is the highest-value thing you will do all day — write down every guess you have to make. Then compare your guesses to the gold answer in `solution/`.

Every guess you had to make is an ambiguity, and ambiguity is never a legitimate difficulty knob. If a careful analyst could land on two different defensible answers, the task is broken, not hard.

Also check:

- Does the prompt leak the answer? Stated totals/counts, a hint-sheet column name like `duplicate_bradley_removed`, or a spelled-out method = leakage.
- Does everything referenced actually exist? Every file, table, policy section.
- Do the verifiers cover every ask, and nothing that was never asked?
- Delete every verifier with `"category": "secondary"`. Only `"core"` ships. Do this before the Oracle run.

**Every verifier maps to exactly one item in the instruction — no more, no less.** That is the client's rule, and it is what *Verifier coverage and fairness* is judged on. Two ways to check it, and you want both:

- **Forward.** Read `instruction.md`, write your own list of the verifiers it implies, then tick each one off against `tests/manifest.json`. Anything on your list with no verifier is a *coverage gap*.
- **Backward.** For every verifier in the manifest that is not on your list, decide whether it genuinely follows from the instruction or should be removed. A verifier grading something the model was never told to do is unfair, and the client fails it.

While you are in there: **minimise strict regex matching**. The client called it out specifically — a verifier that demands exact wording rejects defensible correct answers. Match on the value, not on the phrasing.

Sketch your own rough solution while you're here. It gives you an expected answer to sanity-check the gold against, and shows you which parts are genuinely hard vs. mechanical.

## 5. Oracle run (must be exactly 1.0)

This runs the known-correct solution through the real verifiers. It proves the grading is sound. No model involved.

```
harbor run -p "$TASK" -a oracle \
  --ve OPENAI_API_KEY="$GLM_API_KEY" --ve OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  -o /tmp/harbor-jobs --job-name oracle-<task> -n 1 -y
```

Below 1.0? Open `verifier/test-stdout.txt`, find the failing assertion, and decide which side is wrong — the gold or the verifier. `task.toml` is the tiebreaker. Fix, re-run.

Re-run the Oracle after every single change to gold, verifiers, or instructions. No exceptions.

## Step 3 — 4 GLM-5.2 runs

This is the difficulty measurement. Point your `glm-harbor-config.json` at the task, then:

```
docker ps --format '{{.Names}}'          # ← budget check FIRST, every time
export OPENAI_API_KEY="$GLM_API_KEY"

harbor run -c glm-harbor-config.json -n 1 -y            # smoke test 1 run first
harbor run -c glm-harbor-config.json -n <3-minus-running> -k 4 -y   # the real 4-run battery
```

`-k 4` is total attempts (four, since 25 Aug 2026). `-n` is how many run at once (the budget throttles this, not the total). Running five is not wrong — we only look at four.

The commands above use the **opencode** agent, which is what we default to. QC no longer requires it: **terminus-2 runs are accepted** (2026-08-19) and the harness your runs were made on is recorded in the report rather than scored. Oracle runs use `-a oracle`.

Read the results:

```
cat /tmp/harbor-jobs/<job>/*/verifier/reward.txt   # the four rewards
harbor view /tmp/harbor-jobs                       # browser UI on :8080
```

Per-verifier detail is in `verifier/verifier_summary.json` — read whole `items[]` entries; don't grep for `"passed"`, because the same verifiers appear under multiple groupings and you'll miscount.

## 7. Interpret honestly, then harden

Before you call anything a "pass rate", sanity-check the runs:

| What you see | What it usually means |
| --- | --- |
| Reward exactly 0.0 | Suspect a crash, not difficulty. Check for `exception.txt` / missing `trajectory.json`. A crashed run never counts. |
| 0.9, 0.1, 0.9, 0.1, 0.5 | Bimodal = a coin-flip on an ambiguity. Fix the prompt; this isn't calibrated difficulty. |
| 4/4 passing | Too easy, and rejected outright. Harden. |
| 1/4, 2/4 or 3/4 passing | Inside the band the client accepts. Stop hardening. |
| 0/4 passing | Submittable, but Turing has to re-run it on another frontier model first and only about a third get accepted. Worth a look at whether the task is unfair rather than hard. |
| Steady 0.15–0.35, no full passes | A genuinely hard task. This is what good looks like. |
| >20% of runs infra-broken | Return the task for repair; don't average the noise. |

For each failing run, classify why it failed: **MODEL** (good — real difficulty), **ambiguity** (fix the prompt), **verifier bug** (fix the grader, exclude the run), **infra** (re-run). Only MODEL failures are valid difficulty signal.

**How to actually harden** (the part everyone gets wrong at first):

Stacking more rules and traps does not work. GLM writes a Python script per rule and handles ten independent rules as ten easy checks. Difficulty from independent obligations doesn't compound.

What does work is **coupled reasoning** — make correctness depend on interactions:

- **Chained derivations** — a threshold that must itself be derived from the data, then used as a filter.
- **Cross-source synthesis** — rule in a policy doc, data in a CSV, the exception in a third file. No single source suffices.
- **State across steps** — dedupe before aggregating; an exclusion discovered late invalidates earlier work.
- **Data-shape sabotage** — near-duplicates and edge rows in the input files that a naive per-rule script misclassifies.

Levers, in order of leverage: `instruction.md` (ask for more integration, less recipe) → input data (add edge density) → `tests/manifest.json` (tighten loose verifiers, fairly).

Hardening must stay honest: never through ambiguity, hidden information, a flaky environment, or a tolerance so narrow it rejects defensible correct answers.

Then: sync the `_app/` mirror → re-run Oracle (1.0) → re-run 4 GLM → check the band. Expect several rounds.

You do not always have to re-run the rollouts. A change to the *verifier only* can be re-graded against the existing runs. A major change to `instruction.md` or to what the verifiers ask for invalidates them — re-run the four.

> [!warning] The mirror rule
>
> (silently invalidates everything)
>
> On connector tasks,
>
> `instruction.md`
>
> and
>
> `tests/manifest.json`
>
> also exist under
>
> `environment/_app/`
>
> — and the
>
> `_app/`
>
> copy is the one that actually runs in the container. Edit only the top-level copy and your change never happened; you'll re-run and get identical scores and lose an hour wondering why.
>
> Sync it by running the package's
>
> `sync_app_mirror.sh`
>
> . Never hand-edit the mirror. If the script is missing from a package, flag it — don't build the mirror manually.
>
> (Non-connector tasks have no
>
> `_app/`
>
> mirror and iterate on
>
> `tests/verifier.json`
>
> instead. They must be rewritten to
>
> `tests/manifest.json`
>
> as the last packaging step, with the Oracle re-run afterward.)

## 8. Package and submit

The bundle is one task folder, zipped alone, with the evidence inside:

```
task/                          # ← zip THIS folder only
├── task.toml
├── instruction.md
├── review.csv                 # your QC review — one row per review area
├── qc_report.html             # the Delivery Gate report — client requires it
├── README.md                  # your change summary
├── tests/                     # manifest.json, test_*.py, etc.
├── environment/               # Dockerfile, entrypoint, _app/ mirror, …
├── solution/
│   ├── golden_trajectory.json # required Oracle asset
│   ├── solve.sh / solve.py    # if present
│   └── …
└── evaluations/               # named folders only. Nothing loose in here.
    ├── solvability/           # can a model solve it? ONE reward-1.0 run
    │   └── r1/                # any model — GLM, Claude, GPT… — never the oracle
    │       ├── agent/trajectory.json
    │       ├── result.json
    │       └── verifier/reward.json
    ├── difficulty/            # how often? FOUR independent GLM rollouts
    │   ├── r1/
    │   │   ├── agent/trajectory.json
    │   │   ├── result.json
    │   │   ├── verifier/reward.json
    │   │   ├── verifier/verifier_summary.json
    │   │   └── config.json    # trial-level OK
    │   └── r2/ r3/ r4/
    ├── stability/             # Turing runs this — ship it only if you have it
    │   ├── repeat-01/result.json
    │   └── repeat-02/result.json
    └── platform/              # NOT YET — see the note below
        ├── e2b/oracle/        # required to be 1.0
        ├── e2b/run/           # + sanity_check.md
        ├── modal/oracle/      # required to be 1.0
        └── modal/run/         # + sanity_check.md
```

> [!warning] `evaluations/platform/` is agreed but not yet accepted
>
> It is part of the folder spec settled with the client on 25 Aug 2026, and it will be required. The Delivery Gate does not take it today:
>
> `R17`
>
> allows only
>
> `solvability/`
>
> ,
>
> `difficulty/`
>
> and
>
> `stability/`
>
> under
>
> `evaluations/`
>
> , and a directory the gate cannot name is scored as a
>
> *failed model run*
>
> — which can sink an otherwise clean bundle.
>
> **Do not add `platform/` until we tell you the gate has been updated.**
>
> It is drawn above so you know what is coming.

Copy the four trial folders into `difficulty/` without flattening them, then put one run that scored 1.0 into `solvability/` as well — usually a copy of a passing `difficulty/` rollout, though any model will do (see below). Never include job-level `config.json`, `lock.json`, `job.log`, or the job-root `result.json`, and never put anything directly under `evaluations/` — a folder the gate cannot name is scored as a failed model run.

Each rollout's `result.json` must carry `"model": "GLM-5.2"` exactly, a boolean `overall_pass`, the `final_answer`, the reward, and judge provenance.

**Never put an oracle run in `solvability/`.** An oracle replays your gold answer, so it proves the verifier can grade the gold — not that a model can solve the task. A reward-1.0 oracle sitting in `solvability/` is the single most common way a finished bundle gets handed back.

The bundle ships **no oracle run evidence at all** — there is no `evaluations/oracle/` folder. The golden is graded from `solution/golden_trajectory.json`, which is still required, and the client replays the oracle in their own infrastructure. If you have an `evaluations/oracle/` folder from an earlier bundle, delete it.

**`solvability/` is not GLM-only.** The bar is one reward-1.0 run from *any* non-oracle model — Claude, GPT and Gemini all count, and so does any harness. That is a different question from `difficulty/`, which is specifically about how often **GLM-5.2** solves the task and still needs its four GLM runs. If your GLM rollouts never reach 1.0 but another model does, that other model's run is the solvability evidence.

**Stability is ours now.** Since 25 Aug 2026 Turing re-runs the verifier against the frozen rollout — you do not have to produce that evidence, and a bundle with no `stability/` folder is not a hand-back reason. Its `review.csv` row stopped being required on 26 Aug 2026 too, along with *Cross-trial · Calibration*. The Delivery Gate has not caught up: it will still raise `R3` for missing or thin stability evidence. That finding is **expected** — mark it reviewed with a one-line note saying Turing runs stability, and move on.

If you *do* have real repeats, ship them. Just don't put the four rollouts under `stability/` to fill it: that folder is only for fresh-container re-grades of the same frozen answer, which is a different kind of evidence entirely. Two repeats agreeing is a coin landing the same way twice. If you don't have real stability rechecks, the bundle just has none. Don't fake it.

Then submit the ZIP to the hosted QC platform (one task per ZIP). Runtime is ~5–10 minutes, longer for big verifier sets, and it's silent until done — do the manual checklist review in the right pane while you wait.

You get a report card + `qc_report.html`. It is advisory. A human reviewer makes the Accept / Fix / Reject call, so don't just do whatever the report says. The category report must nevertheless reach **Ready for finalization** before the platform accepts a submission, and the written `review.csv` record ([Step 6](https://qc-api-713053229214.us-central1.run.app/resources#review-csv)) is required alongside it.

### Getting `qc_report.html` into the bundle

The client requires the Delivery Gate's report to ship *inside* the task folder, at the root. You cannot produce it before the run — the platform generates it — so this is the last thing you do:

1. Run the Delivery Gate on your finished bundle.
2. Download `qc_report.html` from that run.
3. Drop it at the task root, next to `review.csv`.
4. Re-zip and upload it as a **new version** of the same task (see [Step 8](https://qc-api-713053229214.us-central1.run.app/resources#new-version)).
5. Run the Delivery Gate once more on that version, and submit *that* one.

The report you ship therefore describes the version before it. That is expected and fine — what the client wants is the QC record travelling with the package, not a self-referential file.

> [!info] Know what the platform adds to the staged delivery ZIP
>
> Submitting stages your uploaded archive for the pipeline. The only artifact the platform may add is an attached
>
> `review.csv`
>
> ; it does not add the generated QC report. Two consequences worth reading twice:
>
> - A `review.csv` you **attached** from the task page is injected into the staged delivery archive at the task root when the uploaded bundle does not already carry one. A bundled copy always wins and is never overwritten.
> - Same for `qc_report.html`. If it is not in the ZIP, it is not in the delivery.
>
> If
>
> `qc_report.html`
>
> is only on the platform and not in the archive, that is not blockable: the report is the Drive record, and once the gate says Ready for finalization you submit the passing version directly — no re-zip, no new version. See [[QC Handoff Guide]] Stage E.

## 9. Write up the review

Two files carry your review out of your head and into the package: `README.md` says what you changed about the *task*, and `review.csv` says what you checked and what you found. Both sit at the task root. The QC platform will not let you submit without the second one.

### README.md — the change summary

Every task needs a `README.md` with a cumulative, humanized summary — what changed from the mined baseline to the final version and why, why the task is genuinely hard, and a justification for any QC flag you left unfixed. Keep it short. You can keep a messy scratchpad while iterating, but the final deliverable is the clean summary.

### review.csv — the QC review record

The client asks for the review itself, not just its outcome: **one row per review area**, each row carrying its current result and, if you fixed something, what you found, what you changed, how you rechecked it, and where the evidence lives. It is the difference between "I reviewed it" and a record someone else can audit.

> [!info] The client reviews against this same rubric
>
> The fourteen areas below are their template, not ours — they accept and reject tasks on it. A task with a missing or thin
>
> `review.csv`
>
> is failed
>
> *even when the task itself is good*
>
> . And it has to be written by you: this is the one artifact we will not generate for you, and a model cannot write it, because the whole point is that a human looked.

The file is a plain CSV named `review.csv`, at the task root. Its first line has to be exactly this — **five** columns, in this order:

```
review_check,status,review_notes,change_made,what_to_record
```

### Write it in the review form

Don't hand-write the file. Use the [review form](https://script.google.com/a/macros/turing.com/s/AKfycbzi9BTJ8iVwPCCEGGaiaII9bKUwVl62mxkRpwGDfRYIKphxiDBfO-oF4B3A8Bc9AgYU/exec) — it holds the fourteen areas, the exact header and the required fields, so the file it produces cannot be refused on its shape. It still asks for all fourteen; the platform only requires twelve, so if the form makes you fill *Layer 2 Stability* or *Cross-trial · Calibration*, a one-line "Turing runs this" is enough.

1. Open the form and sign in with your **Turing Google account**.
2. Paste the Drive link to the **extracted task folder** — the folder, not the zip. The form checks whether a review CSV is already there and tells you if it found none.
3. Fill *Status*, *Review notes*, *Changes made* and *What to record* for the twelve questions that are yours.
4. **Submit and upload to Google** writes `review.csv` straight into that Drive folder, or **Download review CSV** gives you the file to place yourself.
5. To come back and edit, paste the same folder link and **Fetch review CSV** — it reloads what you already wrote.

Then make sure the file is **inside the ZIP you upload**. See the delivery note in [Step 5](https://qc-api-713053229214.us-central1.run.app/resources#step-5): attaching it on the task page unblocks Submit, but only the archive reaches the client.

### The fourteen review areas

Write one row for each of the twelve that are yours. The names are matched loosely — case, spacing and the `·` separators do not matter — but the words do, so keep them as they are here. The two marked **— Turing runs this** are ours: leave them blank or leave them out.

```
Layer 1 · Package consistency
Layer 1 · Clarity and scope
Layer 1 · Realism and leakage
Layer 2 Difficulty
Layer 2 Solvability
Layer 2 Stability            — Turing runs this, may be left blank
Layer 3 Oracle Mode
Layer 4 · Environment and files
Layer 4 · Connectors, MCPs, and CLIs
Layer 4 · Deliverables and artifact quality
Layer 5 · Verifier coverage and fairness
Layer 5 · LLM judge consistency
Layer 5 · Reward hacking and exploitability
Cross-trial · Calibration   — Turing runs this, may be left blank
```

Extra rows are fine — if you checked something the fourteen do not cover, add it. They are held to the same bar as the rest: every row you own has to be resolved. Two rows for the same area is a defect, though, because the file then gives two answers to one question.

### The three statuses that count as resolved

| Status | What it claims |
| --- | --- |
| `PASS` | You checked this area and it was already correct. Nothing was changed. **`review_notes` and `what_to_record` both have to say something** — what you looked at, and what you ran or read to be sure. `change_made` stays empty, because nothing needed changing. |
| `FIXED_AND_VERIFIED` | You checked this area, found a real problem, changed the package, and **re-ran the check that exposed it**. All five columns are filled in, `change_made` included — the status is the claim that something was changed, so a blank there is the row contradicting itself, and Submit refuses. Making the change is not enough either: the original issue has to have stopped reproducing. |
| `N/A` | The check does not apply to this task — the connector row on a task with no connectors, say. **Say why in `review_notes`.** An `N/A` with nothing beside it is treated as unresolved and holds the Submit button. |

> Those three are the only statuses the platform accepts. Anything else in the
>
> `status`
>
> column —
>
> `FAIL`
>
> ,
>
> `NEEDS_FIX`
>
> ,
>
> `TODO`
>
> , or a blank — holds the Submit button, on purpose.
>
> **There is deliberately no `FAIL` status**
>
> : you only submit tasks you finished, so every row lands positive. A row you cannot honestly resolve is work that is not finished, not a status to record.

In practice `N/A` has **one** home: **Layer 4 · Connectors, MCPs, and CLIs**, on a task with no connectors. The client allows it on any check that genuinely does not apply, always with a reason — but if you find yourself writing `N/A` anywhere else, look again. `N/A` on an area that does apply is the version of that claim the client notices, and GLM is asked about it specifically.

### What goes in each column

| Column | What to write |
| --- | --- |
| `review_check` | The review area, named as listed above. |
| `status` | `PASS`, `FIXED_AND_VERIFIED`, or `N/A`. |
| `review_notes` | What you inspected and what you concluded — the running log of your review. For a fix, the original finding, kept. For an `N/A`, why it does not apply. "Checked difficulty" is not a note; "first battery passed 4/4 because the totals were pre-aggregated in the input" is. Put your pointers here too: bundle paths like `evaluations/difficulty/r3/verifier/reward.json`, and Harbor trial or artifact IDs, where they exist. |
| `change_made` | The exact correction, named by file. Blank only when nothing needed changing. |
| `what_to_record` | How you rechecked it — the *What to record* column of the Human Review Guide for that check. The oracle re-run, a fresh GLM battery, and what came back. |

> There is no longer a separate
>
> `evidence`
>
> column, and the three columns
>
> `verification_performed`
>
> ,
>
> `verification_result`
>
> and
>
> `evidence`
>
> are gone — the client folded them into
>
> `what_to_record`
>
> and
>
> `review_notes`
>
> on 24 Aug 2026. A file still using the old seven columns will be refused on its header. The review form always writes the current five.

### A worked pair of rows

```
review_check,status,review_notes,change_made,what_to_record
Layer 1 · Package consistency,PASS,"task.toml, instruction.md and tests/manifest.json name the same three deliverables (recliner_fit_audit.csv, recliner_fit_memo.md, results.json). No drift, nothing renamed.",,"Read all three side by side and compared the declared deliverable names, paths and types."
Layer 2 Difficulty,FIXED_AND_VERIFIED,"First GLM-5.2 battery passed 4/4. Cause: environment/input/invoices.csv shipped a pre-aggregated total_due column, so the model copied a number instead of computing it. Rewards after the fix in evaluations/difficulty/r1..r4/verifier/reward.json; harbor trial 8f2c1e.","Dropped the total_due column from environment/input/invoices.csv, re-ran the _app mirror sync, and rewrote verifier 4 to recompute the total from the line items.","Oracle re-run plus a fresh 4-run GLM-5.2 battery from clean containers. Oracle 1.0; GLM-5.2 passed 2 of 4 — rewards 1.0, 1.0, 0.83, 0.50."
"Layer 4 · Connectors, MCPs, and CLIs",N/A,"This is a native task — no connector manifest, no environment/mcp/ folder, so there is nothing here to review.",,
```

Note the quoting: any cell containing a comma has to be wrapped in double quotes, and a literal double quote inside a cell is doubled (`""`). Writing the file in a spreadsheet and exporting as CSV handles both for you; hand-editing is where "could not be read as CSV" usually comes from.

### What the platform does with it

Two separate readings, deliberately kept apart:

- **Deterministic, and the only thing that blocks.** The file is named `review.csv` and sits at the task root, the header is the template header, the twelve areas you own have exactly one row each, every row has exactly five fields, and every one of those rows — plus any extras you added — is `PASS`, `FIXED_AND_VERIFIED`, or an `N/A` that says why, **with the cells that status obliges actually filled in**. *Layer 2 Stability* and *Cross-trial · Calibration* are exempt: absent, blank or open, they never refuse a submission. Fail any of the rest and Submit refuses, naming which.
- **GLM's reading, which is advisory.** It reads the file against the task and says whether the notes describe *this* package or could have been written about any task. It is reported on the review.csv row and never vetoes anything. That is not a licence to write filler: a human reads these.

A CSV under `environment/` is never mistaken for this file — that is the task's own input data. Both gates still run on a package with no `review.csv`; only submission is held. And if you have already zipped and uploaded, you do not have to re-zip: attach the file to the uploaded version from the **review.csv** row on the task page. A `review.csv` inside the bundle always wins over an attached one, so fix the bundle copy if the package ships both.

## Step 7 — Submit, and what happens next

Submit is a button on the task page, and it refuses in a fixed order. If it is greyed out, it is one of these, and it will say which:

1. the upload is a Harbor bundle (a task-harness JSON cannot be submitted);
2. the Delivery Gate on *this version* came back `PASS`;
3. the score is at or above the floor;
4. `review.csv` is present and every row is resolved;
5. this version has not already been submitted at this score.

"Present and resolved" are two different refusals on purpose — "you did not write it" and "you wrote it and left findings open" are different problems.

### Where it goes

Once submitted, the task moves through the delivery pipeline and the panel follows it:

| State | What it means |
| --- | --- |
| **Queued in pipeline** | Copied into a finalization batch. Nobody has picked it up yet. |
| **Running** | In final QC. |
| **Accepted** | It shipped. |
| **Rejected · N findings** | The client sent it back. Open it — the findings are there in full. |
| **Pipeline error** | **Not a rejection.** The batch died before it judged your task, so there is nothing to fix and it can be run again. |

A rejection is not a badge tucked in a corner — it becomes the headline on the task, expands to the pipeline's full findings, and turns the task list tag and the version trail red. Once the client has sent a task back, your local Delivery Gate `PASS` is no longer the operative answer.

## Step 8 — Updating a task you already submitted

You never edit a submitted task in place. You upload a **new version**, and the old one keeps its runs, its reports and its verdicts exactly as they were — so the record of what was scored stays true.

Two ways in:

- **Upload a new version** from the task itself. This is the one to use.
- Drag the new ZIP in as a fresh upload. The platform will notice it looks like a task you already have — it matches on the name declared in `task.toml` — and offer to link it as the next version. It only ever *offers*; two genuinely different tasks can share a declared name, so you confirm.

**Nothing carries across but the layout override and your worksheet.** Runs, findings you dispositioned, report cards and submissions are deliberately left behind: a new bundle has been scored by nothing, and carrying that across is exactly the confusion versions exist to prevent. Run the Delivery Gate on the new version and submit that one.

Only one version of a task is ever delivered — the one that passed and was submitted.

> Fixing a rejection is the same loop: read the pipeline's findings, fix them in the bundle, refresh
>
> `qc_report.html`
>
> (
>
> [Step 5](https://qc-api-713053229214.us-central1.run.app/resources#step-5)
>
> ), update the affected rows in
>
> `review.csv`
>
> to
>
> `FIXED_AND_VERIFIED`
>
> with the recheck recorded, upload as a new version, re-run the Delivery Gate, and submit that version.

## Errors you will hit in week one

| Symptom | Cause & fix |
| --- | --- |
| Healthcheck failed (rc=7, in_start_period=True) repeating at startup | Normal. Retries 40×. Only a problem if it never passes — then it's memory pressure, lower `-n`. |
| all predefined address pools have been fully subnetted | Leftover Docker networks from finished trials. `docker network prune -f`, then re-run. |
| Same, but after you killed a job | Killed jobs leave exited containers whose networks survive the prune. `docker ps -a --filter status=exited` → `docker rm` the dead `gen-*__env-main-1` containers → then prune → relaunch. |
| AgentSetupTimeoutError after 360s | Concurrent containers slow the opencode install. Add `--agent-setup-timeout-multiplier 3` (already in the template config). |
| NonZeroAgentExitCodeError / text part chatcmpl-… not found / Z.responses is not a function | You used the literal provider name `openai` (e.g. `-m openai/glm-5.2`). opencode special-cases it and calls an OpenAI-only API. Use `-m wandb-glm/glm-5.2` with the custom provider config. |
| Every judged/rubric verifier scores 0 | `JUDGE_MODEL=openai/glm-5.2` wasn't set. The proxy doesn't serve GPT models. |
| Model output truncates mid-reasoning, deliverables missing | Reasoning token ceiling. `"options": {"max_tokens": 96000}` in the model config (already in the template). |
| Your edit had no effect on the scores | The `_app/` mirror wasn't synced. Run `sync_app_mirror.sh`. |
| A run scored 0.0 and you assumed it was hard | Check `exception.txt` / missing `trajectory.json` first. It crashed. |
| A run stops without producing the deliverables | GLM's ceiling is **32,000 tokens**. Token exhaustion on its own is **not** a valid model failure — don't count it as difficulty. But if one rollout solves the task inside 32k while others run out producing nothing, that *is* a real model failure and counts. |
| Reasoning cut off mid-thought | Prefer none. Acceptable as long as at least one run succeeded; cut-offs are not counted as failures either way. |
| You changed a verifier and dread re-running the battery | You may not have to. A verifier-only change can be re-graded against the existing rollouts. Only a major change to the instruction or to what the verifiers ask for invalidates them. |

Never run `docker image prune -a` — it deletes the shared 15 GB benchmark-base image and everyone pays for that.

When you get stuck and then solve it: keep your own running notes file of problem → cause → fix, and write the entry immediately rather than "later." Check it first when something breaks — you've probably hit it before.

## Free points — the one concept worth internalizing early

Every verifier is worth **1/N** of the score. So a verifier that always passes — like "the output file exists" — is a free point, and it raises the score floor for every run.

If 4 of 12 verifiers always pass, no run can score below 0.33, and a "0.48" is really about 0.22 of actual work. Your difficulty number is inflated and the benchmark is lying.

The test: if you can't describe a plausible attempt that would fail this verifier, it's a free point.

The flip side is equally bad — don't move a task into band by adding verifiers rather than adding difficulty. That's denominator gaming, and reviewers look for it.

Related trap: don't grade the model's self-report. Recompute from the artifact it produced, not from a number it wrote into its own summary JSON.

## Your first task, end to end

1. Verify all five package pieces; start `review.csv` — fill rows in as you go, not from memory at the end
2. Cold-read `instruction.md`, list your guesses, sketch your own solution
3. Delete all `"category": "secondary"` verifiers
4. Oracle run → exactly 1.0
5. `docker ps` → smoke-test 1 GLM run → then the 4-run battery
6. Read all four rewards. Expect 4/4. Don't panic.
7. Harden with coupled reasoning → sync mirror → Oracle 1.0 → re-run 4 → repeat until 1, 2 or 3 of 4 pass
8. Golden trajectory present in `solution/` (promote a reward-1.0 run if the original is broken)
9. Assemble bundle with `evaluations/difficulty/r1..r4/` and one reward-1.0 run from any non-oracle model in `evaluations/solvability/`; stability is ours, so ship it only if you have real repeats. Zip the task folder alone
10. Write the `README.md`, and write `review.csv` in the review form — every review area has a row, every row `PASS` or `FIXED_AND_VERIFIED` (or an explained `N/A`). Put the file *in the ZIP*
11. Upload, run the Delivery Gate, fix or justify the findings
12. Download `qc_report.html`, add it to the task root, re-zip, upload as a new version, re-run the gate
13. Submit that version to the pipeline

> [!warning] Corrected 2026-08-27 (the g710 lesson)
> Once the gate says **Ready for finalization**, skip step 12. There is no re-zip and no new version — the report is the Drive record, not a package requirement, and the gate re-validates the *same upload* in place. Click **Submit to pipeline** on the passing version. Detail: [[QC Handoff Guide]] Stage E.

## The five things that will save you the most time

1. Cold-read the prompt and write down your guesses before you run anything. Ambiguity found on day one costs minutes; found after three GLM batteries it costs a day.
2. `docker ps` before every launch.
3. Sync the `_app/` mirror after every edit, via the script.
4. Re-run the Oracle after every change. 1.0 or it isn't done.
5. Expect 4/4 on the first battery. Hardening is the job, not a setback — and stop once 1, 2 or 3 of 4 pass.

---

Related: [[A Guide about `review.csv`.md|review.csv Guide]], [[A Guide About Verifier Quality and Transformation.md]], [[Computer Bench Task Quality Playbook - Daniel Sogbey.md]], [[Non-Connector Guide.md]], [[Set Up Guide.md]].
