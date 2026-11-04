# crates_io_client

## Overview

`crates_io_client` (`tooling/crates_io_client`) provides an async HTTP client trait (`CratesIoClient`), typed data models, a rate-limited production implementation (`RealCratesIoClient`), and an in-memory test double (`SimulatedCratesIoClient`) for interacting with the crates.io v1 API (`https://crates.io/api/v1`).

## Interface

- `[src/crates_io_client.rs:CratesIoClient]`: Async trait (`Send + Sync`) defining `get_site_metadata(&self) -> Result<SiteMetadata>`, `list_crates(&self, query: &CrateListQuery) -> Result<Vec<Crate>>`, `get_crate(&self, name: &CrateName) -> Result<CrateDetails>`, and `update_crate(&self, name: &CrateName, update: &CrateUpdate) -> Result<Crate>`.
- `[src/crates_io_client.rs:CratesIoError]` & `[src/crates_io_client.rs:Result]`: Error enum (`PermissionDenied`, `NotFound`, `Upstream`, `Request`) and `Result<T, E = CratesIoError>` alias.
- `[src/crates_io_client.rs:ApiToken]`: Bearer/API token wrapper (`ApiToken::new`).
- `[src/real_crates_io_client.rs:RealCratesIoClient]`: Production `CratesIoClient` implementation (`RealCratesIoClient::new(http_client: Arc<dyn HttpClient>, api_token: ApiToken)`) backed by `http_client::HttpClient`.
- `[src/simulated_crates_io_client.rs:SimulatedCratesIoClient]` (`#[cfg(any(test, feature = "test-support"))]`) : In-memory `CratesIoClient` test double (`SimulatedCratesIoClient::new`, `token_user`, `test_user`, `test_site_metadata`, `test_crate`, `update_test_crate`, `test_version`).
- `[src/types.rs:UserId]`, `[src/types.rs:VersionId]`, `[src/types.rs:CrateName]`, `[src/types.rs:SiteMetadata]`, `[src/types.rs:CrateListQuery]`, `[src/types.rs:Crate]`, `[src/types.rs:Version]`, `[src/types.rs:User]`, `[src/types.rs:CrateDetails]`, `[src/types.rs:CrateUpdate]`: Request and response data structures for crates.io API endpoints.

## Invariants

- `[src/real_crates_io_client.rs:RequestThrottle]` & `[src/real_crates_io_client.rs:MIN_REQUEST_INTERVAL]`: Enforces crates.io's data-access policy by reserving send times across concurrent callers so requests on a shared `RealCratesIoClient` are spaced at least `Duration::from_secs(1)` apart and carry `USER_AGENT` (`"Zed GPUI releases (https://github.com/zed-industries/zed)"`).
- `[src/real_crates_io_client.rs:RealCratesIoClient::list_crates]`: Requests pages of `MAX_PER_PAGE` (`100`) crates and stops pagination when `page.crates.is_empty()` or `page.meta.next_page` is `None`.
- `[src/types.rs:CrateListQuery]` & `[src/simulated_crates_io_client.rs:SimulatedCratesIoClient::list_crates]`: Setting `keyword` causes crates.io (and `SimulatedCratesIoClient`) to ignore `user_id`; single-crate `keywords` and `categories` fields are stripped (`None`) from `list_crates` and `update_crate` responses via `listed`.
- `[src/simulated_crates_io_client.rs:SimulatedCratesIoClient::update_crate]`: Returns `CratesIoError::PermissionDenied` (`StatusCode::FORBIDDEN`) unless the crate's `owners` contains `token_user`.

## Side Effects

- `[src/real_crates_io_client.rs:RealCratesIoClient::request_response]`: Sleeps until the throttled `Instant` via `smol::Timer::at` and dispatches HTTPS `GET` and `PATCH` requests to `https://crates.io/api/v1` via `Arc<dyn HttpClient>`.
- `[src/simulated_crates_io_client.rs:SimulatedCratesIoClient]`: Mutates in-memory `Mutex<CratesIoState>` registry state during test setup and `update_crate` calls.

## Verification

- Run: `cargo test -p crates_io_client`
