# ServiceNow Knowledge Center Clone Reference

Date: 2026-04-20
Source: `https://dev218807.service-now.com/now/knowledge-center/record/kb_knowledge/-1/params/query/`

## Summary

Inspected the live ServiceNow "Create New Knowledge" page in Chrome using CDP.
This page is the reference UI we plan to clone.

## Observed Layout

- Standard ServiceNow shell with a dark teal global header and a left navigation rail.
- Light blue workspace tab strip below the global header.
- Page header with `Create New Knowledge` on the left and a `Save` button on the right.
- Main workspace split into three columns:
  - left metadata form
  - center document builder canvas
  - right blocks/settings panel

## Design Notes

- Enterprise application styling with compact spacing and strong alignment.
- White content surfaces over pale blue and light gray structural chrome.
- Typography appears to use ServiceNow UI fonts such as `Lato`, `Source Sans Pro`, and `Cabin`.
- Minimal decorative effects; emphasis is on layout, hierarchy, and utility.

## Clone-Critical Details

- Left column contains a two-column metadata form under a `Knowledge` section.
- Center column contains a large white canvas inside a light gray builder workspace.
- Right panel contains `Blocks` and `Settings` tabs with a two-column block picker grid.
- The layout depends heavily on fixed rails, separators, and column proportions rather than visual ornament.

## Note

`AGENTS.md` was not present in this repository at the time of writing, so this entry follows the existing repository convention of dated markdown notes and uses the requested `changes/2026-04-20/` location.
