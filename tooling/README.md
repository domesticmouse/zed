# tooling

## Overview

The `tooling` directory houses repository-level developer automation, custom compiler lints, release compliance auditing, and performance profiling utilities for the Zed workspace. These packages run outside the production Zed editor binary to enforce workspace invariants, generate and validate CI workflows, audit commit histories, and measure benchmark regressions.

## Subdirectories & Boundaries

- `compliance`: [./compliance/README.md](./compliance/README.md) - Audits Git commit ranges against GitHub PR review records, co-author write permissions, and `zed-zippy[bot]` signature invariants to produce release compliance reports.
- `corgi`: `./corgi/patches/scratch` - Vendored crate patch directory referenced by `[patch.crates-io]` in the root `Cargo.toml`.
- `lints`: [./lints/README.md](./lints/README.md) - Standalone pinned-nightly Dylint library implementing custom `LateLintPass` checks for GPUI and Zed codebase anti-patterns.
- `perf`: [./perf/README.md](./perf/README.md) - Test-harness performance profiler and comparator (`perf`) that drives `#[perf]`-annotated tests via Hyperfine and compares `.perf-runs` JSON artifacts.
- `xtask`: [./xtask/README.md](./xtask/README.md) - Workspace automation CLI (`cargo xtask`) for CI workflow generation/validation, Clippy, license/dependency conformity, GPUI release planning, WebRTC setup, and sandbox testing.
