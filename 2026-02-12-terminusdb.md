Correct—Docker just gives you the infrastructure. Once the TerminusDB container is up you still need to push data into it:

1. **Apply serviceIds locally** (if you haven’t already) with `npm run apply-service-ids -- --mapping <path>` so every tasker has the right `serviceIds`.
2. **Run `npm run terminus:setup`** to create or update the schema in the running TerminusDB instance.
3. **Import the taskers** via `npm run terminus:import -- --dry-run` (checks) and then `npm run terminus:import -- --confirm` (actually writes). This script connects to the TerminusDB container using the env vars you set for Compose (`TERMINUSDB_ENDPOINT=http://terminusdb:6363`, etc.), so make sure the app container or your host shell has those values.

Until you perform step 3, the DB will be empty even though the container is running.

