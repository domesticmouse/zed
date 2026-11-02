# Cli

## Overview

Command-line interface (`cli`) and IPC bridge for launching, controlling, and passing file/workspace requests to the Zed application binary across macOS, Linux/FreeBSD, Windows, and WSL.

## Testing

You can test your changes to the `cli` crate by first building the main zed binary:

```
cargo build -p zed
```

And then building and running the `cli` crate with the following parameters:

```
 cargo run -p cli -- --zed ./target/debug/zed.exe
```

## Interface

- Library IPC types: `[src/cli.rs:ipc]`, `[src/cli.rs:IpcHandshake]`, `[src/cli.rs:OpenBehavior]`, `[src/cli.rs:CliBehaviorSetting]`, `[src/cli.rs:CliRequest]`, `[src/cli.rs:CliResponse]`, `[src/cli.rs:CliResponseSink]`, `[src/cli.rs:FORCE_CLI_MODE_ENV_VAR_NAME]`
- Binary entry points & flags: `[Cargo.toml:cli]`, `[Cargo.toml:no-bundled-uninstall]`, `[src/main.rs:Args]`, `[src/main.rs:main]`

## Invariants

- Refuses to execute as root on Unix platforms via `util::prevent_root_execution()` (`[src/main.rs:run]`).
- All file paths and `--diff` paths are canonicalized to absolute paths, and every `--diff` path must exist on disk (`[src/main.rs:parse_path_with_position]`, `[src/main.rs:diff_path_exists]`, `[src/main.rs:run]`).
- `--dev-server-token` and `--system-specs` are rejected on the CLI binary with actionable error messages (`[src/main.rs:Args::dev_server_token]`, `[src/main.rs:Args::system_specs]`, `[src/main.rs:run]`).
- Workspace open-mode flags (`--add`, `--new`, `--reuse`, `--existing`, `--classic`) mutually override one another (`[src/main.rs:Args]`).
- In WSL path parsing (`--wsl`), an `@`-separated user prefix must not be empty (`[src/main.rs:parse_path_in_wsl]`).

## Side Effects

- Establishes a one-shot IPC server (`IpcOneShotServer<IpcHandshake>`) and exchanges `CliRequest`/`CliResponse` messages with the Zed app instance over `ipc-channel`, Unix domain sockets, Windows named pipes, or macOS LaunchServices (`[src/cli.rs:IpcHandshake]`, `[src/main.rs:run]`, `[src/main.rs:linux::App::launch]`, `[src/main.rs:windows::App::launch]`, `[src/main.rs:mac_os::Bundle::launch]`).
- Spawns or execs external processes including the Zed application binary, `wsl.exe`, `osascript`, `sh` (for uninstall scripts), and socket-forwarding helpers for `--askpass` (`[src/main.rs:run]`, `[src/main.rs:parse_path_in_wsl]`, `[src/main.rs:linux::App::boot_background]`, `[src/main.rs:mac_os::Bundle::launch]`).
- Creates temporary files and directories in the OS temp directory for piped `stdin` (`-`), anonymous file descriptors, directory diff stubs, and `zed_dev.log` (`[src/main.rs:expand_directory_pair]`, `[src/main.rs:create_empty_stub]`, `[src/main.rs:run]`).
- Initializes a global Rayon thread pool, mutates process environment variables (`ZED_FORCE_CLI_MODE`, `LD_LIBRARY_PATH`, `ZED_UPDATE_EXPLANATION`), and exits via `std::process::exit` (`[src/main.rs:run]`).

## Verification

- Run: `cargo test -p cli`
