# zed_glsl

## Overview

`zed_glsl` (`extensions/glsl`) provides OpenGL Shading Language (GLSL) syntax highlighting, queries, and `glsl_analyzer` Language Server Protocol (LSP) lifecycle management for Zed.

## Interface

- `[extension.toml:id]`: Registers extension `id = "glsl"`, `version = "0.2.5"`, `schema_version = 1`, grammar `glsl` (`https://github.com/theHamsta/tree-sitter-glsl`), and language server `glsl_analyzer`.
- `[languages/glsl/config.toml:name]`: Configures `GLSL` file suffixes (`vert`, `frag`, `tesc`, `tese`, `geom`, `comp`, `task`, `mesh`, `rgen`, `rint`, `rahit`, `rchit`, `rmiss`, `rcall`, `glsl`) and `#version \d+` first-line matching alongside Tree-sitter queries (`highlights.scm`, `brackets.scm`, `indents.scm`, `injections.scm`).
- `[src/glsl.rs:GlslExtension]`: Implements `zed_extension_api::Extension` and exports it via `zed::register_extension!(GlslExtension)`.
- `[src/glsl.rs:GlslExtension::language_server_command]`: Resolves and returns the `zed::Command` for `glsl_analyzer`.
- `[src/glsl.rs:GlslExtension::language_server_workspace_configuration]`: Reads `LspSettings::for_worktree("glsl_analyzer", worktree)` and returns `{"glsl_analyzer": settings}`.

## Invariants

- `[src/glsl.rs:GlslExtension::language_server_binary_path]`: Resolves `glsl_analyzer` in strict precedence order: first `worktree.which("glsl_analyzer")` on `PATH`, then `self.cached_binary_path` if still present on disk, and finally queries `nolanderc/glsl_analyzer` GitHub releases (skipping the download if `glsl_analyzer-{version}/bin/glsl_analyzer` already exists on disk).

## Side Effects

- `[src/glsl.rs:GlslExtension::language_server_binary_path]`: Queries GitHub releases for `nolanderc/glsl_analyzer`, downloads and extracts `{arch}-{os}.zip` into `glsl_analyzer-{version}/` when not already present, marks `glsl_analyzer-{version}/bin/glsl_analyzer` executable via `zed::make_file_executable`, and removes any entry in `"."` whose filename does not match `version_dir` via `fs::remove_dir_all`.

## Verification

- Run: `cargo check -p zed_glsl`
