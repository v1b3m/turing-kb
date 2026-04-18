
## Major Issue Management

### View Major Cases

View list of major cases affecting multiple customers

Status: Connected to backend API, but only through major-case-state filtering

Frontend usage:
- `frontend/src/components/case/CaseFormPage.tsx`
  Links users to the major-case list.
- `frontend/src/app/cases/page.tsx`
  Reads `sysparm_query` and filters the case list view.
- `frontend/src/hooks/cases/caseApi.ts`
  Supports the backend query params.
- `frontend/src/hooks/cases/useCases.ts`
  Passes list query params to the cases API.

Exact backend route used:
- `GET /api/v1/cases`

Exact frontend query patterns involved:
- Backend-supported API query:
  - `/api/v1/cases?skip=0&limit=250&major_case_state=accepted`
- UI navigation query used by the case form:
  - `/now/nav/ui/classic/params/target/sn_customerservice_case_list.do?sysparm_query=active%3Dtrue%5Emajor_case_state%3Daccepted%5EEQ`

Request body:
- None

Notes:
- This is wired to the real cases list API.
- The backend filter in use is `major_case_state=accepted`.
- The “affecting multiple customers” wording is not represented as a dedicated backend filter or request field.

### Link Case to Major Case

Associate individual case with a major issue

Status: Connected to backend API

Frontend usage:
- `frontend/src/stores/useCaseStore.ts`
  Calls the parent-case and related-case APIs.
- `frontend/src/components/case/CaseFormPage.tsx`
  Opens the major-case link dialogs and triggers the actions.
- `frontend/src/components/case/MajorCaseDialogs.tsx`
  UI for major-case selection.

Primary backend route used for major-case association:
- `PUT /api/v1/cases/{case_id}/parent`

Exact request body used:

```json
{
  "parent": "CS0001234"
}
```

Frontend call site:
- `updateCaseParentApi(sys_id, { parent: nextParent || null })`

Alternative generic case-linking route also present in frontend:
- `POST /api/v1/cases/{case_id}/related-cases/{related_case_id}`

Request body:
- None

Notes:
- The actual major-case association pattern in the current implementation is parent-case linkage, not a separate “major issue” entity.
- The generic related-case API is also wired in the frontend, but the major-case flow is primarily the `parent` update route.


