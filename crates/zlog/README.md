# Zlog

## Overview

Hierarchical scoped logging framework (`zlog`) for Zed applications and libraries, supporting runtime settings refresh, `ZED_LOG`/`RUST_LOG` environment filtering, and rotating file sinks.

Use the `ZED_LOG` environment variable to control logging output for Zed
applications and libraries. The variable accepts a comma-separated list of
directives that specify logging levels for different modules (crates). The
general format is for instance:

```
ZED_LOG=info,project=debug,agent=off
```

- Levels can be one of: `off`/`none`, `error`, `warn`, `info`, `debug`, or
  `trace`.
- You don't need to specify the global level; the default max level is `info` (`LEVEL_ENABLED_MAX_DEFAULT`), while bare scope names in `ZED_LOG` without an explicit `=level` suffix enable up to `trace`.

## Interface

- Initialization & Configuration: `[src/zlog.rs:init]`, `[src/zlog.rs:try_init]`, `[src/zlog.rs:init_test]`, `[src/zlog.rs:process_env]`, `[src/zlog.rs:Zlog]`
- Scoped Logging & Macros: `[src/zlog.rs:SCOPE_DEPTH_MAX]`, `[src/zlog.rs:Scope]`, `[src/zlog.rs:ScopeRef]`, `[src/zlog.rs:ScopeAlloc]`, `[src/zlog.rs:Logger]`, `[src/zlog.rs:Timer]`, `[src/zlog.rs:scoped_logger]`, `[src/zlog.rs:default_logger]`, `[src/zlog.rs:crate_name]`, `[src/zlog.rs:log]`, `[src/zlog.rs:trace]`, `[src/zlog.rs:debug]`, `[src/zlog.rs:info]`, `[src/zlog.rs:warn]`, `[src/zlog.rs:error]`, `[src/zlog.rs:time]`, `[src/zlog.rs:scoped]`
- Filtering & Sinks: `[src/filter.rs:LEVEL_ENABLED_MAX_DEFAULT]`, `[src/filter.rs:LEVEL_ENABLED_MAX_CONFIG]`, `[src/filter.rs:init_env_filter]`, `[src/filter.rs:is_possibly_enabled_level]`, `[src/filter.rs:is_scope_enabled]`, `[src/filter.rs:refresh_from_settings]`, `[src/filter.rs:ScopeMap]`, `[src/filter.rs:ScopeMapEntry]`, `[src/filter.rs:EnabledStatus]`, `[src/sink.rs:Record]`, `[src/sink.rs:init_output_stdout]`, `[src/sink.rs:init_output_stderr]`, `[src/sink.rs:init_output_file]`, `[src/sink.rs:submit]`, `[src/sink.rs:flush]`

## Invariants

- Scope hierarchy depth is fixed at a maximum of 4 (`[src/zlog.rs:SCOPE_DEPTH_MAX]`, `[src/zlog.rs:scoped_logger]`).
- `ENV_FILTER`, `SINK_FILE_PATH`, and `SINK_FILE_PATH_ROTATE` `OnceLock`s can only be initialized once (`[src/filter.rs:init_env_filter]`, `[src/sink.rs:init_output_file]`).
- The log file is rotated when its size reaches or exceeds 1 MB (`1,048,576` bytes) (`[src/sink.rs:SINK_FILE_SIZE_BYTES_MAX]`).
- At most one global max level directive is permitted in an environment filter string, and directive levels must not contain `'='` (`[src/env_config.rs:parse]`).

## Side Effects

- Installs the global logger via `log::set_logger(&ZLOG)` and reads `ZED_LOG`, `RUST_LOG`, and `CI` environment variables (`[src/zlog.rs:try_init]`, `[src/zlog.rs:get_env_config]`).
- Updates global `SCOPE_MAP` and `LEVEL_ENABLED_MAX_CONFIG` state when refreshed from settings (`[src/filter.rs:refresh_from_settings]`).
- Writes formatted log records to stdout, stderr, and/or the configured log file, rotating and truncating the log file on disk when `SINK_FILE_SIZE_BYTES_MAX` is exceeded (`[src/sink.rs:init_output_file]`, `[src/sink.rs:submit]`).

## Verification

- Run: `cargo test -p zlog`
