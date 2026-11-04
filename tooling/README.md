# tooling

## Overview

The `tooling` directory houses repository-level developer automation, custom compiler lints, release compliance auditing, crates.io API integration, and performance profiling utilities for the Zed workspace. These packages run outside the production Zed editor binary to enforce workspace invariants, generate and validate CI workflows, audit commit histories, and measure benchmark regressions.

## Subdirectories & Boundaries

- `compliance`: [./compliance/README.md](./compliance/README.md) - Audits Git commit ranges against GitHub PR review records, co-author write permissions, and `zed-zippy[bot]` signature invariants to produce release compliance reports.
- `corgi`: [./corgi/README.md](./corgi/README.md) - Support files and vendored `[patch.crates-io]` crate overrides (`scratch`) for content-addressed builds.
- `crates_io_client`: [./crates_io_client/README.md](./crates_io_client/README.md) - Rate-limited async crates.io v1 API client (`RealCratesIoClient`) and in-memory test double (`SimulatedCratesIoClient`) for querying and updating published crates.
- `lints`: [./lints/README.md](./lints/README.md) - Standalone pinned-nightly Dylint library implementing custom `LateLintPass` checks for GPUI and Zed codebase anti-patterns.
- `perf`: [./perf/README.md](./perf/README.md) - Test-harness performance profiler and comparator (`perf`) that drives `#[perf]`-annotated tests via Hyperfine and compares `.perf-runs` JSON artifacts.
- `xtask`: [./xtask/README.md](./xtask/README.md) - Workspace automation CLI (`cargo xtask`) for CI workflow generation/validation, Clippy, license/dependency conformity, GPUI release planning, WebRTC setup, and sandbox testing.
