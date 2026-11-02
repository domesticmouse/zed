# zed_html

## Overview

`zed_html` (`extensions/html`) provides HTML syntax highlighting, JavaScript/CSS language injections, outline/indentation queries, and `vscode-html-language-server` LSP integration for Zed.

## Interface

- `[extension.toml:id]`: Registers extension `id = "html"`, `version = "0.3.2"`, `schema_version = 1`, grammar `html` (`https://github.com/tree-sitter/tree-sitter-html`), and language server `vscode-html-language-server` (`HTML` -> `"html"`, `CSS` -> `"css"`).
- `[languages/html/config.toml:name]`: Configures `HTML` file suffixes (`html`, `htm`, `shtml`), Prettier parser `"html"`, and Tree-sitter queries (`highlights.scm`, `injections.scm`, `brackets.scm`, `indents.scm`, `outline.scm`, `overrides.scm`).
- `[src/html.rs:HtmlExtension]`: Implements `zed_extension_api::Extension` and exports it via `zed::register_extension!(HtmlExtension)`.
- `[src/html.rs:HtmlExtension::language_server_command]`: Launches either a `PATH`-resolved `vscode-html-language-server --stdio` binary or `zed::node_binary_path()` running the npm-installed server script with `--stdio`.
- `[src/html.rs:HtmlExtension::language_server_workspace_configuration]`: Returns `LspSettings::for_worktree(server_id.as_ref(), worktree)?.settings`.
- `[src/html.rs:HtmlExtension::language_server_initialization_options]`: Returns `Some(serde_json::json!({"provideFormatter": true}))`.

## Invariants

- `[src/html.rs:BINARY_NAME]` & `[src/html.rs:PACKAGE_NAME]` & `[src/html.rs:SERVER_PATH]`: Uses binary `"vscode-html-language-server"` and npm package `@zed-industries/vscode-langservers-extracted` at `node_modules/@zed-industries/vscode-langservers-extracted/bin/vscode-html-language-server`.
- `[src/html.rs:HtmlExtension::server_script_path]`: Checks `zed::npm_package_installed_version(PACKAGE_NAME)` to skip installation when the latest version is already installed; if `zed::npm_install_package` fails, suppresses the error if `self.server_exists()` is already true, and verifies `self.server_exists()` afterward (returning an error if `SERVER_PATH` is still missing on disk).

## Side Effects

- `[src/html.rs:HtmlExtension::server_script_path]`: Queries `zed::npm_package_installed_version` and `zed::npm_package_latest_version`, and runs `zed::npm_install_package` to install `@zed-industries/vscode-langservers-extracted` into the extension's working directory when missing or outdated.

## Verification

- Run: `cargo check -p zed_html`
