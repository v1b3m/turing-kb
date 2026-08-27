---
title: "Verifier Quality Guide"
tags:
  - computer-bench
  - verifier
  - qc
---
# Writing and Assessing Verifiers

**1\. Purpose**

A high-quality verifier must establish that:

1. the task is clearly defined;  
2. the verifier checks what the task actually requires;  
3. valid solutions are accepted regardless of harmless presentation differences;  
4. substantive errors are rejected;  
5. the verifier is stable and reproducible;  
6. the environment and harness make the task genuinely executable;  
7. rewards reflect task performance rather than infrastructure or evaluator artifacts; and  
8. an agent cannot obtain a high reward without genuinely completing the intended work.

# **2\. The failure taxonomy**

Nearly all feedback maps to one of these seven categories, ordered by how often they appear and how much damage they do.

| \# | Failure mode | Typical share | One-line symptom |
| :---- | :---- | :---- | :---- |
| **1** | Brittle regex / exact-string grading | \~50% | Correct answer rejected over word order, Markdown, a synonym, hyphenation, or numeric notation. |
| **2** | Instruction–verifier mismatch | \~25% | Grades the final chat message (or wrong file) when the instruction sent output elsewhere. |
| **3** | Infrastructure / harness failures scored as model results | \~15% | API error, verifier timeout, or harness quirk → run scored 0.0. |
| **4** | Undocumented policy assumptions | \~7% | Verifier asserts a rule the instruction never states. |
| **5** | All-or-nothing aggregation | \~3% | 63/64 checks pass → total reward 0.0. |
| **6** | LLM judge missed the trajectory | recurring | Judge claims work didn’t happen when the trajectory shows it did. |
| **7** | Dead / hollow deterministic checks | recurring | Empty or disabled buckets silently renormalize onto one rubric item. |

# **3\. Failure mode 1 — Brittle regex and exact-string grading**

This is the dominant problem. Typical false negatives:

* A memo abbreviates “3 August” as “3 Aug”, or writes counts in a different word order — rejected despite being correct.  
* A correct paraphrase fails a narrow synonym whitelist (“do not change capital interest” vs the one accepted phrase).  
* Negation-blind matching: “this is NOT a gap” still matches the bare word “gap” and is scored as a positive finding.  
* Plurals / hyphenation break exact matches: “savings plan” passes but “savings plans” and “savings-plan coverage” fail.  
* Weak alternation: a pattern collapses to its weakest branch, so a wrong answer slips through (or a good one is false-failed).

## **Anti-pattern**

FRAGILE — exact prose, unanchored, negation-blind:  
{  
  "path": "$.text",  
  "comparison": "regex\_match",  
  "expected": "largest-remainder method"     \# rejects "largest remainder"  
}  
{  
  "comparison": "contains",  
  "expected": "gap"                            \# matches "not a gap"  
}  
{  
  "comparison": "regex\_match",  
  "expected": "cap|26 weeks|uncapped"          \# collapses to just "cap"  
}

## **Better**

PREFER — grade the fact structurally from results.json:  
{  
  "source": { "type": "file", "file": { "type": "json",  
              "command": "read\_file",  
              "arguments": { "path": "results.json" } } },  
  "assertion": { "type": "deterministic", "expected": 3,  
                 "deterministic": { "path": "$.invalid\_codes",  
                                    "comparison": "equals" } }  
}

IF you must match prose, anchor \+ lengthen the stem:  
  "expected": "\\\\backnowledg\\\\w\*"        \# word-bounded, meaningful stem

FOR anything semantic, use an LLM judge, not a regex.

## **Rules**

