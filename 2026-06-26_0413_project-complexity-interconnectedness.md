---
created: 2026-06-26 04:13
tags:
  - project-complexity
  - estimation
  - architecture
  - learnings
  - project-planning
---

> [!summary] TL;DR
> **Module count is a poor proxy for project complexity. Interconnectedness is what matters.**
>
> From working on both ReturnMax and ClipStream: the tax project was magnitudes harder despite having only slightly more modules. The gap was wildly disproportionate to the module count difference.
>
> **Why:** In ClipStream, pages are independent islands — a bug on the upload page only breaks uploads. In ReturnMax, pages form a decision tree with state accumulation — a choice on form 2 changes what fields exist on form 4, an amendment ripples across schedules, and a single bug can corrupt the summary, the refund calculation, the "what's next" screen, and the filed return snapshot.
>
> **What this means for planning:**
> - **Estimation**: A 10-module project with deep state coupling can be harder than a 50-module CRUD app
> - **Staffing**: Deep workflow projects need people who think in state machines and directed graphs
> - **Testing**: Unit tests cover nodes — the real bugs live in the edges between them. Flow-level scenario tests matter more than component coverage
> - **Risk**: When scoping, map the state coupling first. If choices on one page affect what's valid on others, you're in deep workflow territory
>
> **Bottom line:** Same number of modules, same number of bugs — radically different blast radius. Rank projects by coupling density, not module count.

# Project Complexity: Module Count vs. Interconnectedness

## The Observation

Having worked on both ReturnMax (tax filing/amendment) and a ClipStream clone, the tax project was **magnitudes harder** — even though it had slightly more modules. The difference in difficulty was wildly disproportionate to the difference in module count. The real driver is how the modules connect.

## The Core Distinction

### Shallow UI (ClipStream clone)

Pages are independent islands. The video player doesn't care what happened on the upload form. A subscription doesn't affect how comments render. Each screen is self-contained — you can build them in isolation and they'll almost always compose correctly at the end.

### Deep Workflow UI (ReturnMax)

Pages are **nodes in a directed graph** where edges are conditional on prior nodes. State accumulates across steps:

- A choice on Form 2 changes what fields exist on Form 4
- An amendment on line 12 of Schedule C ripples into the 1040
- That changes the refund amount → changes the summary page → changes what the user is allowed to amend next
- Selecting one option can hide/unhide entirely different sections downstream

This isn't a collection of pages — it's a **decision tree with state accumulation**.

## Why Module Count Is Misleading

| Dimension | Many modules, low coupling | Fewer modules, high coupling |
|---|---|---|
| **Testing** | Test each module in isolation | Must test flows across modules — edges matter more than nodes |
| **Debugging** | Bugs are local | Bugs propagate — a mistake on page 1 surfaces on page 5 |
| **Estimation** | Linear with module count | Non-linear — each new module wires into existing state graph |
| **Onboarding** | Learn one module at a time | Must understand the full state machine before any module makes sense |
| **Change cost** | Low — changes are contained | High — "small" changes ripple unpredictably |

## Implications for Planning

1. **Estimation**: Stop ranking projects by module count. A 10-module project with deep state coupling can be harder than a 50-module CRUD app.
2. **Timeline**: Teams are fixed (~5 devs including a lead). The lever is time, not composition. Deep workflow projects need a ~1–2 week buffer beyond a baseline estimate of the same scope — not padding, but time for state graph maintenance, flow tests, and coupling reviews that prevent bugs from reaching production.
3. **Architecture**: The most valuable design decisions are about **how state flows**, not how components are organized.
4. **Testing strategy**: Unit tests verify nodes. The real bugs live in the edges — flow-level scenario tests matter more than component test coverage.
5. **Risk assessment**: When scoping a new project, map the state coupling first. If choices on one page affect what's visible/valid on others, you're in deep workflow territory — plan accordingly.

## Concrete Example

In a ClipStream clone, if the "upload video" page has a bug, only uploads break. Everything else works fine.

In a tax filing system, if the amendment form miscalculates a dependent deduction, the 1040 shows the wrong total, the summary shows the wrong refund, the "what's next" screen offers the wrong options, and the filed return snapshot stores incorrect data that affects future amendments.

**Same number of bugs. Radically different blast radius.**

---

## Related Notes

- [[2026-06-26_0425_implementing-coupling-aware-complexity|Implementing Coupling-Aware Project Complexity]] — the full implementation guide: scoring rubric, estimation formula, staffing matrix, architecture patterns, flow testing strategy, and rollout plan
