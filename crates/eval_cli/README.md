# eval-cli

## Overview

Headless Rust binary for running Zed's agent in evaluation and benchmark
environments. It is designed for containerized harnesses such as
[Harbor](https://harborframework.com/) and Pier, where the repository is already
checked out and model API keys are provided via environment variables.

`eval-cli` uses the same `NativeAgent` + `AcpThread` pipeline as the production
Zed editor: a full agentic loop with tool calls, subagents, and retries, without
a GUI.

This directory also contains `zed_eval/`, the Python `zed-eval` package used to
build this binary, launch remote benchmark runs on Modal/Harbor/Pier, and fetch
results. For normal benchmark orchestration, start with
[`./zed_eval/README.md`](./zed_eval/README.md).

## Building

### Native, for local testing on the same OS

```sh
cargo build --release -p eval_cli
```

### Linux x86_64, for Harbor/Pier sandboxes

Harbor and Pier containers run Linux x86_64. From the repository root, use the
Docker-based build script:

```sh
crates/eval_cli/script/build-linux
```

This produces `target/eval-cli`, an x86_64 Linux ELF binary. You can also
specify a custom output path:

```sh
crates/eval_cli/script/build-linux --output ~/bin/eval-cli-linux
```

## Standalone usage

```sh
eval-cli \
  --workdir /testbed \
  --model anthropic/claude-sonnet-4-6-latest \
  --instruction "Fix the bug described in..." \
  --timeout 600 \
  --output-dir /logs/agent
```

`eval-cli` reads provider API keys from environment variables such as
`ANTHROPIC_API_KEY` and `OPENAI_API_KEY`. It writes `result.json`, `thread.md`,
and `thread.json` to the output directory.

### Exit codes

| Code | Meaning |
| --- | --- |
| 0 | Agent finished |
| 1 | Error, such as model/auth/runtime failure |
| 2 | Timeout |
| 3 | Interrupted by SIGTERM or SIGINT |

## Running benchmarks

Most benchmark runs should use the Python `zed-eval` CLI instead of invoking
`eval-cli` directly. From the repository root:

```sh
crates/eval_cli/script/install-zed-eval
zed-eval doctor --create-volume
zed-eval run rf --from local --n-tasks 2
```

For one-off source runs without installing the tool globally, use
`crates/eval_cli/script/zed-eval <args>`.

See [`./zed_eval/README.md`](./zed_eval/README.md) for supported benchmarks, remote
builds, Modal setup, reporting, rejudging, baselines, and Harbor/Pier installed
agent usage.

## Interface

- Binary & CLI arguments: `[Cargo.toml:eval-cli]`, `[src/main.rs:Args]`, `[src/main.rs:EvalResult]`, `[src/main.rs:main]`
- Exit codes & environment constants: `[src/main.rs:EXIT_OK]`, `[src/main.rs:EXIT_ERROR]`, `[src/main.rs:EXIT_TIMEOUT]`, `[src/main.rs:EXIT_INTERRUPTED]`, `[src/main.rs:OPENAI_COMPATIBLE_PROVIDERS_ENV]`, `[src/main.rs:ANTHROPIC_AVAILABLE_MODELS_ENV]`
- Headless initialization: `[src/headless.rs:AgentCliAppState]`, `[src/headless.rs:init]`
- Subdirectories:
  - [`zed_eval`](./zed_eval/README.md) — Python CLI and installed-agent package for building `eval-cli`, launching Modal/Harbor/Pier benchmark runs, and fetching results.

## Invariants

- Instruction text (from `--instruction` or `stdin`) and optional `--instruction-suffix-file` content must not be empty after trimming (`[src/main.rs:read_instruction]`).
- Model discovery polls every 100ms (`MODEL_DISCOVERY_POLL_INTERVAL`) up to 30s (`MODEL_DISCOVERY_TIMEOUT`) and requires the selected model's provider to be authenticated (`[src/main.rs:wait_for_model]`).
- Non-Anthropic providers require exact model ID matches; Anthropic models allow matching after stripping `-latest`, `-thinking`, and `-1m-context` suffixes or matching an 8-digit `-YYYYMMDD` date suffix (`[src/main.rs:model_id_matches_selected]`).
- Configures user settings with `agent.tool_permissions.default = "allow"`, `autosave = "off"`, and `format_on_save = "off"`; if `ZED_EVAL_DISABLE_TOOLS` is set, defines an `"eval"` profile excluding those tools from `WRITE_TOOLS` (`[src/main.rs:run_agent]`).

## Side Effects

- When `--printenv` is passed, prints environment variables as JSON to stdout via `util::shell_env::print_env()` and exits (`[src/main.rs:main]`).
- Initializes `env_logger`, registers a `ctrlc` signal handler, creates `--output-dir` on disk, runs a headless GPUI application with Tokio and HTTP clients, writes `result.json`, `thread.md`, and `thread.json` to `--output-dir`, and exits with code `0`, `1`, `2`, or `3` (`[src/main.rs:main]`, `[src/headless.rs:init]`).

## Verification

- Run: `cargo test -p eval_cli`
