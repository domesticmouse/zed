# db

## Overview

Application SQLite persistence layer (`db`) providing topological domain migrations, scoped database connections, and key-value stores (`KeyValueStore`, `ScopedKeyValueStore`, `GlobalKeyValueStore`) built on `sqlez`.

## Building Queries

First, craft your test data using `open_test_db` (`[src/db.rs:open_test_db]`) in a unit test to create and migrate an isolated SQLite database.

To interactively inspect and test your queries, you can also import a generated SQLite DB file into https://sqliteonline.com/

## Interface

- Database & Migrations: `[src/db.rs:DomainMigration]`, `[src/db.rs:AppDatabase]`, `[src/db.rs:AppMigrator]`, `[src/db.rs:DbScope]`, `[src/db.rs:GlobalDbScope]`, `[src/db.rs:ALL_FILE_DB_FAILED]`, `[src/db.rs:db_path]`, `[src/db.rs:open_db]`, `[src/db.rs:open_test_db]`, `[src/db.rs:static_connection]`, `[src/db.rs:write_and_log]`
- Key-Value Store: `[src/kvp.rs:KeyValueStore]`, `[src/kvp.rs:ScopedKeyValueStore]`, `[src/kvp.rs:GlobalKeyValueStore]`, `[src/kvp.rs:Dismissable]`
- Re-exports: `[src/query.rs:query]`

## Invariants

- `AppMigrator::migrate` executes all `inventory`-registered `DomainMigration` entries in topological dependency order (`[src/db.rs:AppMigrator::migrate]`).
- `AppDatabase::global` panics with `"database not initialized"` in non-test builds if `AppDatabase` is not registered as a GPUI global on `App` (`[src/db.rs:AppDatabase::global]`).
- Every SQLite connection enforces `PRAGMA foreign_keys=TRUE;`, and initialization configures `PRAGMA journal_mode=WAL; PRAGMA busy_timeout=500; PRAGMA case_sensitive_like=TRUE; PRAGMA synchronous=NORMAL;` (`[src/db.rs:CONNECTION_INITIALIZE_QUERY]`, `[src/db.rs:DB_INITIALIZE_QUERY]`).
- `kv_store` (`key TEXT PRIMARY KEY, value TEXT NOT NULL`) and `scoped_kv_store` (`PRIMARY KEY(namespace, key)`) tables are `STRICT` (`[src/kvp.rs:KeyValueStore]`).

## Side Effects

- Creates the database directory on disk under `{db_dir}/0-{scope}/db.sqlite` unless `*ZED_STATELESS` is set; if opening the file database fails, stores `true` in `ALL_FILE_DB_FAILED` and opens a shared in-memory fallback database (`[src/db.rs:open_db]`).
- `AppDatabase::new` and `GLOBAL_KEY_VALUE_STORE` block the current thread via `gpui::block_on` to open and migrate the SQLite database (`[src/db.rs:AppDatabase::new]`, `[src/kvp.rs:GLOBAL_KEY_VALUE_STORE]`).
- `static_connection` registers a `DomainMigration` in the link-time `inventory` registry, and `write_and_log` spawns detached background tasks on the GPUI executor (`[src/db.rs:static_connection]`, `[src/db.rs:write_and_log]`).

## Verification

- Run: `cargo test -p db`
