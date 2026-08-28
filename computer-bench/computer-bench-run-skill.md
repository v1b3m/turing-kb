---
name: computer-bench-run
description: >
  Run and review a local "computer bench" Harbor task — Oracle, then model
  evaluations (GLM-5.2 via the team proxy, or a custom OpenAI-compatible
  endpoint such as a self-hosted DeepSeek), then harden, QC, and package per
  the team playbook. Use when asked to run/evaluate/benchmark a task in the
  computer-bench workspace, check an Oracle run, get a pass rate, review a
  trial's verifier results, debug why a bench run failed, run task QC, or
  prepare an upload package. Triggers: harbor run, oracle, retainer billing
  audit, computer-bench, pass rate, bench task, GLM run, DeepSeek run,
  non-connector task, task QC, unified-qc, hardening, upload package.
  Covers non-connector tasks (no base image / gcloud / W&B / LLM judge)
  verified end-to-end on harbor 0.22.0.
---

# Running a Computer-Bench Task Locally

> **Sync rule: this file is mirrored at `<vault>/computer-bench/computer-bench-run-skill.md`.**
> After any edit to this skill, immediately `cp` it over the vault copy (and
> vice versa when the vault copy changes) — the vault copy is the portable
> source for installing the skill on other machines
> (Start Here step 1). Verified in sync 2026-08-28.

## Read this first

The authoritative docs live in the personal knowledge base (Obsidian vault,
`/home/v1b3m/Dev/Personal/turing-kb/`):

1. `<vault>/computer-bench/Start Here.md` — **the entry point**. A numbered
   10-step map from design through the tracker-sheet log, with one-line
   summaries and a link to the detail doc for each stage (set-up, design,
   verifier audit, hardening, packaging, QC review form, Delivery Gate,
   pipeline submit, labelling, tracker sheet). The working detail doc is
   `<vault>/computer-bench/QC Handoff Guide.md`. Read the map first — it is
   the current truth; where a guide below disagrees with it, Start Here wins.
2. `<vault>/computer-bench/Non-Connector Guide.md` — **preferred** for offline,
   file-deliverable tasks (no gcloud, no W&B, no judge, binary reward). Start
   with its **"The whole lifecycle at a glance"** table + **"Glossary"**; Steps
   4–6 cover hardening, task QC, and upload/packaging; its **"Known gaps"**
   section records the QC tool's format mismatch with non-connector tasks.
3. `<vault>/computer-bench/Computer Bench Task Quality Playbook - Daniel Sogbey.md`
   — the team's design→package methodology (15-step flow, model- vs task-owned
   failure classification, verifier fairness rules, prompt bank, packaging spec
   §13). Use its judgment, not just commands. (Its 1/5–2/5 band is superseded by
   the 2026-08-25 re-cut: 1–3/4 of four, 4/4 rejected — see Start Here step 2.)
4. `<vault>/computer-bench/Set Up Guide.md` — the general guide (connector tasks,
   gcloud + base image + W&B).

If the Non-Connector guide is missing, read the Set Up Guide instead and apply
the corrections below.

## Hard rules (never break)

- **Never print or echo API keys.** Source `.env` and pass env vars; show only
  variable *names* or masked values (first 10 chars + `…`).
- **Never commit `.env`**, and **never include it (or scratch dirs like `pg/`,
  `.DS_Store`, `__pycache__`) in an upload package** — the packaging checklist
  excludes them, and a leaked key is a team incident.
- **Hardening lever order: prompt → verifiers → input, last.** The first surface
  is `instruction.md` (prompt), then `tests/verifier.json` (or the verifier
  scripts), and only then `environment/input/` — which is an editable lever since
  2026-08-26 (lead-confirmed — a register CSV may be rewritten to add difficulty),
  but never the first resort. Never change `input/` silently: surface every input
  diff to the user.
- **Never run a battery (4 runs, per the 2026-08-25 re-cut) without the user's
  explicit approval.** The team shares a model-run cap; one run first, always.
- **Verify the `.env` endpoint matches the model before every run.** DeepSeek
  runs need `api.deepseek.com`, GLM runs need `34.41.10.8:4000` — a swapped-in
  pair crashes the agent with 400 "Invalid model name" mid-run (burns quota and
  burns minutes). Check with `sed -E 's#https?://([^/]+).*#\1#' <<< $OPENAI_BASE_URL`
  right after sourcing `.env`. (Bitten 2026-08-27: r5 "DeepSeek" run dialed GLM
  after the cross-check swap was never reversed.)
- **The run loop: DeepSeek first, then patch, then one GLM run.** A single
  DeepSeek run on the task; patch any task-owned failures (verifier/instruction
  fixes) and re-check on DeepSeek if needed; then **one GLM run** to cross-check
  the patched task on the shared team quota. GLM never comes before the DeepSeek
  run; batteries still require explicit approval.
