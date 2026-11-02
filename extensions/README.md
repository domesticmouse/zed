# Zed Extensions

## Overview

This directory contains extensions for Zed that are largely maintained by the Zed team. They currently live in the Zed repository for ease of maintenance.

If you are looking for the Zed extension registry, see the [`zed-industries/extensions`](https://github.com/zed-industries/extensions) repo.

Currently, Zed includes support for a number of languages without requiring installing an extension. Their grammars, queries, and configs can be found under [`crates/grammars/src`](https://github.com/zed-industries/zed/tree/main/crates/grammars/src), and their language server adapters under [`crates/languages/src`](https://github.com/zed-industries/zed/tree/main/crates/languages/src).

Support for all other languages is done via extensions. This directory ([extensions/](https://github.com/zed-industries/zed/tree/main/extensions/)) contains some of the officially maintained extensions. These extensions use the same [zed_extension_api](https://docs.rs/zed_extension_api/latest/zed_extension_api/) available to all [Zed Extensions](https://zed.dev/extensions) for providing [language servers](https://zed.dev/docs/extensions/languages#language-servers), [tree-sitter grammars](https://zed.dev/docs/extensions/languages#grammar) and [tree-sitter queries](https://zed.dev/docs/extensions/languages#tree-sitter-queries).

You can find the other officially maintained extensions in the [zed-extensions organization](https://github.com/zed-extensions).

See the docs for [Developing an Extension Locally](https://zed.dev/docs/extensions/developing-extensions#developing-an-extension-locally) for how to work with one of these extensions.

## Subdirectories & Boundaries

- `glsl`: [./glsl/README.md](./glsl/README.md) - OpenGL Shading Language (GLSL) grammar, queries, and `glsl_analyzer` language server integration (`zed_glsl`).
- `html`: [./html/README.md](./html/README.md) - HTML grammar, JS/CSS injections, and npm-provisioned `vscode-html-language-server` integration (`zed_html`).
- `proto`: [./proto/README.md](./proto/README.md) - Protocol Buffers (`.proto`) grammar, queries, and multi-server LSP integration for `buf`, `protobuf-language-server`, and `protols` (`zed_proto`).
- `test-extension`: [./test-extension/README.md](./test-extension/README.md) - WASI integration test fixture extension (`zed_test_extension`) used by `extension` and `extension_host` crate tests.
- `workflows`: `./workflows/run_tests.yml` - Generated GitHub Actions CI workflows (`run_tests.yml`, `shared/bump_version.yml`) for extension repositories.
