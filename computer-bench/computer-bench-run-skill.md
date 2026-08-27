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

## Read this first

The authoritative docs live in the personal knowledge base (Obsidian vault,
`/home/v1b3m/Dev/Personal/turing-kb/`):

1. `<vault>/computer-bench/Start Here.md` — **the entry point**. A numbered
   9-step map from design through labelling, with one-line summaries and a link
   to the detail doc for each stage (set-up, design, verifier audit, hardening,
   packaging, QC review form, Delivery Gate, pipeline submit, labelling). The
   working detail doc is `<vault>/computer-bench/QC Handoff Guide.md`. Read the
   map first — it is the current truth; where a guide below disagrees with it,
   Start Here wins.
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
- **Never package a task that does not satisfy the requirements** — the upload/QC
  gate outright rejects it. Only two outcomes exist: the task satisfies the
  requirements (oracle 1.0, inside the 1–3/4 band, findings resolved, human-voiced
  instructions), or it is **abandoned**. Give every task dedicated hardening time;
  if it still cannot be hardened, abandon it and state the reason clearly in the
  labelling tool's Trainer Notes (Stage F — the stated reason is the deliverable
  in that case). A partial, out-of-band, or unverified task is not a package.
- **Instructions must stay human-voiced** — no phrasing engineered to match
  verifier regexes (prompt hacking; unacceptable). Difficulty must come from
  visible domain reasoning (playbook §1).
- **Never `docker image prune -a`.**
- **Never promise a QC verdict for non-connector tasks.** The QC tool's run
  gate currently refuses this family's evidence (see Known gaps in the guide);
  it fails closed. Report partial reviews as partial.

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
  `golden_trajectory.json`), `evaluations/glm-5-2/r1..r5` + `evaluations/stability/r1..r5`
  (each: agent/trajectory.json, result.json, verifier/), `qc/` (Unified QC
  output), `verifications/`, and a root `REWORKED_QC_README.json` trainer note.
  **Never include `.env` or scratch folders.**
- **Harbor platform (older):** `harbor auth login` (GitHub OAuth, once per
  machine) then `harbor upload <job-dir>` — directory in, no zip; records the
  locally computed reward, private by default.

Everything after packaging — QC review form, Delivery Gate, pipeline submit,
labelling, and the no-re-upload rule — is **Start Here steps 6–9** (detail:
`<vault>/computer-bench/QC Handoff Guide.md`).

## Endpoint/model presets

| Endpoint | `.env` base URL | OpenCode model id | `-m` |
|---|---|---|---|
| DeepSeek (user's own, default first run) | `https://api.deepseek.com` (no `/v1` needed) | `deepseek-v4-flash-vision-exp` | `deepseek/deepseek-v4-flash-vision-exp` (matches `--ak` map) |
| GLM proxy (team quota, shared) | `http://34.41.10.8:4000/v1` | `glm-5.2` | `glmproxy/glm-5.2` |

Swapping = edit `.env` + adjust `-m` and the `models:` map inside `--ak`. Everything
else stays identical.
