# Crates

## Overview

The `crates/` directory contains the Cargo workspace members for the Zed editor, GPUI framework, headless evaluation runners, extension APIs, collaboration backend, and developer utilities.

## Subdirectories & Boundaries

### Crates with Component Contracts (`README.md`)

- `agent_skills`: [`./agent_skills/README.md`](./agent_skills/README.md) - Loading, frontmatter validation, discovery, and override merging for `SKILL.md` Agent Skills (`[agent_skills.rs:Skill]`, `[agent_skills.rs:load_skills_from_directory]`).
- `agent_ui_benchmarks`: [`./agent_ui_benchmarks/README.md`](./agent_ui_benchmarks/README.md) - Criterion benchmark suite measuring the `agent_ui` model and ACP configuration-option search pipeline (`[benches/model_search.rs:benches]`).
- `cli`: [`./cli/README.md`](./cli/README.md) - Command-line launcher and IPC bridge (`IpcHandshake`, `CliRequest`, `CliResponse`) for communicating with the Zed application (`[src/cli.rs:IpcHandshake]`, `[src/main.rs:main]`).
- `collab`: [`./collab/README.md`](./collab/README.md) - Zed collaboration and API server backend handling WebSocket RPC connections, room/project transactions, LiveKit integration, and extension indexing (`[src/main.rs:main]`, `[src/rpc.rs:Server]`, `[src/db.rs:Database]`).
- `db`: [`./db/README.md`](./db/README.md) - Application SQLite persistence layer (`AppDatabase`, `DomainMigration`, `KeyValueStore`, `ScopedKeyValueStore`) built on `sqlez` (`[src/db.rs:AppDatabase]`, `[src/kvp.rs:KeyValueStore]`).
- `eval_cli`: [`./eval_cli/README.md`](./eval_cli/README.md) - Headless Rust binary (`eval-cli`) and [`./eval_cli/zed_eval/README.md`](./eval_cli/zed_eval/README.md) Python orchestration package for running Zed's `NativeAgent` + `AcpThread` loop in benchmark containers (`[src/main.rs:main]`, `[src/headless.rs:init]`).
- `eval_utils`: [`./eval_utils/README.md`](./eval_utils/README.md) - Concurrent evaluation runner, progress reporter, and pass-ratio assertion helpers for agent evaluations (`[src/eval_utils.rs:eval]`, `[src/eval_utils.rs:EvalOutput]`).
- `extension_api`: [`./extension_api/README.md`](./extension_api/README.md) - Public Rust API (`zed_extension_api`) and WIT guest bindings for authoring WebAssembly Zed extensions (`[src/extension_api.rs:Extension]`, `[src/extension_api.rs:register_extension!]`).
- `gpui`: [`./gpui/README.md`](./gpui/README.md) - Hybrid immediate and retained mode GPU-accelerated UI framework providing `App`, `Entity`, `Window`, `Element`, `Render`, actions, foreground/background executors, and [`./gpui/examples/README.md`](./gpui/examples/README.md) standalone examples (`[src/app.rs:App]`, `[src/window.rs:Window]`, `[src/element.rs:Element]`).
- `icons`: [`./icons/README.md`](./icons/README.md) - Typed `IconName` enumeration and SVG asset path lookup for Zed UI icons (`[src/icons.rs:IconName]`).
- `inspector_ui`: [`./inspector_ui/README.md`](./inspector_ui/README.md) - Debug-only UI inspector for picking rendered GPUI elements, viewing layout state, and live-editing `Div` styles (`[src/inspector_ui.rs:init]`, `[src/inspector.rs:init]`).
- `sandbox`: [`./sandbox/README.md`](./sandbox/README.md) - Cross-platform shell command sandboxing (`Sandbox`, `SandboxPolicy`, `HostFilesystemLocation`) using macOS Seatbelt, Linux bubblewrap + `SCM_RIGHTS` FD validation + seccomp-BPF, and Windows WSL (`[src/sandbox.rs:Sandbox]`, `[src/sandbox.rs:SandboxPolicy]`).
- `schema_generator`: [`./schema_generator/README.md`](./schema_generator/README.md) - CLI binary generating JSON schemas for Zed themes, icon themes, and project settings (`[src/main.rs:main]`, `[src/main.rs:SchemaType]`).
- `terminal_view`: [`./terminal_view/README.md`](./terminal_view/README.md) - GPUI view (`TerminalView`), custom element (`TerminalElement`), and dock panel (`TerminalPanel`) for Zed's integrated terminal (`[src/terminal_view.rs:TerminalView]`, `[src/terminal_panel.rs:TerminalPanel]`).
- `theme_importer`: [`./theme_importer/README.md`](./theme_importer/README.md) - CLI converter transforming VS Code theme JSON definitions into Zed theme JSON files (`[src/main.rs:main]`, `[src/vscode/converter.rs:VsCodeThemeConverter]`).
- `vim`: [`./vim/README.md`](./vim/README.md) - Vim and Helix modal editing emulation, operators, motions, registers, marks persistence, and Neovim-backed test harness (`[src/vim.rs:init]`, `[src/vim.rs:ModeIndicator]`).
- `zlog`: [`./zlog/README.md`](./zlog/README.md) - Scoped hierarchical logging library with `ZED_LOG` / settings filtering and rotating file sinks (`[src/zlog.rs:init]`, `[src/zlog.rs:Scope]`).

