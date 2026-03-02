**Ref:** main
**Depends:** 2026-03-01_001_more-ui-updates

### Context
There's a UI mismatch in the favorites popover from the ref

**Ref:**

![[Screenshot from 2026-03-02 00-01-43.png]]

**Our implementation:**

![[Pasted image 20260302000305.png]]

**Notes:**

1. Size
	1. The original is a 320 x 235 element
	2. Our implementation is 384 x 256.4
2. The original popover has an arrow touching the anchor/ref element
3. The original popover has the same bg as the header i.e that dark color with the white text
4. The popover inputs have bg rgb(89, 116, 130)
5. "More" text has color  rgb(189, 222, 231)
6. "Remove" button has color rbg(238, 118, 131), this same color is used by the outline border
7. "Done" text is white with outline styling
8. The outline buttons has 0.8px border width
9. The "required" indicator is an svg and is aligned with the text, white in color
10. Clicking the top-level default dropdown reveals a search input

**Required indicator**

```svg
<svg role="img" class="now-icon -sm" aria-label="Required" viewBox="0 0 12 12"><path d="M5.5 1a.5.5 0 0 1 .5.5v3.598l3.235-2.022a.5.5 0 1 1 .53.848L6.443 6l3.322 2.076a.5.5 0 0 1-.53.848L6 6.902V10.5a.5.5 0 0 1-1 0V6.902L1.765 8.924a.5.5 0 0 1-.53-.848L4.557 6 1.235 3.924a.5.5 0 1 1 .53-.848L5 5.098V1.5a.5.5 0 0 1 .5-.5"></path></svg>
```

**Top level dropdown**

![[Pasted image 20260302001238.png]]
### Handoff

| Field | Details |
|---|---|
| **What was done** | Restyled CreateFavoriteDialog from a light Dialog/modal to a dark-themed popover matching the reference. Applied dark background (#0d3849), arrow indicator, SVG required asterisk (white), input bg rgb(89, 116, 130), "More" text rgb(189, 222, 231), "Remove" button rgb(238, 118, 131) with 0.8px outline, "Done" button white with 0.8px outline, custom location dropdown with search input, boxed X close icon, 320px width. |
| **Commit hash(es)** | `aced1f9` |
| **Files touched** | `src/components/knowledge/CreateFavoriteDialog.tsx` |
| **Decisions made** | Converted from Radix Dialog to a custom positioned div (fixed overlay with absolute popover) instead of Radix Popover, because the trigger is in the HamburgerMenuDropdown and the popover needs to appear centered below the header rather than anchored to a specific element. Added outside-click and Escape key handlers manually. |
| **Known limitations** | The popover is always centered horizontally at the top; it doesn't anchor to a specific star button. Only one location option exists ("Top level (default)") — the search dropdown is functional but has only one item. |
| **How to verify** | 1. Run `npm run build` — passes. 2. Click hamburger menu → "Create Favorite" → verify dark popover appears with arrow, dark bg, white text, styled inputs, correct button colors, SVG asterisk. 3. Click the location dropdown — verify search input appears with checkmark on selected option. 4. Test Remove/Done buttons, outside click, and Escape to close. |