- **QC is downstream of acceptance, never adjacent — and never recommended
  unless the task is IN the deliverable band.** The sequence while a task is in
  hardening: runs → classify → **acceptance verification** (requirements check:
  oracle 1.0, ≥1 content run passing, §8 audit verdict with no unresolved
  Missing/Partial/Extra/Brittle checks, findings resolved, human-voiced
  instructions) → battery on approval → **measured 1–3/4 of four** → QC →
  package. "In the deliverable band" means a *measured* 1–3/4 — not an
  inference, not a trend. A task whose content runs all pass (observed 4/4) is
  **above the ceiling**: report it as "in hardening — difficulty pressure
  needed", never as "ready for QC" or "band expected easy end". Do not float
  QC (or packaging, or labelling) in any message while the band is
  unmeasured, unestablished, or measured outside 1–3/4.
- **QC is done on a task worth delivering.** Hardening is iterative — keep
  running pressure passes (levers per g688/g710: boundary cases, coupled
  findings, noise scale, decoy variants; lever order prompt → verifiers →
  input, input diffs surfaced) until the band is measured inside 1–3/4 OR the
  passes are exhausted. **Enough passes** = a documented series of pressure
  passes (record what was tried in `pg/run-log.md`: lever per pass, resulting
  run signal, what it moved). Only when the passes are exhausted and the task
  still measures outside 1–3/4 is it **not worth delivering** — and
  **abandoning is not delivering**: an exhausted hardening record is
  abandonment's precondition (trainer-notes-abandonment.md with what was
  tried), the task is abandoned (not delivered, not packaged, not QC'd), and
  the abandonment note is a record, not a deliverable. QC never runs on it.
  QC is a verdict on a task worth delivering — never a step in the no-band
  path; the no-band path ends in non-delivery, and that end is stated plainly,
  not dressed up as partial delivery.
- **Hardening is mandatory; packaging and abandonment are outcomes, not options.**
  Every task goes through dedicated hardening before either can be considered.
  The gate sequence is: requirements check (oracle 1.0, inside the 1–3/4 band,
  findings resolved, human-voiced instructions) → hardening iteration on
  prompt/verifiers/input as needed → battery on approval → QC → package. A task
  that has not been hardened — no difficulty-band evidence, e.g. 0/2 or 2/2
  content runs — is **in hardening**, never "packageable, if you want", and never
  an abandonment candidate. Never package a task that does not satisfy the
  requirements: the upload/QC gate outright rejects it.
- **Abandonment is last-resort only — and abandoning is not delivering.** It is
  considered solely after dedicated hardening time has been given and the task
  still cannot satisfy the requirements (record what was tried before
  abandoning — the attempt, not the outcome, is the precondition). If it cannot
  be hardened, the task is abandoned: that is a non-delivery — no package, no
  QC, and the Trainer Notes reason is an abandonment record, not a deliverable.
  State the reason clearly in the labelling tool's Trainer Notes (Stage F) and
  mirror it as `trainer-notes-abandonment.md` in the task root (same structure,
  one per task; **TL;DR ≤ 500 chars**). The note's YAML frontmatter
  `task_link:` is generated, not copied — the labelling-tool URL is
  predictable:
  `https://labeling-g.turing.com/conversations/<task-id>/view`, where
  `<task-id>` is the numeric directory in the task path
  (`…/computer-bench/<task-id>/gen-…/`), e.g. 1256596 → `…/conversations/1256596/view`.
  A partial, out-of-band, or unverified task is not a package.
- **Instructions must stay human-voiced** — no phrasing engineered to match
  verifier regexes (prompt hacking; unacceptable). Difficulty must come from
  visible domain reasoning (playbook §1). (Distinct from the voice rule below:
  "human-voiced" here means naturally worded, not verifier-engineered.)
- **Form answers are written in the user's voice — `instruction.md` is the one
  exception.** Every answer text that lands in an online tool — QC review form
  answers (14 sections), `review_form_document.md`, `review.csv`, QC Control
  disposition notes (`human_review.json`), labelling-tool Trainer Notes
  (Stage F), tracker-sheet entries — is drafted in the user's voice: run it
  through the humanize skill's `--me` mode before it ships (profile bundled at
  `~/.claude/skills/humanize/voice.md`). Same for other generated prose
  (README.md, aid files) — user's voice, plain and direct. **`instruction.md`
  is exempt**: it stays professional, domain-specific, and precise. It is the
  prompt a model reads and a client reviews; its power is the domain reasoning
  (§1), not personality. Never "humanize" structured artifacts (verifier JSON,
  task.toml, schema files) — those stay exact.
- **Never `docker image prune -a`.**
- **Never promise a QC verdict for non-connector tasks.** The QC tool's run
  gate currently refuses this family's evidence (see Known gaps in the guide);
  it fails closed. Report partial reviews as partial.