* Grade the fact structurally, not the prose. If it’s a number or exact field, assert equals on a JSON value — never grep the memo for it.  
* When you must match prose, normalize first (strip Markdown, normalize hyphens, whitespace, dates, numeric notation), then accept an equivalence set — not one literal.  
* Prefer an LLM judge for explanations and justifications. If a human would accept a paraphrase, a regex must not be the grader.  
* Anchor and lengthen every regex you keep. Use \\b and a stem long enough to mean something. Require two concepts to co-occur for stronger checks.  
* Judge the weakest branch of any alternation — that is the strength of the whole pattern.  
* Make prose matching negation-aware; evaluate each clause, not a whole section scanned for isolated failure words.  
* For contains, use an identifier, not an English word (“ESC-105”, “2026-06-30” — not “interest”, “term”, “gap”).

# **4\. Failure mode 2 — Instruction–verifier mismatch**

The verifier grades a different deliverable than the instruction asked for — most often the final chat message when the instruction sent the write-up to a file, or an exact label/title the instruction never specified.

MISMATCH — instruction says: "write the report to report.md"  
but the check grades the final chat message:  
  "source": { "target": "final\_answer" }        \# wrong target

FIX — grade the file the instruction named:  
  "source": { "type": "file", "file": {  
     "type": "md", "command": "extract\_text",  
     "arguments": { "path": "report.md" } } }

OR change the instruction to explicitly require the figures in chat.  
Pick one — do not let the two drift.

* Grade the deliverable where the instruction put it (file sources are supported).  
* Do not require literal titles/labels the instruction never specified; accept the names the source data uses.  
* For every check, quote the exact instruction sentence that creates the requirement and confirm the grading target matches.

# **5\. Failure mode 4 — Undocumented policy assumptions**

The verifier must only assert what a capable agent could infer from the instruction and environment alone. A check that encodes a hidden rule fails agents who followed the written policy literally.

* Every verifier requirement must trace to an instruction sentence. If it doesn’t, add it to the instruction or delete the check.  
* Resolve ambiguity before it reaches the grader (e.g. boolean vs count, the scope of a notes section). Make the instruction unambiguous or accept both readings.  
* Don’t re-check in prose what a structured field already proves (e.g. requiring the digit “3” near a keyword in the memo when results.json already has the count).

# **6\. Failure mode 5 — All-or-nothing aggregation**

Reward is 1.0 only if every check passes, so a single defective check zeroes a materially correct run. Most 63/64 cases are a defective check, not an incomplete answer.

* Fix the underlying brittle/mismatched checks first — that removes most 63/64 failures.  
* Where supported, weight and aggregate so one non-critical check cannot zero an otherwise correct run; reserve hard gates for load-bearing facts.  
* Keep the behavioural gate small and structured (a few derived figures in results.json); make secondary prose/format checks non-fatal or semantic.

# **7\. Failure mode 6 — LLM judge missed the trajectory**

The judge must read the actual trajectory and cite observable evidence.

* Judge scoped/search calls and value reuse, not only direct reads or literal hardcoding.  
* Never treat null/missing metadata as evidence of absence — grade whether the target record was retrieved, or fix the metadata.  
* Accept unambiguous aliases, or score identity resolution as a separate, explicit item.  
* Confirm the judge rationale cites the trajectory; run repeats to ensure verdicts don’t flip on identical deliverables.

# **8\. Failure mode 7 — Dead / hollow deterministic checks**

A check that never runs, or that grades a file the model writes about itself, measures nothing while inflating the apparent rigor of the manifest.

HOLLOW — grading a transcript the model authors:  
{  
  "source": { "file": { "type": "text",  
              "arguments": { "path": "pytest\_out.txt" } } },  
  "assertion": { "expected": "9 passed",  
                 "deterministic": { "path": "$.text",  
                                    "comparison": "regex\_match" } }  
}  
\# The model can just TYPE "9 passed". And "99 passed" also matches.

INSTEAD — execute the real suite against the model’s code, and  
grade the output it produces (results.json), not a self-report.

