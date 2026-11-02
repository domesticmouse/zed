# xtask

## Overview

`xtask` (`tooling/xtask`) is Zed's workspace automation binary (`cargo xtask`), providing subcommands for CI workflow generation and security validation, workspace linting, license and manifest conformity checks, GPUI release graph analysis, release compliance auditing, local WebRTC SDK provisioning, GPUI WebAssembly example serving, and Linux/WSL Bubblewrap sandbox testing.

## Interface

- `[src/main.rs:Args]` & `[src/main.rs:CliCommand]`: Top-level `clap` CLI dispatcher for `cargo xtask`.
- `[src/workspace.rs:load_workspace]`: Loads the workspace `cargo_metadata::Metadata` via `cargo metadata`.
- `[src/tasks/clippy.rs:run_clippy]`: Implements `cargo xtask clippy [--fix] [-p <package>]`.
- `[src/tasks/compliance.rs:check_compliance]`: Implements `cargo xtask compliance <version <version_tag> --report-path <report_path> [--branch <branch>] | single <commit_sha>>` using the `compliance` crate and `GITHUB_APP_ID` / `GITHUB_APP_KEY`.
- `[src/tasks/licenses.rs:run_licenses]`: Implements `cargo xtask licenses` verifying `LICENSE-APACHE` or `LICENSE-GPL` symlinks in every workspace crate.
- `[src/tasks/package_conformity.rs:run_package_conformity]`: Implements `cargo xtask package-conformity` checking `[lints] workspace = true` and `[workspace.dependencies]` inheritance.
- `[src/tasks/gpui.rs:run_gpui]`: Implements `cargo xtask gpui <plan --sha <sha> | why <crate_name>>` using `src/tasks/gpui/{crate_graph,publish_plan,validation}.rs`.
- `[src/tasks/workflows.rs:run_workflows]`: Implements `cargo xtask workflows [<sha>]`, regenerating `.github/workflows/*.yml` and `extensions/workflows/**/*.yml` and running `workflow_checks::validate`.
- `[src/tasks/workflow_checks.rs:validate]` & `[src/tasks/workflow_checks.rs:validate_run_command]`: Implements `cargo xtask check-workflows` using `check_permissions.rs` and `check_run_patterns.rs`.
- `[src/tasks/setup_webrtc.rs:run_setup_webrtc]`: Implements `cargo xtask setup-webrtc [--force] [--triple <triple>] [--no-cargo-config]`.
- `[src/tasks/web_examples.rs:run_web_examples]`: Implements `cargo xtask web-examples [--release] [--port <port>] [--no-serve]`.
- `[src/tasks/sandbox_tests.rs:run_sandbox_tests]`: Implements `cargo xtask sandbox-tests [tests...] [--system <system>] [--rebuild] [--list]`.
- `[src/tasks/wsl_sandbox_tests.rs:run_wsl_sandbox_tests]`: Implements Windows `cargo xtask wsl-sandbox-tests [--require-enforced] [--release]`.

## Invariants

- `[src/workspace.rs:tests::no_forbidden_objc_dependencies]`: Legacy Objective-C crates (`block`, `cocoa`, `cocoa-foundation`, `metal`, `objc`, `objc-foundation`, `objc_id`) are forbidden across workspace manifests and resolved dependencies in favor of `objc2*` and `block2`.
- `[src/workspace.rs:tests::no_test_support_in_non_dev_dependencies]`: No workspace crate may enable the `test-support` feature on a non-dev dependency.
- `[src/workspace.rs:tests::no_forbidden_dependencies_between_feature_crates]`: Forbids the 12 directed non-dev dependency edges listed in `FORBIDDEN_DEPENDENCIES` between UI feature crates (`agent_ui`, `git_ui`, `file_finder`, `project_panel`, `open_path_prompt`, `picker`, `editor`, `search`, `sidebar`, `title_bar`).
- `[src/tasks/workflow_checks/check_permissions.rs:validate_permissions]` & `[src/tasks/workflow_checks/check_run_patterns.rs:validate_run_command]`: All GitHub Actions workflows must declare a top-level `permissions:` block granting at most `contents: read` and must not interpolate `${{ ... }}` expressions directly inside `run:` scripts.

## Side Effects

- `[src/tasks/workflows.rs:run_workflows]`: Deletes and regenerates `.yml` files in `.github/workflows/` and `extensions/workflows/`.
- `[src/tasks/setup_webrtc.rs:run_setup_webrtc]`: Downloads prebuilt WebRTC archives from GitHub into `.webrtc-sys/` and updates `.gitignore` and `~/.cargo/config.toml` (`LK_CUSTOM_WEBRTC`).
- `[src/tasks/web_examples.rs:run_web_examples]`: Builds Wasm targets into `target/web-examples/` and spawns a local `python3` HTTP server.
- `[src/tasks/compliance.rs:check_compliance]`: Queries GitHub APIs, mutates PR labels, and writes compliance report files.

## Verification

- Run: `cargo test -p xtask`
