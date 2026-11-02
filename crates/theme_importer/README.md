# Zed Theme Importer

## Overview

Command-line utility (`theme-importer`) that converts VS Code theme JSON files into Zed theme JSON files conforming to the Zed theme schema.

```sh
cargo run -p theme_importer -- dark-plus-syntax-color-theme.json --output output-theme.json
```

## Interface

- Binary & CLI types: `[Cargo.toml:theme-importer]`, `[src/main.rs:Args]`, `[src/main.rs:ThemeAppearanceJson]`, `[src/main.rs:ThemeMetadata]`, `[src/main.rs:main]`
- VS Code theme converter & syntax mapping: `[src/vscode/converter.rs:VsCodeThemeConverter]`, `[src/vscode/theme.rs:VsCodeTheme]`, `[src/vscode/syntax.rs:VsCodeTokenScope]`, `[src/vscode/syntax.rs:VsCodeTokenColor]`, `[src/vscode/syntax.rs:VsCodeTokenColorSettings]`, `[src/vscode/syntax.rs:ZedSyntaxToken]`

## Invariants

- Output JSON always sets `"$schema"` to `"https://zed.dev/schema/themes/v0.2.0.json"` (`[src/main.rs:ZED_THEME_SCHEMA_URL]`, `[src/main.rs:main]`).
- Imported themes default `appearance` to `ThemeAppearanceJson::Dark` and `window_background_appearance` to `WindowBackgroundContent::Opaque` (`[src/main.rs:main]`, `[src/vscode/converter.rs:VsCodeThemeConverter::convert]`).
- Token color candidates without a `foreground` color are ignored during syntax matching, and scopes earlier in `to_vscode()` are weighted higher (`[src/vscode/syntax.rs:ZedSyntaxToken::find_best_token_color_match]`).

## Side Effects

- Initializes `TermLogger` on stderr, reads the input VS Code theme file from `args.theme_path`, and writes the converted Zed theme JSON to `args.output` or stdout (`[src/main.rs:main]`).

## Verification

- Run: `cargo test -p theme_importer`
