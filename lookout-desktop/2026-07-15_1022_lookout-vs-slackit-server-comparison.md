---
tags:
  - lookout-desktop
  - slackit-desktop
  - cross-gym
  - server
  - comparison
  - fastapi
  - electron
  - mock-app
created: 2026-07-15T10:22:00+03:00
---

# LookOut vs SlackIt — Server Implementation Comparison

> [!summary] TL;DR
> Both projects share the same **two-process architecture** (Electron UI thin-client → headless state server on loopback) and the same **OSWorld-V2 deployment contract** (`MOCKAPP_STORE_DIR` / `MOCKAPP_CONFIG_PATH` env vars, `/var/lib/mockapps/<App>` paths, `verification_method=http_api`). Below that surface, **LookOut is a single mature FastAPI server with a real relational schema; SlackIt is a dual-stack project — a live Node `http` server + an in-tree FastAPI rewrite marked "retired" in its migration doc but not wired into `main.js`/`install.sh`.**

## Top-line alignment vs divergence

**Aligned (cross-gym pattern is real):**

1. Two-process model, port-bound to `127.0.0.1`, single env-var contract
2. Same OSWorld installer contract; same systemd (Linux) / scheduled task (Win) lifecycle
3. Same `John Doe` primary mailbox identity (`john.doe@example.com`); LookOut seed accounts and SlackIt's `DEFAULT_USER` agree
4. Same preconfig pipeline shape (`config_reader → config_validator → config_defaults → pickers → data_generator → seeder`)

**Diverge:**