* Delete or fill empty weighted buckets before shipping; empty sql/state buckets silently renormalize weight onto one rubric item.  
* Execute code, don’t grep it. Ship a real test suite and run it against the model’s code at grading time.  
* Never grade a file the model authors about its own run — reduce those to an exists check at most.

# **9\. Decision framework: which grader to use**

| The requirement is… | Use | Never use |
| :---- | :---- | :---- |
| **A derived number / exact field** | equals (approx\_equals for floats) on a JSON value | a regex over the memo |
| **A money / ratio figure** | approx\_equals with a tolerance | exact equals on a float |
| **A record exists in a CSV** | row-anchored, quote-tolerant regex | the id matched anywhere in the blob |
| **A concept must appear in prose** | anchored word/phrase regex with a meaningful stem | bare contains on an English word |
| **An explanation / justification is correct** | LLM judge citing evidence | a synonym-whitelist regex |
| **A behaviour is implemented in code** | execute a real test suite | grep the source |
| **A record must NOT be flagged** | not\_regex\_match on the delimited field | — |
| **Identity / alias resolution** | explicit item accepting valid aliases | exact-name substring |

**10\. Reward-hacking review**

Adversarially review every high-weight verifier. For each, ask: what's the cheapest way to pass without doing the work? Confirm the agent cannot:

\- Hardcode the expected answer

\- Read golden data

\- Write directly to verifier state

\- Fabricate a placeholder or transcript artifact

\- Spoof a required string

\- Satisfy a rubric while skipping a required side effect

\- Exploit stale state

\- Exploit a path mismatch

\- Manipulate the judge via prompt injection in an input

\- Get credit without retrieving the underlying evidence

**11\. Verifier distribution across evidence sources**

Spread the suite across four sources so no single weak surface decides the score:

\- Trajectory : proves the work happened (right records retrieved, side effects issued).

\- Deliverable : the files the instruction named; primary content lives here.

\- DB / environment state : proves side effects actually landed; strongest anti-spoof.

\- Final answer : grade only when the instruction asks for content in the chat reply.

Aim for a small structured gate on the deliverable or DB state, backed by trajectory and secondary checks. Don't concentrate all weight on one source.

**Summary** 

Every verifier check must answer three questions:

**A. What requirement is this checking?**

The reviewer must be able to point to the relevant instruction or task specification.

**B. Why is this check sufficient?**

Explain why passing the check establishes the requirement.

**C. Why is this check appropriately permissive?**

Explain why a valid alternative solution would not be rejected merely because it uses different wording, ordering, formatting, aliases, or an equivalent implementation.

A verifier check without a clear answer to all three questions should not ship.

**Suggestions**

When making changes to verifiers, make use of the following prompt that incorporates the feedback ( NOTE : Don’t accept the changes without reviewing them first, LLM can  make mistakes)

```text

Rewrite one benchmark task's verifiers to be fair, robust, and faithful to the instruction. Output JSON only — no prose, no code fences, no reasoning.

FIX THESE DEFECTS:

1. Brittle regex → accept surface-form variants (markdown, case, hyphen/plural, word order, `3`==`three`). Keep exact values/IDs/counts deterministic; make prose checks rubric.

2. Target mismatch → grade the deliverable the instruction actually names (file vs final message). Never grade an output location the instruction never mentions.

3. Undocumented assumptions → remove any check the instruction/README/policy does not state; list it in `instruction_gaps` instead.

4. All-or-nothing → split bundled checks; weight critical correctness high, minor formatting low.

5. Infra → every regex compiles; every JSONPath is valid; rubric checks include `config.models` (odd count) + `config.temperature`.

DISTRIBUTION: spread checks across the surfaces the task USES (db_state / final_answer / trace / deliverable files) — but only surfaces the instruction actually requires. For connector tasks never leave a weighted `scoring.weights` bucket empty; populate it or rebalance.

HARD RULES: never weaken a real correctness check (numbers, IDs, rows); never invent requirements; keep the given solution passing; preserve `task_id`.

```

