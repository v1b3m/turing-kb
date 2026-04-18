# Products & Assets API Integration Check

Repo checked: `/Users/v1b3m/Dev/Turing/ServiceNow`  
Branch checked: `amazon`  
Goal: verify whether the current frontend is already integrated with backend APIs for the features below, and identify where the wiring exists or where it falls back to frontend-local data.

## Summary

| Feature | Status | Notes |
| --- | --- | --- |
| View Installed Products | Yes | Implemented through the entitlements API. |
| View Assets | No | Current assets UI is driven by a persisted Zustand store, not a backend API. |
| Link Asset to Case | Partial | The case itself is saved through the cases API, but the asset picker reads from local asset store data rather than a backend assets API. |

## 1. View Installed Products

Status: Yes, already integrated with the API.

Interpretation:
- In this codebase, "installed products" is represented most clearly by entitlements.
- The entitlements list includes `product`, `account`, `asset`, `contract`, and consumer metadata, and the frontend entitlements page fetches it from the backend.

Frontend integration:
- List page uses `useAllEntitlementsQuery()` in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/entitlements/page.tsx:14`
- The page renders backend data directly into the list in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/entitlements/page.tsx:66`
- API hook points at `/api/v1/entitlements` in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/entitlements/useEntitlements.ts:10`
- Fetch logic builds query params and calls `getJson(...)` in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/entitlements/useEntitlements.ts:66`
- Primary fetch call is in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/entitlements/useEntitlements.ts:115`

Backend support:
- Entitlements router is mounted under `/api/v1/entitlements` in `/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/__init__.py:65`
- List endpoint is `GET /api/v1/entitlements` in `/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/entitlements.py:21`
- Query/filter logic is implemented in `/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/services/entitlement.py:22`
- Entitlement data model includes `product`, `account`, and `asset` in `/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/models/entitlement.py:16`

Conclusion:
- This feature is already e2e backend -> frontend.
- If "installed products" is expected to mean a different backend resource than entitlements, that separate resource does not appear to exist on the current branch.

## 2. View Assets

Status: No, not integrated with a backend API on the current branch.

Frontend behavior:
- Assets list page reads directly from `useAssetStore()` in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/assets/page.tsx:15`
- The data source is `savedAssets` from the store in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/assets/page.tsx:18`
- The page waits for persisted store hydration instead of awaiting an API query in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/assets/page.tsx:56`

Local persistence:
- Asset data is managed by a persisted Zustand store in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/stores/useAssetStore.ts:306`
- The store seeds default asset rows in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/stores/useAssetStore.ts:126`
- Persistence is explicitly configured with `LOCAL_STORAGE_KEYS.ASSETS` in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/stores/useAssetStore.ts:368`
- The storage key is `assets-storage` in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/stores/local-storage-keys.ts:25`

Backend status:
- I did not find a backend assets router, assets model, or `/api/v1/assets` endpoint.
- The backend search only turned up `asset` as a field on cases and entitlements, not as a standalone asset resource.

Conclusion:
- The frontend assets list is local/demo data today.
- This feature is not yet backend-integrated.

## 3. Link Asset to Case

Status: Partial.

What is integrated:
- The case model supports an `asset` field in `/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/models/case.py:43`
- The request and response schemas include `asset` in `/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/schemas/case.py:9`
- Case create/update endpoints accept the case payload through:
  - `POST /api/v1/cases` in `/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/cases.py:93`
  - `PUT /api/v1/cases/{case_id}` in `/Users/v1b3m/Dev/Turing/ServiceNow/backend/app/api/v1/endpoints/cases.py:104`
- Frontend create/update uses:
  - `createCase(...)` in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/cases/caseApi.ts:57`
  - `updateCase(...)` in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/cases/caseApi.ts:61`
- The form-to-request mapping sends `asset` in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/cases/mappers.ts:343`
- The exact payload mapping is `asset: form.asset || null` in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/hooks/cases/mappers.ts:357`
- The case form displays and updates the asset field in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/components/case/CaseFormPage.tsx:1637`

What is not integrated:
- The asset lookup used by the case form does not call a backend assets API.
- In the lookup popup, `source === 'case' && field === 'asset'` maps directly from `savedAssets` in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/lookup-popup/page.tsx:56`
- Those `savedAssets` come from the persisted local asset store in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/lookup-popup/page.tsx:34`
- The lookup explicitly rehydrates the persisted asset store in `/Users/v1b3m/Dev/Turing/ServiceNow/frontend/src/app/lookup-popup/page.tsx:38`

Conclusion:
- Saving the chosen asset on the case is API-backed.
- Choosing which asset to link is still frontend-local because there is no backend assets API backing the lookup.

## Overall Assessment

Current e2e state:
- Installed products: backend-integrated
- Assets list: frontend-local only
- Link asset to case: backend-integrated for persistence, frontend-local for asset selection

Main gap:
- A real asset resource is missing on the backend, and the frontend assets screens and asset lookup are still built on `useAssetStore` + localStorage.

Minimal backend/frontend work needed for full e2e:
1. Add a backend assets resource, likely `/api/v1/assets`.
2. Replace `useAssetStore` list/form reads with API hooks.
3. Update the case asset lookup popup to read from the assets API instead of `savedAssets`.
