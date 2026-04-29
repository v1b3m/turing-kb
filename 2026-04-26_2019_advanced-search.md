Use this manual QA guide against the running app, mainly on `/results` and the shared header search.

**Setup**
- Start from `/`
- Use the header search input
- Submit with Enter so the app lands on `/results?search_query=...`
- For each case, verify:
  - the input shows the submitted query
  - results load
  - returned videos actually match the intended operator/filter
  - clearing/editing the query still works after submission

**Core operators**
- Basic text:
  - `nextjs`
  - Expect the Next.js video to appear
- Exact phrase:
  - `"RTX 5090"`
  - Expect the RTX 5090 review result
- Required terms:
  - `+react +typescript`
  - Expect React + TypeScript content only
- Excluded terms:
  - `react -typescript`
  - Expect React-related results without the TypeScript video
- OR:
  - `react OR nextjs`
  - Expect results for either side, including both known fixtures
- Hashtag:
  - `#space`
  - Expect space-tagged videos
- Title-only:
  - `intitle:"React"`
  - Expect results whose titles contain `React`
- Date lower bound:
  - `nextjs after:2026-04-01`
  - Expect only results published on or after April 1, 2026
- Date upper bound:
  - `nextjs before:2026-04-10`
  - Expect only results published on or before April 10, 2026

**Compound queries**
- Phrase + exclude:
  - `"review" -iphone`
- OR + title:
  - `intitle:"React" OR nextjs`
- Hashtag + date:
  - `#space after:2025-01-01`
- Required + exclude + phrase:
  - `+react -"Next.js 15"`

For each, verify the returned set still obeys every part of the query.

**Results-page filters**
Use the chip bar on `/results` after a query like `nextjs` or `review`.

- `All`
  - baseline behavior
- `Videos`
  - should exclude live-only results
- `Live`
  - should return only live videos
- `Shorts`
  - should constrain to short-duration results
- `Today`
  - should limit to very recent uploads
- `This week`
  - should limit to the last 7 days
- `Channels`
  - should switch to channel results instead of video rows

Verify that switching filters changes the result type/set correctly and does not break the query in the input.

**Sorting and structured filters**
These are API-backed today even if not all have dedicated UI controls. Test them directly in the URL bar on `/api/search/videos` or by editing query params if you want route-level verification.

Useful examples:
- `/api/search/videos?q=tech&sort=viewCount`
- `/api/search/videos?q=tech&sort=uploadDate`
- `/api/search/videos?q=review&feature=4k`
- `/api/search/videos?q=&duration=short`
- `/api/search/videos?q=&duration=medium`
- `/api/search/videos?q=&duration=long`
- `/api/search/videos?q=&type=live`
- `/api/search/videos?q=&uploadDate=week`
- `/api/search/videos?q=&uploadDate=month`

What to verify:
- `sort=viewCount` is descending by views
- `sort=uploadDate` is newest first
- `feature=4k` only returns 4K-tagged items
- `type=live` only returns live items
- duration buckets are respected

**Header suggestions**
From the header input:
- type `rea`
- verify suggestions appear
- click a suggestion
- verify it navigates correctly and results match
- verify recent searches and server suggestions can coexist
- remove a recent search and confirm it disappears

**Input-state regressions**
These are worth checking carefully:
- Submit a query, select all text in the header, press Backspace
  - field should stay empty
- Type a replacement query immediately after clearing
  - old query should not come back
- Blur and refocus the input on `/results`
  - it should not repopulate unexpectedly while you are editing
- Use browser back/forward between two different searches
  - input and results should stay in sync

**No-results cases**
Try:
- `zzzzzzzzzz`
- `intitle:"definitelynotrealquery"`
- `#notarealtag before:2000-01-01`

Verify:
- no-results UI appears
- page does not crash
- editing the query recovers normally

**Known good fixture queries**
If you want a compact smoke set:
- `nextjs`
- `"RTX 5090"`
- `+react +typescript`
- `react -typescript`
- `react OR nextjs`
- `#space`
- `intitle:"React"`
- `nextjs after:2026-04-01`
- `nextjs before:2026-04-10`
- `review`

If you want, I can turn this into a checkbox QA doc in the repo, for example `docs/manual-advanced-search-qa.md`.