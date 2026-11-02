# Zed Server

## Overview

This crate is what we run at https://collab.zed.dev.

It contains our back-end logic for collaboration, to which we connect from the Zed client via a websocket after authenticating via https://zed.dev, which is a separate repo running on Vercel.

## Local Development

### Database setup

Before you can run the collab server locally, you'll need to set up a zed Postgres database. Follow the steps sequentially:

1. Ensure you have postgres installed. If not, install with `brew install postgresql@15`.
2. Follow the steps on Brew's formula and verify your `$PATH` contains `/opt/homebrew/opt/postgresql@15/bin`.
3. If you hadn't done it before, create the `postgres` user with `createuser -s postgres`.
4. You are now ready to run the `bootstrap` script:

```sh
script/bootstrap
```

This script will set up the `zed` Postgres database, and populate it with some users. It requires internet access, because it fetches some users from the GitHub API.

The script will create several _admin_ users, who you'll sign in as by default when developing locally. The GitHub logins for the default users are specified in the `seed.default.json` file.

To use a different set of admin users, create `crates/collab/seed.json`.

```json
{
  "admins": ["yourgithubhere"],
  "channels": ["zed"]
}
```

### Testing collaborative features locally

In one terminal, run Zed's collaboration server and the livekit dev server:

```sh
foreman start
```

In a second terminal, run two or more instances of Zed.

```sh
script/zed-local -2
```

This script starts one to four instances of Zed, depending on the `-2`, `-3` or `-4` flags. Each instance will be connected to the local `collab` server, signed in as a different user from `seed.json` or `seed.default.json`.

## Deployment

We run two instances of collab:

- Staging (https://staging-collab.zed.dev)
- Production (https://collab.zed.dev)

Both of these run on the Kubernetes cluster hosted in Digital Ocean.

Deployment is triggered by pushing to the `collab-staging` (or `collab-production`) tag in GitHub. The best way to do this is:

- `./script/deploy-collab staging`
- `./script/deploy-collab production`

You can tell what is currently deployed with `./script/what-is-deployed`.

## Interface

- Application & Configuration: `[Cargo.toml:collab]`, `[src/main.rs:main]`, `[src/main.rs:init_tracing]`, `[src/lib.rs:VERSION]`, `[src/lib.rs:REVISION]`, `[src/lib.rs:Config]`, `[src/lib.rs:ServiceMode]`, `[src/lib.rs:AppState]`, `[src/lib.rs:Error]`, `[src/lib.rs:Result]`
- Authentication, Environment & Execution: `[src/auth.rs:validate_header]`, `[src/auth.rs:random_token]`, `[src/env.rs:get_dotenv_vars]`, `[src/env.rs:load_dotenv]`, `[src/executor.rs:Executor]`, `[src/api.rs:CloudflareIpCountryHeader]`, `[src/api.rs:SystemIdHeader]`, `[src/api.rs:fetch_extensions_from_blob_store_periodically]`
- Database Layer: `[src/db.rs:Database]`, `[src/db.rs:DatabaseTestOptions]`, `[src/db.rs:TransactionHandle]`, `[src/db.rs:TransactionGuard]`, `[src/db.rs:Channel]`, `[src/db.rs:ChannelMember]`, `[src/db.rs:ChannelsForUser]`, `[src/db.rs:Project]`, `[src/db.rs:ProjectCollaborator]`, `[src/db.rs:Worktree]`, `[src/db.rs:LanguageServer]`, `[src/db.rs:NewExtensionVersion]`
- RPC & WebSocket Server: `[src/rpc.rs:Server]`, `[src/rpc.rs:ConnectionPool]`, `[src/rpc.rs:ConnectionGuard]`, `[src/rpc.rs:Principal]`, `[src/rpc.rs:ProtocolVersion]`, `[src/rpc.rs:AppVersionHeader]`, `[src/rpc.rs:ReleaseChannelHeader]`, `[src/rpc.rs:ZedVersion]`, `[src/rpc.rs:routes]`, `[src/rpc.rs:handle_websocket_request]`, `[src/rpc.rs:handle_metrics]`, `[src/rpc.rs:RECONNECT_TIMEOUT]`, `[src/rpc.rs:CLEANUP_TIMEOUT]`

## Invariants

- `sea-orm` and `sea-orm-macros` (`=1.1.10`) versions must match (`[Cargo.toml:sea-orm-macros]`).
- Concurrent WebSocket connection upgrades are capped at 512 via `CONCURRENT_CONNECTIONS` and `ConnectionGuard::try_acquire`; exceeding returns `StatusCode::SERVICE_UNAVAILABLE` (`[src/rpc.rs:MAX_CONCURRENT_CONNECTIONS]`, `[src/rpc.rs:ConnectionGuard::try_acquire]`).
- `ProtocolVersion` header (`x-zed-protocol-version`) must equal `rpc::PROTOCOL_VERSION` and `AppVersionHeader` (`x-zed-app-version`) must satisfy `ZedVersion::can_collaborate()`, otherwise WebSocket upgrade returns `StatusCode::UPGRADE_REQUIRED` (`[src/rpc.rs:handle_websocket_request]`).
- Registering two RPC handlers for the same message `TypeId` panics; request and stream handlers returning `Ok(())` must have called `Response::send` or `StreamResponse::end`, otherwise an `ErrorCode::Internal` error response is sent (`[src/rpc.rs:Server::add_handler]`, `[src/rpc.rs:Server::add_request_handler]`, `[src/rpc.rs:Server::add_request_stream_handler]`).
- Foreground message handlers per connection are bounded to `MAX_CONCURRENT_HANDLERS` (256) permits via a semaphore (`[src/rpc.rs:Server::handle_connection]`).
- Database transactions run at `IsolationLevel::ReadCommitted`, and `room_transaction` acquires a per-room `OwnedMutexGuard` in `TransactionGuard` (`!Send`) to serialize client notifications against room/project database transactions (`[src/db.rs:Database::with_transaction]`, `[src/db.rs:Database::room_transaction]`).
- `validate_header` rejects `dev-server-token` authorization headers with `StatusCode::UNAUTHORIZED` and requires `<user_id> <access_token>` validated against `{zed_cloud_url}/client/users/me` (`[src/auth.rs:validate_header]`).

## Side Effects

- Loads `./crates/collab/.env.toml` and optional `.env.secret.toml` into process environment variables (`[src/env.rs:load_dotenv]`).
- Binds a TCP listener on `0.0.0.0:{config.http_port}`, initializes global tracing and panic hooks, connects to Postgres/SQLite, and constructs HTTP, LiveKit, S3 blob store, and Kinesis clients (`[src/main.rs:main]`, `[src/lib.rs:AppState::new]`).
- Performs outbound HTTP requests to `{zed_cloud_url}/client/users/me` for token validation and updates LiveKit rooms and Prometheus metrics (`[src/auth.rs:validate_header]`, `[src/rpc.rs:Server::start]`, `[src/rpc.rs:connection_lost]`, `[src/rpc.rs:handle_metrics]`).

## Verification

- Run: `cargo test -p collab`
