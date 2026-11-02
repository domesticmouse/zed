# perf

## Overview

`perf` (`tooling/perf`) provides the communication protocol, metadata model, and CLI runner/comparator for profiling Zed unit tests marked with `#[perf]` (from `util_macros::perf`).

## Interface

- `[src/implementation.rs:consts]`: Protocol constants shared between the `perf` binary and the `#[perf]` proc-macro (`SUF_NORMAL`, `SUF_MDATA`, `ITER_ENV_VAR`, `MDATA_LINE_PREF`, `MDATA_VER`, `WEIGHT_DEFAULT`, `NOISE_CUTOFF`, `RUNS_DIR`, `ITER_COUNT_LINE_NAME`, `WEIGHT_LINE_NAME`, `IMPORTANCE_LINE_NAME`, `VERSION_LINE_NAME`).
- `[src/implementation.rs:Importance]`: Benchmark priority level (`Critical`, `Important`, `Average`, `Iffy`, `Fluff`).
- `[src/implementation.rs:FailKind]`: Failure reason enum (`Triage`, `Profile`, `VersionMismatch`, `BadMetadata`, `Skipped`).
- `[src/implementation.rs:TestMdata]`: Test metadata struct holding version, optional `NonZero<usize>` iteration count, `Importance`, and `u8` weight.
- `[src/implementation.rs:Timings]`: Mean and standard deviation `Duration` with `Timings::iters_per_sec`.
- `[src/implementation.rs:Output]` & `[src/implementation.rs:PerfReport]`: Collects benchmark results (`Output::blank`, `Output::success`, `Output::failure`, `Output::is_empty`, `Output::sort`, `Output::merge`, `Output::compare_perf`) and formats Markdown tables via `Display`.
- `[src/main.rs:main]` (`feature = "cli"`): CLI binary backing `cargo perf-test` and `cargo perf-compare` (`--json=<ident>`, `--critical`, `--important`, `--average`, `--iffy`, `--fluff`, `--quiet`, `compare [--save=<file>] <new-ident> <old-ident>`).

## Invariants

- `[src/main.rs:parse_mdata]`: If `TestMdata::version <= consts::MDATA_VER`, `parse_mdata` must parse all metadata lines emitted by `*__ZED_PERF_MDATA` functions.
- `[src/main.rs:get_tests]`: Every `#[perf]` test in a compiled test binary emits both a `*__ZED_PERF_FN` test and a matching `*__ZED_PERF_MDATA` test; the filtered test list length must be a multiple of 2.
- `[src/implementation.rs:Output]`: Any entry in `Output` with `Ok(Timings)` must have `Some(TestMdata)` where `mdata.iterations.is_some()`.
- `[Cargo.toml:lints.clippy]`: Enforces `let_underscore_must_use = "forbid"`, `undocumented_unsafe_blocks = "forbid"`, `missing_safety_doc = "forbid"`, `allow_attributes = "deny"`, `allow_attributes_without_reason = "deny"`, and `as_underscore = "deny"`.

## Side Effects

- `[src/main.rs:hyp_profile]` & `[src/main.rs:spawn_and_iterate]`: Spawns the target test binary and `hyperfine` subprocesses (`hyperfine` in `PATH` or `~/.cargo/bin/hyperfine`).
- `[src/main.rs:main]`: Reads and writes JSON benchmark artifacts in `.perf-runs/<ident>.<crate>.json` and optional comparison files via `--save`.

## Verification

- Run: `cargo check -p perf --all-features`
