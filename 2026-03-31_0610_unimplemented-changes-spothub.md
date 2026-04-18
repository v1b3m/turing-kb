# Unimplemented Route Pattern From SpotHub

This note documents how SpotHub handles unimplemented pages by routing users to `/502`, and how to apply the same pattern in another app.

## Repo

SpotHub repo path for verification:

- `/Users/v1b3m/Dev/Turing/turing-spothub-aws`

## What `/502` means in this app

In SpotHub, `/502` is not a true backend `502 Bad Gateway` error page.

It is a deliberate placeholder route for:

- routes that are not part of the currently implemented flows
- disabled navigation items
- buttons and actions that exist in the UI but do not have working behavior yet
- catch-all routes that should not resolve to a real page

The page itself is a simple client-side component in:

- `frontend/src/app/(app)/502/page.tsx`

Behavior:

- sets `document.title` to `502 - Feature not available | SpotHub`
- renders a static “Feature Not Available” UI
- offers `Go Back` and `Back to Home` buttons

Reference:

- `frontend/src/app/(app)/502/page.tsx:8`
- `frontend/src/app/(app)/502/page.tsx:11`
- `frontend/src/app/(app)/502/page.tsx:28`
- `frontend/src/app/(app)/502/page.tsx:45`
- `frontend/src/app/(app)/502/page.tsx:52`

## How users get routed to `/502`

SpotHub uses multiple routing layers, not just one.

### 1. Global route guard in middleware

The main enforcement point is `frontend/src/middleware.ts`.

It defines an explicit allowlist in `ALLOWED_ROUTE_PATTERNS` and then redirects every non-allowed route to `/502`.

Core flow:

1. request comes in
2. if pathname is `/502`, allow it to avoid redirect loops
3. if pathname is in the allowlist, allow it
4. if pathname matches an allowed dynamic base, allow it
5. otherwise redirect to `/502`

References:

- `frontend/src/middleware.ts:21` for `ALLOWED_ROUTE_PATTERNS`
- `frontend/src/middleware.ts:71` for `isAllowedRoute`
- `frontend/src/middleware.ts:83` for `allowedDynamicBases`
- `frontend/src/middleware.ts:102` for `middleware`
- `frontend/src/middleware.ts:106` for the redirect-loop guard
- `frontend/src/middleware.ts:116` for the fallback redirect

Important detail: this is a whitelist model, not a blacklist model.

That means:

- only explicitly approved routes work
- any route forgotten in the whitelist will automatically go to `/502`
- this is safer when only a subset of the app is meant to be usable

### 2. Disabled navigation items push directly to `/502`

The sidebar also routes users to `/502` when they click disabled items.

File:

- `frontend/src/components/hubspot-sidebar.tsx`

Relevant behavior:

- top-level disabled nav item: `router.push("/502")`
- disabled sub-item in flyout menu: `router.push("/502")`

References:

- `frontend/src/components/hubspot-sidebar.tsx:358`
- `frontend/src/components/hubspot-sidebar.tsx:359`
- `frontend/src/components/hubspot-sidebar.tsx:361`
- `frontend/src/components/hubspot-sidebar.tsx:421`
- `frontend/src/components/hubspot-sidebar.tsx:426`

This is a UX improvement over relying only on middleware, because the click immediately goes to the placeholder instead of attempting a route first.

### 3. Catch-all pages redirect to `/502`

Some route files hard-redirect to `/502`.

Example:

- `frontend/src/app/(app)/settings/[...slug]/page.tsx`

This means any settings path caught by that file goes straight to `/502`.

Reference:

- `frontend/src/app/(app)/settings/[...slug]/page.tsx:3`
- `frontend/src/app/(app)/settings/[...slug]/page.tsx:4`

This is useful when a route segment exists structurally, but most of its pages are intentionally unavailable.

### 4. Reusable “coming soon” click wrapper

There is a reusable component that intercepts clicks and redirects to `/502`.

File:

- `frontend/src/components/ui/coming-soon.tsx`

Behavior:

- clones child element(s)
- prevents default click behavior
- stops propagation
- routes to `/502`

References:

- `frontend/src/components/ui/coming-soon.tsx:11`
- `frontend/src/components/ui/coming-soon.tsx:14`
- `frontend/src/components/ui/coming-soon.tsx:17`

This is useful for buttons, cards, and links that should look present but stay non-functional.

### 5. Optional disabled-route registry

SpotHub also has a central disabled-route registry in:

- `frontend/src/lib/constants/disabled-routes.ts`

This file groups disabled routes and exposes `isDisabledRoute(pathname)`.

References:

- `frontend/src/lib/constants/disabled-routes.ts:9`
- `frontend/src/lib/constants/disabled-routes.ts:52`
- `frontend/src/lib/constants/disabled-routes.ts:66`

In the current codebase, the middleware relies on the whitelist in `middleware.ts`, while this file acts more like a maintainable registry for UI-level checks and future consolidation.

## How whitelisting works

SpotHub whitelists routes in two ways.

### Exact route allowlist

`ALLOWED_ROUTE_PATTERNS` contains exact paths such as:

- `/`
- `/502`
- `/login`
- `/crm/contacts`
- `/crm/companies`
- `/crm/deals`
- `/crm/tasks`
- `/crm/orders`
- `/reporting/reports`

Reference:

- `frontend/src/middleware.ts:21`

### Dynamic base allowlist

For detail pages, it allows any route starting with specific prefixes, for example:

- `/crm/contacts/`
- `/crm/companies/`
- `/crm/deals/`
- `/crm/orders/`
- `/reporting/reports/`

