# QC Handoff Guide — computer-bench task, start to pipeline

Companion to [Set Up Guide.md](Set%20Up%20Guide.md), [Non-Connector Guide.md](Non-Connector%20Guide.md),
[Shannon QC Control - 25 Aug Re-cut.md](Shannon%20QC%20Control%20-%2025%20Aug%20Re-cut.md)
and [Verifier Quality Guide.docx.md](Verifier%20Quality%20Guide.docx.md).
Written 2026-08-27 after the full `gen-g710-dev-retainer-billing-audit` journey (task #1251855).
This is the **day-to-day working doc**: the exact steps, files, links and traps.
The Re-cut doc is the spec; this is the loop you actually walk.

---

## 0. The pipeline map

```
design/harden (harbor)  →  verifier guide audit  →  local verify  →  package zip
   →  QC review form (Google Form → review.csv → Drive)  →  QC Delivery Gate (platform)
   →  dispositions (1 only!)  →  Submit to pipeline  →  labelling tool (4 fields + links)
```

Three human-facing surfaces, three artifact stores:

| Surface | URL | What it is |
|---|---|---|
| Harbor | local (`harbor run` in `~/obi-eval`) | oracle + agent runs, job dirs in `~/obi-eval/jobs` |
| QC Control | https://qc-api-713053229214.us-central1.run.app/ (IAP — needs your Google auth; **no public/API links**) | upload zip → Delivery Gate → report download → Submit to pipeline. `…/resources#…` anchors are the run instructions |
| Review form | Google Apps Script (per-task; URL in the guides below) | the 14-section review → `review.csv` |
| Labelling tool | https://labeling-g.turing.com/conversations/<task-id>/view | final metadata submission: pass rate, links, notes |

Also used: Google Drive task folder (the record store: review.csv + qc_report.html + golden-trajectory zip), git **local-only** on the task repo (no remote — the task link for labelling is the Drive folder, not a git URL).

---

## 1. Stage A — design & hardening (the part that takes weeks or minutes)

### Rules of the game (team constraints, current)

- Task must be hard enough that an **untrained-but-intelligent** agent struggles: target **1/5–2/5** (measured on 5 runs) / the re-cut's **1–3/4 band** (4-run batteries). 0/5 impossible, 3+/5 too easy.
- `input/` is **editable** since 2026-08-26 (lead-confirmed) — a legitimate difficulty lever like instructions/verifiers.
- Instructions must read like a human colleague asking for the work. **Difficulty must come from the domain work**, never from phrasing that mirrors verifier checks. If you find yourself writing the instruction to satisfy checks, you're prompt-hacking — stop.
- Stop hardening the moment you're inside the band. No gratuitous rounds.

### The loop

1. Oracle run (gold through the real verifiers) — must be **1.0**. If oracle doesn't pass, fix gold/verifier first.
2. Agent runs (GLM-5.2 via `glmproxy`, DeepSeek) — smoke 1x, then **battery only with explicit approval** (4 runs for the band; single model at a time).
3. Failure analysis every run: classify **task-owned** (verifier representation artifact, instruction ambiguity, gold bug) vs **model-owned** (domain error, strategy death). Fix task-owned; record model-owned honestly.
4. Re-run until inside band → **stop**.

### The g710 lesson — what actually moves difficulty (2026-08-26)

- **Didn't work** (each bought zero difficulty): register exhaustivity (more rows), proration rules, single-rule checklists. Capable models solved every new rule clean first-shot; every 0.0 was a verifier representation artifact or infra.
- **Worked**: *coupled reasoning* — rules split across three files so no single source answers any client-month; state across steps (B6: a July credit is "documented" only if it equals June's *audited* shortfall ⇒ July's variance depends on June's audit); data-shape sabotage (near-dup client names, one blank rate meaning default, a no-change renewal among five varieties); chained derivations (active-flag → proration fraction (30/31-day denominators) → pro-rated commitment → overage → variance).
- **The difficulty lever that actually counted is not domain rules — it's the model's one-shot plan habit.** GLM-5.2 sometimes plans the *entire* audit inside one reasoning block; any such plan is cut off at the per-step token ceiling (`step_finish reason: length`, terminal — the model writes **nothing**, so every verifier fails). The instruction's "do the recomputation in a script or spreadsheet" line is advice a one-shot planner ignores. Measured: pass rate = P(model chooses the tool-interleaved plan) ≈ 1–2/4 at **any** row scale (even 14 rows died once).
- If a re-measure ever lands 4/4: the lever is making the script a firm procedural requirement, human-voiced, no verifier change.

### g710 measurement (the canonical record)

- 4-run GLM-5.2 battery: rewards 0.0 / 0.0 / 1.0 / 0.0 → **1/4** (inside band). Single pass = 96/96.
- Every completed run scored 96/96 — **no rule error in any completed run**. The three failures are all one-shot token-ceiling deaths.
- Solvability: independent reward-1.0 GLM-5.2 rollout. Oracle: 1.0 (96/96, 17s), verified repeatedly.

---

## 2. Stage B — verifier suite & Quality Guide audit

### Verifier quality guide = the audit baseline

The guide (in this folder) defines 7 failure modes (e.g. one-literal anti-pattern, wording-roulette regexes over free text, self-report grading, golden exposure) and the **three-question acceptance test** for every check:
- **A — what requirement?** (which instruction sentence mandates this check)
- **B — why sufficient?** (does it get the full requirement)
- **C — why permissive?** (what legitimate variance it must tolerate — wording, casing, date formats, code variants)

Then run a **forward pass** (instruction items → implied checks, tick each in the manifest) and a **backward pass** (every manifest check → instruction sentence; if it doesn't map, it's an Extra — delete, unless the spec deliberately floors something like stub-memo length).

### Engine mechanics (tests/rl_world_verifiers) — read this before judging a check

- Checks: `check_path_exists`, `extract_text` (memo), `read_rows` (CSV, JSONPath `&` filters per row), `read_file` (results.json).
- Comparisons: `equals` (bool/type-aware: JSON number `"3865.49"` == `3865.49`), `contains` (**case-sensitive** substring), `regex_match` (Python `re`, inline flags `(?i)`, `(?mi)`).
- Expected values live at the **assertion level** (inside each check), NOT in a `.deterministic` dict — don't scan the wrong level when auditing.
- Reward = weighted pass fraction (g710: 96 checks, equal weights ≈ 1/96; Mode 5 partial credit).
- Additive rules: widen acceptance (regex for month variants `(?i)\b(?:june|2026-06|jun)\b`) but never make checks assert more than the instruction mandates (that's the wording-roulette trap — g710 deleted two "explain the trap" memo regexes for it).
- `values_equal` string-vs-number: an agent writing `3865.49` as text must not fail a gold `3865.49` — that was a real 0.0 run.

### The g710 F1–F3 (post-battery fairness pass, no difficulty change)

- **F1**: multi-code finding rows were join-order-sensitive (`OVERAGE_NOT_BILLED|INVOICE_LATE`); instruction didn't pin order → added "in the order listed" to instruction.md.
- **F2**: month-name `contains` on "June"/"July" is a one-literal anti-pattern (fails "june"/"Jun"/"2026-06") → `regex_match` case-insensitive equivalence.
- **F3**: README check-count was wrong (97 vs 96) → fixed.

Only 3 of the checks "always pass" (existence); no meaningful free-point floor.

---

## 3. Local verification

**There is no system pytest** — run the suite inside the task image:

```bash
docker run --rm -v "$PWD/solution/files:/app" -v "$PWD/tests:/tests" \
  -w /tests hb__gen-g710-dev-retainer-billing-audit:local \
  python3 -m pytest test_outputs.py -q   # → 96 passed in ~4s
```

Other standard moves:

- **Verify baked inputs** after every input rebuild: `docker run --rm <img> ls /app/input`.
- **Prebuilt-image mode**: `task.toml` sets `[environment] docker_image = "hb__<task>:local"` + `force_build` default False → harbor skips buildkit's Docker-Hub metadata fetch entirely (no registry access, oracle ~15–17s). Without this, harbor flakes on registry fetches.
- Rebuild the local image and re-run the oracle **after every input/manifest change**; the measurement is only valid on the exact packaged state.
- Negative test: mutate one gold value, expect exactly the corresponding check to fail (proves checks bite).

**Harbor runtime facts (proved empirically — these matter for the reward-hacking dispute):**

- The task image bakes `input/` into `/app/input` via Dockerfile COPY (no /app bind mount).
- Container creation mounts = exactly the 3 `/logs` binds (verifier/agent/artifacts). **`tests/` does NOT exist during the agent phase** — it's `docker cp`'d in only when grading starts.
- Oracle agent phases are sub-2s, so naive execs catch the *verifier* phase; to prove a phase-boundary property, run the check's own verify commands live at T1 (agent) and T2 (post-grading) — that's the evidence pattern.

---

## 4. Stage C — packaging the bundle

### Bundle anatomy (non-connector, deterministic verifiers)

```
gen-<task>/                    ← folder root = zip root; zip name gen-<task>-vN.zip
├── task.toml                  ← prebuilt docker_image; keywords incl. "non-connector","offline","shannon-200"
├── instruction.md             ← human-voiced brief + deliverables block
├── README.md                  ← what changed, measurement, failure signature, flagged items
├── input/                     ← standard/contracts/register data
├── solution/
│   ├── files/                 ← gold deliverables (csv, memo, results.json)
│   └── golden_trajectory.json ← ATIF-v1.7, promoted from a reward-1.0 agent run (use the battery pass)
├── tests/
│   ├── verifier.json          ← what test.sh grades (harbor entrypoint)
│   ├── manifest.json          ← byte-identical copy of verifier.json — the gate reads THIS. Must keep both.
│   └── rl_world_verifiers/    ← engine (sources: csv.py, file.py, text.py; runner.py)
├── evaluations/
│   ├── difficulty/r1..r4/     ← the 4-run battery trials
│   └── solvability/r1/        ← independent reward-1.0 rollout (no verifier_summary/config)
└── review.csv                 ← from the QC form (Turing account)
```

### Assembling `evaluations/` from harbor job dirs (pg/packaging/assemble_evaluations.py)

Trial dir = the subdir containing both `agent/` and `verifier/`. Mapping:

- `agent/trajectory.json` — copy.
- `config.json` — copy (difficulty only; solvability has none).
- **`result.json` must be REBUILT** — harbor's has none of the required fields:
  `model: "GLM-5.2"`, `overall_pass` (reward==1.0), `final_answer` (extract from ATIF `steps[].message` — the field is `message` (string or dict w/ content list), **NOT** `output`), `reward`, judge provenance (deterministic/rl_world/manifest).
- `verifier/reward.json` ← reward.txt, shape `{"reward": 1.0}`.
- `verifier/verifier_summary.json` ← parsed from `test-stdout.txt` (`PASSED|FAILED …test_deliverable[checkname]` lines) → `{items:[{name,passed}], summary:{total,passed,failed,pass_rate}}`.
  **ctrf.json collapses all 96 checks into ONE node — useless. Use test-stdout.txt.**
- Never copy job-level config.json / lock.json / job.log / root result.json.

### The zip

```bash
cd ~/Dev/computer-bench
zip -qr gen-<task>-vN.zip gen-<task> \
  -x '*/pg/*' '*/__pycache__/*' '*/.pytest_cache/*' '*/.git/*' '*/.env'
```

- `.pytest_cache/` is created by running the suite at task root — it's **not** in .gitignore-land, so exclude it explicitly (it slipped into v1 once).
- Versioning: v1, v2, … (v3 was the first gated upload). Zip name/naming convention vs the tracker is **still officially unconfirmed** — tracker is the source of truth.
- After building: `unzip -l` → check count, check no forbidden entries, check root files (task.toml, instruction.md, README.md, review.csv…).

---

## 5. Stage D — QC review form

- Single form URL per task (the Apps Script endpoint; kept in the vault).
- **14 sections** (Layer 1×3, Layer 2×4, Layer 3, Layer 4×3, Layer 5×3, Cross-trial).
- Fields per section — **all required**: `Status` (PASS / FIXED_AND_VERIFIED / N/A), `Review Notes`, `Changes Made`, `What to Record`.
- Conventions that worked:
  - `"None."` as change_made where nothing changed; `"None - <reason>"` for what_to_record on N/A rows.
  - N/A is for genuinely-not-applicable: Connectors (native non-connector task), LLM judge consistency (deterministic only), Stability + Cross-trial Calibration (**"Turing runs this."** one-liner note — these are Turing-side; *do not* write more).
  - FIXED_AND_VERIFIED for the Layer-2 Difficulty redesign.
- g710's final statuses: 9 PASS / 1 FIXED_AND_VERIFIED / 4 N/A.
- Keep a local draft as CSV first (`pg/packaging/review_rows_draft.csv`) for transcription.
- The complete filled document lives at `pg/packaging/review_form_document.md` — below is the whole thing, field by field.

### The 14 sections, field by field (g710's actual filled values)

**1 · Layer 1 Package consistency** — PASS.
Notes: task.toml, instruction.md, tests/manifest.json and the deliverables block name the same three outputs (retainer_billing_audit.csv, retainer_memo.md, results.json); columns/codes match RB-1 v2; solution/golden_trajectory.json (ATIF-v1.7, promoted from a reward-1.0 run) present; evaluations/difficulty/r1..r4 + solvability/r1 assembled per spec; no loose files under evaluations/, no job-level config/lock/log/result.json.
Changes: None. Record: Read task.toml, instruction.md, manifest.json, README.md side by side; checked deliverable names, manifest check names vs instruction items, and the full bundle tree.

**2 · Layer 1 Clarity and scope** — PASS.
Notes: Cold read: one paragraph of context, the asks (recompute per contract terms against RB-1; CSV + memo + results.json), and the variance/credit semantics defined up front. The standard supplies B0-B6 and the finding codes; no step-by-step recipe, no dumped column list.
Changes: None. Record: Cold-read instruction.md + standard; listed every guess; each guess answered by B0/B5/B6 or stated in the instruction; every manifest check traces to a sentence.

**3 · Layer 1 Realism and leakage** — PASS.
Notes: The brief reads as a real finance-committee audit request and the standard is a plausible internal policy. No leakage: no totals or counts stated anywhere, no hint-side columns, no spelled-out result; the finding-code list is the policy's own error taxonomy. The billing months must be read from the register's month headers, not from the prompt.
Changes: None. Record: Swept all inputs and the instruction for stated totals/hint names/methods; confirmed the gold figures appear nowhere outside solution/; verified the agent must derive the month set from the register blocks.

**4 · Layer 2 Difficulty** — **FIXED_AND_VERIFIED**.
Notes: The mined single-month register task was too easy: after its two task-owned round-1 defects were fixed it passed ~100% for GLM-5.2 and DeepSeek through rounds 2-4 (register exhaustivity and proration both solved clean first-shot; every 0.0 run was a verifier representation artifact or infra, never domain reasoning). Redesigned as a coupled two-month audit: terms in contracts.csv, rules in RB-1 v2, credits in the register — no single source suffices; B6 credits make July's variance depend on June's audited shortfall; near-dup client names; one blank overage rate; five renewal varieties plus one no-change guard; proration with 30/31-day denominators. 48 → 32 client-months after evidence the 48-row ledger made GLM-5.2's single-reasoning-block plan hit its token ceiling before writing; all rule interactions preserved (16 clients × 2 months).
Changes: Rewrote environment/input/retainer_billing_standard.md as RB-1 v2 (B0-B6 + finding codes); added environment/input/retainer_contracts.csv (per-client term rows, renewals); re-issued the register as two month-blocked sections with a credit_note_usd column, narrowed to the 16 active clients; rewrote instruction.md (variance definition, results.json structure, recompute-in-a-script guidance); rebuilt tests/verifier.json and wrote tests/manifest.json (96 checks); regenerated solution/files gold.
Record: Oracle re-run on the packaged state: 1.0, 96/96 (17s). Fresh 4-run GLM-5.2 battery: rewards 0.0, 0.0, 1.0, 0.0 — 1/4, inside the accepted 1-3/4 band (evaluations/difficulty/r1..r4/verifier/reward.json). Solvability: independent reward-1.0 rollout (evaluations/solvability/r1).

**5 · Layer 2 Solvability** — PASS.
Notes: The task is solvable: two independent GLM-5.2 rollouts solved it 96/96; the oracle solves it deterministically; 1x (reward 1.0, non-oracle) is promoted as solvability/r1.
Changes: None. Record: Ran the audit via the generator and matched the gold against the 96 checks; verified both reward-1.0 rollouts; solvability/r1/result.json carries model GLM-5.2, overall_pass true, reward 1.0.

**6 · Layer 2 Stability** — **N/A**.
Notes: Turing runs this. Changes: None. Record: None — Turing re-runs the verifier against the frozen rollout.

**7 · Layer 3 Oracle Mode** — PASS.
Notes: Own oracle at 1.0 verified twice on the packaged task. Cross-mode replay is Turing's to run (single-mode deliverable task family).
Changes: None. Record: harbor oracle runs on the final design: 1.0, 96 passed; re-run after the manifest rewrite: 1.0, 96/96, 17s; re-run after the guide fairness pass: 1.0, 96/96, 17s. No reward below 1.0 observed on repeated runs.

**8 · Layer 4 Environment and files** — PASS.
Notes: python:3.12-slim image; inputs baked at /app/input via COPY; the verifier stack and tests are mounted fresh per run (always the packaged manifest/verifier, never stale); pinned local image verified with baked inputs after every input rebuild; agent rollouts complete in 6-8 min; deliverables land in /app.
Changes: None. Record: Rebuilt the image with the final inputs and inspected it; ran oracle + 5 GLM rollouts plus one excluded config crash; no setup timeouts (setup-timeout multiplier 3).

**9 · Layer 4 Connectors, MCPs, and CLIs** — **N/A**.
Notes: Native non-connector task: no connector manifest, no environment/mcp/ folder, no external API/DB — all data is in environment/input/ and grading is an in-container deterministic verifier suite.
Changes: None. Record: None — the check is N/A for native non-connector tasks.

**10 · Layer 4 Deliverables and artifact quality** — PASS.
Notes: retainer_billing_audit.csv: exactly the 8 declared columns, one row per client-month in the register (32 = 16 × 2), findings in the declared code order, paused clients carry only BILLED_WHILE_PAUSED. retainer_memo.md names each finding by client and month and is substantive (>900 chars, uses the computed figures). results.json: months 2026-06/2026-07, six declared keys each, totals recompute from the CSV.
Changes: None. Record: Checked the gold by hand: documented credits (CL-02 200.00, CL-04 650.00) relieve their July shortfalls, unsubstantiated credit (CL-12 July 300.00) does not; zero-usage CL-06 correct-not-flagged; partial-month maths CL-13 350.00 / CL-15 74.19 / CL-22 235.48 / CL-24 43.55; then against all 96 checks via the oracle (1.0).

**11 · Layer 5 Verifier coverage and fairness** — PASS.
Notes: Forward and backward mapping done: every one of the 96 checks maps to exactly one instruction item. No category-secondary checks exist to delete. The only regexes are value-based: an anchored header-column match, a 900+ character floor, and two case-insensitive month-equivalence matches (June/2026-06/Jun, July/2026-07/Jul) — no phrasing sensitivity beyond accepting variant month naming. Nothing grades the model's self-report: every figure is recomputed from the artifact by the grader. Only the 3 existence checks always pass (3/96 — no meaningful free-point floor).
Changes: Audited against the 26-Aug Verifier Quality Guide: widened memo_mentions_june/july to the case-insensitive equivalence regexes above and pinned the finding-code join order in instruction.md; oracle re-run after the change 1.0 (96/96, 17s).
Record: Forward pass: listed the verifiers instruction.md implies, ticked each against manifest.json. Backward pass for every manifest check. Local replay 96/96; negative test: mutating CL-13 July variance and CL-05 July finding failed exactly those two checks.

**12 · Layer 5 LLM judge consistency** — **N/A**.
Notes: Deterministic verifier suite only (check_path_exists, extract_text, read_rows, read_file; equals/regex_match/contains with JSONPath) — no LLM-judged rubric exists, so there is no judge to be inconsistent.
Changes: None. Record: None — there is no LLM-judged rubric in the suite.

**13 · Layer 5 Reward hacking and exploitability** — PASS.
Notes: The agent container sees only /app (writable) and /app/input — no solution/ gold, no verifier state, no golden_trajectory. Expected values are recomputed from the artifact the model writes (read_rows on its CSV, read_file on its results.json, extract_text on its memo), so a hardcoded value would have to satisfy 64 per-row plus 12 derived-month checks coherently. No trajectory/self-report grading; no placeholder or transcript surface.
Changes: None. Record: Ran the reward-hacking audit from the guide: gold unreadable from the container, no writes to verifier state, no spoofable strings (identifiers only for contains), all checks recompute; probed a minimal solution (3 existence checks only = 3/96, no floor rescue).

**14 · Cross-trial Calibration** — **N/A**.
Notes: Turing runs this. Changes: None. Record: None — Turing runs cross-trial calibration.
- **The Apps Script pitfall**: the form's "Submit and upload to Google" errors with `You do not have permission to call UrlFetchApp.fetch… script.external_request`. Workaround: write the form anyway, then **download review.csv manually** and **add it to the Drive task folder manually**.
- The exported review.csv carries a UTF-8 BOM — it's normal, don't "fix" it.

---

## 6. Stage E — the Delivery Gate (the traps lived here)

### The loop (exact steps)

1. Upload the zip to QC Control → a delivery run starts. The run folder is downloaded to `~/Dev/computer-bench/gen-<task>-v3-<uploadid>-v1-delivery-<runid>/` — the `uploadid` (e.g. `c40a4a`) is stable; the `-v<idx>` is the **delivery index** (v1 → v2 on gate re-runs of the same upload); `-delivery-<runid>` is the run.
2. The folder contains: `qc_report.html` (the report), `report.json` (full 2.2 MB raw), `human_review.json` (your dispositions — the proof of what you submitted), `run_owner.json`.
3. Read the report: header = task + run id + engine sha (`vendored-…`); verdict banner either "Review required — findings await the reviewer's disposition" or **"Ready for finalization"**; 14 category results; a Findings section.
4. **"Ready for finalization" = the pass.** Only the two Turing-owned findings may remain (l2.stability, l3.oracle_mode) — the spec says Turing runs stability/replays, so they don't block and the platform **ignores your disposition on them** (the QC page accepts dispositions for exactly one finding — the trainer-owned one, e.g. l5.reward_hacking: confirm vs dispute + note).

### The reward-hacking finding — what it claimed, what we did

Finding (l5.reward_hacking): "tests/verifier.json contains all 96 expected answers as literal values, and test_outputs.py is designed for local execution in the agent's container, so an agent can read the spec and copy the expected findings…"

**Empirical refutation** (run the check's own verify steps live):
- `docker inspect <container>` at creation → mounts = exactly the 3 `/logs` binds; **no `/tests`**.
- Agent phase: `ls /tests` → "no such file"; `find / -name verifier.json` → nothing.
- After grading starts: `/tests/verifier.json` exists (grade-time injection only).
- The passing rollout's own log: `ls /app` shows only `input/` in the agent phase.
- `test_outputs.py` is the verifier-phase grader, not an agent-side script; literal expected values are the **client-mandated manifest shape** (QC Control's worked example), not a leak.
- Plus: negative test (mutate 2 values → exactly those 2 checks fail); oracle 1.0.

**The dispute note** (what goes in `human_review.json.dispositions[<finding-id>].note` — that IS the delivered record):

```
**Dispute — premise verified false.** Ran the check's own verify step in the agent container: agent phase `ls /tests` → no such file; `find / -name "verifier.json"` → nothing. `/tests` appears only when grading starts — grade-time injection only; `docker inspect` shows no `/tests` mount at creation. The passing GLM-5.2 rollout's own log confirms `ls /app` shows only `input/`. `test_outputs.py` is the verifier-phase grader, not an agent-side loop; literal `expected` values are the client-mandated manifest shape. Oracle 1.0 (96/96); negative test fails exactly the two mutated checks. **Caveat:** if the agent platform ever mounts the bundle into the agent workspace, the finding would hold for the whole family — platform-side mounts should exclude grading inputs. **Resolution: disputed.**
```

After the dispute, re-running the gate on the same upload: the finding disappears, category renders **Pass**, stays 2 findings, verdict "Ready for finalization".

### THE PAINFUL LESSON — no re-upload after the pass

**Do NOT re-zip / re-upload after a pass.** The gate runs against the already-uploaded package *in place*; the report and review.csv do **not** need to be inside the package for the platform to accept it (the passing run graded a package without the report inside). Once the run says "Ready for finalization", the QC page shows **Submit to pipeline** — click that, and you're done at QC. (g710 went through v3 → v4 → v5 rebuilds chasing the report inside the bundle; the user correctly refused the v5 upload: wasted cycle. The report + review.csv are the *Drive record* and the repo's task root, not a gate requirement.)

---

## 7. Drive folder & manual artifacts

The Drive task folder holds the **record** (same content as the repo task root):

| File | Source | Notes |
|---|---|---|
| `review.csv` | manual download from the form | BOM'd, 5 columns, 14 rows |
| `qc_report.html` | download from the final delivery folder | the *Ready for finalization* one |
| `gen-<task>-golden-trajectory.zip` | build it: `zip golden_trajectory.json` (zip root = the JSON) | for the labelling link |

The form's auto-upload is broken (Apps Script permission); **add all of these manually.**

---

## 8. Stage F — labelling tool (the FINAL step)

URL: `https://labeling-g.turing.com/conversations/<task-id>/view` (g710: task **#1251855**).

| Field | Type | What we submitted | Gotcha |
|---|---|---|---|
| Pass Rate | **SINGLE_CHOICE** — only `1/5, 2/5, 3/5, 0/5` | `1/5` | **There is no 1/4.** Measured 1/4 (25%) → closest tier is 1/5. The trainer notes then carry the actual measurement ("4-run battery 1/4") so the two cohere |
| Modified Task Link | FREE_TEXT | Drive **folder** link (e.g. `https://drive.google.com/drive/folders/<id>?usp=drive_link`) | No git remote exists — the repo is local-only (git added just to track changes). The Drive folder IS the task link |
| Golden Trajectory Zip File | FREE_TEXT | Drive **file** link to the golden-trajectory zip | The tool only accepts a **link**, not a file — upload the zip to Drive first |
| QC Proof | FREE_TEXT | Drive **file** link to `qc_report.html` | The QC API is IAP-protected ("Invalid IAP credentials" on anonymous calls) — no public/API link exists; the report file on Drive is the proof. The field description names the QC tool URL |
| Trainer Notes | FREE_TEXT | concise summary | Rules learned: **no verdict line**, no "Turing will handle stability/calibration" (obvious — the work is being done for Turing), keep the narrative = design change → measurement → guide audit → review form → gate dispute outcome |

Trainer notes (g710 final, ~1.1k chars) — submitted verbatim:

```
Redesigned from a mined single-month register audit into a coupled two-month task: terms in retainer_contracts.csv, rules in RB-1 v2 (B0-B6, incl. B5 proration and B6 compensation credits coupling June's audited shortfall to July's variance), two-month register, 32 client-months. Verifier suite rebuilt to 96 checks; oracle 1.0 (96/96, 17s), verified three times. Measured difficulty: 4-run GLM-5.2 battery 1/4, inside the 1-3/4 band; every completed run scored 96/96; failures are the model's single-reasoning-block plan hitting the per-step token ceiling before writing anything (strategy, not rule errors); independent solvability run 1.0. Audited against the 2026-08-25 Verifier Quality Guide: month-name checks widened to case-insensitive June/2026-06/Jun + July/2026-07/Jul, finding-code join order pinned, README count corrected. Review: 9 PASS / 1 FIXED_AND_VERIFIED / 4 N/A. Delivery gate: reward_hacking finding disputed with container-level evidence (tests/ injected only at grade time; literal expectations are the client-mandated manifest shape), cleared on re-run.
```

The g710 form metadata (from the downloaded `task-1251855-form-data.json`, `conversation_data`):

- passRate → `1/5`
- modifiedTaskLink → `https://drive.google.com/drive/folders/1_aL6I7pynqKm_PuFHRvCJ3KC_yv5CdhS?usp=drive_link`
- goldenTrajectoryZipFile → `https://drive.google.com/file/d/1x14riZFR5Wx1KjESn7oSiG_PneejUtEc/view?usp=sharing`
- qcLink → `https://drive.google.com/file/d/1ORQxpaestudP_Su2c7OxJPHgLOqTto-B/view?usp=sharing`
- trainerNotes → the block above
- Other ids: `conversationId: 1251855`, `trainerQuestionVersionId: 1605`

**After submitting, download the task metadata JSON** (`task-<id>-form-data.json` at task root): `conversation_data` holds the **live** values; `csv_data` is a **stale export** — don't panic when they disagree.

**Batch/slot id**: from `task.toml` — keywords `shannon-200`, `source_config = "infra/task-harness/configs/shannon-200/task_gen_G710_…"` → batch **shannon-200**, generator index **G710** (that's where `gen-g710-…` comes from).

---

## 9. Discipline checklist (never break these)

- Never commit or print API keys; masked prints only (names/URLs/lengths).
- `.env` never enters a zip or a commit (and `pg/`, `__pycache__/`, `.pytest_cache/` never enter a zip).
- Never `docker image prune -a`.
- Never write the literal provider name of the GLM endpoint (use "glmproxy").
- Never hand-edit the `_app/` mirror; no oracle run inside `evaluations/solvability/`; no `evaluations/platform/` (R17 doesn't accept it).
- Batteries only with explicit approval; single-model runs otherwise.
- Inside the band → stop hardening.
- Instructions stay human-voiced, always.

---

## 10. Gotchas — one-liners (the pain list)

1. No system pytest → run the suite inside the task image.
2. `tests/manifest.json` must be a **byte-identical** copy of `tests/verifier.json`; keep both in the bundle.
3. Expected values sit at the assertion level — don't read the `.deterministic` dict.
4. ctrf.json collapses all checks — build `verifier_summary.json` from `test-stdout.txt`.
5. ATIF steps carry `message`, not `output`.
6. `final_answer` must be extracted when rebuilding `result.json`; harbor's result.json has none of the needed fields.
7. review.csv has a BOM; 5 columns; `None.` fills.
8. The Apps Script `UrlFetchApp` error → manual CSV download → manual Drive upload.
9. Only **one** disposition is accepted (the trainer-owned finding); stability/oracle-mode dispositions are ignored — don't fight it.
10. **No re-upload after "Ready for finalization"** — click Submit to pipeline.
11. Same-upload gate re-runs bump the delivery index (v1→v2) and keep the upload id.
12. `values_equal` is type-aware; JSON number vs numeric string must be equal.
13. `contains` is case-sensitive; use `regex_match` with `(?i)` for month/code variants.
14. QC API is IAP-only — no public proof links; Drive files are the proof.
15. The labelling Pass Rate has no 1/4 — pick the closest fifth and say the measurement in the notes.
16. No git remote — Drive folder link for anything the tool wants as a "task link".
17. Oracle runs to inspect mounts: the agent phase is sub-2s, so snapshot the phase boundary, not a moment-after.
18. `evaluations/platform/` absent; stability/calibration Turing-side from the start.
19. Zip from the parent dir so the zip root is the task folder (`cd ~/Dev/computer-bench && zip -qr …`), not a nested copy.
20. `.pytest_cache` must be in the zip exclusions, and it is created by test runs at the task root.

---

## 11. Links index

| Link | Use |
|---|---|
| https://qc-api-713053229214.us-central1.run.app/ | QC Control (IAP-auth). `…/resources#step-5`, `#new-version`, `#review-csv`, `#free-points`, `#division-of-labour` = run instructions |
| https://labeling-g.turing.com/conversations/<id>/view | labelling tool (g710: 1251855) |
| https://script.google.com/a/macros/turing.com/s/AKfycbzi9BTJ8iVwPCCEGGaiaII9bKUwVl62mxkRpwGDfRYIKphxiDBfO-oF4B3A8Bc9AgYU/exec | the Apps Script review form engine (per-task URL kept in the vault) |
| https://drive.google.com/drive/folders/1_aL6I7pynqKm_PuFHRvCJ3KC_yv5CdhS?usp=drive_link | g710 Drive record folder (task link) |
| g710 QC-proof file | `https://drive.google.com/file/d/1ORQxpaestudP_Su2c7OxJPHgLOqTto-B/view?usp=sharing` |
| g710 golden-trajectory file | `https://drive.google.com/file/d/1x14riZFR5Wx1KjESn7oSiG_PneejUtEc/view?usp=sharing` |
| g710 deliveries | `…-v3-c40a4a-v1-delivery-4fc024ad/` (review-required) → `…-v3-c40a4a-v2-delivery-302e24b3/` (ready) |
| Docs (specs) | `https://docs.google.com/document/d/1D35bBfrF6so0Y8-w5fqdB5ywxoIjfHhATpOMYwSL4-0/edit`, `…/1KVqG9gSf4oRIsY3rGoxcfWred-Z4Gj1_wCl9o7U5SWM`, `…/resources#the-job`, review.csv spec + QC Control resources |

---

## 12. g710 one-page after-action (for scale)

- Rounds 1–4 (single-month register): every fix (variance convention, overbilled, exhaustive 14–24-row registers, proration) bought ~0 difficulty — both models solved clean every time; all 0.0s were task-owned representation/infra bugs.
- Round 5: two-month coupled redesign (3 files, B0–B6 incl. B6 month-coupling, 16 clients × 2 months = 32 rows) + measurement of the one-shot plan habit → measured 1/4. Hardening stopped.
- Guide audit → F1–F3 (pure widening). Verifier suite 96 checks. Oracle 1.0 ×3 on packaged state.
- Package: manifest = byte-copy of verifier; evaluations/ assembled from `glm-r5d-slim-1x..5x` battery + promoted solvability run; golden_trajectory.json = 4x battery pass (ATIF-v1.7, 20 steps).
- Review form: 14 rows, 9/1/4, "Turing runs this." one-liners, "None." fills.
- Gate: 3 findings (stability, oracle-mode, reward-hacking). Dispute l5.reward_hacking empirically → re-run → 2 findings, category Pass, "Ready for finalization" → Submit to pipeline.
- Labelling (1251855): 1/5, Drive links ×3, concise notes. Batch: shannon-200 / G710.

---

*Last updated 2026-08-27. Corrections welcome — this guide exists so the next task doesn't need to relive the pain.*
