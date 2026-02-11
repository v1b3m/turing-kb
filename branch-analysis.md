# Remote Branch Analysis

_Last updated: 2026-01-31_

## Summary

14 remote branches. 7 are fully merged into main (0 commits ahead) and can be deleted. 6 have unmerged work.

## Active Branches (unmerged commits)

| Branch                           | Commits Ahead | Last Activity | Description                                    |
| -------------------------------- | ------------- | ------------- | ---------------------------------------------- |
| `fb-furniture-assembly`          | 9             | 2025-11-14    | Payment and recommendations page updates       |
| `feature/details_page_component` | 4             | 2025-10-24    | Reusable details page component                |
| `feature/details_page`           | 3             | 2025-10-24    | Header with progress bar, steps made optional  |
| `Electrical-Work`                | 2             | 2025-11-13    | InfoBanner styling, moving page implementation |
| `fb-cleaning`                    | 2             | 2025-11-17    | Cleaning feature, font-size updates            |
| `feature/landscape`              | 1             | 2025-10-23    | Landscape task details page                    |

## Stale / Merged Branches (can be deleted)

These branches have 0 commits ahead of main:

| Branch | Last Activity | Notes |
|--------|---------------|-------|
| `buil-error-configs` | 2025-11-17 | Merged via PR #12 |
| `Heavy-Lifting-&-Loading` | 2025-11-06 | Merged or rebased |
| `TV-Mounting` | 2025-10-30 | Merged or rebased |
| `feature/navbar-pages` | 2025-10-23 | Merged or rebased |
| `feature/recommendations` | 2025-10-25 | Merged or rebased |
| `feature/tasks` | 2025-11-05 | Merged or rebased |
| `setup/mui-config` | 2025-10-23 | Merged or rebased |

## Cleanup Command

```bash
# Delete merged remote branches (review before running)
git push origin --delete buil-error-configs "Heavy-Lifting-&-Loading" TV-Mounting feature/navbar-pages feature/recommendations feature/tasks setup/mui-config
```