- **Hardening mechanisms: follow the Hardening Guide** (vault:
  `computer-bench/Hardening Guide.md`, 2026-08-28 re-cut). The legacy lever set
  (boundary cases, coupled findings, noise scale, decoy variants) saturates —
  family-wide, exactly one model-owned content error exists; every completing
  run scores full marks. The only measured difficulty so far was local 32k
  step-ceiling deaths, which the remote pipeline erased (g709 solved all 4 runs
  remotely). Design difficulty that makes a **completing** run fail
  (identification / integration / acceptance-condition errors) — never token-budget
  deaths — and measure with n≥5 before trusting any local number.
- **Every user-facing web action carries its URL inline.** If a step needs the
  user to act in a web tool (QC Control, labelling tool, review form, Drive,
  tracker), include the exact link in the same message — never "open QC
  Control" without the URL. Index below.

## Web tool URLs (always inline when a step needs them)

| Tool | URL |
|---|---|
| QC Control (IAP-auth; `#new-version`, `#review-csv`, `#division-of-labour`) | `https://qc-api-713053229214.us-central1.run.app/` |
| Labelling tool (per task id) | `https://labeling-g.turing.com/conversations/<id>/view` |
| Tracker sheet (gid=3003) | `https://docs.google.com/spreadsheets/d/1bbphEa7oIHfRhMPHEt_TmAbHcej1_ArImtsW43QaRyY/edit?gid=3003#gid=3003` |
| Review form engine (per-task exec URL) | `https://script.google.com/a/macros/turing.com/s/AKfycbzi9BTJ8iVwPCCEGGaiaII9bKUwVl62mxkRpwGDfRYIKphxiDBfO-oF4B3A8Bc9AgYU/exec` |

## Procedure

### 1. Identify the task and its type

- Task folder: under `/home/v1b3m/Dev/computer-bench/<task-name>/` (contains
  `task.toml`, `instruction.md`, `tests/`, `environment/`, optionally `solution/`).
- Classify in one pass:
  - `task.toml` keywords contain `non-connector`/`offline`, and
    `environment/Dockerfile` starts `FROM python:...` (public image) and comments
    say "these tasks never call a judge" →
    **non-connector**: no gcloud, no W&B, no `JUDGE_MODEL`, no base image pull.
  - Otherwise (Dockerfile `FROM ${BASE_IMAGE}`, GAR registry) → **Set Up Guide flow**
    (gcloud auth + W&B key + judge model).
- Note the deliverable filenames from `instruction.md` (e.g. `retainer_billing_audit.csv`,
  `retainer_memo.md`, `results.json`).

### 2. Preflight checks (parallel, all cheap)

```sh
docker info --format '{{.ServerVersion}}'          # daemon running
harbor --version                                    # ≥0.20; 0.22.0 verified
free -h | head -2                                   # RAM → concurrency = ⌊(GB-4)/4⌋
ls <task-dir>/.env && sed -E 's/=.*$/=<redacted>/' <task-dir>/.env   # OPENAI_BASE_URL + OPENAI_API_KEY exist
git -C <task-dir> status --porcelain                # surface unexpected input/ changes
```

If the `.env` is missing, ask the user for the endpoint + key (they may paste it;
store it into `.env` yourself with a mask-disciplined edit — or better, have the
user create the file).

### 3. Credential sanity check (one tiny model call)

```sh
cd <task-dir> && set -a && . ./.env && set +a
curl -s -m 30 "$OPENAI_BASE_URL/chat/completions" \
  -H "Authorization: Bearer $OPENAI_API_KEY" -H "Content-Type: application/json" \
  -d '{"model":"<model-id>","messages":[{"role":"user","content":"hi"}],"max_tokens":5}' | head -c 300
```

- Success = a real completion (checked id, model echo). Failures: 401 → key
  expired/incorrect; 404 → base URL wrong (GLM-style proxies need `/v1`;
  `https://api.deepseek.com` works bare); connection refused → endpoint
  unreachable. Fix in `.env` before any harbor run — model runs burn quota,
  curl does not.

### 4. Oracle (validates grading; needs no model; must be exactly 1)

```sh
harbor run -p <task-dir> -a oracle -o ~/obi-eval/jobs --job-name oracle-<task> -y
```

- First build takes minutes (small public image + pip installs); no 15 GB pull.
- Success: `<job>/<trial>/verifier/reward.txt` == `1`. Count tests in
  `verifier/test-stdout.txt` (`16 passed` here).
- If `0`: task defect (verifier rejects the known answer) — report, don't run a
  battery. Check whether someone modified `environment/input/` vs the git-committed
  original and surface it.

### 5. Single model run (never skip this step)

