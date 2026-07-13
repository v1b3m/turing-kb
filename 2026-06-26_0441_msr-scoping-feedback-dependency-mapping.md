---
created: 2026-06-26 04:41
tags:
  - msr
  - feedback
  - scoping
  - estimation
  - project-planning
  - dependency-mapping
---

> [!summary] TL;DR
> **Teams should assess the complexity of a gym before they begin work, so expectations aren't superficial.**
>
> On ReturnMax, the scoping process assessed modules in isolation and missed the deep coupling between them. A week into development, we discovered that a single field change ripples across 6 downstream surfaces — but the timeline was already committed. The fix is straightforward: before work starts, the dev team explores real applications, changes inputs, and traces what happens downstream. If a change to one field recalculates 4+ downstream surfaces, add a coupling buffer to the timeline.

# MSR Feedback: Assess Complexity Before Work Begins

## The Feedback

**Teams should assess the complexity of a gym before they begin work, so expectations aren't superficial.**

On ReturnMax, we started working and realized about a week in that the project was magnitudes more complex than the scoping suggested. Tax forms aren't a flat list of independent screens — they're deeply nested and interconnected. A change to one field recalculates six downstream surfaces. But we didn't know that going in, because the scoping process only assessed modules in isolation — what each module does, how many there are. It didn't ask how they connect.

By the time we understood the real complexity, the timeline was already committed and the team absorbed the difference.

## What Went Wrong

The current scoping process tells you *what* you're building. It doesn't tell you how the pieces connect — and that's where the real difficulty lives. In ReturnMax, this chain was discoverable before any code was written:

```
Amend Schedule C, line 12
  → Recalculate 1040
    → Update refund amount
      → Update summary screen
        → Update filed return snapshot
          → Change what the user is allowed to amend next
```

No one asked "if the user changes this field, what else needs to update?" If someone had, the 6-hop chain would have been obvious. The estimate would have accounted for it.

## What Should Happen Instead

Before work begins on a gym, the dev team spends a focused session mapping the dependencies. The goal is simple: know whether you're walking into a shallow or deep project before you commit to a timeline.

A domain expert is ideal but not always available. The team can self-serve: log into real applications in the domain, navigate them intentionally, change inputs, and observe what updates downstream. Online resources and LLMs fill in the gaps.

The session asks one question repeatedly: **"If this value changes, what else needs to update?"**

**Pick 3–5 key data points in the system and trace every screen, module, or calculation that reads them or recalculates based on them.**

If a single field change ripples across 4 or more downstream surfaces, the project needs a **coupling buffer** on the timeline — at least 1 extra week for a team of ~5.

### What This Looks Like in Practice

| Step | Who | Output |
|---|---|---|
| 1. List major data points | Dev team | A shortlist of the 3–5 most important fields/entities in the system |
| 2. Trace downstream consumers | Dev team (explore real apps, change inputs, use online resources + LLMs) | For each data point, a list of every screen/module that depends on it |
| 3. Count the hops | Gym lead | The longest chain: how many steps from source to furthest consumer? |
| 4. Decide buffer | Gym lead | If any chain ≥ 4 hops → add coupling buffer to timeline |

### What This Produces: A Dependency Map

Here is what the output of a dependency mapping session would have produced for ReturnMax — had one been done before estimation.

#### Primary Chain: Amendment to Schedule C, Line 12

```
[User amends Schedule C, line 12]
         │
         ▼
   ┌─────────────────┐
   │ Amendment Review │  ◄── Hop 1: Displays the before/after values
   │    Screen        │
   └────────┬────────┘
            │ triggers recalculation
            ▼
   ┌─────────────────┐
   │  1040 Form       │  ◄── Hop 2: AGI recalculates (line 12 feeds Schedule C total → 1040 line 8)
   │  Recalculation   │
   └────────┬────────┘
            │ changes AGI
            ▼
   ┌─────────────────┐
   │  Refund / Tax    │  ◄── Hop 3: New AGI → new taxable income → new refund or amount due
   │  Calculation     │
   └────────┬────────┘
            │ changes refund amount
            ▼
   ┌─────────────────┐
   │  Filing Summary  │  ◄── Hop 4: Displays the updated refund/tax-due to the user
   │     Screen       │
   └────────┬────────┘
            │ persists amended data
            ▼
   ┌─────────────────┐
   │ FiledReturn      │  ◄── Hop 5: Stores the amended values, original values, and metadata
   │ Snapshot         │      for audit trail and future amendment eligibility
   └────────┬────────┘
            │ determines available actions
            ▼
   ┌─────────────────┐
   │ Next-Amendment   │  ◄── Hop 6: Rules engine checks which lines have already been
   │ Eligibility      │      amended and excludes them from the "amend again" options
   └─────────────────┘
```

**Blast radius of a bug at the source:** A miscalculation of line 12 corrupts 6 downstream surfaces. A unit test on the amendment form component catches none of them.

#### Secondary Chains

```
[Classification Result]
  → Form Selection (Hop 1)
    → Validation Rule Set (Hop 2)
      → Available Deductions (Hop 3)
        → Dependent Schedules (Hop 4)

[Filed Return Values]
  → Amendment Options List (Hop 1)
    → Per-Category Amendment Forms (Hop 2)
      → Recalculation Engine (Hop 3)
        → Filing Summary (Hop 4)
```

#### Dependency Density Table

| Source Data Point | Downstream Chain | Hops | Affected Screens |
|---|---|---|---|
| Amendment value (Schedule C, line 12) | Review → 1040 → Refund → Summary → Snapshot → Eligibility | 6 | 6 |
| Classification result | Form selection → Validation → Deductions → Schedules | 4 | 4 |
| Filed return values | Amendment options → Category forms → Recalculation → Summary | 4 | 4 |
| Validation rules | Error display → Blocked next steps → Recovery path | 3 | 3 |
| Per-category form selection | Available line items → Data entry → Recalculation | 3 | 3 |

#### How to Read This Map

- **Arrow (→)** means "feeds into" or "causes a recalculation in"
- **Hops** count how many steps from the source data to the furthest downstream consumer
- **Affected screens** is the blast radius — if the source data is wrong, how many surfaces show incorrect information
- **4+ hops = coupling buffer warranted.** Every major data point in ReturnMax hit this threshold.

This is what a focused exploration session produces. No code access needed — just hands-on use of real applications, online research, and asking at each node: *"If this value changes, what else needs to update?"*

---

## The Ask

1. **Add a dependency mapping step** to the scoping process for projects involving forms, workflows, or multi-step user journeys. Not every project needs it — but if users move through screens where earlier choices affect later options, it applies.

2. **Give gym leads the authority to flag coupling risk** and request a timeline buffer based on the mapping output, not just module count.

3. **Use the language of coupling density** (not just "complexity") so the conversation is specific: "This project has high coupling density — a change to one field recalculates 6 downstream surfaces. We need an extra week to handle the blast radius."

## Related Notes

- [[2026-06-26_0413_project-complexity-interconnectedness|Project Complexity: Module Count vs. Interconnectedness]] — the conceptual case
- [[2026-06-26_0425_implementing-coupling-aware-complexity|Implementing Coupling-Aware Project Complexity]] — the full implementation guide with CDS rubric, timeline calibration, architecture patterns, and flow testing strategy
