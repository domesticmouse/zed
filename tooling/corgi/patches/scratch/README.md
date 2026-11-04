# scratch

## Overview

`scratch` (`tooling/corgi/patches/scratch`) is a corgi-compatible `[patch.crates-io]` replacement for the upstream `scratch` crate that resolves build-script scratch directories from `OUT_DIR` at runtime rather than baking a single compile-time `env!("OUT_DIR")` path shared across build actions.

## Interface

- `[src/lib.rs:path]`: `pub fn path(suffix: &str) -> PathBuf` — Returns `PathBuf::from(std::env::var_os("OUT_DIR").unwrap()).join(suffix)` after ensuring the directory exists via `fs::create_dir_all`.
- `[build.rs:main]`: Build script that clears and recreates `OUT_DIR` via `fs::remove_dir_all` and `fs::create_dir`.

## Invariants

- `[src/lib.rs:path]` & `[build.rs:main]`: Reads `OUT_DIR` at runtime via `std::env::var_os("OUT_DIR")` (panicking if unset) so each crate's build script receives an isolated directory inside its own action output rather than mutating cross-action shared state in content-addressed builds.

## Side Effects

- `[src/lib.rs:path]`: Reads the `OUT_DIR` environment variable and creates the target directory tree on disk via `std::fs::create_dir_all`.
- `[build.rs:main]`: Removes and recreates `OUT_DIR` on disk during build script execution.

## Verification

- Run: `cargo check -p scratch`
