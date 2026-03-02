# Knowledge Article Detail Page

## Description
Implement the detail page at `/knowledge/[id]` that shows the full KB article — metadata fields, article body, action buttons, related links, and bottom tabs — matching the ServiceNow form layout.

Parent task: [[2026-02-26_knowledge-articles-table]]

## Progress
- [x] Add per-article detail fields to `knowledge-articles.json` (knowledgeBase, articleType, workflowState, published, validTo, articleBody, etc.)
- [x] Create `/knowledge/[id]/page.tsx` with header, metadata grid, article body, tabs
- [x] Update list page Number column link to route to detail page
- [ ] Tweak metadata form layout to pixel-match ServiceNow screenshots
- [ ] Tweak header bar styling
- [ ] Tweak tab bar styling
- [ ] Tweak article body rendering
- [ ] Tweak Related Links section
- [ ] Tweak action buttons styling
- [ ] Verify all fields match the reference HTML

## Decisions
- Using row index as the route param (`/knowledge/[idx]`) since KB numbers repeat across versions
- `dangerouslySetInnerHTML` for article body — data is from our own JSON, not user input
- KB99999999 gets the full article body from `pg/knowledge-item.html`; others get placeholder text

## Outcome