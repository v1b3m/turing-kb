# ServiceNow: Author Knowledge Article from Case Resolution

_Date: March 13, 2026_

---

## Goal

Author a new knowledge article directly from a resolved CSM case (CS0001212) in ServiceNow, so that resolution details are captured and linked to the source case.

---

## Environment

- **Interface:** CSM/FSM Configurable Workspace
- **Case:** CS0001212 (State: Resolved)
- **Resolution notes:** "This has been fixed"

---

## What We Tried

### 1. `...` (More Options) button on case form

Clicked the ellipsis menu top-right of the case. Options: Compose Email, Escalate Case, Report Knowledge Gap, Special Handling Notes, Delete. No "Create Knowledge" option present.

### 2. More ▼ → Attached Knowledge

Opened a form titled "Create New Knowledge Applied to Tasks" with two fields: Knowledge article (search) and Task (pre-filled as CS0001212). This form links an _existing_ article to the case — not for authoring a new one.

### 3. Standalone Create Knowledge form

Navigated to Knowledge → Create Article independently. Found a valid "Create New Knowledge" form (KB0010010) with the right fields. However, **Source task was disabled** — it only auto-populates when launched from within a case context.

### 4. More ▼ → Knowledge Gaps

Reports a gap against an existing article. Not a path to author a new one.

---

## Blockers

- **No "Create Knowledge" button on the case form.** The one-click UI action that opens a pre-linked article draft from a resolved case has not been configured in this workspace. Requires admin setup.
    
- **Source task field is disabled on the standalone form.** The field only becomes editable/auto-filled when article creation is triggered from within a case. Creating an article independently breaks the automatic case linkage.
    
- **Attached Knowledge only links existing articles.** The form under More → Attached Knowledge is a relationship form, not an article authoring form.
    

---

## Current Workaround

Two-step manual process:

1. Create the article independently via Knowledge → Create Article, manually copying resolution notes from the case.
2. Return to CS0001212 → More ▼ → Attached Knowledge and link the newly created article.

---

## Recommended Admin Action

Request a ServiceNow admin to add the **"Create Knowledge"** UI action button to the CSM Configurable Workspace case form. Once enabled, agents can author a new article in a single click from a resolved case — with Source task pre-populated and resolution notes auto-imported.
