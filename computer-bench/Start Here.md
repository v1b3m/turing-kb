---
title: Computer-bench — Start Here
tags:
  - computer-bench
  - qc
  - guide
---

# Computer-bench — Start Here

The end-to-end path for a computer-bench task, from design to pipeline submission. Each step has a one-line summary; the detailed docs are linked. The **working detail doc is [[QC Handoff Guide]]** — it carries every command, file, link and trap from the g710 journey. The **spec is [[Shannon QC Control - 25 Aug Re-cut.md]]**; this note is the map between the two.

---

1. **Set up the environment** — harbor, docker image, model endpoints, QC Control access. `harbor run` lives in `~/obi-eval`, jobs in `~/obi-eval/jobs`.
   ➜ [[Set Up Guide]]

2. **Design the task** — pick the difficulty target (1/5–2/5; re-cut band 1–3/4), keep the instruction human-voiced, remember `input/` is an editable lever since 2026-08-26. If the task is non-connector, follow the non-connector conventions.
   ➜ [[Computer Bench Task Quality Playbook - Daniel Sogbey.md]] · [[Non-Connector Guide]]

3. **Write the verifier suite, then audit it** — deterministic checks (equals / contains / regex_match, JSONPath rows); each check must pass the *what requirement / why sufficient / why permissive* test. Why and how in the two verifier guides; the engine mechanics are in [[QC Handoff Guide]] Stage B.
   ➜ [[Verifier Quality Guide.docx.md]] · [[A Guide About Verifier Quality and Transformation.md]]

4. **Harden until inside the band** — oracle must be 1.0 on the packaged state; single-model smoke runs, then a battery only with explicit approval; classify every failure task-owned vs model-owned; **stop the moment you're inside the band**.
   ➜ [[QC Handoff Guide]] Stage A — including the g710 lesson: difficulty comes from coupled reasoning, not register exhaustivity.

5. **Package the bundle** — the exact anatomy (manifest.json = byte-copy of verifier.json, solution/ gold + promoted golden trajectory, evaluations/ assembled from harbor job dirs), and the zip with its exclusions.
   ➜ [[QC Handoff Guide]] Stage C · [[Non-Connector Guide]]

6. **Fill the QC review form** — 14 sections × 4 fields, **all 56 values required**. Build the aid files first (document + 5-column draft CSV, template: `review_form_draft_template.csv`); watch the Apps Script upload error — export the CSV manually to Drive.
   ➜ [[A Guide about `review.csv`.md]] · [[QC Handoff Guide]] Stage D

7. **Run the Delivery Gate** — upload once, read the report, dispose findings: **only one disposition is accepted** (the trainer-owned finding; stability/oracle-mode are Turing-side and ignored). If reward-hacking is flagged, the empirical refutation + dispute-note recipe is in the guide.
   ➜ [[QC Handoff Guide]] Stage E · [[Shannon QC Control - 25 Aug Re-cut.md]]

8. **Submit to pipeline — without re-uploading.** "Ready for finalization" + the Submit-to-pipeline button are the end of QC. The report/review.csv are the Drive record, not a gate requirement.
   ➜ [[QC Handoff Guide]] Stage E (the painful lesson)

9. **Labelling tool** — 5 fields: Pass Rate (choices are 1/5, 2/5, 3/5, 0/5 — no 1/4), Modified Task Link (Drive folder — there is no git remote), Golden Trajectory Zip (Drive file link), QC Proof (Drive link to `qc_report.html` — the QC API is IAP-only), Trainer Notes (concise, no verdict line). Batch id comes from `task.toml` (e.g. `shannon-200`, generator `G710`).
   ➜ [[QC Handoff Guide]] Stage F

---

> [!warning] The one-upload rule
> Once a delivery says **Ready for finalization**, click **Submit to pipeline**. Do **not** re-zip, re-upload, or chase the report inside the bundle — the gate validates the uploaded package in place, and the report is the Drive record, not a package requirement.

> [!tip] Build the aids first
> Always draft `review_form_document.md` + `review_rows_draft.csv` (copy `review_form_draft_template.csv`) before opening the form; every one of the 56 fields is required and the form fails silently on empties.

> [!info] The record
> Drive task folder and the task repo root carry the same three files: `review.csv` (BOM'd form export), `qc_report.html` (final delivery's), golden-trajectory zip. The git repo is local-only — the Drive folder link IS the task link downstream.

> [!example] g710 (the reference run)
> Generator batch **shannon-200 / G710**, labelling task **#1251855**; 96-check suite, 1/4 measured, oracle 1.0; one finding disputed (reward hacking) and cleared; submitted 2026-08-27. Full detail: [[QC Handoff Guide]] §12 and the one-page after-action.
