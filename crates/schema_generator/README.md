# Zed Schema Generator

## Overview

Prints various Zed JSON schemas (`theme`, `icon_theme`, `project`) to stdout or an output file.

## Usage

```sh
cargo run -p schema_generator -- --help

cargo run -p schema_generator -- theme
cargo run -p schema_generator -- icon_theme
```

## Interface

- Binary & CLI types: `[Cargo.toml:schema-generator]`, `[src/main.rs:Args]`, `[src/main.rs:SchemaType]`, `[src/main.rs:main]`

## Invariants

- `#[clap(rename_all = "snake_case")]` restricts valid CLI `schema_type` arguments to `theme`, `icon_theme`, and `project` (`[src/main.rs:SchemaType]`).

## Side Effects

- Initializes the global logger via `env_logger::init()` and writes the generated JSON schema to `--output` (`File::create`) or stdout (`[src/main.rs:main]`).

## Verification

- Run: `cargo test -p schema_generator`
