---
created: 2026-06-26 04:25
tags:
  - project-complexity
  - estimation
  - architecture
  - learnings
  - project-planning
  - implementation
aliases:
  - coupling-density-framework
  - complexity-implementation-guide
---

# Implementing Coupling-Aware Project Complexity

> [!summary] TL;DR
> This document is the *how* — concrete processes, rubrics, and checklists to replace module-count-based estimation with a coupling-aware approach. Each section includes actionable steps a team can adopt immediately.

## Table of Contents

1. [[#1 Complexity Assessment Framework]]
2. [[#2 Estimation Process]]
3. [[#3 Staffing Decision Matrix]]
4. [[#4 Architecture Containment Patterns]]
5. [[#5 Flow-Level Testing Strategy]]
6. [[#6 Risk Assessment & Early Detection]]
7. [[#7 Onboarding for Deep Workflow Projects]]
8. [[#8 Metrics to Track]]
9. [[#9 Rollout Plan]]

---

## 1. Complexity Assessment Framework

**Goal:** Replace "count the modules" with a reproducible score calculated *before* estimation begins. The score informs how much buffer to add, who to staff, and what kind of testing is mandatory.

### The Coupling Density Score (CDS)

Score each dimension 1–5 (1 = low, 5 = extreme). Multiply by weight. Sum for final score.

| # | Dimension | Weight | 1 (Low) | 3 (Medium) | 5 (High) |
|---|-----------|--------|---------|------------|----------|
| A | **State accumulation depth** | ×3 | Stateless pages; each request is independent | Session state persists; later pages reference earlier choices | Full decision tree; choices on step N determine what exists on step N+5 |
| B | **Cross-module data dependency** | ×3 | Modules read/write their own data only | Modules share a data layer but don't transform each other's output | One module's output is another module's input; transformations chain ≥3 deep |
| C | **Conditional visibility/behavior** | ×2 | All UI is static; no conditional rendering | Simple show/hide based on direct user input | Options selected on page 2 determine which *forms* appear on page 6; branching paths |
| D | **Ripple risk** | ×2 | A bug affects only its own screen | A bug affects the current flow step + 1 adjacent step | A single miscalculation corrupts downstream calculations, summaries, and stored snapshots |
| E | **Recovery complexity** | ×2 | Errors are caught and handled at point of failure | Errors require rolling back to a known checkpoint | Errors require unwinding a chain of dependent state changes across multiple modules |
| F | **Onboarding surface area** | ×1 | A new dev can be productive in one module within days | A new dev needs to understand 3–5 connected modules to be effective | A new dev cannot contribute meaningfully without understanding the full state graph |

**Scoring:**
- **6–20**: Shallow UI territory. Standard estimation, generalist team, unit tests sufficient.
- **21–40**: Mixed. Add 30% buffer to estimates, require at least one flow-level test per critical path, prefer 1–2 team members with state-machine experience.
- **41–60**: Deep workflow territory. Add 50%+ buffer, flow-level tests mandatory for all critical paths, staff with at least one senior who has done this before, require a state graph document before coding starts.

### How to Run the Assessment

1. **Who:** Tech lead + product owner + 1 senior engineer, together. Not solo.
2. **When:** During scoping, before any estimate is committed.
3. **Input:** Wireframes, user stories, or flow diagrams — whatever exists. If none exist, score a 5 on Dimension A and flag it.
4. **Output:** A CDS score + a one-paragraph narrative explaining the main coupling risks. Both go into the project brief.
5. **Re-score:** At the end of discovery/design phase. If the score moved by more than one tier, re-estimate.

---

## 2. Estimation Process

**Goal:** Replace "N modules × X days/module" with an estimate that accounts for coupling tax.

### The Coupling-Aware Estimation Formula

```
Base estimate = sum of (module complexity × team familiarity factor)
Coupling tax    = Base estimate × (CDS / 20) × 0.15
Integration tax = Base estimate × (module count above 5) × 0.05
Total estimate  = Base estimate + Coupling tax + Integration tax
```

**Example:**
- 12 modules, average 3 days each → Base = 36 days
- CDS = 42 → Coupling tax = 36 × (42/20) × 0.15 = 11.3 days
- Integration tax = 36 × (12-5) × 0.05 = 12.6 days
- Total = 36 + 11.3 + 12.6 = **59.9 days** (vs. the naive 36)

### Milestone Structure for Deep Workflow Projects

Don't milestone by module ("auth done," "profile done"). Milestone by **completed flow**:

| Naive milestone | Flow-aware milestone |
|---|---|
| "Amendment form complete" | "User can amend Schedule C line 12 and see the updated 1040 and summary" |
| "Validation module done" | "Validation errors on step 3 correctly block step 5 and show recoverable error states" |
| "API integration done" | "Full filing flow: classify → extract → validate → file, with rollback on failure" |

Each milestone must include:
- Entry state (where does the user start?)
- The path (which steps do they take?)
- Exit state (what should be true at the end?)
- Failure modes (what breaks, and does the user land somewhere recoverable?)

### Estimation Checklist

Before committing an estimate, confirm:
- [ ] CDS score calculated and documented
- [ ] State graph (even a rough one) exists
- [ ] Critical paths enumerated
- [ ] Coupling tax applied to the estimate
- [ ] At least one flow-level test per critical path is in the plan
- [ ] Timeline buffer matches the CDS tier

---

## 3. Timeline Calibration

**Goal:** Translate CDS tier into a delivery timeline. Teams are fixed (~5 devs including a lead) — complexity doesn't change who's on the project, it changes how long they get.

### The Fixed-Team Reality

Teams are consistently ~5 developers including one lead. The lever available to planning is **time**, not composition. The CDS tier determines how much additional time a project needs beyond a baseline estimate.

### Timeline Buffer by CDS Tier

The baseline for a project of a given scope assumes a CDS ≤ 20 (shallow UI, independent modules). Every tier above that adds buffer:

| CDS Tier | Timeline Adjustment | Rationale |
|---|---|---|
| **Shallow (6–20)** | Baseline | Modules are independent; standard velocity applies. |
| **Mixed (21–40)** | Baseline + ~1 week | State coupling adds debugging overhead, cross-module coordination, and flow testing. |
| **Deep (41–60)** | Baseline + ~1.5–2 weeks | Every change has blast radius. Flow tests catch bugs unit tests can't. Back-navigation and recovery paths multiply the test surface. |

### What the Extra Time Buys

The buffer isn't padding — it funds specific activities that prevent coupling bugs from reaching production:

- **State graph production and maintenance** (see [[#4.1 The State Graph Document (Mandatory for CDS ≥ 21)]])
- **Flow-level test authoring** — one per critical path, plus back-navigation and failure-injection variants (see [[#5 Flow-Level Testing Strategy]])
- **Flow walkthroughs** — weekly 30-minute sessions where the lead walks the team through one critical path end-to-end
- **Coupling review** — before merging any PR that touches a data point with ≥3 downstream consumers, a second team member traces the ripple

### Red Flags

If these are true, flag the project for re-estimation or scope reduction:
- CDS ≥ 35 and the timeline doesn't include at least a 1-week buffer
- No one on the team can draw the state graph from memory by sprint 3
- "We'll figure out the flows as we go" is the stated approach
- The lead hasn't worked on a deep workflow project before and the buffer is ≤1 week

### Working Within a Fixed Team

Since team composition can't be changed per project, optimize within the constraint:

- **The lead owns the state graph.** They're the single point of truth for how modules connect. If they can't draw the full graph on a whiteboard, no one can.
- **Pair on cross-module work.** When a task touches two modules that feed into each other, pair the module owner with someone from the consuming side. Don't let one person wire modules together in isolation.
- **Rotate flow-test authorship.** Every team member writes at least one flow test per project. It's the fastest way to internalize the coupling graph.
- **Use flow walkthroughs as team calibration.** Weekly, not just during onboarding. Keeps the full coupling map in everyone's head — not just the lead's.

---

## 4. Architecture Containment Patterns

**Goal:** Design the system so coupling is explicit, traceable, and bounded — even when the domain is inherently coupled.

### 4.1 The State Graph Document (Mandatory for CDS ≥ 21)

Before any code is written, produce a single document containing:

1. **Nodes**: Every distinct state/screen/step the user can be in
2. **Edges**: Every possible transition, labeled with the condition that triggers it
3. **Data dependencies**: For each node, what data it consumes and what it produces
4. **Ripple map**: For each piece of data, every node that reads it or depends on it
5. **Error states**: For each node, what happens on failure and where the user lands

Format: Mermaid, XState definition, or a canvas. The format matters less than the fact that it exists and is maintained. A stale graph is worse than no graph — it must be updated when flows change.

### 4.2 Explicit State Management

| Pattern | When to use | Implementation |
|---|---|---|
| **Single source of truth** | Always | One store/context holds the canonical flow state. No component owns flow state locally. |
| **State machine (XState, Robot)** | CDS ≥ 21 | Use a finite state machine library. Transitions are explicit and guarded. Impossible states are literally impossible to represent. |
| **Event sourcing** | CDS ≥ 41 or audit trail required | Every state change is an event. Replay the event log to reconstruct any state. Enables debugging of "how did we get here?" |
| **Derived/ computed state** | When one value feeds many consumers | Calculate downstream values from the source, never store derivations independently. If the amendment changes line 12, the 1040 total is recomputed, not updated separately. |

### 4.3 Interface Contracts Between Modules

For any pair of modules where Module A produces data Module B consumes:

1. Define the contract as a type/schema. Both sides import it.
2. If Module A changes what it produces, the contract version bumps.
3. Module B explicitly declares which contract version it expects.
4. At integration time, version mismatches fail at build time or startup, not at runtime deep in a flow.

In practice (TypeScript example):
```typescript
// contracts/amendment-output.v2.ts
export interface AmendmentOutputV2 {
  schedule: "C" | "E" | "SE";
  lineItem: string;
  amendedValue: number;
  originalValue: number;
  // v2 added:
  reasonCode: AmendmentReason;
}

// Module A (amendment form) exports AmendmentOutputV2
// Module B (1040 recomputation) imports and expects AmendmentOutputV2
// If A upgrades to v3, B fails to compile until it explicitly adopts v3
```

### 4.4 Containment Boundaries

Even in a deeply coupled system, draw containment lines:

- **Horizontal slices** (by flow phase): The "data entry" phase doesn't import from the "review" phase directly — they communicate through the shared state machine.
- **Vertical slices** (by tax category): Schedule C logic doesn't import from Schedule E logic.
- **Shared kernel**: A small set of types, constants, and pure functions that both sides can depend on. Keep it small — if it grows beyond ~20 exports, you've probably broken containment.

---

## 5. Flow-Level Testing Strategy

**Goal:** Catch the bugs that live in the edges between modules, not inside them.

### 5.1 The Test Pyramid Inversion

For deep workflow projects, invert the traditional test pyramid:

| Test type | Traditional allocation | Deep workflow allocation | Why |
|---|---|---|---|
| **Flow/scenario tests** | 10% | 40% | The real bugs are in state transitions and cross-module data flow |
| **Integration tests** | 20% | 30% | Module-to-module contracts need verification |
| **Unit tests** | 70% | 30% | Still valuable for pure logic, but not where the risk lives |

### 5.2 How to Write a Flow Test

A flow test follows a complete user journey from entry to exit. Template:

```
GIVEN the user is [entry state with specific data]
WHEN they [perform sequence of actions across multiple steps]
THEN [exit state should be true]
AND [intermediate states should hold at each step]
AND [failure at step N should land the user at recoverable state R]
```

**Example (ReturnMax amendment flow):**
```
GIVEN a filed 2025 return with:
  - Schedule C, line 12 (supplies): $5,000
  - 1040, line 8 (AGI): $45,000
  - Refund: $2,100

WHEN the user:
  1. Navigates to Amend → selects 2025 return
  2. Chooses Schedule C
  3. Changes line 12 from $5,000 to $6,200
  4. Reviews the amendment summary
  5. Submits

THEN:
  - The amendment summary shows: line 12 $5,000 → $6,200
  - The recalculated 1040 shows AGI: $46,200
  - The updated refund is $2,340
  - The FiledReturnSnapshot stores the new values with amendment metadata
  - The "amend again" screen correctly excludes Schedule C line 12 (already amended)

AND IF the user goes back at step 4 and changes line 12 to $4,800:
  - AGI recalculates to $44,800
  - Refund recalculates to $1,860
  - No stale $6,200 values appear anywhere
```

### 5.3 Test Infrastructure

What you need to make flow tests practical:

1. **Flow test harness**: A helper that sets up initial state, runs a sequence of actions, and asserts at each step. Don't repeat setup boilerplate in every test.
2. **State snapshot utility**: A function that captures the full application state at any point. Flow tests assert against snapshots, not individual DOM elements.
3. **Back-navigation fuzzer**: A test utility that, at each step of a flow, randomly navigates back 1–3 steps, then forward again, then asserts nothing is corrupted. Run this on every critical path.
4. **Failure injection**: The ability to force any step in the flow to fail (network error, validation error, timeout) and assert the user lands in a recoverable state with no data corruption.

### 5.4 Flow Test Checklist

For each critical path, confirm these tests exist:

- [ ] Happy path: full flow, start to finish
- [ ] Back + forward: navigate back at step N, change a value, go forward, assert correctness
- [ ] Failure at each step: force an error at each step, assert recoverable state
- [ ] Abandon + resume: start the flow, abandon midway, return later, assert state is intact
- [ ] Concurrent flows: start flow A, switch to flow B, complete B, return to A, assert no cross-contamination

### 5.5 When Flow Tests Run

- On every PR that touches a module participating in a critical path
- Nightly on the full suite
- Before every release, with a human reviewing the test report

---

## 6. Risk Assessment & Early Detection

**Goal:** Surface coupling risks before they become bugs, and contain them when they do.

### 6.1 Pre-Development Risk Audit

Run this during scoping/discovery, before any code:

#### Dependency Graph Audit

For each module, list:
- What data it reads (and from where)
- What data it writes (and who reads it)
- What conditions gate its visibility or behavior

Then draw the graph. If the graph looks like a tree, you're in good shape. If it looks like a mesh, every node is a risk multiplier.

#### Ripple Map

Choose the 3 most "upstream" data points in the system. For each one, trace every downstream consumer. Count the hops.

- 1–2 hops: Low risk
- 3–4 hops: Medium — requires flow tests
- 5+ hops: High — this data point is a blast-radius multiplier. Flag it for extra review on every change.

### 6.2 During-Development Signals

Watch for these during sprints. Each is a leading indicator that coupling is biting you:

| Signal | What it means | Action |
|---|---|---|
| "This should be a quick fix" takes 3+ days | The fix propagated through unexpected modules | Stop. Update the ripple map. The map is wrong or incomplete. |
| A bug reported on screen 5 was caused by a change on screen 2 | The team doesn't understand the coupling graph | Run a flow walkthrough. Have the person who changed screen 2 trace the data path to screen 5 for the team. |
| QA finds 3+ related bugs in one flow test run | The coupling is denser than estimated | Re-score the CDS. Consider adding more flow tests before continuing feature work. |
| "Let's just add a flag" becomes the fix twice in one sprint | State is being patched, not modeled | Pause. The state model is insufficient. Invest in making the state machine explicit rather than accumulating flags. |

### 6.3 Pre-Release Flow Audit

Before any release that touches flow logic:

1. **Run the full flow test suite.** Any failure blocks the release.
2. **Manual walkthrough** of the top 3 critical paths by a human who didn't write the code.
3. **State graph diff**: If the state graph changed, require explicit sign-off from the tech lead. A changed graph means changed behavior somewhere — find it.

---

## 7. Onboarding for Deep Workflow Projects

**Goal:** Get new team members to the point where they can hold the state graph in their head before they write code.

### 7.1 The Onboarding Sequence

**Day 1–2: User-perspective immersion**
- Run through every critical flow as a user. Not watching a demo — hands on keyboard.
- Document one flow end-to-end in their own words.
- Result: they know what the system does, even if they don't know how.

**Day 3–4: State graph study**
- Read the state graph document. Then reproduce it from memory on a whiteboard.
- The lead reviews: what did they miss? What edges did they draw that don't exist?
- Repeat until they can draw it correctly without reference.
- Result: the state graph is in their head.

**Day 5: Trace exercise**
- Given a specific data point (e.g., "Schedule C, line 12"), trace every module that reads it.
- Given a specific bug report, trace the data path from source to symptom.
- Result: they can navigate the coupling graph.

**Day 6–10: Shadow + small task**
- Pair on a real task with someone who already knows the system.
- First solo task: a flow test, not a feature. Writing a flow test forces them to understand a path end-to-end without the risk of introducing bugs into production code.
- Result: their first contribution validates their understanding.

### 7.2 Knowledge Persistence

Deep workflow projects lose knowledge fast when people leave. Mitigations:

- **The state graph is checked in.** It lives next to the code, not in someone's head or a stale wiki.
- **Flow tests are the executable spec.** If the tests pass, the flows work — even if no original team member remains.
- **Decision records for coupling choices:** When you choose to couple Module A and Module B (or choose *not* to), write a one-paragraph ADR explaining why. Future devs will thank you.

---

## 8. Metrics to Track

**Goal:** Know whether your approach to coupling complexity is improving over time.

### 8.1 Lead Metrics (Predictive)

| Metric | How to measure | Target |
|---|---|---|
| **CDS estimation accuracy** | Compare pre-project CDS to re-score at project end. Was the tier correct? | ≥80% of projects within correct tier |
| **Flow test coverage** | Critical paths with ≥1 flow test ÷ total critical paths | 100% for CDS ≥ 21 |
| **State graph freshness** | Days since last update to the state graph document | Updated within 3 days of any flow change |
| **Back-navigation bugs** | Bugs filed where the repro steps include "go back and change X" | Trending down sprint over sprint |

### 8.2 Lag Metrics (Outcome)

| Metric | How to measure | Target |
|---|---|---|
| **Coupling bug rate** | Bugs traced to cross-module state inconsistency ÷ total bugs | <20% (lower is better; this will never be zero in deep workflow) |
| **Estimate accuracy** | Actual ÷ estimated (at the CDS-adjusted estimate, not the naive one) | 0.8–1.2 |
| **Ramp time to productivity** | Days from new dev start to first merged PR that touches flow logic | ≤10 working days |
| **Release rollback rate** | Releases rolled back due to flow bugs ÷ total releases | <5% |

---

## 9. Rollout Plan

**Goal:** Adopt this framework incrementally. Don't boil the ocean.

### Phase 1: Pilot (Next project, CDS ≥ 21)

- Run the CDS assessment before estimation.
- Produce a state graph before coding.
- Write flow tests for the top 2 critical paths.
- Apply the coupling-aware estimation formula.
- Track the metrics in [[#8 Metrics to Track]].
- **Success criteria:** The estimate was more accurate than the previous 3 projects. At least one flow test caught a bug that unit tests missed.

### Phase 2: Standardize (After 2 successful pilots)

- CDS assessment becomes a required step in project scoping.
- State graph document becomes a required artifact (like a PRD or tech spec).
- Flow tests become required for all CDS ≥ 21 projects.
- Timeline buffers by CDS tier are used in resource planning.

### Phase 3: Institutionalize (After 6 months)

- Historical CDS scores + outcomes inform a calibrated estimation model.
- The coupling-aware formula is tuned with real data from your teams.
- "Flow test coverage" appears on project dashboards alongside code coverage.
- Onboarding templates for deep workflow projects are maintained and improved.

---

## Appendix A: Quick Reference Cards

### The CDS Scorecard (Print This)

| Dimension | Weight | Score (1–5) | Weighted |
|---|---|---|---|
| A — State accumulation depth | ×3 | | |
| B — Cross-module data dependency | ×3 | | |
| C — Conditional visibility/behavior | ×2 | | |
| D — Ripple risk | ×2 | | |
| E — Recovery complexity | ×2 | | |
| F — Onboarding surface area | ×1 | | |
| **CDS Total** | | | |

**Tiers:** Shallow 6–20 | Mixed 21–40 | Deep 41–60

### The Estimation Cheat Sheet

```
Coupling tax    = Base × (CDS / 20) × 0.15
Integration tax = Base × (modules - 5) × 0.05
Total           = Base + Coupling tax + Integration tax
```

### The Flow Test Template

```
GIVEN [entry state with data]
WHEN [sequence of actions]
THEN [exit state]
AND [intermediate assertions]
AND IF [failure at step N] THEN [recoverable state R]
```

---

## Appendix B: Real Example (ReturnMax Amendment Flow)

### CDS Assessment

| Dimension | Score | Rationale |
|---|---|---|
| A — State accumulation | 5 | Full decision tree: amendment type → category → form → line item → review → submit, each step depends on all prior choices |
| B — Data dependency | 5 | Amend line 12 → recalculate 1040 → update refund → update summary → update snapshot. 5-hop chain. |
| C — Conditional behavior | 4 | Per-category amendment forms; available amendments change based on what was previously amended |
| D — Ripple risk | 5 | One miscalculation corrupts: 1040, refund amount, summary display, next-amendment eligibility, filed return snapshot |
| E — Recovery complexity | 4 | Recovering from a mid-amendment error requires unwinding to the correct checkpoint and preserving partial state |
| F — Onboarding surface | 4 | New dev must understand routing, per-category forms, recalculation chain, snapshot persistence, and the amendment state machine |
| **CDS** | **46** | **Deep workflow territory** |

### What This Meant in Practice

- **Estimation:** A naive module-count estimate for the amendment feature would have been ~15 days. The coupling-aware estimate: ~28 days. The actual took 26 days.
- **Timeline:** The amendment feature got a ~1.5-week buffer beyond baseline. Without it, the 4 coupling bugs caught by flow tests would have reached production.
- **Testing:** The flow tests caught 4 bugs that unit tests missed — all related to state corruption on back-navigation.
- **Architecture:** The route-based amendment flow with per-category forms was a direct result of recognizing the coupling density and designing explicit containment boundaries.

---

## Related Notes

- [[2026-06-26_0413_project-complexity-interconnectedness|Project Complexity: Module Count vs. Interconnectedness]] — the summary/conceptual version
- [[2026-06-26_0441_msr-scoping-feedback-dependency-mapping|MSR Feedback: Dependency Mapping in Scoping]] — the specific feedback to MSR about what the scoping process missed
