Check out the branch `jagadeesh/common-components-create-case-sample`

- Study how they have structured their code-components, state, etc
- We need to make sure we also align with their structure in this PR

Their PR is titled `State, Form elements, case create page` with the description below:

```
- Common form elements ui/components/form-*
- Zustand store
- Sample state for initial values
- Case create page
```

---

I am more interested in the component structure and the stores, the rest not so much.



## Outcome

Only concrete misalignment was store file naming: `knowledge-store.ts` → `useKnowledgeStore.ts` to match their `use<Domain>Store.ts` convention. Everything else (export names, store location, UI components) was already aligned or complementary.

Commit: 1fb72a2