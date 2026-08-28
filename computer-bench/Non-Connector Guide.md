---
title: "Non-Connector Guide: Running an Offline, File-Deliverable Task Locally"
tags:
  - computer-bench
  - non-connector
---
# Non-Connector Guide: Running an Offline, File-Deliverable Task Locally

**Audience:** anyone running a Harbor "non-connector" task (no simulated company, no MCP connectors) on their own machine, and anyone reviewing or designing such tasks.

This is the companion to [[Set Up Guide]]. Read the Set Up Guide first for the common install steps (Docker, Harbor CLI, the oracle concept); this guide covers what is **different** about non-connector tasks, plus verified corrections to the Set Up Guide.

**Team playbook (the authoritative design-to-package flow):** [[Computer Bench Task Quality Playbook - Daniel Sogbey]] — 15-step flow (cold-read instruction → inputs/policy → plain-English understanding → one representative case → pressure-test questions → expected-verifier checklist → compare with real verifier → fill gaps → harden → Oracle 1.000 → GLM target 1/5–2/5 → failure review → QC → package), plus the **Upload_reworked-QC zip spec** (§13) and a fixed decision table for model-owned vs task-owned failures (§11). This guide's Step 4/5/6 are the mechanics of those stages for non-connector tasks; the playbook supplies the judgment.

By the end you can run a non-connector task's Oracle check and its model evaluation, and read the results — and you will know which parts of the Set Up Guide do **not** apply.

## The whole lifecycle at a glance

A task goes through **six stages**. Read top to bottom; each stage has a goal, a command, and an output. Earlier stages gate later ones (never battery before a single run, never upload before a QC verdict).

| # | Stage | Question it answers | Command / surface | Output |
| :- | :--- | :--- | :--- | :--- |
| 0 | **Author** | Is the task *built*? | `task.toml`, `instruction.md`, `tests/verifier.json`, `environment/Dockerfile`, `solution/` (gold answer) | the task folder |
| 1 | **Oracle** | Does a correct answer exist — and does the grader accept it? | `harbor run -a oracle` | `verifier/reward.txt` = **1** (must be exactly 1) |
| 2 | **Model run** | Can an agent do the work? | `harbor run -a opencode -m <model>` — one run first | reward 0/1 + named checks in `test-stdout.txt` |
| 3 | **Analyze** | *Why* 0? Which checks failed — capability or task defect? | `verifier/test-stdout.txt`, `agent/trajectory.json`, `agent/opencode.txt` | postmortem findings |
| 4 | **Harden** | Is the task calibrated to 1/5–2/5? | only `instruction.md` + `tests/verifier.json` may change (never `environment/input/`) | pass rate = mean of a 5-run battery |
| 5 | **QC** | Does an independent review approve the task? | `unified_qc.py` (dry run → live) | advisory **Keep / Fixable / Reject** |
| 6 | **Upload** | Record or share the results with the platform/team? | `harbor auth login` + `harbor upload <job-dir>` | platform record (private by default) |

