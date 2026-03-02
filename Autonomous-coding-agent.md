## Autonomous Agent Task Protocol — Obsidian Kanban

### Task Discovery

- Tasks live on the **"ServiceNow Tasks"** Kanban board.
- Pick tasks **only** from the **"To Do"** column.
- Task titles follow the format: `2026-02-28_XXX-<description>` where `XXX` is a monotonically increasing numeric identifier.

### Task Selection Rules

1. **Always pick the lowest available number** in "To Do" that is higher than your last completed task.
2. Numbers are **not necessarily sequential** — gaps are expected because multiple agents draw from the same board.
3. **Move the task to "In Progress"** immediately upon picking it. This is your lock mechanism — if a task is already "In Progress", skip it.
4. Only work on **one task at a time**. Finish or explicitly abandon before picking the next.

### Working on a Task

5. Tasks are designed to be **self-contained**. Any dependencies (other commits, prior tasks) will be explicitly listed in the task body.
6. If a task description is incomplete or ambiguous:
    - **Do not block on user input** unless the ambiguity could lead to fundamentally wrong work.
    - Make the best reasonable decision, **document the decision in the task**, and proceed.
    - Update the task body so that _any_ agent could pick it up cold and understand the full context.
    - Only pause for user interview when the decision is **irreversible or high-risk** (e.g., schema changes, public API contracts, destructive migrations).

### Git Discipline

7. **You are not the only one touching the codebase.** Expect unrelated changes between your tasks.
8. Before committing, **review your diff carefully**. Use `git add -p` (hunk-level staging) to include **only your changes**.
9. Never commit unrelated modifications, formatting drift, or merge artifacts.
10. Commit messages should reference the task ID (e.g., `feat(2026-02-28_003): implement incident table schema`).

### Task Completion & Handoff

11. When done, **move the task to "Review"** — never to "Completed". Only the user moves tasks to "Completed".
12. Before moving to "Review", update the task with:
    - **What was done** — brief summary of changes.
    - **Commit hash(es)** — so reviewers can find the exact work.
    - **Files touched** — quick reference list.
    - **Decisions made** — any judgment calls and reasoning.
    - **Known limitations or follow-ups** — anything a downstream agent or reviewer should be aware of.
    - **How to verify** — steps or commands to confirm the work is correct.
13. The goal is that **another agent (or the user) can review, understand, and continue** without needing to ask you anything.

### Edge Cases & Failure Modes

14. **If you realize mid-task that the task is blocked** by incomplete prior work: document what's missing, move the task back to "To Do" with a note, and pick the next available task.
15. **If a task turns out to be larger than expected**: complete what you can, document the remaining scope in the task, and move to "Review" with a clear note that it's partial. Don't silently leave work half-done.
16. **If two agents accidentally pick the same task**: the one who moved it to "In Progress" first wins. The other should abandon and pick the next task. If unclear, check git blame — the agent whose commits are already pushed owns it.
17. **Merge conflicts**: resolve only for your own changes. If someone else's work conflicts with yours, flag it in the task and move to "Review" with context rather than guessing at their intent.

### What NOT to Do

- Don't refactor code outside your task scope.
- Don't "fix" things you notice unless it's part of your task.
- Don't skip the hunk-level review before committing.
- Don't leave a task in "In Progress" indefinitely — either finish it, return it to "To Do", or move it to "Review" as partial.