Reference:

- `frontend/src/middleware.ts:83`

This is how a detail page like `/crm/contacts/123` is allowed without listing every possible concrete URL.

## How to implement this in another app

This pattern is best when:

- only part of the app is production-ready
- design wants unfinished sections visible in nav
- you need a controlled placeholder instead of broken routes or 404s
- you want to ship limited flows while keeping the broader IA visible

## Recommended structure

### 1. Create a placeholder route

Example for Next.js App Router:

```tsx
// app/502/page.tsx
"use client"

import { useRouter } from "next/navigation"

export default function FeatureUnavailablePage() {
  const router = useRouter()

  return (
    <main>
      <h1>502</h1>
      <h2>Feature Not Available</h2>
      <p>This feature has not been implemented yet.</p>
      <button onClick={() => router.back()}>Go Back</button>
      <button onClick={() => router.push("/")}>Back to Home</button>
    </main>
  )
}
```

If you want to avoid semantic confusion, consider using `/coming-soon` or `/unavailable` instead of `/502`.

SpotHub uses `/502`, but that name can be misleading because it resembles a real infrastructure failure.

### 2. Add a middleware whitelist

Example:

```ts
// src/middleware.ts
import type { NextRequest } from "next/server"
import { NextResponse } from "next/server"

const ALLOWED_ROUTES = [
  "/",
  "/502",
  "/login",
  "/dashboard",
  "/projects",
]

const ALLOWED_DYNAMIC_BASES = [
  "/projects/",
]

function isAllowedRoute(pathname: string) {
  if (ALLOWED_ROUTES.includes(pathname)) return true
  return ALLOWED_DYNAMIC_BASES.some((base) => pathname.startsWith(base))
}

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl

  if (pathname === "/502") {
    return NextResponse.next()
  }

  if (isAllowedRoute(pathname)) {
    return NextResponse.next()
  }

  const url = request.nextUrl.clone()
  url.pathname = "/502"
  return NextResponse.redirect(url)
}

export const config = {
  matcher: [
    "/((?!api|_next/static|_next/image|favicon.ico).*)",
  ],
}
```

Implementation rules:

- always allow the placeholder route itself
- always exclude API and static asset paths from the matcher
- separate exact routes from dynamic route bases
- keep the whitelist short and intentional

### 3. Mark disabled nav items explicitly

In the navigation config, add a `disabled` flag.

Example:

```ts
const navigationItems = [
  { title: "Home", href: "/", disabled: false },
  { title: "Analytics", href: "/analytics", disabled: true },
]
```

Then route disabled clicks to the placeholder:

```ts
const handleItemClick = (item: NavItem) => {
  if (item.disabled) {
    router.push("/502")
    return
  }

  router.push(item.href)
}
```

This gives immediate, intentional UX and keeps disabled items visible.

### 4. Add a reusable wrapper for unfinished actions

Example:

```tsx
"use client"

import { cloneElement, ReactElement } from "react"
import { useRouter } from "next/navigation"

export function ComingSoon({ child }: { child: ReactElement }) {
  const router = useRouter()

  return cloneElement(child, {
    onClick: (e: React.MouseEvent) => {
      e.preventDefault()
      e.stopPropagation()
      router.push("/502")
    },
  })
}
```

Use this for:

- disabled buttons
- cards in feature galleries
- “coming soon” settings tabs
- toolbar actions with no implementation yet

### 5. Add catch-all redirects where appropriate

If a whole route branch is mostly unavailable, add a catch-all page:

```tsx
// app/settings/[...slug]/page.tsx
import { redirect } from "next/navigation"

export default function SettingsFallback() {
  redirect("/502")
}
```

Use this when:

- the route tree exists for IA consistency
- only a few concrete pages under that section are implemented
- the rest should collapse into a placeholder

## Routing hierarchy to copy

For another app, the clean version of the pattern is:

1. Middleware is the hard gate.
2. Navigation and buttons should redirect proactively for better UX.
3. Catch-all routes protect partially implemented route trees.
4. A shared wrapper handles unfinished actions consistently.

That combination gives both enforcement and good user experience.

## Suggested implementation checklist

1. Create the placeholder page.
2. Add the route to the middleware allowlist.
3. Create an exact-route allowlist.
4. Create dynamic-prefix allowlist rules.
5. Exclude API and static paths from middleware matching.
6. Add `disabled: true` support in navigation config.
7. Route disabled clicks to the placeholder.
8. Add a reusable `ComingSoon` wrapper for buttons and links.
9. Add catch-all redirect pages for mostly unimplemented route groups.
10. Decide whether to keep `/502` naming or use a clearer route like `/coming-soon`.

## Tradeoffs

Advantages:

- prevents accidental access to unfinished areas
- makes partial releases safer
- keeps IA visible without fake functionality
- centralizes route availability decisions

Risks:

- `/502` can be mistaken for a real platform error
- whitelist maintenance can become tedious if routes change often
- missing a legitimate route in the allowlist will silently send users to the placeholder
- middleware-only enforcement can be confusing if UI still looks fully active

## My recommendation for other apps

Use the same architecture, but rename the destination route unless you specifically want the fake-error aesthetic.

Preferred naming:

- `/coming-soon`
- `/feature-unavailable`
- `/not-implemented`

Then keep the exact same mechanics:

- middleware whitelist
- disabled navigation redirects
- reusable “coming soon” click wrapper
- catch-all fallback routes

That preserves the implementation benefits while making the intent clearer to users and engineers.
