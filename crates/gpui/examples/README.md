# GPUI Examples

## Overview

Standalone desktop and WebAssembly example applications demonstrating GPUI windows, layout, styling, input, animations, and headless/windowed runtime modes.

Examples can be run from the Zed repository root:

```sh
cargo run -p gpui --example hello_world
```

## Running in a browser

A selection of these examples is also served by the standalone web gallery:

```sh
cd crates/gpui_web/examples/hello_web
trunk serve
```

Open <http://localhost:8080/> to choose an example:

| Path | Example |
| --- | --- |
| `/hello-world` | Hello world |
| `/text` | Styled text |
| `/text-layout` | Alignment and decorations |
| `/text-wrapper` | Wrapping and truncation |
| `/input` | Text input and selection |
| `/dynamic-fonts` | Download a font on missing-glyph notifications |
| `/prime-sieve` | Background task demo |

The server supports direct links and reloads at each path. Only the selected
example's WASM module is loaded. Follow **All examples** to return to the gallery;
navigation reloads the page so the previous app and its workers are released.

Install [Trunk](https://trunk-rs.github.io/trunk/) if needed. The gallery's toolchain file selects
nightly Rust, the WASM target, and `rust-src`. Its Trunk configuration supplies the
cross-origin-isolation headers needed for shared memory and watches the GPUI
example and renderer sources for automatic rebuilds and browser reloads. Use a
browser with WebGPU or WebGL2 support.

The gallery builds separate Cargo binaries directly from the existing example
sources. To add another browser-compatible example, register its binary in the
gallery's `Cargo.toml`, add an auxiliary Rust asset in `index.html`, and add its
gallery link with the corresponding `data-module` name.

### Dynamic font loading

Open `/dynamic-fonts` to see `App::on_missing_glyphs` fetch a versioned
[Noto Sans Devanagari TTF](https://fonts.gstatic.com/s/notosansdevanagari/v30/TuGoUUFzXI5FBtUq5a8bjKYTZjtRU6Sgv3NaV_SNmI0b8QQCQmHn6B2OHjbL_08AlXQly-A.ttf)
from Google Fonts' CDN. The 219 KB font is licensed under the
[SIL Open Font License](https://github.com/google/fonts/blob/main/ofl/notosansdevanagari/OFL.txt).
The CDN permits cross-origin requests and caches the font for one year.
No sample text is sent to the host.

The example bundles only IBM Plex Sans, downloads the fallback once on demand,
calls `add_fonts`, and refreshes the windows to reshape the text. Emoji continues
to use the default Canvas fallback. The page displays loading and failure states;
use browser network throttling or block `fonts.gstatic.com` and reload to
exercise them. Reload to retry a failed request.

This is a small script-to-font mapping, not a general font discovery service.
For production, host versioned font assets on your own CDN and choose an explicit
coverage, caching, and retry policy. Use raw TTF/OTF assets rather than passing
Google Fonts CSS or WOFF2 responses directly to `add_fonts`.

### Canvas font fallback

The web text system uses browser fonts for eligible missing CJK and emoji
graphemes. It renders each complete grapheme independently, retaining the primary
font's baseline and line-spacing metrics. Contextual CJK spacing and arbitrary
OpenType features are not reproduced. Other scripts remain on Cosmic Text.
Browser font availability is assumed stable for the application's lifetime.

With the gallery server running, a browser smoke test checks native-font
preservation, whole-grapheme rendering, emoji atlas reuse, and input/deletion:

```sh
node crates/gpui_web/examples/hello_web/test_canvas_fallback.mjs
```

This requires Node.js 22 or newer and Chrome/Chromium. On macOS it uses the
standard Google Chrome application path; elsewhere it uses `chromium`. Set
`CHROME` to override the executable, and pass an optional server URL as the first
argument. Screenshots and logs are saved under the gallery's ignored `target`
directory. The test intercepts clipboard writes rather than modifying the system
clipboard.

## Where to start

- `hello_world` shows the basic shape of a GPUI application: create an
  `Application`, open a window, create a root view, and render a `div`.
- `input` demonstrates text input, focus, selections, clipboard actions, and
  keyboard bindings.
- `uniform_list` shows how to render a simple virtualized list.
- `testing` demonstrates `#[gpui::test]`, `TestAppContext`, actions, focus, and
  window-based tests.

## Layout and styling

- `grid_layout` demonstrates CSS-grid-style layout.
- `opacity` demonstrates opacity styling.
- `pattern` shows patterned backgrounds.
- `shadow` demonstrates box shadows.
- `text` shows styled text rendering.
- `text_layout` demonstrates text alignment, decoration, weights, and wrapping.
- `text_wrapper` shows wrapping text content.

## Interaction

- `anchor` demonstrates anchored positioning.
- `data_table` combines virtualized list rendering with table-style rows and a
  custom scrollbar.
- `drag_drop` shows draggable elements and drop targets.
- `focus_visible` demonstrates keyboard-visible focus styling.
- `mouse_pressure` demonstrates pressure-sensitive pointer input where supported.
- `popover` shows floating layers with `deferred` and `anchored`.
- `scrollable` demonstrates scrollable content.
- `tab_stop` shows keyboard tab navigation.

## Images, drawing, and animation

- `animation` demonstrates GPUI animations and animated SVG transforms.
- `gif_viewer` shows GIF rendering.
- `gradient` demonstrates linear gradients and color spaces.
- `image` shows local and remote image loading, image sizing, and asset setup.
- `image_gallery` demonstrates image caching and loading remote images.
- `image_loading` shows image loading states and asset loading.
- `painting` demonstrates custom drawing with paths and canvas.
- `svg` shows SVG rendering.

## Windows and application behavior

- `move_entity_between_windows` shows moving an entity between windows.
- `on_window_close_quit` demonstrates quitting when a window closes.
- `set_menus` shows application menu setup.
- `system_notifications` demonstrates posting, replacing, dismissing, and responding to operating-system notifications.
- `window` demonstrates creating normal, dialog, popup, and floating windows.
- `window_positioning` demonstrates window bounds and placement.
- `window_shadow` demonstrates window shadow styling.

## Specialized examples

These examples are useful when working on GPUI itself, but they may not be the
best starting point for new applications:

- `active_state_bug` is a focused active-state reproduction.
- `toggle_headless` starts without a display and preserves its app and todo
  entities while attaching to or detaching from Wayland or X11. Run it on Linux
  with `cargo run -p gpui --example toggle_headless`; its stdin commands are
  `ls`, `create [message]`, `open`, `close`, and `quit`. `open` attaches to the
  display server in the process environment and opens the window; while headless,
  `open DISPLAY=:1` (or `WAYLAND_DISPLAY=… XDG_RUNTIME_DIR=…`) targets another one.
  `close` closes the window and returns to headless mode.
- `layer_shell` demonstrates Linux layer-shell windows.
- `list_example` demonstrates bottom-aligned list state and scrollbar behavior.
- `ownership_post` supports the ownership and data-flow documentation.
- `paths_bench` is a path rendering benchmark.
- `tree` renders a deep tree of nested elements.

## Interface

- **Starter & Core Examples**: `[hello_world.rs:main]` / `[hello_world.rs:start]`, `[input.rs:main]`, `[uniform_list.rs:main]`, `[testing.rs]`, `[view_example/view_example_main.rs:main]`
- **Layout, Styling & Text**: `[grid_layout.rs:main]`, `[opacity.rs:main]`, `[pattern.rs:main]`, `[shadow.rs:main]`, `[text.rs:main]`, `[text_layout.rs:main]`, `[text_wrapper.rs:main]`
- **Interaction, Accessibility & Navigation**: `[a11y.rs:main]`, `[anchor.rs:main]`, `[data_table.rs:main]`, `[drag_drop.rs:main]`, `[focus_visible.rs:main]`, `[mouse_pressure.rs:main]`, `[popover.rs:main]`, `[scrollable.rs:main]`, `[tab_stop.rs:main]`
- **Graphics, Images & Animation**: `[animation.rs:main]`, `[gif_viewer.rs:main]`, `[gradient.rs:main]`, `[image/image.rs:main]`, `[image_gallery.rs:main]`, `[image_loading.rs:main]`, `[painting.rs:main]`, `[svg/svg.rs:main]`
- **Windows & Platform Integration**: `[move_entity_between_windows.rs:main]`, `[on_window_close_quit.rs:main]`, `[set_menus.rs:main]`, `[system_notifications.rs:main]`, `[window.rs:main]`, `[window_movable.rs:main]`, `[window_positioning.rs:main]`, `[window_shadow.rs:main]`, `[toggle_headless.rs:main]`, `[layer_shell.rs:main]`
- **Specialized & Diagnostic Examples**: `[active_state_bug.rs:main]`, `[list_example.rs:main]`, `[ownership_post.rs:main]`, `[paths_bench.rs:main]`, `[tree.rs:main]`

## Invariants

- `[hello_world.rs:run_example]` & `[example_support/fonts.rs:load_fonts]`: Examples sharing `example_support` must load bundled fonts via `example_support::load_fonts(cx)` before opening a window, returning early if font loading fails.
- `[hello_world.rs:start]`: WebAssembly-compatible examples mark `#![cfg_attr(target_family = "wasm", no_main)]` and export a `#[wasm_bindgen(start)]` entrypoint that calls `gpui_platform::web_init()` before invoking `run_example()`.

## Side Effects

- `[hello_world.rs:run_example]`: Initializes `gpui_platform::application()`, loads font/image/SVG assets into `App`, and opens native OS or browser WebGPU/WebGL2 windows.
- `[toggle_headless.rs:main]` & `[system_notifications.rs:main]`: Reads interactive commands from `stdin`, attaches/detaches Wayland or X11 display connections on Linux, and dispatches OS notifications.

## Verification

- Run: `cargo check -p gpui --examples`