Verified on this machine (harbor 0.22.0, 2026-08-26): stages 1–3 end-to-end on the g710 task, including a custom-endpoint (DeepSeek) swap; stage 5 works for the **design gates** (prompt + verifier) but its run gate is blocked on non-connector evidence — see [[#Known gaps and open questions]]. Stage 6 is documented but not yet authorized on this machine.

## Glossary (plain English)

| Term | Meaning |
| :--- | :--- |
| **Task** (a.k.a. *Harbor bundle*) | The folder you run: instructions + input data + verifier + gold solution. |
| **Trial** | One single run of the task. Its results live in `~/obi-eval/jobs/<job>/<trial>/`. |
| **Job** | A batch of trials from one `harbor run` command (e.g. 1 oracle trial, or 5 model trials). |
| **Agent** (`-a opencode`) | The model-driven worker inside the container that does the task. |
| **Oracle** | A privileged agent that *installs the known-good answer* instead of solving — used to verify the grader works. |
| **Verifier** | The deterministic grading checks (`tests/verifier.json`). No LLM judge in non-connector tasks. |
| **Reward** | The grade: **binary 1/0** for this task family (everything must pass). |
| **Trajectory** | The agent's full logged work (`agent/trajectory.json`, ATIF format, and `agent/opencode.txt` raw log). |
| **Artifacts** | The files the agent produced (`artifacts/` in the trial dir; may be empty — see reading results). |
| **Hardening** | The iterate-to-calibration loop: run → analyze → tune *instruction/verifier only* → re-run until 1/5–2/5. |
| **QC (Unified QC)** | The team's independent task review tool (`unified_qc.py`) → advisory Keep/Fixable/Reject, human confirms. |
| **Keep / Fixable / Reject** | QC verdicts: keep as-is / cited editable defect / structural redesign needed. |
| **Dry run** | QC's free pass: deterministic checks only, no model call, no verdict. |
| **Run gate** | QC's third gate: "did the model fail for capability or for task/verifier reasons?" — needs trajectory evidence. |
| **Upload** | Putting a job's results on the Harbor platform (`harbor upload`); shares/records them. |

## Contents

- [[#The whole lifecycle at a glance]]
- [[#Glossary (plain English)]]
- [[#What "non-connector" means here]]
- [[#How to tell your task is non-connector (check these, cheap and definitive)]]
- [[#Prerequisites]]
- [[#Credentials: one .env file, any OpenAI-compatible endpoint]]
- [[#Step 1: Oracle (no model, must be exactly 1.0)]]
- [[#Step 2: One model run (always do this before spending five)]]
- [[#Step 3: The 5-run battery (after the single run is reviewed)]]
- [[#Reading the results (this harness's actual layout)]]
- [[#Step 4: Hardening — iterating to the 1/5–2/5 target]]
- [[#Step 5: Task QC (unified_qc.py) — with the format caveat]]
- [[#Step 6: Uploading and sharing results (harbor upload)]]
- [[#Known gaps and open questions]]
- [[#Troubleshooting]]
- [[#Quick reference]]

## What "non-connector" means here

A non-connector task is an offline "file-deliverable" task. There is no simulated company with seven systems, no healthcheck loops, and no connectors (Slack, Gmail, Drive, …). The agent works in a plain writable `/app`, starting from read-only fixtures at `/app/input`, and must **write deliverable files** (e.g. a CSV, a memo, a JSON summary) into `/app`. Grading replays the same verify-only checks used by the harness.

The practical consequences:

| Resource                | Standard task (Set Up Guide)                                         | Non-connector task                                                                                                                             |
| :---------------------- | :------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------- |
| Docker                  | Required                                                             | Required (same)                                                                                                                                |
| **Base image**          | ~15 GB private image pulled from Google Artifact Registry via gcloud | **None.** The task's `environment/Dockerfile` starts from a public image (e.g. `python:3.12-slim-bookworm`). Build takes minutes, not an hour. |
| **gcloud / GAR access** | Required to pull the base image                                      | **Not needed.** No gcloud install, no `gcloud auth login`, no `configure-docker`.                                                              |
| **W&B API key**         | Model calls and rubric grading                                       | **Not needed for grading.** Model calls use whatever endpoint you configure (see Credentials).                                                 |
| **LLM judge**           | Some criteria graded by an LLM judge (`JUDGE_MODEL`)                 | **None.** All verifiers are deterministic. **Do not set `JUDGE_MODEL`; do not pass `--ve JUDGE_MODEL=…`.**                                     |
| Disk needed             | ~64 GB                                                               | A few GB (small base image + pip deps).                                                                                                        |
| First-run wait          | ~15 GB pull                                                          | Minutes (small image pull + `pip install` at build time).                                                                                      |

## How to tell your task is non-connector (check these, cheap and definitive)

1. `task.toml` keywords include `non-connector` and/or `offline`:
   ```toml
   keywords = ["harbor", "shannon-200", "gen", "non-connector", "offline"]
   ```
2. `environment/Dockerfile` starts from a **public** image and says so:
   ```dockerfile
   # Harbor CLI task image — offline, file-deliverable.
   FROM python:3.12-slim-bookworm
   ```
   If it starts with `FROM ${BASE_IMAGE}` / a GAR path (`us-central1-docker.pkg.dev/…`), it is a **connector-style** task — use the Set Up Guide (gcloud needed).
3. The Dockerfile comment is explicit when there is no judge:
   ```
   # … even though these tasks never call a judge —
   ```
4. `tests/verifier.json` contains only deterministic assertions (types `file` + `json` sources, `regex_match`/`equals` comparisons). No rubric/judge types. All checks must pass; see Reading results for the exact rule.
5. No `mcp_servers` in `task.toml` (`mcp_servers = []`), `network_mode = "public"`.

It is safe to skip gcloud **and** the W&B key **and** `JUDGE_MODEL` when all five hold.

## Prerequisites

- **Docker** running (see Set Up Guide Step 1).
- **Harbor CLI 0.20.0+** — verified on **0.22.0**; all commands below were run on 0.22.0. Install via `uv tool install harbor` (Set Up Guide Step 3). Expect the same on the team's version.
- **Python 3.12+** — pipx/uv-installed harbor ships its own venv (3.13 observed), so system Python is only needed for the little ad-hoc JSON/python snippets below.
- **A `.env` with model credentials** — see next section.
- **~10 GB free disk** is plenty. **4 GB RAM per running container** (the container is a full Python image + agent bootstrap; the agent's node/npm install is the heavy part — see Slow first run).

## Credentials: one `.env` file, any OpenAI-compatible endpoint

The Set Up Guide's W&B pair (`OPENAI_API_KEY` + `OPENAI_BASE_URL` pointing at `api.inference.wandb.ai`) is replaced by a single pair kept in a `.env` file at the **task directory root**:

```sh
# .env — at the root of the task folder. Never commit or paste this file.
OPENAI_BASE_URL="http://<your-endpoint>:4000/v1"
OPENAI_API_KEY="<your-key>"
```

Verified working combinations:

- **GLM proxy (default, shared team quota):** `OPENAI_BASE_URL=http://34.41.10.8:4000/v1` with the team GLM key. Serves `glm-5.2`.
- **Any OpenAI-compatible endpoint** (e.g. a custom DeepSeek deployment, another provider's `/v1`): same shape — base URL + key. The OpenCode agent configuration below only needs `{env:OPENAI_BASE_URL}` / `{env:OPENAI_API_KEY}` to exist in the container, so swapping endpoints is just editing `.env` (and the model name in the `--ak` block).

Two hard-won rules:

1. **The base URL must end in `/v1`** (for proxies like LiteLLM style `/v1/chat/completions`). An IP/port without the path gets 404s.
2. **Set both or neither.** Without `OPENAI_BASE_URL` the key goes to `api.openai.com` and every call fails with an auth error. Same failure mode as the W&B note in the Set Up Guide.

**Verify the credential before running anything** (a one-token call — costs almost nothing, catches expired keys early):

```sh
set -a && . ./.env && set +a
curl -s -m 30 "$OPENAI_BASE_URL/chat/completions" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"glm-5.2","messages":[{"role":"user","content":"hi"}],"max_tokens":5}' | head -c 300
```

We caught an expired key this way (both Z.AI public endpoints returned `401 "token expired or incorrect"`); the team proxy answered with a live `glm-5.2` completion.

> [!info] Where does the key end up?
> Harbor normalizes env values that match the ambient process env into `${OPENAI_API_KEY}` references in the job's `config.json` (secret hygiene) and resolves them at runtime from the harness process env. This is by design: source `.env` into the *same shell* that runs `harbor`, and the agent process gets the real key. Confirm with: `docker exec <container> bash -c 'env | grep ^OPENAI'` — the values are there for the agent.

## Step 1: Oracle (no model, must be exactly 1.0)

```sh
harbor run \
  -p /path/to/<task-name> \
  -a oracle \
  -o ~/obi-eval/jobs \
  --job-name oracle-<task-name> \
  -y
```

- `-p/--path` accepts a local **task directory** (on 0.22.0 it is listed under the "Dataset" panel; `harbor exec -p` is the separate ad-hoc-task compiler — don't confuse them).
- **First build** downloads `python:3.12-slim-bookworm` (~120 MB) and runs the Dockerfile's `pip install` — a few minutes total, not the 15 GB ritual.
- No healthcheck loops, no "Healthcheck failed" lines, no `agent/oracle.txt` replay log (there is no simulated environment to replay) — the oracle just installs the gold deliverables and grades them.
- Success: `~/obi-eval/jobs/oracle-<task-name>/<trial>/verifier/reward.txt` contains **`1`** (`16/16` tests passed in our case).

**Oracle must be exactly `1`.** Zero means a defect in the task, not your setup.

## Step 2: One model run (always do this before spending five)

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

What each part does, and what to know:

| Flag | Meaning | Note |
| :--- | :--- | :--- |
| `-a opencode` | Agent harness | Alternative: `terminus-2`, or any agent in `harbor run --help` |
| `-m glmproxy/glm-5.2` | Model id passed to the agent | Must match the model name inside `--ak` |
| `--ak opencode_config=…` | OpenCode provider config | The `{env:…}` placeholders resolve inside the container — do **not** hardcode secrets here |
| `--agent-setup-timeout-multiplier 3` | Extra setup time | **Required on first run**: OpenCode bootstraps `node/npm` (via nvm) inside the container |
| `--ae …` / `--ve …` | Agent / verifier env vars | Pass both key and URL; **omit `JUDGE_MODEL`** entirely |
| `--n-attempts -k`, `--n-concurrent -n` | Battery knobs | See Step 3 |
| `-o ~/obi-eval/jobs --job-name … -y` | Output dir, label, autoconfirm | Same as Set Up Guide |

**Quota etiquette (we share a cap):** run **one** attempt first; never launch a battery without reviewing the single run and confirming with your lead. One task at a time. Model consumption is on the team's proxy, so a fresh 24k-token attempt is a cost.

### Swapping to a custom model (e.g. DeepSeek testing)

Only three things change: `-m`, the model name(s) inside the `--ak` JSON, and (usually) nothing in `.env` — the provider name (`"glmproxy"`) is arbitrary; what matters is `npm: @ai-sdk/openai-compatible`, `baseURL`/`apiKey` from `{env:…}`, and `models: {"<model-id>": …}` matching `-m`. Full verified shape for a DeepSeek-compatible endpoint (`deepseek-v4-flash-vision-exp` in this example — use whatever your deployment serves; `.env` then holds the custom endpoint's own key/URL):

```sh
set -a && . ./.env && set +a

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

Start it, then confirm the agent is actually talking to the model (see Step 4) — a wrong provider name or a model the endpoint doesn't serve fails at first model call, which a liveness check catches.

## Step 3: The 5-run battery (after the single run is reviewed)

```sh
harbor run \
  -p "$TASK" -a opencode -m glmproxy/glm-5.2 \
  --ak 'opencode_config={"provider":{"glmproxy":{"npm":"@ai-sdk/openai-compatible","name":"GLM via LiteLLM","options":{"baseURL":"{env:OPENAI_BASE_URL}","apiKey":"{env:OPENAI_API_KEY}"},"models":{"glm-5.2":{"name":"GLM 5.2"}}}}}' \
  --n-attempts 5 --n-concurrent 2 -r 3 \
  --agent-setup-timeout-multiplier 3 \
  --ae OPENAI_API_KEY="$OPENAI_API_KEY" --ae OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  --ve OPENAI_API_KEY="$OPENAI_API_KEY" --ve OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  -o ~/obi-eval/jobs --job-name "glm-5x-opencode-<task>" -y
```

- **Concurrency:** each container asks ~4 GB. With 23 GB RAM use `--n-concurrent 2`; formula ≈ `(RAM GB − 4) / 4`.
- **Reward is binary per run** — see the next section. The reported **pass rate = mean of the five rewards** (= number of runs that scored 1, divided by 5).
- `-r 3` retries transient failures (image pull, container start). Healthcheck-style retries are a connector-task concern; don't expect them here.

## Reading the results (this harness's actual layout)

```sh
~/obi-eval/jobs/<job-name>/
  job.log / result.json          # job-level metadata, task_checksum
  <task-name>__<id>/             # one folder per trial
    trial.log                    # build + container logs ("Running command: apt-get … nodejs npm" = agent setup)
    result.json                  # trial metadata (config, checksum)
    agent/
      trajectory.json            # ATIF v1.7: schema_version, session_id, steps[], final_metrics
      opencode.txt               # raw agent log (JSON lines, tool calls + token usage + delays)
      exit-code.txt              # ONLY present if the agent crashed -> number is meaningless
    verifier/
      reward.txt                 # the number that matters: "1" or "0"
      test-stdout.txt            # pytest -rA: PASSED/FAILED per verifier with names
      ctrf.json                  # machine-readable pytest-json-ctrf output
```

Differences from the Set Up Guide worth memorizing:

- **There is no `verifier/reward.json` and no `verifier_summary.json`** in this harness version. The score is `verifier/reward.txt`.
- **Reward is all-or-nothing.** `tests/test.sh` runs the whole pytest suite and writes `1` only if **every** test passed, else `0`:
  ```sh
  if [ $status -eq 0 ] && [ $suite_status -eq 0 ]; then
      echo 1 > /logs/verifier/reward.txt
  else
      echo 0 > /logs/verifier/reward.txt
  fi
  ```
  The Set Up Guide's "six verifiers with four passing gives 0.6667" is **wrong for this task format**. 14/16 passing still rewards `0`. If you need partial credit semantics, the verifier engine supports per-verifier weights — the *task* must decide; the runner does not.
- **Per-verifier detail lives in `test-stdout.txt`** — the `PASSED/FAILED workbench::…[name]` lines name exactly which check failed. The failure message is generic (`assertion failed`), so for the specifics read the failing definition in the task's `tests/verifier.json` (each has a `how_justification`/`why_justification` and the exact `expected` value, e.g. `"expected": 800.0` with `"comparison": "equals"`).
- **The container is deleted after the trial.** The agent's deliverables live only inside the container unless they were collected as artifacts (`artifacts/manifest.json` — often empty for these tasks). For a postmortem you have `agent/opencode.txt` (tool calls contain the file contents the agent wrote, inspected with the `cat`/`ls` steps) and `trajectory.json`; alternatively add `--artifact /app/retainer_billing_audit.csv --artifact /app/results.json …` to the run command so the files are copied out.

### Telling a broken run from a genuine failure

Same rule as the Set Up Guide; in practice for non-connector tasks:

| Symptom                                                                  | Meaning                                                        |
| :----------------------------------------------------------------------- | :------------------------------------------------------------- |
| `agent/exit-code.txt` exists                                             | Crashed. Number is meaningless. Read `trial.log`.              |
| `trajectory.json` missing / empty                                        | No tool calls were recorded → every verifier fails by default. |
| `trajectory.json` present with steps, some tests pass, no exit-code file | **Genuine run.** Analysis, not infra.                          |

Our first GLM run (14/16): genuine — 12 steps, ~24k tokens, no crash. It flagged all four findings correctly; it failed *only* on (a) `results.json` `total_billing_variance_usd` not exactly `800.0` (`equals`, no tolerance) and (b) the memo regex `\bCL-06.*(not an underbilling|is correct|no rollover)`.

The playbook §11.5 (added 2026-08-27) classifies the exceptions this table does not cover — **the ones that are NOT task evidence at all**:

- **Agent-broke** (verifiable incompletion): the transcript is complete, the agent worked (read/reasoned) and then produced no output — e.g. step-length death (`step_finish reason: "length"`; g688 GLM r1 died at 45.2k tokens after 17 file reads, wrote nothing). Counted neither pass nor fail; replace.
- **Network / kill is a different class**, classified by where the record ends. Died in setup/preflight (g688 GLM r2 `NetworkConnectionError` in `_setup_agent`) → known no-work: the agent never ran, replace, uncounted. Died mid-agent → **unknown completion**: the record stops before the final state is knowable, and the agent may have completed just fine — never classify it as "couldn't complete", never count it as a pass; inspect the trajectory (deliverables written → recover the writes and grade locally, flagged "trajectory-recovered"; absent → rerun, document as unknown).
- Only **model-owned** (content exists, wrong) and **task-owned** (fix the task) outcomes are task evidence. Floor: never conclude anything with zero passing content runs — replace broken/unknown runs until ≥1 passes; the solvability gate needs a reward-1.0 non-oracle run.

### Liveness checks while / after a run

```sh
docker ps --format '{{.Names}}'          # expect <trial>__env-main-1
docker exec <container> tail -c 1200 /logs/agent/opencode.txt
docker exec <container> bash -c 'for d in /proc/[0-9]*; do case "$(tr "\0" " " < $d/cmdline 2>/dev/null)" in *opencode*) tr "\0" "\n" < $d/environ | grep -E "^OPENAI";; esac; done'
```

Real model traffic looks like `step_finish` records with `tokens` (input/output/reasoning) and incrementing `messageID`s. E.g. after 12 steps the log showed ~24k total tokens. If a run exits with reward `0` and `0` tokens, check credentials first (the endpoint, not the task).

## Step 4: Hardening — iterating to the 1/5–2/5 target

> [!important] Mechanism re-cut (2026-08-28): [[Hardening Guide]]
> The loop below is the *mechanics* (run → analyze → tune → re-run). Which mechanisms actually move difficulty — and the calibration requirements (remote matrix, n≥5, canary anchors, sandbox parity) — live in [[Hardening Guide]]. The legacy levers (register scale, boundary density, decoy variants) saturate; step-ceiling deaths do not transfer to the remote pipeline.

Hardening is the loop that turns a working task into a *calibrated* task. It is the stage the team meeting rules govern:

1. **Run** (single, then analyze — never jump to a battery).
2. **Analyze** failures: which checks failed, and *why* — capability gap or task-definition issue (Use the run gate of Step 5 when it is available; until then do the postmortem by hand from `test-stdout.txt` + `verifier.json` + trajectory).
3. **Tune** — editing allowed only on **`instruction.md`** and **`tests/verifier.json`**. **Never touch `environment/input/`.**
4. **Re-run** (single run, then a 5-run battery) until the pass rate lands at **1/5–2/5**.

Meeting rules that constrain every iteration:

- Hard enough that agents without domain training fail; **an intelligent agent should land 1/5–2/5**. 0/5, 3/5, 4/5, 5/5 are all unacceptable.
- Instructions must read like a human asking for the work — **not steps stitched together to trip verifiers** (prompt hacking, not acceptable). Difficulty must come from the domain work itself.
- Watch **brittle details** under the binary reward: an `equals`-compare on an exact aggregate sum or a strict memo regex turns a 95%-correct agent into a `0`. Our g710 runs are the canonical example (GLM 14/16, DeepSeek 13/16, both reward 0): the analysis was right, the *number and wording* weren't. Whether that brittleness is the desired calibration is a design decision, judged on whole-run outcomes.

## Step 5: Task QC (unified_qc.py) — with the format caveat

The team's QC tool lives in `` `qc-script/` `` (a clone of `infra/unified-qc/`; standalone copy here: `~/Dev/computer-bench/qc-script/`). It reviews a task + optional run evidence through three **North Star gates** (prompt & environment; verifier coverage/fairness; run failure attribution) plus a reconciliation stage, and emits an advisory **Keep / Fixable / Reject** — a human verifies each cited finding last. It never edits tasks.

```sh
cd <qc-script-dir>
python3 unified_qc.py --input <task-dir> --dry-run                    # free: deterministic checks only
python3 unified_qc.py --input <task-dir> --trajectory ~/obi-eval/jobs  # live: adds GLM/QC-model review
```

- It reads `.env` from its own directory (`OPENAI_API_KEY`, `OPENAI_BASE_URL`, `QC_MODEL` — use the same values as the task's `.env` for whatever endpoint you want; `--model`/`--base-url`/`--api-key-env` override). Default endpoint is the team GLM gateway.
- Output: `runs/<UTC-timestamp>/{results.json, results.csv, summary.md}`. A dry run never emits a verdict.
- Verified locally (2026-08-26): with the one-line discovery fix (see gaps) the **prompt and verifier gates pass** on g710. The **run gate is not supported for this task format** — see [[#Known gaps and open questions]]. Because of that, live QC on non-connector tasks currently ends as `partial` with **no verdict** — the tool fails closed rather than guess.
- The QC verdict is advisory. Treat a "pass" as one more reviewer's opinion, not a submission gate; the team's human-review checklist in the script's README still applies.

## Step 6: Uploading, packaging, and sharing results

Optional for local calibration — pass rates are read from `~/obi-eval/jobs` directly. Two different "submit" flows exist; use the one matching the current tracker column.

**A. Upload_reworked-QC (current flow per the playbook §13): one self-contained zip** named `<task-id>-upload-reworked-qc.zip`, opening directly at the task package:

```text
task.toml  instruction.md  README.md  environment/  tests/  solution/
evaluations/glm-5-2/       r1/…r5/   each: agent/trajectory.json, result.json, verifier/
evaluations/stability/     r1/…r5/   frozen gold answer re-run 5×, same layout
qc/                        summary.md, results.json, results.csv        (Unified QC output)
verifications/             report.html, summary.json, report.json, full_report.json,
                           issues.csv, unified-qc/, REWORKED_QC_README.json
```

Also required by the checklist: `solution/golden_trajectory.json` (kept inside `solution/` **and** a root copy), `tests/test.sh`/`test_outputs.py`/`verifier.json`, no scratch folders, and a `REWORKED_QC_README.json` trainer note explaining what was changed and why. (g710 is missing `golden_trajectory.json` and `README.md` — see [[#Known gaps and open questions]].)

**B. Harbor platform upload (older flow):** `harbor auth login` (GitHub OAuth, once per machine) then `harbor upload <job-dir>` — directory in, no zip; tars each trial, records the locally computed reward, private by default (`--public`/`--share-org`/`--share-user`). `harbor hub` browses uploaded jobs. Not needed if flow A is the tracker's column.

Nothing is uploaded automatically — `harbor run` never submits anywhere.

## Known gaps and open questions

1. **QC tool lags the current task format (genuine upstream bug).** The team playbook's own packaging checklist (§13) specifies `tests/verifier.json` and `tests/test.sh`/`test_outputs.py` — i.e. our non-connector layout **is** the current standard. `unified_qc.py` however was written for task-harness JSON and connector-style bundles: it expects `tests/manifest.json`, named verifier units (or `verifier_spec` nesting), and run artifacts that contain per-check names. Three concrete gaps:
   - verifier discovery omits `tests/verifier.json` — **one-line fix applied locally** (`unified_qc.py:502`; belongs upstream);
   - the parser collapses our list-of-check-objects `verifier.json` into one anonymous unit (`verifier_1`) — names like `schedule_exists` never register;
   - Harbor 0.22 trial `result.json` carries only the scalar reward; per-check pass/fail lives in `verifier/ctrf.json` + `test-stdout.txt`, which the tool never reads.
   Consequence: the **run gate always refuses** this task family's evidence (`CONFIGURED_VERIFIERS_NOT_OBSERVED_IN_RUNS`) → no verdict. Since the playbook requires `qc/` in the upload zip, this should go to the infra/QC owner — the tool needs a small adapter (unit parsing + ctrf ingestion), not a local hack.
2. **`harbor upload`** — not authenticated on this machine yet; and whether the platform re-grades an upload (vs trusting the recorded local reward) is unverified — check with the platform owner before relying on it as proof.
3. **Battery** — 5-run batteries are still gated behind the single-run review + lead approval (shared quota).
4. **Upload destination** — the playbook names only the zip file (`<task-id>-upload-reworked-qc.zip`) and says "use the current tracker column as the source of truth"; no folder/link is inside it. The Set Up Guide says task folders come from a Drive link in the tracker — i.e. the upload destination lives in the team tracker, not in any doc we have. Confirm the g710 row/column (and where the zip should go) with the lead before packaging.
5. **Artifacts** — `artifacts/` is often empty for these tasks, so postmortems rely on `opencode.txt`/`trajectory.json` or re-running with `--artifact /app/<file>` (see Reading results).

## Troubleshooting

**Image builds fine but the run fails immediately / reward 0 with zero tokens** — credentials. Re-run the curl check (Step 2, Credentials). Check `.env` was sourced in the same shell as `harbor`; check the URL has `/v1`; expired keys return `401 token expired or incorrect`.

**`harbor: command not found` / unknown `-p` flag** — installs to `~/.local/bin` (add to PATH); on 0.22.0 `-p/--path` is under the "Dataset" panel. `harbor run --help` lists everything.

**reward 0 but most tests passed** — not a bug: binary reward (see Reading results). Look at `test-stdout.txt` for the failing names, then the matching `verifier.json` entry for the exact expected value.

**Agent setup hangs** — first run installs node/npm inside the container; `--agent-setup-timeout-multiplier 3` gives it room. Watch `trial.log`.

**Out of RAM** — lower `--n-concurrent`; close other applications (this machine had ~23 GiB with ~4–5 GiB available under use; a single container fits, two is the ceiling).

**Container gone, want the agent's files** — re-run with `--artifact /app/<file>` flags, or extract from `agent/opencode.txt`.

**Never run `docker image prune -a`** — same rule as the Set Up Guide; with tiny images it's less catastrophic but still deletes what the next run needs.

## Quick reference

```sh
# One-time: Docker running, uv tool install harbor, .env at task root (OPENAI_BASE_URL + OPENAI_API_KEY)

# 0. Verify credential (1-token call) — then:
set -a && . ./.env && set +a

# 1. Oracle (must be 1)
harbor run -p <task-dir> -a oracle -o ~/obi-eval/jobs --job-name oracle-<task> -y

# 2. One model run (review before a battery)
harbor run -p <task-dir> -a opencode -m glmproxy/glm-5.2 \
  --ak 'opencode_config={"provider":{"glmproxy":{"npm":"@ai-sdk/openai-compatible","name":"GLM via LiteLLM","options":{"baseURL":"{env:OPENAI_BASE_URL}","apiKey":"{env:OPENAI_API_KEY}"},"models":{"glm-5.2":{"name":"GLM 5.2"}}}}}' \
  --agent-setup-timeout-multiplier 3 \
  --ae OPENAI_API_KEY="$OPENAI_API_KEY" --ae OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  --ve OPENAI_API_KEY="$OPENAI_API_KEY" --ve OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  -o ~/obi-eval/jobs --job-name glm-1x-opencode-<task> -y

# 3. Results
cat ~/obi-eval/jobs/<job>/<trial>/verifier/reward.txt        # 1 or 0
grep -E 'PASSED|FAILED' ~/obi-eval/jobs/<job>/<trial>/verifier/test-stdout.txt

# 4. Hardening loop (until pass rate = 1/5-2/5): tune ONLY instruction.md + tests/verifier.json,
#    re-run a single attempt, then the 5-run battery after review + lead approval.

# 5. Task QC (dry run first, then live; verdict is advisory)
cd <qc-script-dir> && python3 unified_qc.py --input <task-dir> --dry-run
python3 unified_qc.py --input <task-dir> --trajectory ~/obi-eval/jobs

# 6. Upload results (optional; needs harbor auth login once per machine)
harbor upload ~/obi-eval/jobs/<job-name>
```

**The two numbers that matter:** Oracle reward must be **1**; the mean of the GLM rewards is the pass rate you report — and for this task format each run is a **0 or 1**, not a fraction.
