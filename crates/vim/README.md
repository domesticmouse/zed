# Vim

## Overview

This contains the code for Zed's Vim emulation mode.

Vim mode in Zed is supposed to primarily "do what you expect": it mostly tries to copy vim exactly, but will use Zed-specific functionality when available to make things smoother. This means Zed will never be 100% vim compatible, but should be 100% vim familiar!

The backlog is maintained in the `#vim` channel notes.

## Testing against Neovim

If you are making a change to make Zed's behavior more closely match vim/nvim, you can create a test using the `NeovimBackedTestContext`.

For example, the following test checks that Zed and Neovim have the same behavior when running `*` in visual mode:

```rust
#[gpui::test]
async fn test_visual_star_hash(cx: &mut gpui::TestAppContext) {
    let mut cx = NeovimBackedTestContext::new(cx).await;

    cx.set_shared_state("ˇa.c. abcd a.c. abcd").await;
    cx.simulate_shared_keystrokes(["v", "3", "l", "*"]).await;
    cx.assert_shared_state("a.c. abcd ˇa.c. abcd").await;
}
```

To keep CI runs fast, by default the neovim tests use a cached JSON file that records what neovim did (see crates/vim/test_data),
but while developing this test you'll need to run it with the neovim flag enabled:

```sh
cargo test -p vim --features neovim test_visual_star_hash
```

This will run your keystrokes against a headless neovim and cache the results in the test_data directory. Note that neovim must be installed and reachable on your $PATH in order to run the feature.


## Testing zed-only behavior

Zed does more than vim/neovim in their default modes. The `VimTestContext` can be used instead. This lets you test integration with the language server and other parts of zed's UI that don't have a NeoVim equivalent.

## Interface

- Initialization, UI & Settings: `[src/vim.rs:init]`, `[src/vim.rs:ModeIndicator]`, `[src/mode_indicator.rs:ModeIndicator::new]`, `[src/vim.rs:InsertModeCursorShape]`, `[src/vim.rs:CursorShapeSettings]`, `[src/vim.rs:ModeContent]`, `[src/vim.rs:UseSystemClipboard]`, `[Cargo.toml:neovim]`
- Mode Switching Actions: `[src/vim.rs:ToggleVimMode]`, `[src/vim.rs:ToggleHelixMode]`, `[src/vim.rs:SwitchToNormalMode]`, `[src/vim.rs:SwitchToInsertMode]`, `[src/vim.rs:SwitchToReplaceMode]`, `[src/vim.rs:SwitchToVisualMode]`, `[src/vim.rs:SwitchToVisualLineMode]`, `[src/vim.rs:SwitchToVisualBlockMode]`, `[src/vim.rs:SwitchToHelixNormalMode]`
- Operator, Register, Mark & Pane Actions: `[src/vim.rs:ClearOperators]`, `[src/vim.rs:ClearExchange]`, `[src/vim.rs:Tab]`, `[src/vim.rs:Enter]`, `[src/vim.rs:InnerObject]`, `[src/vim.rs:PushChange]`, `[src/vim.rs:PushDelete]`, `[src/vim.rs:Exchange]`, `[src/vim.rs:PushYank]`, `[src/vim.rs:PushReplace]`, `[src/vim.rs:PushDeleteSurrounds]`, `[src/vim.rs:PushMark]`, `[src/vim.rs:ToggleMarksView]`, `[src/vim.rs:PushForcedMotion]`, `[src/vim.rs:PushIndent]`, `[src/vim.rs:PushOutdent]`, `[src/vim.rs:PushAutoIndent]`, `[src/vim.rs:PushRewrap]`, `[src/vim.rs:PushShellCommand]`, `[src/vim.rs:PushLowercase]`, `[src/vim.rs:PushUppercase]`, `[src/vim.rs:PushOppositeCase]`, `[src/vim.rs:PushRot13]`, `[src/vim.rs:PushRot47]`, `[src/vim.rs:ToggleRegistersView]`, `[src/vim.rs:PushRegister]`, `[src/vim.rs:PushRecordRegister]`, `[src/vim.rs:PushReplayRegister]`, `[src/vim.rs:PushReplaceWithRegister]`, `[src/vim.rs:PushToggleComments]`, `[src/vim.rs:PushToggleBlockComments]`, `[src/vim.rs:MaximizePane]`, `[src/vim.rs:ResetPaneSizes]`, `[src/vim.rs:ResizePaneRight]`, `[src/vim.rs:ResizePaneLeft]`, `[src/vim.rs:ResizePaneUp]`, `[src/vim.rs:ResizePaneDown]`, `[src/vim.rs:MenuSelectNext]`, `[src/vim.rs:MenuSelectPrevious]`, `[src/vim.rs:ToggleProjectPanelFocus]`, `[src/vim.rs:PushHelixMatch]`, `[src/vim.rs:PushHelixSurroundAdd]`, `[src/vim.rs:PushHelixSurroundReplace]`, `[src/vim.rs:PushHelixSurroundDelete]`

## Invariants

- When `HelixModeSetting` is enabled, `VimSettings::default_mode` must be `Mode::Normal` or `Mode::Insert` (`[src/vim.rs:Vim::new]`).
- `Vim::pop_operator` requires an operator to be present on `operator_stack` when popped (`[src/vim.rs:Vim::pop_operator]`).
- Pre-count and post-count accumulation is bounded below `isize::MAX as usize` (`[src/vim.rs:Vim::push_count_digit]`).
- Pushing `AddSurrounds`, `ChangeSurrounds`, `DeleteSurrounds`, or `Exchange` clears any prior operators from `operator_stack` before pushing (`[src/vim.rs:Vim::push_operator]`).
- `VimDb` enforces SQLite unique index invariants on `vim_marks(workspace_id, mark_name, path)` and `vim_global_marks_paths(workspace_id, mark_name)` (`[src/state.rs:VimDb]`).

## Side Effects

- Registers global `VimGlobals` state, observes `SettingsStore`, `Editor`, and `Workspace` entities, and updates user settings files when toggling Vim or Helix mode (`[src/vim.rs:init]`, `[src/state.rs:VimGlobals::register]`).
- Reads and writes the system clipboard and primary selection via `cx.write_to_clipboard` and `cx.write_to_primary` (`[src/state.rs:VimGlobals::write_registers]`).
- Persists and deletes local and global Vim marks in SQLite tables `vim_marks` and `vim_global_marks_paths` via `VimDb`, saves buffers to disk via `VimSave`, and spawns external shell commands via `ShellExec::run` (`[src/state.rs:VimDb]`, `[src/command.rs:VimSave]`, `[src/command.rs:ShellExec::run]`).

## Verification

- Run: `cargo test -p vim`
