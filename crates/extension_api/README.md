# The Zed Rust Extension API

## Overview

This crate lets you write extensions for Zed in Rust, compiled to WebAssembly (`wasm32-wasip2`) against the WIT extension guest interface.

## Extension Manifest

You'll need an `extension.toml` file at the root of your extension directory, with the following structure:

```toml
id = "my-extension"
name = "My Extension"
description = "..."
version = "0.0.1"
schema_version = 1
authors = ["Your Name <you@example.com>"]
repository = "https://github.com/your/extension-repository"
```

## Cargo metadata

Zed extensions are packaged as WebAssembly files. In your Cargo.toml, you'll
need to set your `crate-type` accordingly:

```toml
[dependencies]
zed_extension_api = "0.7.0"

[lib]
crate-type = ["cdylib"]
```

## Implementing an Extension

To define your extension, create a type that implements the `Extension` trait, and register it.

```rust
use zed_extension_api as zed;

struct MyExtension {
    // ... state
}

impl zed::Extension for MyExtension {
    // ...
}

zed::register_extension!(MyExtension);
```

## Testing your extension

To run your extension in Zed as you're developing it:

- Make sure you have [Rust installed](https://www.rust-lang.org/learn/get-started)
- Have the `wasm32-wasip2` target installed (`rustup target add wasm32-wasip2`)
- Open the extensions view using the `zed: extensions` action in the command palette.
- Click the `Install Dev Extension` button in the top right
- Choose the path to your extension directory.

## Compatible Zed versions

Extensions created using newer versions of the Zed extension API won't be compatible with older versions of Zed.

Here is the compatibility of the `zed_extension_api` with versions of Zed (note: `0.8.0` is the current in-tree version staged on `Dev` and `Nightly` release channels via `wit/since_v0.8.0`, while `Stable` and `Preview` support up to `0.7.0` via `wit/since_v0.6.0`):

| Zed version | `zed_extension_api` version |
| ----------- | --------------------------- |
| `Dev` / `Nightly` | `0.0.1` - `0.8.0`     |
| `Stable` / `Preview` | `0.0.1` - `0.7.0`  |
| `0.192.x`   | `0.0.1` - `0.6.0`           |
| `0.186.x`   | `0.0.1` - `0.5.0`           |
| `0.184.x`   | `0.0.1` - `0.4.0`           |
| `0.178.x`   | `0.0.1` - `0.3.0`           |
| `0.162.x`   | `0.0.1` - `0.2.0`           |
| `0.149.x`   | `0.0.1` - `0.1.0`           |
| `0.131.x`   | `0.0.1` - `0.0.6`           |
| `0.130.x`   | `0.0.1` - `0.0.5`           |
| `0.129.x`   | `0.0.1` - `0.0.4`           |
| `0.128.x`   | `0.0.1`                     |

## Interface

- Extension trait & registration: `[src/extension_api.rs:Extension]`, `[src/extension_api.rs:register_extension!]`, `[src/extension_api.rs:register_extension]`, `[src/extension_api.rs:Result]`, `[src/extension_api.rs:ZED_API_VERSION]`, `[src/extension_api.rs:Guest]`
- Core WIT & Host types: `[src/extension_api.rs:LanguageServerId]`, `[src/extension_api.rs:ContextServerId]`, `[src/extension_api.rs:Project]`, `[src/extension_api.rs:Worktree]`, `[src/extension_api.rs:KeyValueStore]`, `[src/extension_api.rs:Command]`, `[src/extension_api.rs:EnvVars]`, `[src/extension_api.rs:Range]`, `[src/extension_api.rs:CodeLabel]`, `[src/extension_api.rs:CodeLabelSpan]`, `[src/extension_api.rs:CodeLabelSpanLiteral]`, `[src/extension_api.rs:LanguageServerInstallationStatus]`, `[src/extension_api.rs:set_language_server_installation_status]`, `[src/extension_api.rs:DownloadedFileType]`, `[src/extension_api.rs:download_file]`, `[src/extension_api.rs:make_file_executable]`, `[src/extension_api.rs:ContextServerConfiguration]`
- Platform, GitHub, Node & Slash Commands: `[src/extension_api.rs:Architecture]`, `[src/extension_api.rs:Os]`, `[src/extension_api.rs:current_platform]`, `[src/extension_api.rs:GithubRelease]`, `[src/extension_api.rs:GithubReleaseAsset]`, `[src/extension_api.rs:GithubReleaseOptions]`, `[src/extension_api.rs:github_release_by_tag_name]`, `[src/extension_api.rs:latest_github_release]`, `[src/extension_api.rs:node_binary_path]`, `[src/extension_api.rs:npm_install_package]`, `[src/extension_api.rs:npm_package_installed_version]`, `[src/extension_api.rs:npm_package_latest_version]`, `[src/extension_api.rs:SlashCommand]`, `[src/extension_api.rs:SlashCommandArgumentCompletion]`, `[src/extension_api.rs:SlashCommandOutput]`, `[src/extension_api.rs:SlashCommandOutputSection]`
- Debug Adapter & Tasks: `[src/extension_api.rs:DebugAdapterBinary]`, `[src/extension_api.rs:DebugConfig]`, `[src/extension_api.rs:DebugRequest]`, `[src/extension_api.rs:DebugScenario]`, `[src/extension_api.rs:DebugTaskDefinition]`, `[src/extension_api.rs:LaunchRequest]`, `[src/extension_api.rs:AttachRequest]`, `[src/extension_api.rs:BuildTaskDefinition]`, `[src/extension_api.rs:BuildTaskDefinitionTemplatePayload]`, `[src/extension_api.rs:BuildTaskTemplate]`, `[src/extension_api.rs:TaskTemplate]`, `[src/extension_api.rs:StartDebuggingRequestArguments]`, `[src/extension_api.rs:StartDebuggingRequestArgumentsRequest]`, `[src/extension_api.rs:TcpArguments]`, `[src/extension_api.rs:TcpArgumentsTemplate]`, `[src/extension_api.rs:resolve_tcp_template]`
- Submodules (`lsp`, `http_client`, `process`, `settings`): `[src/extension_api.rs:lsp::Completion]`, `[src/extension_api.rs:lsp::CompletionKind]`, `[src/extension_api.rs:lsp::InsertTextFormat]`, `[src/extension_api.rs:lsp::Symbol]`, `[src/extension_api.rs:lsp::SymbolKind]`, `[src/http_client.rs:HttpMethod]`, `[src/http_client.rs:HttpRequest]`, `[src/http_client.rs:HttpRequestBuilder]`, `[src/http_client.rs:HttpResponse]`, `[src/http_client.rs:HttpResponseStream]`, `[src/http_client.rs:RedirectPolicy]`, `[src/http_client.rs:fetch]`, `[src/http_client.rs:fetch_stream]`, `[src/process.rs:Command]`, `[src/process.rs:Output]`, `[src/settings.rs:LanguageSettings]`, `[src/settings.rs:LspSettings]`, `[src/settings.rs:ContextServerSettings]`, `[src/settings.rs:CommandSettings]`

## Invariants

- Global static `EXTENSION` must be initialized via `register_extension` before any WIT `Guest` callbacks are invoked (`[src/extension_api.rs:extension]`).
- On `target_os = "wasi"`, `register_extension!` overrides C `chdir` to set `errno = 58` (`NOTSUP`) and return `-1`, forbidding extensions from changing the working directory, and initializes `__wasilibc_cwd` from `PWD` (`[src/extension_api.rs:register_extension!]`).
- `HttpRequestBuilder::build` requires both `method` and `url` to be set and defaults `redirect_policy` to `RedirectPolicy::NoFollow` (`[src/http_client.rs:HttpRequestBuilder::build]`).
- `LanguageSettings::tab_size` must be a `NonZeroU32` (`[wit/since_v0.8.0/settings.rs:LanguageSettings]`).

## Side Effects

- `build.rs` writes the 6-byte big-endian `CARGO_PKG_VERSION` to `$OUT_DIR/version_bytes`, which `ZED_API_VERSION` embeds into the `"zed:api-version"` Wasm link section on `wasm32` (`[build.rs:main]`, `[src/extension_api.rs:ZED_API_VERSION]`).
- Invokes WIT host imports to mutate global extension state, update language server installation status, perform HTTP requests, spawn host child processes, and query host settings (`[src/extension_api.rs:register_extension]`, `[src/extension_api.rs:set_language_server_installation_status]`, `[src/http_client.rs:HttpRequest::fetch]`, `[src/process.rs:Command::output]`, `[src/settings.rs:get_settings]`).

## Verification

- Run: `cargo test -p extension_api`