Exact command shapes. **DeepSeek (the user's own endpoint) is the default first
run — always DeepSeek before any GLM run on a new task**; after the DeepSeek run
and any patches from it, run **one GLM** run to cross-check the patched task.

**Custom endpoint (DeepSeek, verified)** — the `.env` holds the DeepSeek base URL
+ key; only `-m`, the `--ak` provider/model ids, and the job name change relative
to the GLM shape:

```sh
harbor run \
  -p /path/to/<task-name> \
  -a opencode \
  -m deepseek/deepseek-v4-flash-vision-exp \
  --ak 'opencode_config={"provider":{"deepseek":{"npm":"@ai-sdk/openai-compatible","name":"DeepSeek","options":{"baseURL":"{env:OPENAI_BASE_URL}","apiKey":"{env:OPENAI_API_KEY}"},"models":{"deepseek-v4-flash-vision-exp":{"name":"DeepSeek"}}}}}' \
  --agent-setup-timeout-multiplier 3 \
  --ae OPENAI_API_KEY="$OPENAI_API_KEY" --ae OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  --ve OPENAI_API_KEY="$OPENAI_API_KEY" --ve OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  -o ~/obi-eval/jobs --job-name "deepseek-1x-opencode-<task>" -y
```

**GLM proxy (team quota, shared)** — same shape; swap `.env` to the GLM base URL +
key; only `-m`, the `--ak` provider/model ids, and the job name change:

```sh
set -a && . ./.env && set +a

harbor run \
  -p /path/to/<task-name> \
  -a opencode \
  -m glmproxy/glm-5.2 \
  --ak 'opencode_config={"provider":{"glmproxy":{"npm":"@ai-sdk/openai-compatible","name":"GLM via LiteLLM","options":{"baseURL":"{env:OPENAI_BASE_URL}","apiKey":"{env:OPENAI_API_KEY}"},"models":{"glm-5.2":{"name":"GLM 5.2"}}}}}' \
  --agent-setup-timeout-multiplier 3 \
  --ae OPENAI_API_KEY="$OPENAI_API_KEY" --ae OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  --ve OPENAI_API_KEY="$OPENAI_API_KEY" --ve OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  -o ~/obi-eval/jobs --job-name "glm-1x-opencode-<task>" -y
```

```sh
harbor run \
  -p /path/to/<task-name> \
  -a opencode \
  -m deepseek/deepseek-v4-flash-vision-exp \
  --ak 'opencode_config={"provider":{"deepseek":{"npm":"@ai-sdk/openai-compatible","name":"DeepSeek","options":{"baseURL":"{env:OPENAI_BASE_URL}","apiKey":"{env:OPENAI_API_KEY}"},"models":{"deepseek-v4-flash-vision-exp":{"name":"DeepSeek"}}}}}' \
  --agent-setup-timeout-multiplier 3 \
  --ae OPENAI_API_KEY="$OPENAI_API_KEY" --ae OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  --ve OPENAI_API_KEY="$OPENAI_API_KEY" --ve OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  -o ~/obi-eval/jobs --job-name "deepseek-1x-opencode-<task>" -y
```

Fixed parameters that matter:

- `-p <task-dir>` accepts a local task directory on 0.22.0 (Dataset panel).
- `{env:OPENAI_BASE_URL}`/`{env:OPENAI_API_KEY}` placeholders resolve inside the
  container — **never hardcode secrets** in `--ak`.
- `--agent-setup-timeout-multiplier 3` (node/npm bootstrap into python:3.12-slim).
- `--ae`/`--ve` both key and URL. **Never `--ve JUDGE_MODEL`** for non-connector tasks.
- Run in the background; it can take 20–45 min (agent budget up to 1800 s).
- Per the playbook, when the task is being **designed/hardened** this step is
  preceded by the design loop: cold-read `instruction.md` → inputs/policy →
  plain-English understanding → one representative case solved by hand →
  pressure-test questions → expected-verifier checklist → compare with the real
  verifier → fill gaps (§2, §8 of the playbook; full prompts in §15). For what
  *actually* moves difficulty, read the QC Handoff Guide §1's **"the g710 lesson"**
  (coupled reasoning — stacking rules buys nothing; the model's one-shot plan habit
  is the real lever) and **"the g688 pilot lesson"** (saturation = open-book
  determinism; the axes that discriminate are noise discipline and definitional
  margins, with decoys as the error generator, not domain expertise).

### 6. Liveness checks while it runs

```sh
docker ps --format '{{.Names}}'        # <trial>__env-main-1 exists
docker exec <c> tail -c 800 /logs/agent/opencode.txt      # step_finish records + token counts = real traffic
docker exec <c> bash -c 'for d in /proc/[0-9]*; do case "$(tr "\0" " " < $d/cmdline)" in *opencode*) tr "\0" "\n" < $d/environ | grep -E "^OPENAI";; esac; done'
```

The last one proves the credentials reached the agent process (Harbor normalizes
them to `${OPENAI_API_KEY}` refs in `config.json` — expected, not a bug).

### 7. Review results

Path: `~/obi-eval/jobs/<job>/<trial>/`.

- `verifier/reward.txt` → `1` or `0`. **Binary**, even if 14/16 tests pass
  (this task family's `test.sh` is all-or-nothing; only a full `1.000` counts as
  a pass per the playbook).
- `verifier/test-stdout.txt` → PASSED/FAILED per named verifier.
- `verifier/ctrf.json` → machine-readable pytest results.
- `agent/trajectory.json` (ATIF v1.7; steps, final_metrics) + `agent/opencode.txt`
  → postmortem source. Container is deleted after the trial; if the agent's
  deliverables are needed, re-run with `--artifact /app/<file>` or extract from the
  log's tool-call outputs.
- Broken-run triage: `agent/exit-code.txt` present ⇒ crashed; trajectory
  missing ⇒ verifiers failed by default; both absent with real steps and some
  tests passing ⇒ genuine failure.
- Run-failure taxonomy (playbook §11.5, added 2026-08-27): **agent-broke** is a
  *verifiable incompletion* — transcript complete, agent worked (read/reasoned),
  wrote nothing (e.g. step-length death `step_finish reason: "length"`), counted
  neither pass nor fail, replace. **Network/kill is a different class**: died in
  setup/preflight → known no-work (replace, uncounted); died mid-agent →
  **unknown completion** — the agent may have completed; never classify as broken
  (that would read as model evidence it isn't), never count as pass; inspect the
  trajectory (deliverables present → recover + grade locally, flagged
  "trajectory-recovered"; absent → rerun, document as unknown). Only
  **model-owned** and **task-owned** outcomes are task evidence. Floor: ≥1 content
  run must pass before any conclusion — 0 passing runs = no difficulty evidence
  either way.
- For failing checks, read the exact expected value in `tests/verifier.json`
  (e.g. `"expected": 800.0`, `"comparison": "equals"`; regexes like
  `\bCL-06.*(not an underbilling|is correct|no rollover)`).

### 8. Report and classify

Per-run: reward + which verifiers passed/failed (names) + genuine-vs-crash verdict +
one-line cause summary for any failure. Then classify failures per playbook §11:
**model-owned** (missed a visible rule with files present — valid difficulty) vs
**task-owned** (ambiguous instruction / hidden requirement / verifier checks unasked
behavior — fix the task, never count it as difficulty). For brittle checks, playbook
§8's table prescribes "make less wording-sensitive". For a battery: pass rate = mean
of the four binary rewards; hook to the 1–3/4 band (4/4 is rejected as too easy).
List agent-broken/unknown-completion attempts **separately** from the rewards, as
exceptions (attempt, reason, recovered-or-not) — they count neither as pass nor
fail, and never as task evidence (playbook §11.5).

**Report the state, not the ambitions.** The report names whether the task
satisfies the requirements (oracle 1.0, **measured** 1–3/4 band, findings
resolved, human-voiced instructions). If the band is unestablished (no battery)
or measured outside 1–3/4 — including observed **4/4** (all content runs pass),
which is above the ceiling per the 2026-08-25 re-cut — the honest state is
**"in hardening — difficulty pressure needed"**, and the report says so instead
of floating QC or packaging as a next step. Never recommend QC unless the band
is measured inside 1–3/4; never "package if you want" while the requirement gate
is open; no abandonment phrasing either (hardening comes first — see hard rules).

Fairness rule (playbook §8): the verifier is only the marking scheme — assert
**exactly** what the prompt requires, no less, no more: Missing → add a
deterministic check; Partially covered → strengthen; **Extra (checks something
not asked) → remove or align**; Brittle (equivalent correct answers fail) →
make less wording-sensitive; Covered → keep. "The verifier should grade the real
task, not hidden preferences."

### 9. Task QC (`unified_qc.py`)

The team QC tool (standalone copy: `~/Dev/computer-bench/qc-script/`). Advisory
Keep/Fixable/Reject; human verifies cited evidence last; never edits tasks.

```sh
cd <qc-script-dir>
python3 unified_qc.py --input <task-dir> --dry-run                    # free: deterministic checks only
python3 unified_qc.py --input <task-dir> --trajectory ~/obi-eval/jobs  # live: adds model review
```

- Reads `.env` from the qc-script dir; defaults to the team GLM gateway. For a
  custom endpoint: `QC_MODEL=deepseek-v4-flash-vision-exp` + DeepSeek
  `OPENAI_BASE_URL`/`OPENAI_API_KEY` in that `.env` (verified working 2026-08-26),
  or `--model`/`--base-url`/`--api-key-env` flags.
- Output: `runs/<UTC-timestamp>/{results.json,results.csv,summary.md}`.
- **Known limitation:** the tool's run gate refuses non-connector evidence
  (it expects `tests/manifest.json` + named verifier units + per-check names in
  run results; local one-line discovery patch applied at `unified_qc.py:502`
  added `tests/verifier.json`). Expect `partial` / **no verdict** for this task
  family — report that, don't fake a Keep/Fixable/Reject. The upstream report
  (infra owner) is the real fix; see the guide's Known gaps.

### 10. Upload / package

Two flows — use the one matching the current tracker column (the tracker is the
source of truth; the upload location is not documented in the vault — confirm
with the lead before packaging):

- **Upload_reworked-QC (current per playbook §13):** one self-contained zip
  `<task-id>-upload-reworked-qc.zip` with the task package (task.toml,
  instruction.md, README.md, environment/, tests/, solution/ +
  `golden_trajectory.json`), `evaluations/difficulty/r1..r4` +
  `evaluations/solvability/r1` (each: agent/trajectory.json, result.json,
  verifier/), and **`review.csv` at the bundle root — always** (the QC form
  record ships in every upload; the accepted g710 bundle anatomy is the
  reference: no stability dirs — Turing-side, no REWORKED_QC_README.json).
  **Never include `.env` or scratch folders.**
- **Zip location: inside the task's batch folder** (e.g. `~/Dev/computer-bench/1256598/`
  for batch 1256598), alongside the task dir — never loose in
  `~/Dev/computer-bench/`. The task dir is the single bundle root: task files +
  `evaluations/` + `review.csv` + `README.md` + `.gitignore` merged into it
  (layout 2026-08-28). Build: `cd <batch-folder> && zip -qr <task>-vN.zip
  <task-dir> -x '*/pg/*' '*/__pycache__/*' '*/.pytest_cache/*' '*/.git/*' '*/.env'`.
- **One source of truth: the task directory.** The zip is a build artifact,
  never a second source — never hand-modify a zip (`zip -u` a file in) and never
  add a file to the zip only. Any change (adding `qc_report.html`, editing
  review.csv) goes into the canonical task dir first, then the zip is **rebuilt
  from it** and round-trip-checked (`unzip` to temp, `diff -r` vs the canonical
  dir minus the excluded paths).

### The Delivery Gate loop (review.csv in, report rides the next upload)

1. Zip **always contains `review.csv` at root** before any upload.
2. Upload the zip to QC Control (`#new-version`) → a delivery run starts; its
   folder (`…-v<idx>-delivery-<runid>/`) contains `qc_report.html`,
   `report.json`, `human_review.json`, `run_owner.json`.
3. Read the report: "Ready for finalization" = the pass — **stop uploading.**
   Anything else (findings awaiting disposition): dispose the trainer-owned
   finding (confirm/dispute + note into `human_review.json`), then **add the
   downloaded `qc_report.html` to the zip root** and re-upload (the delivery
   index `-v<idx>` increments) — repeat until the tool verifies readiness.
4. After the final run: upload **`review.csv` AND the final `qc_report.html` to
   the Drive task folder** (the record — repo task root keeps the same two
   files). No uploads after "Ready for finalization", ever (g710's lesson).
- **Harbor platform (older):** `harbor auth login` (GitHub OAuth, once per
  machine) then `harbor upload <job-dir>` — directory in, no zip; records the
  locally computed reward, private by default.

### QC finding classes & the fairness rule (what the gate checks)

**Who acts (findings ship in `report.json` / `qc_report.html`, keyed by owner):**

- **trainer-owned** (`l5.verifier_fairness`, `l5.reward_hacking`, …): must be disposed —
  confirm or dispute with a note in `human_review.json.dispositions[<finding-id>]`.
  Never judge from the summary: **run the finding's verify step with the vendored
  engine** before writing the note. Dispute = the g710 recipe (container-level
  evidence; see guide §6). Confirm = reproduced, fixed, re-verified; the note records
  all three.
- **finalization-owned** (`l2.stability`, `l3.oracle_mode`, `l2.difficulty`): no action,
  never block, and may remain in an accepted delivery. `l2.difficulty` "above ideal
  band" is advisory (it says "trainer may submit as-is") — the client band (1–3 of 4)
  is the spec; never reshape rewards to please it.

**The fairness rule — a deterministic check may assert only what the instruction
actually requires.** A check over free text or a format must accept every output the
instruction permits. Three flavors (all three shipped tasks got hit in one delivery,
g709 2026-08-28):

1. **Vocabulary pinning** — memo checks demanding fixed phrases or order ("one of four
   phrases after the TIX IDs") when the instruction pins no wording. Fix: concept-based,
   order-independent — two lookaheads (concept A present AND concept B present, any
   order); phrase set = the instruction's own terms + standard synonyms + what the gold
   text actually uses.
2. **Orthographic variant rejection** — `\broot\s+cause` rejects the hyphen compound
   "root-cause"; `subject\s+line` rejects the instruction's own "subject wording".
   Fix: accept all standard variants of instruction-family terms
   (`[\s-]+`, `(?:line|wording|text|title)`).
3. **Format-surface inconsistency** — sibling checks on one source disagree on
   acceptance (finding checks tolerate `\x22?` quoting, the `hours_open` checks do
   not): a fully quoted RFC-4180 CSV with exact gold values passed 40 checks and
   failed 6. Fix: **one acceptance surface per source type** — if any check tolerates
   quoting, all do; build row/field patterns once (shared regex builder), never
   hand-roll one check's tail.

**Round-2 lesson (g709 2026-08-28, same check, one level deeper):** a phrase-set
fix inside flavor 1 is whack-a-mole — the gate returned on `memo_explains_traps`
rejecting "not members of any tracked cohort" (member-before-cohort), "does not
match any tracked cohort" (intervening verb), "doesn't belong" (contraction), "no
cohort membership", even "false positives" (plural). The QC's remediation lists
its own forms only. Fixing it right: stop enumerating phrases, adopt a **semantic
construct** — a negation marker (`not`/`never`/`no`, contraction family) within
one sentence clause of a cohort/member word, either order, clause-bounded window
(`[^.!?\n;:]{0,40}` so "not" and "cohort" in unrelated clauses or different
sentences don't pair). Keep the old phrase family as alternates so the acceptance
surface only ever grows (acceptance-superset monotonicity is what keeps recorded
battery rewards valid).

**Round-3 lesson (g709 2026-08-28, the class, not the check):** the round-2
semantic construct still had a phrase-list core, so the gate moved to the
same check's remaining holes — "clean traps … outside the cohort": `clean`/`no
finding`/`false positive` is the instruction-own negation vocabulary (the task
says "naming every ticket whose subject line *looks like* a cohort match but is
*not*"), and "outside the cohort" is the relative-clause variant of the same
statement, matched by no marker in the negation window. The class was live in
every sibling memo check too: the instruction's own phrasings ("the SLA policy
sets a two-day limit", "the code-table change effective 2026-08-24",
"membership comes from the cause code, never the subject") all failed the
shipped verifier. Fix: **anchor vocabulary to instruction-own ∪ task-schema ∪
gold-synonym** (never one phrase list), relations as order-free clause-window
co-occurrence, and **fix the whole class in one commit** — widen every sibling
check on the same source the same way; re-verify against every prior QC
construction (r1 through rN) plus newly generated ones, and confirm acceptance
is a strict superset of the previous delivery.

**Round-4 lesson (g709 2026-08-28, flavor 2 one round deeper):** the round-1
orthographic fix (`root\s+cause` → `root[\s-]+cause`) closed only *that* literal
— every other space-only join in the same checks kept the defect: `false\s+positives?`
rejected the standard hyphenated "false-positives" (and `no\s+finding` → "no-finding",
`cause\s+code` → "cause-code", `subject\s+line` → "subject-line", `logged\s+cause`,
`first\s+…rule` → "first-applicable-rule", `business\s+hours`, `prior\s+code`,
`effective\s+…` — the last three masked behind sibling literals but still classes).
Fix: when correcting flavor 2, **grep every free-text check for ALL remaining
`\s+` joins inside multi-word literals and widen them in the same commit**
(`[\s-]+`), one judgment per join — widen when the hyphenated form is standard
English, keep space-only when it isn't ("the cause", "one finding" never
hyphenate). Never fix one literal of an orthographic family; a
space-only join is a variant-rejection bug anywhere it appears.

**Round-5 lesson (g684 2026-08-28, the "no less" direction + dead alternates):**
the gate also graded *under*-assertion on a hardened task — every claim the
instruction makes needs at least one check that can actually fail:

- Value fields must be asserted, not just vocabulary: a CSV with a recomputed
  price column was ungraded until per-order deterministic row checks were added
  (wrong `expected_unit_price` mutated → check failed cleanly).
- Decoy rows must be asserted clean: the preferred-name decoy was ungraded — a
  model catching it as `NAME_MISMATCH` (the commonest error) scored 1 until a
  `none` assertion was added. If a trap can be missed and still pass, the trap
  is not graded.
- Dropped rows: add presence assertions; they also catch row-count defects.
- **Dead OR-alternates are a real structural defect**: a figure alternate
  appended as `...|\b22\.(?:5|50)\b` to a check whose `\boverbill` alternate
  already matched every overbilling memo never failed — the mutant
  (memo: "overbilled $100.00") passed 25/25. After adding any alternate,
  mutate the claim and confirm the check can fail; assert each required claim
  in **its own check** instead of OR-ing it under one that already passes.
- Optional-article pins: `(?:a\s+)?` rejects the instruction's own
  "an already-approved proof" — use `an?\s+` or drop the article and window the
  relation (semantic construct, round-2 style).
- Build new deterministic checks with **one shared quote-tolerant field builder**
  (`\x22?<value>\x22?`, `\s*` separators, `\s*$` for CRLF tolerance) — g709
  flavor 3's one-acceptance-surface rule applied at authoring time, not only
  when widening. Validate in-process via the vendored engine
  (`SourceRegistry(Path(ws))` + `verify_definition`; deps in a local venv —
  `pydantic`, `jsonpath-ng`, `tenacity`, no pytest needed) across gold,
  natural paraphrases, wrong-value mutants, and a fully RFC-4180-quoted gold.

**Prevention habits (cheap, and they catch what the QC tool will):**

- Authoring: for every free-text/format check ask *"what would the instruction-permitted
  paraphrase look like?"* — then run 3–5 mutants of the gold text (paraphrases,
  hyphen/quote variants, reordered clauses) through the engine before shipping.
  The mutant matrix must include **order swaps, intervening verbs, contractions,
  plural/inflection variants** — the gate will probe all four. A mutant that fails
  and reads like natural instructed English is a check bug, not a lucky catch.
- Sweep: fixing one instance of a category does not fix the category — after any
  fairness fix, grep the manifest for siblings **and fix them in the same commit**
  (the `\bcredits\b` widening left two memo + six hours checks with the same
  defect class; the gate found them, and the r3 class was found this way).
- Post-widening re-test, both directions: valid-variant mutants now PASS **and**
  wrong-value mutants still FAIL exactly the intended checks. Validate even a
  QC-suggested fix against the finding's full construction (their quoting fix alone
  still failed the quoted row — the `,<code>` tail was quote-intolerant too).
- A QC-suggested remediation is a floor, not a spec: implement its forms *and* the
  semantic generalization that dominates them — otherwise the next delivery comes
  back with the sibling of the same finding.

### Form-answer voice (what's written into web forms)

Any answer text that lands in an online tool is authored **in the user's voice,
not in task-doc register** — humanize it with the built-in skill's `--me` mode
(profile: `~/.claude/skills/humanize/voice.md`) on the way out. What that means
concretely for this workflow:

- **Direct and plain.** "Verifier was reading for a fixed phrase the task never
  required" beats "The verifier asserted a narrowly-scoped lexical construct
  that the instruction did not stipulate."
- **Contractions always** (`don't`, `wasn't`, `they're`), **no em dashes**
  (the user's pet peeve — use periods or commas), **no preamble**
  ("The task:", "In summary:"), **sentence fragments fine**. No rule-of-three
  flourish, no "It's worth noting", no hedge openers.
- **Facts stay verbatim**: verifier names, check ids, reward numbers, file
  paths, URLs — never reworded or stylized. This is voice on top of the facts,
  not a rewrite of them.
- **Do not lower precision to sound casual.** The register transfers, the
  accuracy does not. A finding note still names the check and the exact
  construction; it just says so in plain speech.
- **The one and only exception: `instruction.md`.** Professional,
  domain-specific, precise — no personality, no voice treatment, ever.

### Review-form discipline (initial commit vs delivered form)

Review notes describe the **delivered form** and, where it matters, the contrast
against the **initial commit** — never the versioned hardening journey
(v0→v1→v2 narration). Versions are trainer-local: do not reference versioned
artifacts (zip names, "reference vN anatomy") in review notes, aid files, or the
bundle. The only acceptable version strings are factual schema labels
(ATIF-v1.7) and Drive delivery labels. The 14 sections read as the final state:
what the task is, how it measures, what changed from the initial commit, and
what was fixed during hardening — without version numbers. Author the notes in
the user's voice (the form-answer voice rule above): plain, direct,
contractions, no em dashes — facts verbatim.

**review.csv can be generated locally** — no web app needed: run
`<qc-script-dir>/generate_review_csv.py <draft.csv> [out.csv]` (the script emits
the form's exact byte format: UTF-8 BOM, CRLF, standard quoting) and it is
byte-identical to the Apps Script export when the draft carries the form's
section labels (four Layer 2/3 labels omit the middle dot — verified
2026-08-28). **Canonical copy: `~/Dev/computer-bench/000000/gen-g688-loan-file-risk-screen/pg/packaging/generate_review_csv.py`**
(task 000000, the reference task); per-machine copies sit in each batch's
qc-script dir and should be re-synced from the canonical one. The qc-script dir
is machine/batch-local and may differ per task — locate it in the task
workspace first (or copy the generator + unified_qc.py into the task's
qc-script dir) before relying on it. Author the
draft thoroughly (review_form_document.md is the source of truth), sync labels,
generate, and upload the file to Drive manually; the form itself can be skipped.

Everything after packaging — QC review form, Delivery Gate, pipeline submit,
labelling, and the no-re-upload rule — is **Start Here steps 6–9**, plus the
final step: log the task in the tracker sheet (gid=3003,
`https://docs.google.com/spreadsheets/d/1bbphEa7oIHfRhMPHEt_TmAbHcej1_ArImtsW43QaRyY/edit?gid=3003#gid=3003`).
Detail: `<vault>/computer-bench/QC Handoff Guide.md` §8.

## Endpoint/model presets

| Endpoint | `.env` base URL | OpenCode model id | `-m` |
|---|---|---|---|
| DeepSeek (user's own, default first run) | `https://api.deepseek.com` (no `/v1` needed) | `deepseek-v4-flash-vision-exp` | `deepseek/deepseek-v4-flash-vision-exp` (matches `--ak` map) |
| GLM proxy (team quota, shared) | `http://34.41.10.8:4000/v1` | `glm-5.2` | `glmproxy/glm-5.2` |

Swapping = edit `.env` + adjust `-m` and the `models:` map inside `--ak`. Everything
else stays identical.
