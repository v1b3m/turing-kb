Check what the branch `emmanuel-ui-bugs` is trying to bring into `ots_dev`.

Below is the PR description:

```
## Summary

This PR fixes core playlist consistency issues across the app, so UI state now reflects user actions immediately and predictably.

- Playlist cards now stay in sync with live viewer state for playlists, so counts no longer drift from reality after actions like like/save/download. Previously, if you liked a video, the count on the 'Liked videos' card on the playlist card remained the same
- Card cover behaviour for playlists was improved to show the most recently added item, matching expected YouTube-like recency semantics.
- Empty playlists are filtered out on the playlists page, reducing noise and preventing misleading cards.
- Watch playlist panel ordering is now recency-first across queued playlist types (not just Watch Later), so newest additions appear first consistently.
- Panel playlist selection logic was refactored into a dedicated helper for clarity and maintainability, reducing branching complexity in the page component.

## Why this is significant

- **User trust:** playlist counts/covers now reliably reflect current actions in real time.
- **Behavioural consistency:** recency ordering aligns cards and watch-panel queues with expected product behaviour.
  
```

## Review handoff - 2026-05-05 09:35

Status: Handoff / Review

Base: `ots_dev` at `7f226fbd`
Branch reviewed: `emmanuel-ui-bugs` at `c6f9c27e`
Local-only patch commit: `0ac151726483c7d44f0f3c5c274e7e1b40cbac34` (do not push)

Scope reviewed:

- `app/feed/playlists/page.tsx`
- `app/results/page.tsx`
- `app/watch/page.tsx`
- `src/components/layout/Sidebar.tsx`
- `src/components/playlist/PlaylistCard.tsx`
- `src/lib/local-playlists.ts`
- `src/lib/playlist-watch-url.ts`

Summary:

The branch is trying to make playlist cards and watch playlist panels use live viewer state for system playlists (`LL`, `WL`, `PPSV`), hide empty playlists on `/feed/playlists`, and display/watch playlist items in recency-first order. Most of the change is in `src/lib/local-playlists.ts`, `app/feed/playlists/page.tsx`, `src/components/playlist/PlaylistCard.tsx`, and `app/watch/page.tsx`.

Findings:

1. False positive for the current UI: Downloads recency was flagged for playlist cards and card links.
   - `stores/useViewerStore.ts:2157` stores new downloads at the front of the array.
   - `app/feed/playlists/page.tsx:52` and `src/components/playlist/PlaylistCard.tsx:76` were sorting downloads newest-first before passing IDs into playlist helpers.
   - `src/lib/local-playlists.ts:379` then selects `videos[videos.length - 1]` for `watchUrlForLocalPlaylist`, and `src/components/playlist/PlaylistCard.tsx:104` does the same for card cover preview.
   - `app/watch/page.tsx:60` orders the Downloads panel newest-first.
   - However, `PPSV` is not in `src/data/playlists.json`, is not seeded into default local playlists, and `/feed/playlists` does not normally render a Downloads playlist card. The visible Downloads UI is `/feed/downloads`, where newest downloads already appear at the top.
   - Local patch `0ac1517` was created before confirming render reachability and should not be pushed.

Decision:

- Merge the original branch as-is. The flagged Downloads-card issue does not block the current visible UI.

Verification:

- `git diff --check ots_dev...emmanuel-ui-bugs` passed.
- `npm run typecheck` failed before validating the branch because `@radix-ui/react-popover` is declared in `package.json` but missing from `node_modules`.
- `npm run lint` failed with repo-wide existing lint/prettier issues; examples include `app/[channelName]/page.tsx`, `app/localStorage/page.tsx`, `app/results/page.tsx`, `app/shorts/[id]/page.tsx`, and many other unrelated files.
- After patch: `git diff --check` passed.
- After patch: `npx eslint app/feed/playlists/page.tsx src/components/playlist/PlaylistCard.tsx` passed.
- After patch: `npm run typecheck` still failed on the same missing `@radix-ui/react-popover` dependency.

Limitations:

- Local-only production code was changed in `app/feed/playlists/page.tsx` and `src/components/playlist/PlaylistCard.tsx`, but that patch commit should not be pushed.
- I did not move this note to Completed.
