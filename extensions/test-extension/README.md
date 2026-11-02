# Test Extension

## Overview

This is a test extension (`extensions/test-extension`, crate `zed_test_extension`) that we use in the tests for the `extension` and `extension_host` crates.

Originally based off the Gleam extension.

## Interface

- `[extension.toml:id]`: Registers extension `id = "test-extension"`, `version = "0.1.0"`, `schema_version = 1`, language server `gleam` (`Gleam LSP`), grammar `gleam`, and `process:exec` capabilities for `echo "hello from a child process!"` / `cmd /C echo "hello from a child process!"`.
- `[languages/gleam/config.toml:name]`: Configures `Gleam` language grammar and Tree-sitter query files (`highlights.scm`, `indents.scm`, `outline.scm`).
- `[src/test_extension.rs:TestExtension]`: Implements `zed_extension_api::Extension` (`zed::register_extension!(TestExtension)`).
- `[src/test_extension.rs:TestExtension::language_server_command]`: Exercises WASI filesystem operations, capability-gated child process execution (`process::Command`), and GitHub release asset downloading before returning the `gleam lsp` command.
- `[src/test_extension.rs:TestExtension::label_for_completion]`: Formats Gleam LSP completion items with `detail` (`CompletionKind::Function | CompletionKind::Constructor` and other kinds) into `CodeLabel` spans.

## Invariants

- `[src/test_extension.rs:TestExtension::language_server_binary_path]`: Asserts that `std::env::current_dir()` ends with `"test-extension"`, that relative and absolute directories/files (`dir-created-with-abs-path`, `./dir-created-with-rel-path`, `file-created-with-rel-path`, `file-created-with-abs-path`) can be created and read back inside the WASI sandbox, and that the capability-allowed `echo` / `cmd` subprocess executes via `process::Command::output()`.

## Side Effects

- `[src/test_extension.rs:TestExtension::language_server_binary_path]`: Creates test directories/files (`dir-created-with-abs-path`, `./dir-created-with-rel-path`, `file-created-with-rel-path`, `file-created-with-abs-path`), spawns `echo` / `cmd` via `zed_extension_api::process::Command`, downloads a `.tar.gz` release archive from `gleam-lang/gleam` (mocked in tests), and prunes older `gleam-*` directories.

## Verification

- Run: `cargo test -p extension_host`
