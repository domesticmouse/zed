# Zed

[![Zed](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/zed-industries/zed/main/assets/badge/v0.json)](https://zed.dev)
[![CI](https://github.com/zed-industries/zed/actions/workflows/run_tests.yml/badge.svg)](https://github.com/zed-industries/zed/actions/workflows/run_tests.yml)

Welcome to Zed, a high-performance, multiplayer code editor from the creators of [Atom](https://github.com/atom/atom) and [Tree-sitter](https://github.com/tree-sitter/tree-sitter).

---

### Installation

On macOS, Linux, and Windows you can [download Zed directly](https://zed.dev/download) or install Zed via your local package manager ([macOS](https://zed.dev/docs/installation#macos)/[Linux](https://zed.dev/docs/linux#installing-via-a-package-manager)/[Windows](https://zed.dev/docs/windows#package-managers)).

Other platforms are not yet available:

- Web ([tracking discussion](https://github.com/zed-industries/zed/discussions/26195))

### Developing Zed

- [Building Zed for macOS](./docs/src/development/macos.md)
- [Building Zed for Linux](./docs/src/development/linux.md)
- [Building Zed for Windows](./docs/src/development/windows.md)

### Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for ways you can contribute to Zed.

Also... we're hiring! Check out our [jobs](https://zed.dev/jobs) page for open roles.

### Licensing

Zed source code is licensed primarily under GPL-3.0-or-later, with Apache-2.0 components where marked.

License information for third party dependencies must be correctly provided for CI to pass.

We use [`cargo-about`](https://github.com/EmbarkStudios/cargo-about) to automatically comply with open source licenses. If CI is failing, check the following:

- Is it showing a `no license specified` error for a crate you've created? If so, add `publish = false` under `[package]` in your crate's Cargo.toml.
- Is the error `failed to satisfy license requirements` for a dependency? If so, first determine what license the project has and whether this system is sufficient to comply with this license's requirements. If you're unsure, ask a lawyer. Once you've verified that this system is acceptable add the license's SPDX identifier to the `accepted` array in `script/licenses/zed-licenses.toml`.
- Is `cargo-about` unable to find the license for a dependency? If so, add a clarification field at the end of `script/licenses/zed-licenses.toml`, as specified in the [cargo-about book](https://embarkstudios.github.io/cargo-about/cli/generate/config.html#crate-configuration).

## Sponsorship

Zed is developed by **Zed Industries, Inc.**, a for-profit company.

If you’d like to financially support the project, you can do so via GitHub Sponsors.
Sponsorships go directly to Zed Industries and are used as general company revenue.
There are no perks or entitlements associated with sponsorship.

## System Overview

The Zed monorepo is organized as a Cargo workspace (`Cargo.toml`) comprising the desktop editor binary (`crates/zed`), the GPU-accelerated UI framework (`crates/gpui`), the real-time collaboration and API server (`crates/collab`), the headless remote development server (`crates/remote_server`), first-party WASI extensions (`extensions/`), developer automation and compiler lints (`tooling/`), and cross-platform packaging/CI scripts (`script/`).

## Local Interface & Invariants

- **Workspace & Toolchain Configuration**: Root Cargo workspace (`[Cargo.toml:workspace]`, default member `crates/zed`, edition `2024`, `publish = false`), pinned Rust toolchain (`[rust-toolchain.toml:toolchain]`, `channel = "1.98.1"`), Clippy lint policy (`[clippy.toml:disallowed-methods]`, `[clippy.toml:disallowed-types]`), and Corgi content-addressed build configuration (`[corgi.toml:corgi_version]`, `[corgi.toml:extra-inputs]`).
- **Local Services & Container Definitions**: Docker Compose local backing services (`[compose.yml:services.blob_store]`, `[compose.yml:services.livekit_server]`), Foreman process definitions (`[Procfile:collab]`, `[Procfile.web:website]`), and container build definitions (`[Dockerfile-collab:builder]`, `[Dockerfile-distros:BASE_IMAGE]`).
- **Root Invariants**: `clippy.toml` forbids `std::process::Command` methods (use `smol::process::Command`), `smol::Timer::after` (use `gpui::BackgroundExecutor::timer`), and `serde_json::from_reader` / `serde_json_lenient::from_reader` (use `from_slice`); `[Cargo.toml:patch.crates-io]` patches `scratch` to `tooling/corgi/patches/scratch` for per-crate runtime `OUT_DIR` isolation and keeps `tree-sitter-language` unified with `tree-sitter`.

## Subdirectories & System Boundaries

- `crates`: [`./crates/README.md`](./crates/README.md) - Rust workspace crates implementing the GPUI framework ([`./crates/gpui/README.md`](./crates/gpui/README.md), [`./crates/gpui/examples/README.md`](./crates/gpui/examples/README.md)), text/CRDT engine (`rope`, `sum_tree`, `text`, `multi_buffer`), language intelligence (`language`, `lsp`, `grammars`), editor and modal editing (`editor`, [`./crates/vim/README.md`](./crates/vim/README.md)), AI agents and evaluation (`agent`, `acp_thread`, [`./crates/agent_skills/README.md`](./crates/agent_skills/README.md), [`./crates/eval_cli/README.md`](./crates/eval_cli/README.md), [`./crates/eval_utils/README.md`](./crates/eval_utils/README.md)), sandboxing ([`./crates/sandbox/README.md`](./crates/sandbox/README.md)), SQLite persistence ([`./crates/db/README.md`](./crates/db/README.md)), CLI launcher ([`./crates/cli/README.md`](./crates/cli/README.md)), and collaboration backend ([`./crates/collab/README.md`](./crates/collab/README.md)).
- `extensions`: [`./extensions/README.md`](./extensions/README.md) - First-party and integration-test WebAssembly (`wasm32-wasip2`) extensions ([`./extensions/glsl/README.md`](./extensions/glsl/README.md), [`./extensions/html/README.md`](./extensions/html/README.md), [`./extensions/proto/README.md`](./extensions/proto/README.md), [`./extensions/test-extension/README.md`](./extensions/test-extension/README.md)) built against `zed_extension_api`.
- `script`: [`./script/README.md`](./script/README.md) - Cross-platform build, bundling (`bundle-mac`, `bundle-linux`, `bundle-windows.ps1`, [`./script/flatpak/README.md`](./script/flatpak/README.md)), linting and PR review hygiene (`clippy`, `check-licenses`, `check-keymaps`, `check-todos`, [`./script/danger/README.md`](./script/danger/README.md)), benchmarking (`gpui-bench-compare`), shared libraries ([`./script/lib/README.md`](./script/lib/README.md)), Sentry crash triage, and GitHub automation ([`./script/update_top_ranking_issues/README.md`](./script/update_top_ranking_issues/README.md)).
- `tooling`: [`./tooling/README.md`](./tooling/README.md) - Repository automation CLI ([`./tooling/xtask/README.md`](./tooling/xtask/README.md)), custom Dylint static analysis passes ([`./tooling/lints/README.md`](./tooling/lints/README.md)), PR review compliance verifier ([`./tooling/compliance/README.md`](./tooling/compliance/README.md)), crates.io API client ([`./tooling/crates_io_client/README.md`](./tooling/crates_io_client/README.md)), `#[perf]` benchmark profiler ([`./tooling/perf/README.md`](./tooling/perf/README.md)), and Corgi crate patches ([`./tooling/corgi/README.md`](./tooling/corgi/README.md)).
- `docs`: [`./docs/README.md`](./docs/README.md) - mdBook user and developer documentation (`docs/src/`), `docs_preprocessor` pipeline configuration (`docs/book.toml`), legacy anchor redirects (`docs/redirects/`), and client-side theme/analytics plugins (`docs/theme/`).

## Global Invariants

- **Workspace Package & License Conformity**: Every workspace package must inherit `[lints] workspace = true` and `[workspace.dependencies]` (except standalone `extensions/*` and `zed_extension_api`), and every crate directory must contain a valid `LICENSE-GPL` or `LICENSE-APACHE` symlink (`[tooling/xtask/src/tasks/package_conformity.rs:run_package_conformity]`, `[tooling/xtask/src/tasks/licenses.rs:run_licenses]`, `[script/check-licenses:check_license]`).
- **Dependency Graph Isolation**: Non-dev dependencies must never enable the `test-support` feature, legacy Objective-C crates (`block`, `cocoa`, `cocoa-foundation`, `metal`, `objc`, `objc-foundation`, `objc_id`) are forbidden in favor of `objc2*`/`block2`, and forbidden directed dependency pairs between UI feature crates (`FORBIDDEN_DEPENDENCIES`) are rejected (`[tooling/xtask/src/workspace.rs:tests]`).
- **Foreground Thread & Render Discipline**: Synchronous GPUI foreground contexts (`App`, `Context<T>`, `Window`) and `Render::render` / `RenderOnce::render` implementations must not perform blocking filesystem/network/process/sync I/O or synchronous entity updates/notifications during render (`[tooling/lints/src/blocking_io_on_foreground.rs:BLOCKING_IO_ON_FOREGROUND]`, `[tooling/lints/src/entity_update_in_render.rs:ENTITY_UPDATE_IN_RENDER]`, `[tooling/lints/src/notify_in_render.rs:NOTIFY_IN_RENDER]`).
- **Workflow Security Policy**: All generated GitHub Actions workflows (`.github/workflows/*.yml`) must declare top-level `permissions:` granting at most `contents: read` and must not interpolate `${{ ... }}` expressions directly inside `run:` shell blocks (`[tooling/xtask/src/tasks/workflow_checks.rs:validate]`).

## Global Side Effects

- **Local Persistence & Logs**: The Zed application and CLI create and mutate SQLite databases under the platform application support directory (`{db_dir}/0-{scope}/db.sqlite`), rotating log files (`zed.log`), IPC sockets/named pipes, and user configuration files (`[crates/db/src/db.rs:open_db]`, `[crates/zlog/src/sink.rs:init_output_file]`, `[crates/cli/src/main.rs:run]`).
- **Sandboxed & Host Process Execution**: Subsystems spawn language servers, debug adapters, PTY shells, WASI extensions, and sandboxed agent commands using macOS Seatbelt, Linux Bubblewrap (`SCM_RIGHTS` FD validation + seccomp-BPF), or Windows WSL (`[crates/sandbox/src/sandbox.rs:Sandbox]`, `[crates/extension_host/Cargo.toml:extension_host]`).
- **External Network & Cloud Integrations**: Client, server, and automation components communicate with `collab.zed.dev`, `cloud.zed.dev`, LiveKit, LLM provider APIs, GitHub REST/GraphQL APIs, Sentry, and DigitalOcean Spaces (`[crates/collab/src/main.rs:main]`, `[script/lib/blob-store.sh:upload_to_blob_store_with_acl]`).

## System Verification

- Run: `./script/clippy` and `cargo test --workspace`
