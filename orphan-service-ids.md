# Orphan Service IDs Bug

## Bug

Some `bookingId`s from `service-list.json` returned **zero taskers** on the recommendations page (`/book/[taskId]/recommendations`). Users selecting those services would see an empty results page.

## Root Cause

`service-list.json` contains entries at two levels of specificity:

- **Parent skills** that taskers actually have (e.g. `assemble-furniture`, `deep-cleaning`, `local-movers`)
- **Sub-services** that are more specific subtypes (e.g. `chair-assembly`, `airbnb`, `help-moving`)

Taskers only have `skillIds` matching the parent skills. The `assign-service-ids.ts` script mapped `skillIds` to `bookingIds` to produce each tasker's `serviceIds` array. But sub-service `bookingId`s had no corresponding skill on any tasker, so queries for those IDs returned nothing.

There were **63 orphan sub-services** across all categories with this problem.

## Fix

**Commit:** `50c9896` — *"Map orphan sub-services to parent skills and filter taskers by location"*

Added an `ORPHAN_MAPPING` dictionary in `assign-service-ids.ts` that maps each orphan sub-service slug to its parent skill. During serviceId assignment, taskers who have the parent skill also receive the `bookingId`s of all mapped orphan sub-services.

### How it works

1. Build `skill -> bookingId` map from `service-list.json`
2. For each tasker, assign `bookingId`s for their `skillIds` (direct mapping)
3. For each orphan in `ORPHAN_MAPPING`, look up its `bookingId` and associate it with the parent skill
4. For each tasker, if they have a parent skill, also add all orphan `bookingId`s for that parent to their `serviceIds`

### Mapping (63 entries)

| Category | Orphan Sub-service | Parent Skill |
|---|---|---|
| Furniture Assembly | `chair-assembly`, `couch-assembly`, `table-assembly`, `wardrobe-assembly`, `furniture-disassembly` | `assemble-furniture` |
| Moving | `help-moving`, `full-service-movers` | `local-movers` |
| Moving | `in-home-furniture-movers`, `one-piece-of-furniture`, `rearranging-furniture` | `furniture-movers` |
| Moving | `pool-table-movers` | `heavy-lifting` |
| Moving | `appliance-removal`, `furniture-removal` | `junk-removal` |
| Moving | `unpacking` | `packing` |
| Cleaning | `airbnb`, `disinfecting-service`, `move-out`, `spring-cleaning` | `deep-cleaning` |
| Cleaning | `one-time` | `house-cleaning` |
| Cleaning | `mobile-car-wash` | `pressure-washing` |
| Yardwork | `landscaping`, `produce-gardening`, `urban-gardening`, `vacation-plant-watering` | `gardening` |
| Yardwork | `lawn-care`, `lawn-fertilizer` | `lawn-mowing` |
| Yardwork | `gutter-cleaning` | `leaf-removal` |
| Yardwork | `patio-cleaning`, `hot-tub-cleaning` | `pressure-washing` |
| Yardwork | `shed-maintenance` | `deck-restoration` |
| Winter | `residential-snow-removal`, `sidewalk-snow-removal`, `winter-yardwork` | `snow-removal` |
| Winter | `ac-winterization`, `pipe-insulation` | `water-heater-maintenance` |
| Winter | `storm-door-installation` | `window-winterization` |
| Winter | `winter-deck-maintenance` | `deck-restoration` |
| Winter | `tree-removal` | `tree-trimming` |
| Shopping/Delivery | `baby-food-delivery`, `dog-food-delivery` | `grocery-shopping` |
| Shopping/Delivery | `breakfast-delivery`, `coffee-delivery`, `shipping` | `delivery-service` |
| Shopping/Delivery | `wait-for-delivery` | `waiting-in-line` |
| Office | `conference-room` | `computer-help` |
| Office | `movers` | `local-movers` |
| Office | `office-furniture-moving` | `furniture-movers` |
| Office | `office-interior-design` | `organization` |
| Office | `office-mounting` | `mounting-all` |
| Office | `organization-setup` | `organization` |
| Office | `supply-snack-delivery` | `delivery-service` |
| Holidays | `christmas-light-installation` | `christmas-decorator` |
| Holidays | `toy-assembly` | `assemble-furniture` |
| Baby Prep | `need-something-else-cleaned` | `house-cleaning` |
| Baby Prep | `organize-a-room` | `home-organization` |
| Handyman | `appliance-repairs`, `install-air-conditioner` | `general-handyman` |
| Handyman | `cabinet-installation` | `carpentry-construction` |
| Online/PA | `data-entry-clerk` | `computer-help` |
| Online/PA | `virtual-assistant` | `personal-assistant` |
| Online/PA | `interior-design` | `home-organization` |
| Online/PA | `prescription-delivery` | `pick-up-delivery` |
| Contactless | `yard-work-removal` | `gardening` |

## Key Files

- `src/data/assign-service-ids.ts` — the `ORPHAN_MAPPING` dict and assignment logic
- `src/data/service-list.json` — source of all `bookingId` / `serviceSlug` pairs
- `src/data/taskers/taskers-{1..4}.json` — chunked tasker data with assigned `serviceIds`

## Notes

- If new sub-services are added to `service-list.json`, they need a corresponding entry in `ORPHAN_MAPPING` or they'll hit the same bug
- The same commit also added location-based filtering to the recommendations page (filters by city from the booking store)
