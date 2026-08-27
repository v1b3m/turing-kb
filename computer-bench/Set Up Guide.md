---
GLM_API_KEY: <your API key here>
GLM_OPENAI_BASE: http://34.41.10.8:4000/v1
DEEPSEEK_API_KEY: <your API key here>
DEEPSEEK_OPENAI_BASE: https://api.deepseek.com
DEEPSEEK_MODEL: deepseek-v4-flash-vision-exp
---
# Company Bench: Running a Task Locally, From Scratch

**Audience:** anyone setting up a fresh machine to run Company Bench tasks, runner or QC reviewer.

By the end of this you will be able to run a task's Oracle check and a 5-run GLM-5.2 evaluation on your own machine, and read the results.

Work through it once. After setup, running a task is two commands.

## Contents

- [[#What you are setting up]]
- [[#Prerequisites at a glance]]
- [[#Step 1: Docker]]gcloud artifacts docker images list us-central1-docker.pkg.dev/delivery-g-obi/data-obi-rl-gym/benchmark-base --limit=1
- [[#Step 2: Access to the task image]]
- [[#Step 3: The Harbor CLI]]
- [[#Step 4: Credentials]]
- [[#Step 5: Your first run (Oracle)]]
- [[#Step 6: GLM-5.2 runs]]
- [[#Reading the results]]
- [[#Troubleshooting]]
- [[#Quick reference]]

## What you are setting up

A task is a folder (a "Harbor package") that you download from Google Drive. You run it with the **Harbor CLI**, which builds a Docker container from a shared base image, starts seven simulated company systems inside it, runs an agent against the task, and grades the result.
*I*
Everything runs on your machine. Nothing is submitted anywhere.

Two kinds of run matter:

| Run         | What it does                                                                              | Needs a model? |
| :---------- | :---------------------------------------------------------------------------------------- | :------------- |
| **Oracle**  | Replays the task's reference solution through the graders. Proves the grading is correct. | No             |
| **GLM-5.2** | Runs the real model against the task. Produces the pass rate you report.                  | Yes, via W&B   |

## Prerequisites at a glance

| Requirement                               | Why                                    | Notes                   |
| :---------------------------------------- | :------------------------------------- | :---------------------- |
| Docker                                    | Runs the task container                | ~64 GB free disk needed |
| Google account with delivery-g-obi access | Pulls the task base image              | Ask your lead if unsure |
| Python 3.12 or newer                      | Harbor CLI requires it                 |                         |
| Harbor CLI                                | Runs and views evaluations             |                         |
| W&B API key                               | GLM-5.2 model calls and rubric grading | Ask your lead           |

   
**Disk:** the shared base image is about 15 GB to download and expands to roughly 50 GB on disk. It is downloaded **once** and shared by every task, so only your first run is slow. Budget 64 GB free.

**RAM:** each running task container asks for 4 GB. This matters when you choose how many runs to do in parallel. See Step 6.

## Step 1: Docker

### Windows

1. Download **Docker Desktop for Windows**:  
 [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

2. Run the installer. When prompted, keep the **WSL 2** backend enabled. This is the default and the supported configuration.

3. Reboot if the installer asks.

4. Launch **Docker Desktop** from the Start menu and wait for the whale icon in the system tray to stop animating.

 
Docker Desktop must be **running** whenever you run a task. It does not always start automatically after a reboot. To make it do so: Docker Desktop, Settings,  General, tick "Start Docker Desktop when you sign in".

Verify, in PowerShell:

```sh
 docker run --rm hello-world	  
```

### macOS

1. Download **Docker Desktop for Mac**:  
 https://www.docker.com/products/docker-desktop/  
 Pick the **Apple silicon** build for M1 through M4 Macs, or the **Intel chip** build for older ones.

2. Open the .dmg and drag Docker to Applications.

3. Launch Docker from Applications and wait for the menu-bar whale to settle.

Verify:

```sh
docker run --rm hello-world
``` 

### Linux

Install **Docker Engine** (Docker Desktop is not required). Follow the guide for your distribution: https://docs.docker.com/engine/install/

Then allow your user to run Docker without sudo:

```sh
sudo usermod -aG docker $USER
newgrp docker
docker run --rm hello-world
```

### Confirm you have the disk space

```sh
docker system df      	# what Docker is currently using
df -h                 	# macOS and Linux: free space
Get-PSDrive C         	# Windows: free space on C:
```

## Step 2: Access to the task image

The base image lives in a private Google Artifact Registry. You need the Google Cloud CLI to authenticate Docker against it.

### Install the gcloud CLI

| Platform | How |
| :---- | :---- |
| Windows | Installer: https://cloud.google.com/sdk/docs/install#windows |
| macOS | Installer: https://cloud.google.com/sdk/docs/install#mac or brew install --cask google-cloud-sdk |
| Linux | https://cloud.google.com/sdk/docs/install#linux |

### Authenticate

Same commands on every platform:

```sh
gcloud auth login
gcloud auth configure-docker us-central1-docker.pkg.dev
gcloud config set project delivery-g-obi
```

`gcloud auth login` opens a browser. Sign in with your **Turing** account.

Two notes:

- `gcloud auth configure-docker` is the one that actually matters for pulling images. It teaches Docker how to authenticate to that registry.
- `gcloud config set project` is not needed for pulling, but set it anyway so other tooling behaves.

### Verify access

```sh
gcloud artifacts docker images list us-central1-docker.pkg.dev/delivery-g-obi/data-obi-rl-gym/benchmark-base --limit=1
```

If that lists an image, you are set. If it returns a permission error, your account  does not have access yet. Ask your lead rather than trying to work around it.

## Step 3: The Harbor CLI

Harbor is a Python package. It needs **Python 3.12 or newer**.

Check what you have:

```sh
python --version
```

If it is older than 3.12, install a newer Python from https://www.python.org/downloads/ before continuing.

### Install (recommended: uv)

uv installs the tool into an isolated environment so it cannot conflict with your other Python packages.

**Windows (PowerShell):**

```sh
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
uv tool install harbor
```

**macOS and Linux:**

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh	  
uv tool install harbor	  
```

### Install (alternative: pipx or pip)

```sh
pipx install harbor
```

or, if you prefer plain pip and accept it landing in your user site-packages:

```sh
python -m pip install --user harbor
```

### Verify

```sh
harbor --version
```

Expect 0.20.0 or newer.

**If the command is not found**, the install directory is not on your `PATH`.

● **Windows:** it installs to `C:\Users\<you>\.local\bin`. Add that to your `PATH` via Settings, then open a **new** terminal.

● **macOS and Linux:** it installs to `~/.local/bin`. Add  
 `export PATH="$HOME/.local/bin:$PATH"` to your `~/.zshrc` or `~/.bashrc`, then open a new terminal.

Harbor also installs two short aliases, hb and hr. This document always uses the full harbor.

## Step 4: Credentials

Two environment variables point the model calls at W&B, which hosts GLM-5.2. The same pair serves both the agent and the rubric grader.

Get the API key from your lead. It starts with `wandb_v1_`.

**Windows (PowerShell), current session only:**

```sh
$env:OPENAI_API_KEY  = "wandb_v1_your_key_here"	  
$env:OPENAI_BASE_URL = "https://api.inference.wandb.ai/v1" 
$env:JUDGE_MODEL     = "openai/glm-5.2"  
```

**Windows, persist across sessions:**

```sh 	  
setx OPENAI_API_KEY  "wandb_v1_your_key_here"     
setx OPENAI_BASE_URL "https://api.inference.wandb.ai/v1"	
setx JUDGE_MODEL = "openai/glm-5.2"  
```

`setx` affects only **new** terminals. Close and reopen PowerShell afterwards.

**macOS and Linux, current session only:**

```sh
export OPENAI_API_KEY="wandb_v1_your_key_here"
export OPENAI_BASE_URL="https://api.inference.wandb.ai/v1"
export JUDGE_MODEL="openai/glm-5.2"
```

**macOS and Linux, persist:** add those two lines to `~/.zshrc` (macOS default) or  `~/.bashrc` (most Linux), then `source ~/.zshrc`.

### Why both variables

`OPENAI_API_KEY` is the credential. `OPENAI_BASE_URL` redirects calls from OpenAI's servers to W&B. **Without the base URL the key is sent to OpenAI and every model call fails**, so set both or neither.

### Optional: `JUDGE_MODEL`

Tasks grade some criteria with an LLM judge, which defaults to GLM-5.2. If your lead tells you to point the judge somewhere else, override it:

**Linux/Mac**

```sh 	  
export JUDGE_MODEL="some-other-model-id"
```

**Windows**

```sh
$env:JUDGE_MODEL = "some-other-model-id"	  
```

Leave it unset unless told otherwise.

### **Verify**

```sh
echo $OPENAI_API_KEY	  
echo $OPENAI_BASE_URL	  
```

```sh  	  
$env:OPENAI_API_KEY	  
$env:OPENAI_BASE_URL	  
```

**Never commit these or paste them into a shared document.**

## Step 5: Your first run (Oracle)

Download a task folder from the Drive link in the tracker and unzip it. You should see this inside:

```sh
<task-name>/	  
  instruction.md	  
  task.toml	  
  tests/	  
  solution/	  
  environment/
```

If any of those are missing, the package is incomplete. Say so rather than trying to repair it.

Now run Oracle. It needs no model, so it is the cheapest way to confirm your setup works.

**macOS and Linux:**

```sh
harbor run \
  -p /path/to/<task-name> \
  -a oracle \
  -o ~/obi-eval/jobs \
  --job-name oracle-<task-name> \
  -y
```

**Windows (PowerShell):**

```sh
harbor run `
  -p C:\path\to\<task-name> `
  -a oracle `
  -o $HOME\obi-eval\jobs `
  --job-name oracle-<task-name> `
  -y	  
```

What the flags mean:

| Flag | Meaning |
| :---- | :---- |
| -p | Path to the task folder |
| -a oracle | Use the Oracle agent, which replays the reference solution |
| -o | Where results are written |
| --job-name | A label for this run, becomes the results folder name |
| -y | Skip confirmation prompts |

### What to expect the first time

**The first run downloads about 15 GB and takes a while.** That is the shared base image. Every later run on any task reuses it and starts far faster.

The run then proceeds through:

1. Building the task image (quick after the first time)
2. Starting the container
3. Booting seven simulated systems inside it, which takes a couple of minutes

While that is happening you will see repeated lines like:

```sh
Healthcheck failed (rc=7, in_start_period=True)
```

**That is normal.** It is Harbor waiting for the systems to finish starting. It retries up to 40 times. You only have a problem if it never says Healthcheck passed.

### Success looks like

**Linux/Mac**
```sh
cat ~/obi-eval/jobs/oracle-<task-name>/*/verifier/reward.json	  
```

**Windows**
```sh
Get-Content $HOME\obi-eval\jobs\oracle-<task-name>\*\verifier\reward.json	  
```

**Output**
```json
{"reward": 1.0}	  
```

**Oracle must be exactly 1.0.** Anything lower means a verifier rejects the known correct answer, which is a defect in the task, not in your setup. Once you see any number at all, your setup is working.

## Step 6: GLM-5.2 runs

This produces the number you report.

### A single run first

Before spending five runs, do one to check the task behaves:

```sh
export OPENAI_API_KEY="<OPENAI_API_KEY>"
export OPENAI_BASE_URL="<OPENAI_BASE_URL>"
export GLM_API_KEY="${GLM_API_KEY:-$OPENAI_API_KEY}"
export JUDGE_MODEL="openai/glm-5.2"

TASK="<TASK_FOLDER>"

harbor run \
  -p "$TASK" \
  -a opencode \
  -m glmproxy/glm-5.2 \
  --ak 'opencode_config={"provider":{"glmproxy":{"npm":"@ai-sdk/openai-compatible","name":"GLM via LiteLLM","options":{"baseURL":"{env:OPENAI_BASE_URL}","apiKey":"{env:OPENAI_API_KEY}"},"models":{"glm-5.2":{"name":"GLM 5.2"}}}}}' \
  --agent-setup-timeout-multiplier 3 \
  --ae OPENAI_API_KEY="$GLM_API_KEY" --ae OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  --ve OPENAI_API_KEY="$GLM_API_KEY" --ve OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  --ve JUDGE_MODEL="$JUDGE_MODEL" \
  -o ~/obi-eval/jobs --job-name "glm-5x-opencode-<task>" -y	  
```

### The 5-run battery

```sh
export OPENAI_API_KEY="<OPENAI_API_KEY>"
export OPENAI_BASE_URL="<OPENAI_BASE_URL>"
export GLM_API_KEY="${GLM_API_KEY:-$OPENAI_API_KEY}"
export JUDGE_MODEL="openai/glm-5.2"

TASK="<TASK_FOLDER>"

harbor run \
  -p "$TASK" \
  -a opencode \
  -m glmproxy/glm-5.2 \
  --ak 'opencode_config={"provider":{"glmproxy":{"npm":"@ai-sdk/openai-compatible","name":"GLM via LiteLLM","options":{"baseURL":"{env:OPENAI_BASE_URL}","apiKey":"{env:OPENAI_API_KEY}"},"models":{"glm-5.2":{"name":"GLM 5.2"}}}}}' \
  --n-attempts 5 --n-concurrent 2 -r 3 \
  --agent-setup-timeout-multiplier 3 \
  --ae OPENAI_API_KEY="$GLM_API_KEY" --ae OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  --ve OPENAI_API_KEY="$GLM_API_KEY" --ve OPENAI_BASE_URL="$OPENAI_BASE_URL" \
  --ve JUDGE_MODEL="$JUDGE_MODEL" \
  -o ~/obi-eval/jobs --job-name "glm-5x-opencode-<task>" -y      	
```

**Windows:** identical, but use a backtick (\`) at the end of each line instead of a backslash, as in Step 5.

New flags:

| Flag                      | Meaning                                 |
| :------------------------ | :-------------------------------------- |
| -a terminus-2             | The agent harness that drives the model |
| -m openai/zai-org/GLM-5.2 | The model                               |
| --ak api_base=...         | Points the agent at W&B                 |
| --n-attempts 5            | Run the task five times                 |
| --n-concurrent 3          | How many run at the same time           |

### Choosing `--n-concurrent`

Each running container asks for **4 GB of RAM**. Set concurrency to fit your machine, leaving a few GB for the operating system:

```sh
--n-concurrent  ≈  (your RAM in GB - 4) / 4
```

| Your RAM | Suggested |
| :---- | :---- |
| 16 GB | 2 to 3 |
| 32 GB | 5 |
| 64 GB | 5 (capped by the run count) |

Setting it too high makes the machine swap, and a starved container can look like a failed task when it is really a resource problem.

**Run one task at a time.** Five parallel attempts already sit near the shared W&B rate limit, and your teammates draw on the same quota.

## Reading the results

### Where things land

```sh
~/obi-eval/jobs/<job-name>/     
job.log                     top-level log for the whole job     
result.json	  
<task-name>__<id>/         	one folder per run	  
  agent/	  
    trajectory.json        	every tool call the agent made	  
    final_answer.md        	what it replied	  
    oracle.txt             	Oracle runs only: step-by-step replay log	  
  verifier/	  
    reward.json             {"reward": 0.6} <- the score	  
    reward.txt             	the same number, plain text	  
    verifier_summary.json   per-verifier detail. The important one.	  
  result.json              	run metadata, including task_checksum	  
  trial.log                	setup and container log
```

With `--n-attempts 5` you get five run folders.

### The browser UI, easiest option

```sh
harbor view ~/obi-eval/jobs	  
harbor view $HOME\obi-eval\jobs
```

Opens at [http://127.0.0.1:8080](http://127.0.0.1:8080), or the next free port up to 8089. It shows jobs, individual runs, rewards, trajectories and verifier output, and lets you compare runs side by side.

**Point it at the same folder you passed to `-o`.**

### The score

```sh
cat ~/obi-eval/jobs/<job-name>/*/verifier/reward.json
```

Each run produces a reward between $0$ and $1$: the fraction of that task's verifiers that passed. Six verifiers with four passing gives $0.6667$.

**The pass rate you report is the mean of the five run rewards.** For example $0.6, 0.5, 0.7, 0.5, 0.4$ gives $0.54$.

### Which verifiers passed, and why

`verifier_summary.json` is where the real information is.

```sh 	  
python -c "
import json,sys
d=json.load(open(sys.argv[1]))	  
print('reward:', d['reward']['total'])     
print(json.dumps(d['verification_summary'], indent=2))	  

for it in d['reward']['rubric']['items']:	  
  print(('PASS' if it.get('passed') else 'FAIL'), it.get('name'), '|', it.get('verifier_type'))	  
  if it.get('motivation'): print('	', it['motivation'][:200])	  
"

~/obi-eval/jobs/<job-name>/<run-folder>/verifier/verifier_summary.json
``` 

The same command works in PowerShell with Windows-style paths.

**Useful fields:**

| Field | What it tells you |
| :---- | :---- |
| reward.total | This run's score |
| verification_summary.passed / .total | How many verifiers passed |
| verification_summary.type_scores | Pass rate broken out by verifier type |
| reward.rubric.items[] | One entry per verifier, with motivation explaining LLM-judged decisions |
| statistics.individual_verifier_stats | Per-verifier pass rate across runs |

**One warning:** this file lists the same verifiers several times under different groupings. Do not grep for "passed" and pair it with a nearby "name". You will attribute a failure to the wrong verifier. Read whole entries from `items[]`, as the snippet above does.

### Telling a broken run from a genuine failure

**This matters. A reward of 0.0 usually means something crashed, not that the task is hard.**

Check for these two things in the run folder:

| Symptom                              | Meaning                                                          |
| :----------------------------------- | :--------------------------------------------------------------- |
| agent/exit-code.txt exists           | The run crashed. The number is meaningless.                      |
| agent/trajectory.json is **missing** | No tool calls were recorded, so every verifier failed by default |

When both appear, read `agent/oracle.txt` (Oracle runs) or `trial.log`. The last lines usually name the cause outright.

A genuine failure looks different: `trajectory.json` is present and populated, some verifiers pass, and the failures carry real motivation text explaining what was wrong with the answer.

For the exception classes — an **agent-broke** (complete transcript, agent worked, produced nothing — a *verifiable incompletion*, neither pass nor fail, replace) and a **network/kill death** (died in setup → known no-work; died mid-agent → **unknown completion**: never classify as broken, never count as pass, inspect the trajectory and recover the writes if present, else rerun and document) — see the **playbook §11.5**. Only model-owned and task-owned outcomes are task evidence, and never draw a conclusion with zero passing runs.

## Troubleshooting

**Docker daemon is not running**
Start Docker Desktop and wait for the whale icon to settle. On Linux,  `sudo systemctl start docker`.

**harbor: command not found**
The install directory is not on your `PATH`. See Step 3.  
Remember to open a new terminal after changing `PATH`.

**Image pull fails with permission denied**  
Re-run gcloud auth login and `gcloud` auth configure-docker `us-central1-docker.pkg.dev`.

If it still fails, your account lacks access to `delivery-g-obi`. Ask your lead.

**Every model call fails with an authentication error**  
`OPENAI_BASE_URL` is almost certainly unset, so the W&B key is going to OpenAI.  

Check both variables are set in the terminal you are actually running from.

**Healthcheck keeps failing and never passes**
The simulated systems did not finish starting. Usually memory pressure. Lower  `--n-concurrent`, close other applications, and retry.

**The run is slow the first time**
Expected. About 15 GB of image download. Only the first run pays this.

**Out of disk**
The base image needs roughly 50 GB. Check with docker system df.

**Never run `docker image prune -a`**  
It deletes the shared base image along with everything else, and you will download 15 GB again. If you must clean up, remove specific images by name.

**Results folder is empty**  
The job failed before producing anything. Read job.log in the job folder.

## Quick reference

**One-time setup**

```sh
# Docker: install and start Docker Desktop (or Docker Engine on Linux)
gcloud auth login
gcloud auth configure-docker us-central1-docker.pkg.dev
gcloud config set project delivery-g-obi
      	  
uv tool install harbor
harbor --version
```

**Every session**

```sh
export OPENAI_API_KEY="wandb_v1_..."
export OPENAI_BASE_URL="https://api.inference.wandb.ai/v1"
$env:OPENAI_API_KEY  = "wandb_v1_..."
$env:OPENAI_BASE_URL = "https://api.inference.wandb.ai/v1"
```

**Run Oracle** (no model, must score exactly 1.0)

```sh  	  
harbor run -p <task-folder> -a oracle -o ~/obi-eval/jobs --job-name oracle-<task> -y
```

**Run GLM-5.2 five times** (the reported pass rate)

```sh
harbor run -p <task-folder> -a terminus-2 -m openai/zai-org/GLM-5.2 \
--ak api_base=https://api.inference.wandb.ai/v1 \
--n-attempts 5 --n-concurrent 3 \
-o ~/obi-eval/jobs --job-name glm-5x-<task> -y
```

**Look at results**

```sh
harbor view ~/obi-eval/jobs                          	# browser UI, port 8080
cat ~/obi-eval/jobs/<job-name>/*/verifier/reward.json	# the type_scores
```

**The two numbers that matter**

|   |   |
| :---- | :---- |
| Oracle reward | must be exactly **1.0** |
| Mean of 5 GLM rewards | the pass rate you report |
