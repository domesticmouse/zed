# lints

## Overview

`lints` (`tooling/lints`) is a standalone [dylint](https://github.com/trailofbits/dylint) dynamic library (`cdylib`) that flags Zed- and GPUI-specific performance and correctness anti-patterns using `rustc_lint::LateLintPass`.

Install `dylint`, a pinned nightly toolchain and the necessary tools with

```sh
cargo install cargo-dylint dylint-link
cd tooling/lints
rustup toolchain install
```

The channel and its components (`rustc-dev`, `rust-src`, `llvm-tools-preview`)
are declared in `tooling/lints/rust-toolchain.toml`, so `rustup toolchain install`
picks them up automatically when run from that directory.

## Interface

- `[src/lib.rs:register_lints]`: Dylint entrypoint exported via `dylint_library!()` that registers all late lint passes into `rustc_lint::LintStore`.
- `[src/lib.rs:SHARED_STRING_FROM_STR_LITERAL]` (`shared_string_from_str_literal`) — Flags `SharedString::new/from` etc. where `SharedString::new_static` should be used instead.
- `[src/lib.rs:ASYNC_BLOCK_WITHOUT_AWAIT]` (`async_block_without_await`) — Flags `async { … }` blocks whose body contains no `.await` expression.
- `[src/entity_update_in_render.rs:ENTITY_UPDATE_IN_RENDER]` (`entity_update_in_render`) — Flags `Entity::update`/`WeakEntity::update` mutating an entity inside `Render::render` / `RenderOnce::render`.
- `[src/map_lookup_then_insert.rs:MAP_LOOKUP_THEN_INSERT]` (`map_lookup_then_insert`) — Flags map value lookups that branch into an insertion for the same key, on any map with an `entry` method (`HashMap`, `BTreeMap`, `FxHashMap`, `hashbrown`, `indexmap`, ...), and the same pattern on `HashSet`/`BTreeSet` (where a bare `insert` already searches only once). Keys are matched up to borrows/derefs, value-preserving conversions (`clone`, `to_owned`, `to_string` on string types, `to_vec` on slices, `String::from` on `&str` — each resolved by definition, not name), and immutable local aliases, so looking up `&key` and inserting `key.clone()` is caught.
- `[src/notify_in_render.rs:NOTIFY_IN_RENDER]` (`notify_in_render`) — Flags `Context::notify()` called inside `Render::render` / `RenderOnce::render`.
- `[src/owned_string_into_shared.rs:OWNED_STRING_INTO_SHARED]` (`owned_string_into_shared`) — Flags `String::from(<lit>).into()` / `<lit>.to_string().into()` / `<lit>.to_owned().into()` whose target is `SharedString`, `Arc<str>`, `Rc<str>`, or `Cow<'_, str>`.
- `[src/blocking_io_on_foreground.rs:BLOCKING_IO_ON_FOREGROUND]` (`blocking_io_on_foreground`) — Catches blocking IO and sync primitive calls (`std::fs`, `std::thread::sleep`, `std::process::Command`/`Child`, `std::path::Path`, `std::net`, `std::sync::{Mutex, RwLock, Condvar, Barrier}`, `std::sync::mpsc::{Receiver, SyncSender}`) that are called on the main thread (functions taking `App`, `Context<T>`, or `Window` by reference or value, or inside `Render::render` / `RenderOnce::render`), stopping at closure boundaries.
- `[single-lint:DYLINT_RUSTFLAGS]`: Helper script (`tooling/lints/single-lint`) that cleans target package fingerprints (or removes the check target directory on workspace runs) and runs a single named lint via `DYLINT_RUSTFLAGS="-A warnings --force-warn <lint>" cargo dylint --all`.

## Invariants

- `[Cargo.toml:workspace]` & `[rust-toolchain.toml:toolchain]`: Defines its own standalone `[workspace]` and pinned nightly toolchain (`rustc-dev`, `rust-src`, `llvm-tools-preview`) matching `clippy_utils`; must not be added to the root workspace `members`.
- `[src/render_helpers.rs:is_directly_in_render_method]` & `[src/blocking_io_on_foreground.rs:BlockingIoOnForeground]`: Render and foreground lint passes bail out on macro-expanded spans (`span.from_expansion()`) and stop at closure boundaries so callbacks and background tasks are not falsely flagged.
- `[single-lint:DYLINT_RUSTFLAGS]`: `--force-warn` is required because after `-A warnings` silences the group, a plain `-W <lint>` does not reliably re-enable a driver-registered lint; targeted packages are `cargo clean -p`ed (or `$check_target` removed when no `-p` is passed) first because `DYLINT_RUSTFLAGS` is not part of Cargo's fingerprint.

## Side Effects

- `[src/lib.rs:tests::ui]` & `[src/lib.rs:tests::ui_shared_string]`: Runs `dylint_testing::ui::Test::src_base("lints", "ui")` and `dylint_testing::ui_test_examples("lints")` during `cargo test`, building `test_fixture` (`gpui_fixture_rustc_flags`) via `cargo build --package gpui`.
- `[single-lint:cargo clean]`: Cleans target crate build fingerprints (`cargo clean -p` or `rm -rf "$check_target"`) before invoking `cargo dylint`.

## How to run

Ideally you run this as part of the `clippy` script in the `zed/script` directory since this will also run our other linters.

### Prerequisites

Install both tools (version 6 or later):

```sh
cargo install cargo-dylint dylint-link
```

- `cargo-dylint` is the `cargo` subcommand that builds and runs the lints; `dylint-link` is the linker used to build the lint library.

The workspace registers this library under `[workspace.metadata.dylint]` in the
root `Cargo.toml`, so Dylint discovers it automatically — you do not pass a
`--path`. The first run builds the library against its pinned nightly (see
`rust-toolchain.toml`) and is slow; later runs are cached.

### Run all lints against the whole repo

```sh
cargo dylint --all -- --workspace
```

### Run all lints against a single crate

```sh
cargo dylint --all -- -p project_panel
```

### Run a single lint

The library loads every lint at once. To run just one, use the `single-lint`
helper, which silences the rest and force-enables the one you name:

```sh
tooling/lints/single-lint blocking_io_on_foreground -p project_panel
```

The first argument is the lint name (one of the snake_case identifiers under
[Interface](#interface)); everything after it is passed to `cargo check`
and defaults to `--workspace`. Under the hood the script runs:

```sh
DYLINT_RUSTFLAGS="-A warnings --force-warn <lint>" cargo dylint --all -- <args>
```

## Verification

- Run: `cd tooling/lints && cargo test`
