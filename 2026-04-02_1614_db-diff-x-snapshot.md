# DB Diff & Snapshot Routes — Implementation Plan

## Description
Wire the turing_backend_common.db_snapshot.router factory into the ServiceNow backend to expose database snapshot, diff, schema, session status, and drop endpoints.

## Current State

The heavy lifting was **already done**:

- **backend/app/api/v1/endpoints/db_snapshot.py** — fully wired with ServiceNow-specific config
- **shared-libs/turing_backend_common/db_snapshot/** — shared library (router.py, diff.py, utils.py)
- **backend/app/db/run_router.py** — provides run_db_exists, get_run_db_name, drop_run_database, ensure_run_database
- **backend/app/db/registry.py** — provides _admin_engine, ensure_registry_table

## Endpoints Provided

| Endpoint | Method | Purpose |
|---|---|---|
| /db_snapshot | GET | Full snapshot of all tables/rows in a run DB |
| /db_diff | GET | Seed vs run DB diff — added/modified/deleted rows |
| /db_schema | GET | Static schema metadata (tables, columns, types) |
| /session_status | GET | Check if session DB is active (90-min timeout) |
| /get_session_id | GET | Exchange JWT token for session_id |
| /db_drop | DELETE | Destroy a run database (with protection for seed DBs) |

## Implementation Steps

- [x] **Register the router** in backend/app/api/v1/__init__.py — added db_snapshot import and include_router call
- [x] **Verify database_schema.json** — no file exists; /db_schema falls back to seed DB introspection (acceptable)
- [x] **Verify ignored tables** — alembic_version is the only relevant one; api_logs/sessions/audit_logs/prompt_tasks don't exist in ServiceNow schema (harmless no-ops)
- [ ] **Smoke test** — start backend and hit /db_schema, /db_snapshot, /db_diff

## Config Reference

| Parameter | Value |
|---|---|
| registry_table_name | servicenow_run_registry |
| template_db_name | servicenow_seed (from env) |
| db_inactive_minutes | 90 |
| ignore_tables | alembic_version, api_logs, sessions, audit_logs, prompt_tasks |

## Decisions
- No database_schema.json needed — seed DB introspection is sufficient for /db_schema
- Kept the ignore_tables set from spothub as-is — non-existent tables are simply skipped

## Outcome
Router registered in backend/app/api/v1/__init__.py (line 19 import, line 40 include_router). Single file change — all 6 endpoints now live under /api/v1/.