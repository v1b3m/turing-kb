# Configurable Filter Breadcrumb Links

## Description
Extract the filter breadcrumb row from KnowledgeArticlesHeader into a reusable FilterBreadcrumb component that accepts an array of link segments via props.

## Progress
- [x] Create FilterBreadcrumb component in src/components/ui/filter-breadcrumb.tsx
- [x] Update KnowledgeArticlesHeader to use FilterBreadcrumb
- [x] Build passes

## Decisions
- Placed in `src/components/ui/` since it's a generic UI component, not knowledge-specific
- Props: `segments: { label: string; tooltip?: string }[]` and optional `defaultTooltip`
- Keeps the same visual behavior (chevron separators, hover tooltips)

## Outcome
Created reusable FilterBreadcrumb component. KnowledgeArticlesHeader now uses it instead of inline JSX.