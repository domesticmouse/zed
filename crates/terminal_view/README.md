# Terminal View

## Overview

GPUI view, element, and dock panel integration (`terminal_view`) for Zed's integrated terminal emulator.

## Design Notes

Terminal support in Zed is split across two crates:
- `crates/terminal` (`terminal.rs` and `src/mappings/`), which contains the code for interacting with terminal emulator backends and maintaining the PTY event loop. Some behavior in `crates/terminal` is constrained by terminal protocols and standards.
- `crates/terminal_view`, which integrates the `Terminal` model from `crates/terminal` into GPUI workspaces and panes. The main entry points for GPUI are `src/terminal_view.rs` (`terminal_view::init`, `TerminalView`) and `src/terminal_panel.rs` (`TerminalPanel`).

ttys are created externally, and so can fail in unexpected ways. However, GPUI currently does not have an API for models than can fail to instantiate. `TerminalBuilder` solves this by using Rust's type system to split tty instantiation into a 2 step process: first attempt to create the file handles with `TerminalBuilder::new()`, check the result, then call `TerminalBuilder::subscribe(cx)` from within a model context.

The TerminalView struct abstracts over failed and successful terminals, passing focus through to the associated view and allowing clients to build a terminal without worrying about errors.

## Backend Boundary

`terminal.rs` exposes backend-neutral domain types such as terminal content, cells, modes, points, ranges, scroll commands, vi motions, hyperlinks, and search matches. UI code should depend on those types instead of importing backend-specific terminal types directly.

The current implementation is still Alacritty-backed, but the abstraction boundary keeps backend details concentrated in the terminal crate:

- `terminal_view` renders `TerminalContent` and dispatches backend-neutral actions.
- Panels and tools use terminal-domain types for mode, cursor, range, hyperlink, and search behavior.
- Backend-specific conversions stay near the terminal event loop and render snapshot code.

This keeps the user-facing terminal behavior unchanged while making future backend experiments reviewable as backend implementations instead of UI-wide refactors.

## Input

There are currently many distinct paths for getting keystrokes to the terminal:

1. Terminal specific characters and bindings. Things like ctrl-a mapping to ASCII control character 1, ANSI escape codes associated with the function keys, etc. These are caught with a raw key-down handler in the element and are processed immediately. This is done with the `try_keystroke()` method on Terminal

2. GPU Action handlers. GPUI clobbers a few vital keys by adding bindings to them in the global context. These keys are synthesized and then dispatched through the same `try_keystroke()` API as the above mappings

3. IME text. When the special character mappings fail, we pass the keystroke back to GPUI to hand it to the IME system. This comes back to us in the `EntityInputHandler::replace_text_in_range()` method (implemented on `TerminalInputHandler` in `src/terminal_element.rs`), and we then send that to the terminal directly, bypassing `try_keystroke()`.

4. Pasted text has a separate pathway.

Generally, there's a distinction between 'keystrokes that need to be mapped' and 'strings which need to be written'. I've attempted to unify these under the '.try_keystroke()' API and the `.input()` API (which try_keystroke uses) so we have consistent input handling across the terminal

## Interface

- Initialization & View: `[src/terminal_view.rs:init]`, `[src/terminal_view.rs:TerminalView]`, `[src/terminal_view.rs:TerminalMode]`, `[src/terminal_view.rs:ContentMode]`, `[src/terminal_view.rs:BlockProperties]`, `[src/terminal_view.rs:BlockContext]`, `[src/terminal_view.rs:ScrollTerminal]`, `[src/terminal_view.rs:default_working_directory]`
- Actions: `[src/terminal_view.rs:SendText]`, `[src/terminal_view.rs:SendKeystroke]`, `[src/terminal_view.rs:RerunTask]`, `[src/terminal_view.rs:RenameTerminal]`, `[src/terminal_panel.rs:Toggle]`, `[src/terminal_panel.rs:ToggleFocus]`
- Element & Layout: `[src/terminal_element.rs:TerminalElement]`, `[src/terminal_element.rs:LayoutState]`, `[src/terminal_element.rs:LayoutPoint]`, `[src/terminal_element.rs:BatchedTextRun]`, `[src/terminal_element.rs:BlockElementLayoutRect]`, `[src/terminal_element.rs:LayoutRect]`, `[src/terminal_element.rs:TerminalLayoutCell]`, `[src/terminal_element.rs:is_blank]`, `[src/terminal_element.rs:convert_color]`
- Panel & Scrollbar: `[src/terminal_panel.rs:init]`, `[src/terminal_panel.rs:TerminalPanel]`, `[src/terminal_panel.rs:prepare_task_for_spawn]`, `[src/terminal_panel.rs:new_terminal_pane]`, `[src/terminal_scrollbar.rs:TerminalScrollHandle]`

## Invariants

- Embedded terminals fall back to `ContentMode::Scrollable` when `total_lines > 1_000` (`[src/terminal_view.rs:TerminalView::MAX_EMBEDDED_LINES]`, `[src/terminal_view.rs:TerminalView::content_mode]`).
- When `read_only` is true, mouse input uses `MouseInputMode::LocalSelection` and PTY-mutating operations (`set_marked_text`, `commit_text`, `paste`, `send_text`, `clear`, `rerun_task`) are no-ops (`[src/terminal_view.rs:TerminalView::with_read_only]`).
- Task terminals (`terminal.task().is_some()`) cannot be renamed or serialized (`[src/terminal_view.rs:TerminalView::rename_terminal]`, `[src/terminal_view.rs:TerminalView::serialize]`, `[src/persistence.rs:serialize_pane]`).
- Block elements are laid out on an 8x24 subcell grid per cell (`BLOCK_SUBCELL_COLUMNS = 8`, `BLOCK_SUBCELL_LINES = 24`) and terminal width is clamped to at least `cell_width * 2.0` (`[src/terminal_element.rs:BLOCK_SUBCELL_COLUMNS]`, `[src/terminal_element.rs:TerminalElement::prepaint]`).
- Collaborative guest projects (`project.is_via_collab()`) cannot spawn tasks, and panel serialization is suppressed while `self.restoring` is true (`[src/terminal_panel.rs:TerminalPanel::spawn_task]`, `[src/terminal_panel.rs:TerminalPanel::serialize]`).

## Side Effects

- Creates and migrates the SQLite `terminals` table via `TerminalDb` and persists pane layout JSON to `KeyValueStore` under `TERMINAL_PANEL_KEY` (`[src/persistence.rs:TerminalDb]`, `[src/terminal_panel.rs:TerminalPanel::serialize]`, `[src/terminal_panel.rs:TerminalPanel::restore_serialized_state]`).
- Persists dock position and flexible size updates to the user settings file via `settings::update_settings_file` (`[src/terminal_panel.rs:TerminalPanel::set_position]`, `[src/terminal_panel.rs:TerminalPanel::set_flexible_size]`).
- Spawns PTY shells/tasks via `Project`, writes input bytes to the PTY, reads the OS clipboard, plays the system bell, and opens URLs and workspace file paths (`[src/terminal_view.rs:TerminalView]`).

## Verification

- Run: `cargo test -p terminal_view`