### Other Core Workspace Crates

- `zed` (`./zed/Cargo.toml`) - Main Zed desktop application binary, bootstrapping windows, workspace state, CLI IPC handling, and subsystem initialization.
- `workspace` (`./workspace/Cargo.toml`) - Multi-pane workspace layout, docks, status bar, modal management, item navigation, and project coordination.
- `editor` (`./editor/Cargo.toml`) - Core text editor view, multi-buffer rendering, selections, completions, diagnostics, inline diffs, and code actions.
- `project` (`./project/Cargo.toml`) - Project model managing worktrees, buffers, LSP servers, DAP sessions, task execution, git repositories, and remote/collab synchronization.
- `language` (`./language/Cargo.toml`) - Buffer text representation, Tree-sitter syntax parsing, grammar registry, auto-indentation, and language server protocol (LSP) abstractions.
- `agent` (`./agent/Cargo.toml`) & `agent_ui` (`./agent_ui/Cargo.toml`) - Native AI agent loop, built-in tool implementations, thread history, and assistant panel UI.
- `acp_thread` (`./acp_thread/Cargo.toml`) & `acp_tools` (`./acp_tools/Cargo.toml`) - Agent Communication Protocol (ACP) thread state, tool execution, and UI views.
- `gpui_platform` (`./gpui_platform/Cargo.toml`), `gpui_macos` (`./gpui_macos/Cargo.toml`), `gpui_linux` (`./gpui_linux/Cargo.toml`), `gpui_windows` (`./gpui_windows/Cargo.toml`), `gpui_wgpu` (`./gpui_wgpu/Cargo.toml`) - Platform windowing, text system, and GPU rendering backends for `gpui`.
- `extension` (`./extension/Cargo.toml`) & `extension_host` (`./extension_host/Cargo.toml`) - Extension trait abstractions and Wasmtime host runtime for executing Zed extensions.
- `remote` (`./remote/Cargo.toml`) & `remote_server` (`./remote_server/Cargo.toml`) - SSH, WSL, and dev-container remote development client and headless remote server binary.
- `git` (`./git/Cargo.toml`) & `git_ui` (`./git_ui/Cargo.toml`) - Git repository operations, blame, staging, commit workflows, and version control UI panels.
- `dap` (`./dap/Cargo.toml`) & `debugger_ui` (`./debugger_ui/Cargo.toml`) - Debug Adapter Protocol client, session state, breakpoint management, and debugger UI panels.
- `terminal` (`./terminal/Cargo.toml`) - Terminal emulator backend and PTY event loop abstraction consumed by `terminal_view`.
- `rope` (`./rope/Cargo.toml`), `sum_tree` (`./sum_tree/Cargo.toml`), `text` (`./text/Cargo.toml`), `multi_buffer` (`./multi_buffer/Cargo.toml`) - Foundational CRDT text buffer, B-tree summary data structures, and multi-buffer excerpt composition.
- `settings` (`./settings/Cargo.toml`) & `settings_ui` (`./settings_ui/Cargo.toml`) - Configuration schema store, JSON settings file merging, and interactive settings editor UI.
- `theme` (`./theme/Cargo.toml`) & `ui` (`./ui/Cargo.toml`) - Design system tokens, color themes, typography, and shared GPUI component library.
- `client` (`./client/Cargo.toml`), `rpc` (`./rpc/Cargo.toml`), `proto` (`./proto/Cargo.toml`) - Protobuf message definitions, peer-to-peer RPC framing, and Zed cloud/collaboration client connection management.
- `fs` (`./fs/Cargo.toml`) & `worktree` (`./worktree/Cargo.toml`) - Abstracted real/fake filesystem layer and worktree file indexing, watching, and `.gitignore` filtering.