- **LookOut = single FastAPI stack.** **SlackIt = two stacks** (live Node `server.js` + in-tree FastAPI `server/` package that's "retired now that parity is validated" per `server/docs/MIGRATION.md`, but `main.js`/`install.sh` still only invoke `server.js`).
- **Per-action HTTP routes:** LookOut has 14 verb routes + 8 bulk variants on emails (`commit b7b205f`); SlackIt Node has none; SlackIt FastAPI is converging on LookOut's shape.
- **Runtime schema:** LookOut's `schema_v2.dbml` is realized as `backend/app/schema.sql` and ORM models. **SlackIt's `schema_v2.dbml` is aspirational** — the runtime uses a single `KVStore` table holding JSON envelopes per collection.
- **Concurrency:** LookOut has two `threading.Lock`s (`_receive_lock`, `_auth_lock`) + IntegrityError retry; SlackIt Node has zero locks; SlackIt FastAPI has one `asyncio.Lock` in the triggers service.

> [!warning] Latent bug worth flagging
> SlackIt's `manifest.json` declares `verification_endpoint: "http://localhost:5070/api/v1/verify"` — the **FastAPI** path. But `install.sh` only invokes `server.js` (Node). At install time, the manifest endpoint points to a route the live server doesn't serve. Works in the harness only if the harness falls back to `/verify` at root, or if SlackIt's FastAPI rewrite gets wired into the lifecycle. LookOut has no such inconsistency.

---

## 1. Server stack

| Aspect | LookOut | SlackIt (live) | SlackIt (in-tree, unwired) |
|---|---|---|---|
| Stack | FastAPI + uvicorn | Node `http` + plain `sqlite3` | FastAPI + uvicorn + SQLAlchemy |
| Entrypoint | `backend/app/main.py` (841 lines) | `server.js` (940 lines) | `server/app/main.py` (`create_app()`) |
| Lifecycle handler | `@asynccontextmanager async def lifespan(app)` | Top-level `main()` sequential calls | `create_app()` + lifespan → `get_context().startup()` |
| Actually run by `main.js`? | ✓ | ✓ | ✗ |
| Migration doc | n/a | n/a | `server/docs/MIGRATION.md` says "retired now that parity is validated" — but `server.js` still depends on legacy services |

> [!note] Shared trick
> Both apps reuse Electron's bundled binary in Node mode for the trainer build. LookOut: spawns `backend/.venv/bin/python -m uvicorn` from the trainer asar. SlackIt: spawns `process.execPath` with `ELECTRON_RUN_AS_NODE=1` against `server.js`. Trainers need neither Python nor Node installed.

---

## 2. Process layout / lifecycle

**Identical:**

- `/health` probe before spawning (`lookout-desktop/main.js:104-162`, `slackit-desktop/main.js:103-105`).
- Single-instance lock on the Electron app.
- OSWorld deploy = systemd unit (Linux) / scheduled task (Win) running only the headless server; UI is launched on demand. **OSWorld's controller on port 5000 is untouched.**
- Trainer fallback: `ensureUserWritablePaths()` redirects `MOCKAPP_STORE_DIR` to `app.getPath('userData')/state-server-store` so no sudo needed.

**Differences:**

- LookOut sets `cwd = path.dirname(backendDir)` so `backend.app.main:app` resolves on sys.path; SlackIt has no Python sys.path concern.
- LookOut teardown: `SIGTERM` POSIX / `taskkill /t /f` Win (`main.js:164-176`). SlackIt's `server.js` exits on `EADDRINUSE` (`server.js:926`); no explicit stop path.
- LookOut has a real venv story: `scripts/build-venv.{sh,ps1,js}` builds `backend/.venv` on every electron-builder prebuild (`npm run backend:build`).

---

## 3. HTTP API surface

| Method | LookOut | SlackIt Node | SlackIt FastAPI |
|---|---|---|---|
| `/health` | ✓ | ✓ | ✓ (also `SELECT 1`; 503 if DB unreachable) |
| `/state`, `/seed`, `/snapshot` | ✓ | ✓ | ✓ |
| `/diff` (HTML viewer) | ✓ (serves `tools/db-diff.html`) | ✓ (serves `tools/db-diff.html`) | ✓ (serves `tools/db-diff.html`) |
| `/reset` | ✓ | ✓ | ✓ (`POST /admin/reset`) |
| `/verify` (GET + POST) | ✓ | ✓ | ✓ |
| `/verify?collection=<n>` (view shapes) | ✓ — each branch declares `payloadKey` (single data list) | ✓ | ✓ |
| `/auth/{user,accounts,login,logout}` | ✓ | ✓ | ✓ (`/auth/session` instead of `/auth/user`) |
| `/collection/<name>` whole-blob | Retained for singletons only | ✓ | **Removed** — granular only |
| Per-entity CRUD `/{coll}/{id}` | ✓ generated for 9 collections | ✗ | ✓ via `build_entity_router` |
| Per-action verbs `/emails/{id}/<action>` | ✓ (14 actions + 8 bulk) | ✗ | ✗ |
| External inbox `/receive` | ✓ (with `_receive_lock`) | n/a | n/a |

**Envelope-shape uniformity (LookOut):** every collection in `/state`, `/seed`, `/collection/<n>`, and the per-branch `/verify?collection=<n>` response exposes a `payloadKey` field — `"entities"` for entity-shaped (default) or `"data"` for the four singletons (`auth`, `settings`, `sender-lists`, `search`). The `events` branch of `/verify?collection=` also declares `payloadKeySecondary: "holidayEvents"`. This is what lets the diff viewer (`tools/db-diff.html`) treat singletons and entity collections through one algorithm — no special-casing by collection name. Source of truth: `PAYLOAD_KEY_BY_COLLECTION` in `backend/app/storage.py`.

**Most consequential divergence:** LookOut's recent refactor (`commit b7b205f`) converted email actions from `PATCH /collection/emails` into 14 named verbs. SlackIt FastAPI is converging toward the same shape (`build_entity_router` per feature); SlackIt Node has not migrated.

See [[lookout-desktop/About/Http-Api]] for the LookOut route map.

---

## 4. Storage layer

| Aspect | LookOut | SlackIt Node | SlackIt FastAPI |
|---|---|---|---|
| ORM / driver | SQLAlchemy 2.0 ORM | raw `sqlite3` | SQLAlchemy 2.0 ORM |
| Backend abstraction | `Storage` ABC + `_BACKENDS` registry + `SqliteOrmStorage` + `PostgresOrmStorage` stub (`storage.py:685-688`) | none — direct sqlite calls | `StorageRepository` Protocol + `SqlAlchemyStorage` (no `PostgresStorage` class; `DATABASE_URL` env-var swap) |
| Tables actually used at runtime | 35 ORM models + 1 kv_store fallback (40+ truncate list) | Single SQLite file, kv-by-collection | **Single `KVStore` table** — every collection is a JSON envelope. DBML is aspirational only. |
| Sync vs async | All storage methods are `def` (sync) | Sync | Sync (FastAPI runs them in threadpool) |
| Auto-id race retry | 5× on `IntegrityError` (`storage.py:451-460`) | n/a | n/a |
| FK handling | `_with_fk_off(session)` per write; engine-wide `begin` listener flips back on (`engine.py:88-110`) | n/a | SQLite WAL + `foreign_keys=ON` connect hook |
| Optimizations | `insertmanyvalues=False` on SQLite (self-ref FK in batch) | n/a | `check_same_thread=False` on SQLite |

**LookOut-only patterns:**

- `_BACKENDS` registry — one-line backend swap. SlackIt FastAPI has a Protocol but no registry.
- Per-transaction FK toggling is **dialect-aware**: SQLite → `PRAGMA foreign_keys=OFF`, Postgres → `SET session_replication_role='replica'`. Necessary because LookOut's writers do arbitrary delete-then-insert order with no dependency sort.
- IntegrityError retry is a runtime defense against the auto-id generator racing — SlackIt doesn't need it because its FastAPI rewrite uses JSON envelopes (no FK graph to break).

---

## 5. Preconfig / seed pipeline

| Concern | LookOut | SlackIt Node | SlackIt FastAPI |
|---|---|---|---|
| `config_reader` | `preconfig/config_reader.py` | `src/main/preconfig/config-reader.js` | `server/app/features/preconfig/config_reader.py` |
| `config_validator` | `preconfig/config_validator.py` (rejects unknown folder keys) | ✗ | `config_validator.py` |
| `config_defaults` | `preconfig/config_defaults.py` | ✗ | `config_defaults.py` |
| Pickers | `email_picker.py` (336 lines), `contact_picker.py`, `calendar_picker.py`, `name_pool.py`, `email_templates.py` (469 lines) | ✗ none | `member_picker.py`, `message_picker.py` (per-channel templates), `name_pool.py`, `data_generator.py` |
| Generator | `preconfig/data_generator.py` orchestrator → `pick_all_emails`, `pick_all_contacts`, `pick_all_events` | ✗ | `data_generator.generate_from_config()` mirrors LookOut's pipeline 1:1 |
| Seed vs preconfig choice | `POST /reset` reads `MOCKAPP_CONFIG_PATH`; falls back to `reset_to_seed()` | `applyConfigIfPresent()` (`server.js:90`) | `features/state/service.py:apply_config_if_present()` (line 35) |
| Per-task account switch | ✓ `data["authenticatedUser"]` survives reset (LO-012) | ✓ `setAuthUser(...)` after config apply | ✓ Same |
| Pool files | `data/email-pool.json` (404 KB), `contact-pool.json`, `calendar-pool.json` | none | none — built per-config |
| ID allocation | `id_generator.SequenceCounter` (stateful `prefix-NNN`) | n/a | n/a |

**Picker-layer alignment:** LookOut and SlackIt FastAPI are most aligned here. Both have validator → resolver → picker → generator → seeder pipelines with near-identical file names. SlackIt Node has no pickers — `applyConfigIfPresent()` just reads the JSON and writes it verbatim.

See [[lookout-desktop/About/Seeding]] for the LookOut pre-seed flow.

---

## 6. Verification (`/verify`)

> [!info] Domain-specific shape
> The `/verify` payload diverges because the domains are different (Outlook folders vs Slack channels). Listed side-by-side below.

| LookOut | SlackIt |
|---|---|
| `inboxCount`, `inboxEmails[]`, `lastSentEmail`, `sentEmailsCount`, `lastDraft` | `channelCount`, `channels{}` (per-channel `messages`, `pinnedCount`, `topic`), `totalMessageCount`, `lastMessage` |
| `spamCount`, `junkCount`, `archiveCount`, `deletedCount` | `directMessageCount`, `directMessages{}` (per-DM `messages[]`, `unreadCount`, `isGroup`) |
| `contacts[]` (flattened), `contactsCount` | `memberStatuses`, `activeTriggers{legacy, autoReplies}` |
| `calendarCount`, `calendarEvents[]` (merged), `userCalendarEvents[]`, `holidayEventsCount` | `settings`, `activeChannelId: null`, `threadOpen: false` |
| `filters[]`, `filterCount` | `history`, `historySummary`, `stats` |
| `signature`, `autoReplyEnabled`, `categories[]` | n/a |
| UI placeholders: `currentFolder="inbox"`, `selectedEmailCount=0`, `readingPanePosition="right"` (no-op in Python) | n/a |

**Assertion types:**

- **LookOut:** generic (`exists`, `count`, `equals`, `contains`) plus sugar (`email-in-folder`, `email-marked-read`, `contact-created`, `folder-created`).
- **SlackIt Node/FastAPI:** Slack-specific (`message-in-channel`, `channel-created`, `message-pinned`, `member-status-set`, `dm-exists`, `history-contains`).
- **SlackIt FastAPI** joins reactions/replies/bookmarks/pins/forwards from their own normalized collections (`_build_message_index()`); `message-pinned` checks the `message_pins` collection (`service.py:389`) instead of trusting inlined fields.

The LookOut `equals` assertion walks a dotted path through the full `/state` snapshot and returns `passed:false, error:"path not found"` rather than crashing — nicer failure mode than SlackIt's missing-field errors.

See [[lookout-desktop/About/Verification]] for the LookOut verify details.

---

## 7. Persistence middleware / write path

| Concern | LookOut | SlackIt FastAPI |
|---|---|---|
| File | `src/store/middleware/persistenceMiddleware.ts` (200 lines) | `src/store/middleware/apiSyncMiddleware.ts` |
| Pattern | Optimistic thunks (`createOptimisticThunk`) + targeted per-action HTTP | Diff-based: compute `{added, changed, removed}` against baseline; one HTTP call per row |
| Debounce | 1000 ms (legacy middleware only, for singletons) | 300 ms |
| Per-action HTTP | `serverClient.emails.markRead/pin/snooze/...` (14 actions + 8 bulk) | Generic `resource.create/patch/remove` per collection |
| In-flight guard | `pendingWrites: Set<string>` + `markPending/clearPending` | `pendingCollections: Set<string>` (mark window during scheduled flush) |
| Self-write guard | `selfWrittenLastModified: Map<coll, lastModified>` + `consumeSelfWrite` — server's `_lastModified` echo is recognized and skipped during next poll | Less granular: `meta.skipPersist === true` (boot hydration and poller) |
| `markSeen` short-circuit | ✓ per collection `_lastModified` | ✓ |
| Skip whole-slice write when targeted write in flight | ✓ via `meta.skipPersist` | ✓ via `meta.skipPersist` |

**LookOut-only pattern:** the **two-guard combination** (`pendingWrites` for in-flight edits + `selfWrittenLastModified` for the server's echo). SlackIt FastAPI only has the first; relies on `skipPersist` flags during hydration/poller rather than per-write echo recognition.

**SlackIt-only pattern:** **dual-layer view models** — 10 legacy camelCase *view* slices (`channelsSlice`, `messagesSlice`, …) re-projected from the normalized `entities` reducer via `src/store/thunks/reproject.ts`. LookOut does not have this dual layer — its slices are entity-shaped directly.

---

## 8. Concurrency

| Lock / mechanism | LookOut | SlackIt Node | SlackIt FastAPI |
|---|---|---|---|
| `_receive_lock` (`threading.Lock`) | ✓ `main.py:701`, around `/receive` | n/a | n/a |
| `_auth_lock` (`threading.Lock`) | ✓ `auth.py:27` | n/a | n/a |
| Trigger bookkeeping lock | n/a | n/a | `asyncio.Lock` in `triggers/service.py:33` |
| SQLite WAL + `foreign_keys=ON` | ✓ | n/a (plain sqlite3) | ✓ |
| Transaction per write | ✓ (`s.begin()` per call) | n/a (one-shot calls) | ✓ (`SessionLocal.begin()`) |
| `IntegrityError` retry | ✓ 5× (`storage.py:451-460`) | n/a | n/a |
| FK toggling per write | ✓ `_with_fk_off(session)` + engine `begin` listener | n/a | n/a |
| Renderer-side concurrency | `pendingWrites` Set + per-collection debounce | n/a | `pendingCollections` Set during scheduled flush |

The LookOut `_receive_lock` has an explanatory docstring (`main.py:706-707`): *"concurrent /receive requests don't lose emails on the read-modify-write of `emails`"*. This is the one real-world concurrency hazard LookOut explicitly defends against. SlackIt's `/receive` analogue doesn't exist because Slack has DMs, not an inbox.

---

## 9. Config / paths

**Identical conventions:**

- `MOCKAPP_STORE_DIR` overrides everything
- `MOCKAPP_CONFIG_PATH` overrides the preconfig path
- Linux → `/var/lib/mockapps/<App>` + `/var/lib/appconfigs/<App>.json`
- Win → `C:\ProgramData\MockApps\<App>` + `C:\ProgramData\AppConfigs\<App>.json`
- macOS dev → `~/.mockapps-dev/<App>` + `<projectRoot>/test-config/<App>.json`
- Port bound to `127.0.0.1`, never public interface
- `chmod 1777` (sticky world-writable) on the config dir so the gym can drop `<App>.json` in

**Differences:**

- LookOut: hardcoded constants in `paths.py`. Single source of truth.
- SlackIt Node: same convention in `src/main/paths.js`.
- SlackIt FastAPI: pydantic-settings with `SLACKIT_*` prefix; `model_post_init` (`config.py:70`) explicitly falls back to `MOCKAPP_*` vars first. Supports `SLACKIT_DATABASE_URL` for the Postgres swap. Has a `server/.env.example`.

LookOut's `STORAGE_BACKEND` env var (`storage.py:698`) is the cleaner abstraction — `sqlite` vs `postgres` is a single switch. SlackIt FastAPI requires understanding `DATABASE_URL` semantics.

See [[lookout-desktop/About/Paths]] for LookOut's fixed-OS paths.

---

## 10. Manifest & deployment

| Aspect | LookOut | SlackIt |
|---|---|---|
| Port | 5050 | 5070 |
| `gym_type` | `electron_app` | `electron_app` |
| `verification_endpoint` | `http://localhost:5050/verify` | `http://localhost:5070/api/v1/verify` ⚠ shape mismatch (see TL;DR callout) |
| `health_endpoint` | `http://localhost:5050/health` | `http://localhost:5070/health` |
| `reset_endpoint` | `http://localhost:5050/reset` | `http://localhost:5070/api/v1/admin/reset` |
| Test accounts in manifest | 2 (John Doe, Sarah Johnson) | 5 (John Doe, Sarah Johnson, Mark Jason, John Smith, Emily Rodriguez) |
| Tasks in manifest | 25 (`login-001`, `send-email-001`, …, `complex-workflow-001`) | 6 (`send-message-001`, `create-channel-001`, …) |
| Profiles | `clean`, `logged_in_john`, `logged_in_frida` (deprecated), `logged_in_sarah` | `clean`, `logged_in_john`, `logged_in_sarah`, `logged_in_mark` |

Both installers use `chmod 1777` (sticky bit) on the config dir so non-privileged processes can drop `<App>.json` there. Both create a dedicated service user (`lookout` / `slackit`) with `nologin` shell on Linux, randomly generated password on Windows.

---

## 11. Schema

| Aspect | LookOut | SlackIt |
|---|---|---|
| DBML | `docs/schema_v2.dbml` (35 tables, 69 FK relationships, PostgreSQL dialect declared but SQLAlchemy portable via `FlexString`) | `docs/schema_v2.dbml` (22 tables, SQLite dialect declared, identifier convention `{entity}-wsNN-NNN`) |
| Runtime DDL | `backend/app/schema.sql` — full SQLite DDL run via `raw.executescript()` on first boot | **DBML not realized at runtime.** Single `KVStore` table holds every collection as a JSON envelope. |
| Identifier convention | `folder-001`, `msg-NNN`, `account-001`, `event-NNN`, `contact-NNN` | `channel-ws01-017`, `member-ws01-001`, `dm-ws02-001`, `msg-ws03-0004`, `workspace-01..03`; composite PKs `::`-joined |
| Empty by design | `account_sender_lists`, `mail_rules`, `email_signatures`, `auto_reply_settings`, `user_preferences` — runtime-only | `drafts`, `app_state` |
| Notable tables | `email_message_references` (composite pk with `sequence_order` for thread ordering), `email_message_flags` (1:1 with message_id for follow-up dates), `mail_rules` (polymorphic via `action_type`), `calendar_event_recurrence_days` (composite pk per weekday) | `message_forwards`, `message_thread_replies`, `message_reaction_users`, `message_bookmarks`, `message_pins`, `message_attachments`, `custom_channel_sections`, `custom_channel_section_items`, `member_settings`, `conversation_member_state` |

**Most striking divergence:** LookOut's DBML **is** the runtime schema (modulo portability swaps in `FlexString`). SlackIt's DBML is a **target design** — the current runtime stores everything as JSON envelopes in a single table, so the relational integrity SlackIt's DBML promises (e.g. `message_thread_replies.thread_id → message_threads.id`) isn't enforced by SQLite today. SlackIt FastAPI reads/joins across these collections at verify time (`_build_message_index()`) and at API time, treating the envelope JSON as the source of truth and the DBML as documentation.

---

## Summary

**Where LookOut is more mature than SlackIt (live):**

1. **Per-action HTTP routes** — 14 verb routes + 8 bulk variants on emails; generated routes on 9 collections. SlackIt Node has none.
2. **Real relational schema at runtime** — DBML ↔ `schema.sql` ↔ ORM models are aligned. SlackIt's runtime is JSON envelopes.
3. **Two concurrency locks** + `IntegrityError` retry. SlackIt Node has zero locks.
4. **Self-write guard** in the renderer (server's `_lastModified` echo recognized). SlackIt FastAPI only has the simpler in-flight guard.
5. **Backend registry** (`_BACKENDS`) — adding a new backend is one line. SlackIt FastAPI has a Protocol but no registry.

**Where SlackIt FastAPI mirrors LookOut more closely than SlackIt Node does:**

1. **Feature-based routers** with per-entity CRUD — same direction as LookOut's per-action refactor.
2. **Picker layer** (`member_picker`, `message_picker`) — same pattern as LookOut's `email_picker` / `contact_picker`.
3. **Pydantic-settings / Protocol-based storage abstraction** — same direction as LookOut's `Storage` ABC.

**Where SlackIt is more developed than LookOut:**

1. **Slack-specific verification shape** — `directMessages{}` with full nested message arrays, `activeTriggers{legacy, autoReplies}`, `memberStatuses`, `historySummary`. LookOut's verify summary is shallower on this dimension because Outlook doesn't have an equivalent.
2. **History tracking** — `history-tracker.js` (Node) captures action history; LookOut's `/state` doesn't track action history.
3. **Dual-layer view models** — 10 legacy view slices reprojected from normalized entities. Useful pattern for migration scenarios.

**Practical gaps:**

- SlackIt's `manifest.json` points at `/api/v1/verify` (FastAPI path) but the install lifecycle runs `server.js` (Node). Latent bug.
- SlackIt's FastAPI rewrite is "retired" per its own migration doc but remains un-wired — anyone reading SlackIt today has to choose which server to study. A clear deprecation (delete the tree, or delete `server.js`) would make the project easier to read.
- The picker-layer file naming differs slightly: LookOut uses `email_picker.py` / `contact_picker.py` / `calendar_picker.py`; SlackIt FastAPI uses `member_picker.py` / `message_picker.py`. Aligning these would help anyone cross-referencing both projects.

---

## Update — 2026-07-15 evening

Three small follow-ups after this comparison was first written:

1. **LookOut `/diff` now serves the HTML viewer** (was returning a broken JSON envelope whose `_lastModified` comparison always flagged everything as different because the seed snapshot stamps `_lastModified=None`). New behaviour mirrors SlackIt Node's `server.js:881-892` — `getDiffViewerHtml()` lazy-loads `tools/db-diff.html` once per process; 404 + JSON error if the file is missing.
2. **LookOut singleton coverage in the diff viewer** — `auth`, `settings`, `sender-lists`, `search` were invisible to the page because they store their payload under `data`, not `ids`/`entities`. Now every collection envelope declares a `payloadKey` (`storage.PYLOAD_KEY_BY_COLLECTION`); the page's `_normalize()` helper synthesises a synthetic entity for singletons.
3. **Uniform `payloadKey` across all collection-returning endpoints** — `/state`, `/seed`, `/collection/<n>`, `/snapshot`, and every branch of `/verify?collection=<n>` declare `payloadKey` (single data-list key) and `payloadKeySecondary` (only on the `events` branch which has both `userEvents` and `holidayEvents`). All additive — no renderer breakage.

Related commit branch: see `docs/INSTALL-TRAINERS.md` §4.1 for the trainer-side access pattern (`http://127.0.0.1:5050/diff`); the server binds `--host 127.0.0.1` so OSWorld-VM access requires VM-side port forwarding.

---

## Sources

**LookOut:** `backend/app/main.py`, `backend/app/storage.py`, `backend/app/preconfig/*`, `backend/app/verification_server.py`, `backend/app/auth.py`, `backend/app/paths.py`, `main.js`, `install.sh`, `install.ps1`, `electron-builder.trainer.json`, `manifest.json`, `docs/schema_v2.dbml`, `backend/app/schema.sql`, `src/store/middleware/persistenceMiddleware.ts`, `src/store/thunks/_optimistic.ts`, `src/api/serverClient.ts`, `scripts/build-venv.{sh,ps1,js}`.

**SlackIt:** `server.js`, `src/main/services/sqlite.service.js`, `src/main/preconfig/*`, `server/app/main.py`, `server/app/api.py`, `server/app/common/repository.py`, `server/app/features/preconfig/*`, `server/app/features/state/service.py`, `server/app/features/verify/service.py`, `server/app/core/{config,constants}.py`, `main.js`, `install.sh`, `install.ps1`, `electron-builder.trainer.json`, `manifest.json`, `docs/schema_v2.dbml`, `server/docs/MIGRATION.md`, `server/docs/API.md`, `src/api/{http,client}.ts`, `src/store/middleware/apiSyncMiddleware.ts`, `src/store/registry/collections.ts`, `src/store/thunks/dataThunks.ts`.

## Related

- [[lookout-desktop/About]] — LookOut codebase map
- [[lookout-desktop/About/Architecture]] — two-process split
- [[lookout-desktop/About/Http-Api]] — HTTP API contract
- [[lookout-desktop/About/Seeding]] — pre-seed and preconfig flow
- [[lookout-desktop/About/Verification]] — `/verify` endpoint
- [[lookout-desktop/About/Paths]] — fixed OS paths
- [[lookout-desktop/About/Data-Layer]] — SQLite schema and seed collections