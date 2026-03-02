**Ref:** main
**Depends:** none
### Context

While grouping articles by "Article ID", I noticed this blue bar that we should definitely get rid of.

![[Screenshot from 2026-03-01 23-47-55.png]]

### Acceptance Criteria
- [x] No "Show just this group" bar in article groups

### Constraints
- None

### Resources
- None

---

### Handoff

| Field | Details |
|---|---|
| **What was done** | Removed the blue "Show just this group" banner that appeared below each expanded group header in the knowledge articles table. |
| **Commit hash(es)** | `78caa95` |
| **Files touched** | `src/components/knowledge/KnowledgeArticlesTable.tsx` |
| **Decisions made** | The banner had no onClick handler and no functional purpose — pure deletion was appropriate. Merged `feat/knowledge-articles` into the worktree branch since the code only exists there. |
| **Known limitations** | None |
| **How to verify** | Navigate to Knowledge Articles, group by any field (e.g., "Article ID"), expand a group — the blue "Show just this group" bar should no longer appear. |
