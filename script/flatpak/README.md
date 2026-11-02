# flatpak

## Overview
Packaging and metadata automation scripts for building Zed Flatpak bundles across release channels (`dev`, `nightly`, `preview`, `stable`) and converting GitHub release notes into AppStream XML `<release>` entries.

## Interface
- `[bundle-flatpak:CHANNEL]`: Exported environment variable populated from `crates/zed/RELEASE_CHANNEL`.
- `[bundle-flatpak:ARCHIVE]`: Exported environment variable matching `zed(-[a-zA-Z0-9]+)?-linux-$(uname -m)\.tar\.gz` in `target/release`.
- `[bundle-flatpak:APP_ID]`: Exported environment variable (`dev.zed.ZedDev`, `dev.zed.ZedNightly`, `dev.zed.ZedPreview`, or `dev.zed.Zed`).
- `[bundle-flatpak:APP_NAME]`: Exported environment variable (`Zed Devel`, `Zed Nightly`, `Zed Preview`, or `Zed`).
- `[bundle-flatpak:BRANDING_LIGHT]`: Exported environment variable (`#99c1f1` or `#e9aa6a`).
- `[bundle-flatpak:BRANDING_DARK]`: Exported environment variable (`#1a5fb4`).
- `[bundle-flatpak:ICON_FILE]`: Exported environment variable (`app-icon-dev`, `app-icon-nightly`, `app-icon-preview`, or `app-icon`).
- `[convert-release-notes.py:sys.argv]`: CLI positional arguments `<tag>` and `<channel>`.
- `[convert-release-notes.py:clean_line]`: `clean_line(line: str, in_code_fence: bool) -> str` for sanitizing Markdown lines and escaping XML entities.
- `[convert-release-notes.py:convert_body]`: `convert_body(body: str) -> str` for converting Markdown release note bodies into AppStream HTML/XML fragments.
- `[convert-release-notes.py:get_release_info]`: `get_release_info(tag: str)` for fetching GitHub release metadata by tag.
- `[deps:fd_version]`: Shell variable specifying the Freedesktop Flatpak runtime and SDK version (`23.08`).

## Invariants
- `[bundle-flatpak:set -euo pipefail]`: Execution aborts immediately on any command failure, unset variable, or pipeline error.
- `[bundle-flatpak:channel]`: Contents of `crates/zed/RELEASE_CHANNEL` must be `dev`, `nightly`, `preview`, or `stable`; otherwise the script prints an error and exits.
- `[bundle-flatpak:archive_match]`: A release archive matching `zed(-[a-zA-Z0-9]+)?-linux-$(uname -m)\.tar\.gz` must exist in `target/release` after running `script/bundle-linux --flatpak`.
- `[convert-release-notes.py:sys.argv]`: Exactly two CLI arguments (`<tag>` and `<channel>`) must be provided (`len(sys.argv) == 3`), or the script exits with code `1`.
- `[convert-release-notes.py:get_release_info]`: The GitHub release API request must return HTTP status `200`, or the script terminates via `quit()`.

## Side Effects
- `[bundle-flatpak:cd]`: Changes working directory to the repository root (`$(dirname "$0")/../..`).
- `[bundle-flatpak:script/bundle-linux --flatpak]`: Invokes the Linux bundle build script with `--flatpak`.
- `[bundle-flatpak:envsubst]`: Generates `$APP_ID.json` in the repository root from `crates/zed/resources/flatpak/manifest-template.json`.
- `[bundle-flatpak:flatpak-builder]`: Builds and installs the Flatpak into the user installation (`--user --install --force-clean`).
- `[bundle-flatpak:flatpak build-bundle]`: Outputs a single-file bundle at `target/release/$APP_ID.flatpak` from `~/.local/share/flatpak/repo`.
- `[convert-release-notes.py:os.chdir]`: Changes the working directory to the script directory (`sys.path[0]`).
- `[convert-release-notes.py:requests.get]`: Sends an HTTP GET request to `https://api.github.com/repos/zed-industries/zed/releases/tags/{tag}`.
- `[convert-release-notes.py:open]`: Reads and prepends formatted XML `<release>` entries to `../../crates/zed/resources/flatpak/release-info/{channel}`.
- `[deps:flatpak remote-add]`: Registers the `flathub` user remote (`https://dl.flathub.org/repo/flathub.flatpakrepo`) if not already present.
- `[deps:flatpak install]`: Installs `org.freedesktop.Platform/${arch}/23.08`, `org.freedesktop.Sdk/${arch}/23.08`, and `org.freedesktop.Sdk.Extension.rust-stable/${arch}/23.08` for the current user.

## Verification
- Run: `bash -n script/flatpak/bundle-flatpak script/flatpak/deps && python3 -m py_compile script/flatpak/convert-release-notes.py`
