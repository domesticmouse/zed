# Welcome to GPUI!

## Overview

GPUI is a hybrid immediate and retained mode, GPU accelerated, UI framework
for Rust, designed to support a wide variety of applications.

## Getting Started

GPUI is still in active development as we work on the Zed code editor, and is still pre-1.0. There will often be breaking changes between versions. You'll also need to use the latest version of stable Rust. Add `gpui`, and optionally `gpui_platform`, to your `Cargo.toml`:

```toml
gpui = { version = "*" }
gpui_platform = { version = "*", features = ["font-kit", "wayland", "x11"] }
```

Everything in a standalone GPUI app starts with an `Application`. You can create one with `gpui_platform::application()`, which picks the windowing and text backends for the host OS, and kick off your application by passing a callback to `Application::run()`. Inside this callback, you can create a new window with `App::open_window()` and register your first root view.

```rust,no_run
use gpui::*;

fn main() {
    gpui_platform::application().run(|cx: &mut App| {
        // ..
    });
}
```

### `gpui_platform`

The features on `gpui_platform` are platform-specific, so the list above is a safe cross-platform default. If you build for a single platform, you can trim it:

- **macOS** — Rendering uses Metal and is always available, but glyph rasterization needs `font-kit`. Without it, GPUI falls back to a placeholder text system that lays text out but renders no glyphs.

    ```toml
    gpui_platform = { version = "*", features = ["font-kit"] }
    ```

- **Linux / FreeBSD** — enable at least one windowing backend for desktop windows: `wayland`, `x11`, or both. These features also compile the renderer and text system, so no separate text feature is needed.

    ```toml
    gpui_platform = { version = "*", features = ["wayland", "x11"] }
    ```

- **Windows** — no features are required. Windowing uses Win32 and text uses DirectWrite. `font-kit` has no effect here.

### Additional Topics

- [Ownership and data flow](_ownership_and_data_flow)
- [Accessibility](_accessibility)

### Dependencies

GPUI has various system dependencies that it needs in order to work.

#### macOS

On macOS, GPUI uses Metal for rendering. In order to use Metal, you need to do the following:

- Install [Xcode](https://apps.apple.com/us/app/xcode/id497799835?mt=12) from the macOS App Store, or from the [Apple Developer](https://developer.apple.com/download/all/) website. Note this requires a developer account.

> Ensure you launch Xcode after installing, and install the macOS components, which is the default option.

- Install [Xcode command line tools](https://developer.apple.com/xcode/resources/)

  ```sh
  xcode-select --install
  ```

- Ensure that the Xcode command line tools are using your newly installed copy of Xcode:

  ```sh
  sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
  ```

## The Big Picture

GPUI offers three different [registers](<https://en.wikipedia.org/wiki/Register_(sociolinguistics)>) depending on your needs:

- State management and communication with `Entity`'s. Whenever you need to store application state that communicates between different parts of your application, you'll want to use GPUI's entities. Entities are owned by GPUI and are only accessible through an owned smart pointer similar to an `Rc`. See the `app::context` module for more information.

- High level, declarative UI with views. All UI in GPUI starts with a view. A view is simply an `Entity` that can be rendered, by implementing the `Render` trait. At the start of each frame, GPUI will call this render method on the root view of a given window. Views build a tree of `elements`, lay them out and style them with a tailwind-style API, and then give them to GPUI to turn into pixels. See the `div` element for an all purpose swiss-army knife of rendering.

- Low level, imperative UI with Elements. Elements are the building blocks of UI in GPUI, and they provide a nice wrapper around an imperative API that provides as much flexibility and control as you need. Elements have total control over how they and their child elements are rendered and can be used for making efficient views into large lists, implement custom layouting for a code editor, and anything else you can think of. See the `element` module for more information.

Each of these registers has one or more corresponding contexts that can be accessed from all GPUI services. This context is your main interface to GPUI, and is used extensively throughout the framework.

## Other Resources

In addition to the systems above, GPUI provides a range of smaller services that are useful for building complex applications:

- Actions are user-defined structs that are used for converting keystrokes into logical operations in your UI. Use this for implementing keyboard shortcuts, such as cmd-q. See the `action` module for more information.

- Platform services, such as `quit the app` or `open a URL` are available as methods on the `app::App`.

- An async executor that is integrated with the platform's event loop. See the `executor` module for more information.

- The `[gpui::test]` macro provides a convenient way to write tests for your GPUI applications. Tests also have their own kind of context, a `TestAppContext` which provides ways of simulating common platform input. See `app::test_context` and `test` modules for more details.

Currently, the best way to learn about these APIs is to read the Zed source code or drop a question in the [Zed Discord](https://zed.dev/community-links). We're working on improving the documentation, creating more examples, and will be publishing more guides to GPUI on our [blog](https://zed.dev/blog).

## Interface

- Application & Contexts: `[src/app.rs:Application]`, `[src/app.rs:ApplicationHandle]`, `[src/app.rs:App]`, `[src/app.rs:AppCell]`, `[src/app.rs:SHUTDOWN_TIMEOUT]`, `[src/app/context.rs:Context]`, `[src/app/async_context.rs:AsyncApp]`, `[src/app/async_context.rs:AsyncWindowContext]`, `[src/gpui.rs:AppContext]`, `[src/gpui.rs:VisualContext]`, `[src/gpui.rs:BorrowAppContext]`
- Entities, Events & Globals: `[src/app/entity_map.rs:Entity]`, `[src/app/entity_map.rs:WeakEntity]`, `[src/app/entity_map.rs:AnyEntity]`, `[src/app/entity_map.rs:AnyWeakEntity]`, `[src/app/entity_map.rs:EntityId]`, `[src/gpui.rs:Reservation]`, `[src/gpui.rs:EventEmitter]`, `[src/subscription.rs:Subscription]`, `[src/subscription.rs:SubscriberSet]`, `[src/global.rs:Global]`, `[src/global.rs:ReadGlobal]`, `[src/global.rs:UpdateGlobal]`
- Windows, Views & Elements: `[src/window.rs:Window]`, `[src/window.rs:WindowHandle]`, `[src/window.rs:AnyWindowHandle]`, `[src/window.rs:WindowId]`, `[src/window.rs:WindowOptions]`, `[src/window.rs:DispatchPhase]`, `[src/window.rs:ElementId]`, `[src/view.rs:View]`, `[src/view.rs:AnyView]`, `[src/view.rs:AnyWeakView]`, `[src/view.rs:ViewElement]`, `[src/element.rs:Element]`, `[src/element.rs:IntoElement]`, `[src/element.rs:Render]`, `[src/element.rs:RenderOnce]`, `[src/element.rs:ParentElement]`, `[src/element.rs:AnyElement]`, `[src/element.rs:GlobalElementId]`
- Styling, Layout, Input & Actions: `[src/styled.rs:Styled]`, `[src/style.rs:Style]`, `[src/interactive.rs:InteractiveElement]`, `[src/interactive.rs:StatefulInteractiveElement]`, `[src/interactive.rs:Focusable]`, `[src/interactive.rs:FocusHandle]`, `[src/action.rs:Action]`, `[src/action.rs:actions!]`, `[src/keymap.rs:Keymap]`, `[src/keymap.rs:KeyBinding]`, `[src/color.rs:Hsla]`, `[src/color.rs:Rgba]`, `[src/geometry.rs:Bounds]`, `[src/geometry.rs:Point]`, `[src/geometry.rs:Size]`, `[src/geometry.rs:Pixels]`, `[src/geometry.rs:DevicePixels]`, `[src/geometry.rs:ScaledPixels]`, `[src/geometry.rs:Rems]`, `[src/scene.rs:Scene]`
- Executors, Platform, Text & Assets: `[src/executor.rs:BackgroundExecutor]`, `[src/executor.rs:ForegroundExecutor]`, `[src/executor.rs:SendForegroundExecutor]`, `[src/executor.rs:Task]`, `[src/executor.rs:TaskExt]`, `[src/executor.rs:Priority]`, `[src/platform.rs:Platform]`, `[src/platform.rs:PlatformWindow]`, `[src/platform.rs:PlatformDisplay]`, `[src/platform.rs:PlatformDispatcher]`, `[src/text_system.rs:TextSystem]`, `[src/text_system.rs:WindowTextSystem]`, `[src/asset_cache.rs:Asset]`, `[src/assets.rs:AssetSource]`, `[src/gpui.rs:GpuSpecs]`, `[src/prelude.rs:*]`
- Subdirectories: [examples](./examples/README.md) (`./examples/README.md`)

## Invariants

- `App` state is wrapped in an `AppCell` (`RefCell`); overlapping mutable/shared borrows panic with `AppBorrowError` reporting the existing borrow's caller location (`[src/app.rs:AppCell]`).
- Leasing an entity removes it from `EntityMap` onto the stack; attempting to read or update the same `EntityId` while leased panics via `double_lease_panic` (`[src/app/entity_map.rs:EntityMap::lease_erased]`).
- Entities must only be read or updated with the `App` context that created them (`[src/app/entity_map.rs:EntityMap::assert_valid_context]`).
- Registering multiple actions with the same name or reading an unassigned `Global` type panics (`[src/action.rs:Action]`, `[src/global.rs:ReadGlobal::global]`).

## Side Effects

- `build.rs` emits `gpui_leak_detection` and `gpui_hot_reload` rustc cfg flags and compiles Windows manifest resources when `target_os == "windows"` and `windows-manifest` is enabled (`[build.rs:main]`).
- `BackgroundExecutor` spawns tasks onto background worker threads (or a dedicated realtime audio thread for `Priority::RealtimeAudio`) and manages OS App Nap activity guards (`[src/executor.rs:BackgroundExecutor]`).
- `WindowInvalidator` marks views/windows dirty, records profiler frame journal events when `feature = "profiler"` is enabled, pushes `Effect::Notify` to `App`, and invokes platform frame wakers (`[src/window.rs:WindowInvalidator]`).

## Verification

- Run: `cargo test -p gpui`
