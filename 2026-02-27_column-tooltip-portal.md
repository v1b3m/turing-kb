# Column Options Tooltip: Portal Above

## Description
The 'Column options' tooltip on table column headers currently renders below the icon because the scroll container's `overflow: auto` clips content above the sticky thead. Ideally it should render above the icon like the reference design.

## Solution
Use a React portal to render the tooltip outside the overflow container, positioning it absolutely relative to the icon button. This lets it appear above without being clipped.

## Priority
Low — only update if we get complaints about the current below-icon positioning.