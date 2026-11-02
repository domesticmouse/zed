# zed_proto

## Overview

`zed_proto` (`extensions/proto`) provides Protocol Buffers (`.proto`) Tree-sitter grammar and query support along with multi-server LSP dispatch for `buf`, `protobuf-language-server`, and `protols`.

## Interface

- `[extension.toml:id]`: Registers extension `id = "proto"`, `version = "0.3.4"`, `schema_version = 1`, grammar `proto` (`https://github.com/coder3101/tree-sitter-proto`), and language servers `buf`, `protobuf-language-server`, and `protols`.
- `[languages/proto/config.toml:name]`: Configures `Proto` `.proto` file recognition and Tree-sitter queries (`highlights.scm`, `indents.scm`, `injections.scm`, `outline.scm`, `textobjects.scm`).
- `[src/proto.rs:ProtobufExtension]`: Implements `zed_extension_api::Extension` (`zed::register_extension!(ProtobufExtension)`), routing `language_server_command`, `language_server_workspace_configuration`, and `language_server_initialization_options` across `BufLsp`, `ProtobufLanguageServer`, and `ProtoLs`.
- `[src/language_servers/buf.rs:BufLsp]`: Language server adapter for `buf` (`SERVER_NAME = "buf"`, default args `["lsp", "serve"]`).
- `[src/language_servers/protobuf_language_server.rs:ProtobufLanguageServer]`: Language server adapter for `protobuf-language-server` (`SERVER_NAME = "protobuf-language-server"`, default args `["-logs", ""]`).
- `[src/language_servers/protols.rs:ProtoLs]`: Language server adapter for `protols` (`SERVER_NAME = "protols"`).
- `[src/language_servers/util.rs:remove_outdated_versions]`: Helper that removes outdated `{server_id}-*` directories after a new release is downloaded.

## Invariants

- `[src/language_servers/buf.rs:BufLsp::language_server_binary]` & `[src/language_servers/protols.rs:ProtoLs::language_server_binary]` & `[src/language_servers/protobuf_language_server.rs:ProtobufLanguageServer::language_server_binary]`: Explicit `LspSettings::for_worktree(...).binary` settings take precedence over cached binaries and `worktree.which(...)` before any GitHub release download is attempted.
- `[src/language_servers/protobuf_language_server.rs:ProtobufLanguageServer::language_server_binary]`: `protobuf-language-server` is never auto-downloaded; it must be configured in settings or present on `PATH`.

## Side Effects

- `[src/language_servers/buf.rs:BufLsp::language_server_binary]` & `[src/language_servers/protols.rs:ProtoLs::language_server_binary]`: Queries GitHub releases (`bufbuild/buf` and `coder3101/protols`), downloads release binaries/archives into the extension working directory, marks binaries executable, and deletes older cached version directories via `util::remove_outdated_versions`.

## Verification

- Run: `cargo check -p zed_proto`
