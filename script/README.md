# script

## Overview
Build, packaging, release automation, database bootstrapping, code-quality linting, crash triage, and GitHub repository/project automation scripts for the Zed repository across macOS, Linux, FreeBSD, and Windows.

## Interface
- **Subdirectories**:
  - [danger](./danger/README.md): DangerJS pull-request hygiene, release notes validation, agent self-review marker enforcement, and schema/fixture modification attestations.
  - [flatpak](./flatpak/README.md): Flatpak packaging scripts and AppStream XML release note conversion.
  - [lib](./lib/README.md): Shared Bash, PowerShell, and Node.js helper functions for blob storage uploads, version bumping, release notes parsing, and deployment environment configuration.
  - `licenses` (`./licenses/zed-licenses.toml`): `cargo-about` SPDX license configuration and Handlebars templates (`template.md.hbs`, `template.csv.hbs`).
  - `terms` (`./terms/terms.json`): macOS DMG license agreement configuration (`terms.json`, `terms.rtf`).
  - [update_top_ranking_issues](./update_top_ranking_issues/README.md): Python CLI tool (`uv` / `typer`) that ranks top-voted open GitHub issues by section and updates a reference tracking issue.
- **Bootstrap & Developer Environment**:
  - `[bootstrap:sqlx]` & `[bootstrap.ps1:sqlx]`: Installs local development dependencies (`script/linux` or `foreman`, `minio`, `sqlx-cli` `0.7.2`) and initializes local databases (`DATABASE_URL`, `LLM_DATABASE_URL`) defined in `crates/collab`.
  - `[linux:finalize]`: Installs Linux system build dependencies and `rustup`.
  - `[freebsd:finalize]`: Installs FreeBSD system build packages via `pkg` and `rustup`.
  - `[install-cmake:$1]` / `[install-cmake:CMAKE_VERSION]`: Installs CMake on Linux (requires positional version argument `$1`; sets `CMAKE_VERSION="${CMAKE_VERSION:-${1:-3.30.4}}"`).
  - `[install-rustup.ps1:$env:CARGO_HOME]`: Installs `rustup` and `cargo` on Windows.
  - `[download-wasi-sdk:WASI_SDK_VERSION]` / `[download-wasi-sdk:WASI_SDK_DIR]`: Downloads and extracts the WASI SDK into `./target/wasi-sdk`.
  - `[setup-sccache:install_sccache]`, `[setup-sccache:configure_sccache]`, `[setup-sccache:show_config]`: Installs and configures `sccache` `v0.16.0` for Cloudflare R2.
  - `[setup-sccache.ps1:Install-Sccache]`, `[setup-sccache.ps1:Configure-Sccache]`, `[setup-sccache.ps1:Show-Config]`: Windows PowerShell `sccache` `v0.16.0` setup.
  - `[setup-dev-driver.ps1:script]`: Sets up a 100GB Windows Dev Drive VHDX (`C:/zed_dev_drive.vhdx`) for CI builds.
  - `[new-crate:$1]` / `[new-crate:$2]`: Scaffolds a new workspace crate `<crate_name>` with optional `apache` license flag (defaults to `GPL-3.0-or-later`).
  - `[zed-local:--help]`, `[zed-local:--release]`, `[zed-local:-<N> / --<N>]`, `[zed-local:--stateful]`, `[zed-local:--top]`, `[zed-local:--stable]`, `[zed-local:--preview]`: Launches 1–6 tiled local Zed client instances connected to a local collab server.
  - `[debug-cli:$@]`: Builds `zed` and runs the `cli` crate in foreground mode.
  - `[metal-debug:$@]`: Runs `cargo run` with Metal frame-capture and validation environment variables enabled.
  - `[mitm-proxy.sh]`: Starts `mitmproxy/mitmproxy` on port `9876` via Docker/Podman and runs `cargo run` through `http_proxy=http://localhost:9876`.
  - `[run-local-minio]`: Starts a local MinIO object storage server backed by `.blob_store`.
  - `[seed-db]`: Runs `cargo run -p collab migrate`.
  - `[reset_db]`: Drops local `zed` and `zed_llm` PostgreSQL databases and re-runs `script/bootstrap`.
  - `[drop-test-dbs:databases]`: Drops all local PostgreSQL databases matching `zed-test-%`.
- **Build, Cargo Wrappers & Benchmarking**:
  - `[cargo:main]`, `[cargo:--init]`, `[cargo:--target-dir]`, `[cargo:--timings]`: Node.js Cargo wrapper that injects `--timings` on `build`/`check`/`run`/`test` in the Zed repo and spawns background timing extraction.
  - `[cargo-timing-info.js:analyzeTimings]`, `[cargo-timing-info.js:writeBuildTimingJson]`: Parses Cargo HTML timing files (`UNIT_DATA`) and writes build timing JSON summaries into `<ZedDataDir>/build_timings`.
  - `[clear-target-dir-if-larger-than:<MAX_SIZE_IN_GB> [SMALL_CLEAN_SIZE_IN_GB]]` / `[clear-target-dir-if-larger-than.ps1:$MAX_SIZE_IN_GB [$SMALL_CLEAN_SIZE_IN_GB]]`: Cleans `target` directory when size thresholds are exceeded.
  - `[exit-ci-if-dev-drive-is-full.ps1:$MAX_SIZE_IN_GB]`: Exits with status `1` if the CI `target` directory exceeds `$MAX_SIZE_IN_GB`.
  - `[bench-agent-model-search:$@]`: Runs `cargo bench --locked -p agent_ui_benchmarks --bench model_search --profile release-fast`.
  - `[generate-merge-conflict-benchmark:destination]`, `[generate-merge-conflict-benchmark:--preset]`: Generates a synthetic Git repository with unresolved merge conflicts (`small`, `medium`, `severe`).
  - `[run-unit-evals:UNIT_EVAL_COMMIT]`: Runs `cargo nextest run` for `test(::eval_)`.
  - `[test-wsl-sandbox.ps1:-NoProvision]`, `[test-wsl-sandbox.ps1:-Release]`, `[test-wsl-sandbox.ps1:-AllowSkip]`: Provisions `bubblewrap` in WSL and runs `cargo xtask wsl-sandbox-tests`.
  - `[histogram:parse_log_file(file_path)]`, `[histogram:create_histograms(df, measurement)]`: Parses timing log files and plots normalized step histograms.
  - `[crate-dep-graph:graph_file]`: Generates and opens `target/crate-graph.html` using `cargo depgraph` and Graphviz `dot`.
- **Linting, Verification & Metadata Generation**:
  - `[clippy:-p]`, `[clippy:--package]` / `[clippy.ps1:-p]`, `[clippy.ps1:--package]`: Runs `cargo clippy --release --all-targets --all-features -- --deny warnings` (plus `cargo shear`, `typos`, and `buf` checks locally).
  - `[shellcheck-scripts:$1]`: Runs `shellcheck` across shell scripts in `script/` (`error` or `warning` mode).
  - `[squawk:script]`: Runs Squawk `v0.26.0` linter on PostgreSQL migrations in `crates/collab/migrations/*.sql` and `crates/collab/migrations_llm/*.sql`.
  - `[prettier:--write]`: Checks or formats `assets/settings/default.json` and `docs/` with `prettier@3.5.0`.
  - `[check-keymaps:pattern]`: Checks non-macOS keymap JSON files under `assets/keymaps/` for `cmd-` and checks all keymap files with `git grep --fixed-strings` against `pattern='super-|win-|fn-'`.
  - `[check-licenses:check_license]`, `[check-licenses:check_symlink_target]`, `[check-licenses:check_manifest_for_agpl]`, `[check-licenses:check_no_agpl_license_file]`: Verifies every subdirectory `Cargo.toml` has a valid `LICENSE-GPL` or `LICENSE-APACHE` symlink, no `Cargo.toml` references `agpl`/`affero` or `license-file`, and no `LICENSE-AGPL` file exists.
  - `[check-links:local]`, `[check-links:all]`: Checks documentation links in `./docs/src/**/*` via `lychee`.
  - `[check-todos:pattern]`: Verifies no tracked files contain `todo!` or `FIXME`.
  - `[verify-macos-document-icon:$1]`: Validates `CFBundleDocumentTypes` and `Contents/Resources/Document.icns` inside a built `Zed.app` bundle.
  - `[generate-action-metadata:cargo]`: Dumps all Zed actions to `crates/docs_preprocessor/actions.json`.
  - `[generate-licenses:$1 / OUTPUT_FILE]` / `[generate-licenses.ps1:$args[0] / $outputFile]` / `[generate-licenses-csv:$1 / OUTPUT_FILE]`: Generates Markdown or CSV license attribution files using `cargo-about` `0.8.2`.
  - `[generate-terms-rtf:pandoc]`: Converts `./legal/terms.md` into `./script/terms/terms.rtf` via `pandoc`.
  - `[update-json-schemas:$1]`: Fetches and updates bundled `tsconfig.json` and `package.json` schemas from `SchemaStore/schemastore`.
  - `[import-themes:$@]`: Forwards arguments to `cargo run -p theme_importer --`.
- **Bundling, Installation & Release Management**:
  - `[bundle-mac:-d]`, `[bundle-mac:-o]`, `[bundle-mac:-i]`, `[bundle-mac:-l]`, `[bundle-mac:-h]`, `[bundle-mac:architecture]`: Builds, signs, notarizes, and packages `Zed.app`, `Zed-${arch_suffix}.dmg`, and `zed-remote-server-macos-${arch_suffix}.gz`.
  - `[bundle-linux:--flatpak]`, `[bundle-linux:--require-sentry]`, `[bundle-linux:--skip-sentry]`, `[bundle-linux:-h|--help]`: Builds and packages `zed-linux-${arch}.tar.gz` and `zed-remote-server-linux-${arch}.gz`.
  - `[bundle-freebsd:-h]`, `[bundle-freebsd:CARGO_TARGET_DIR]`, `[bundle-freebsd:REMOTE_SERVER_TARGET]`: Builds and packages `zed-remote-server-freebsd-x86_64.gz`.
  - `[bundle-windows.ps1:-Install|-i]`, `[bundle-windows.ps1:-Architecture|-a]`, `[bundle-windows.ps1:-Name]`, `[bundle-windows.ps1:-Help|-h]`, `[bundle-windows.ps1:BuildInstaller]`: Builds, signs, packages `.appx` and Inno Setup installer, and uploads symbols on Windows.
  - `[build-docker:BASE_IMAGE]`: Builds container image from `Dockerfile-distros` via `podman build`.
  - `[snap-build:$1]` / `[snap-try:$1]`: Builds or tests the Zed Snap package for `<release_version>`.
  - `[install.sh:main]` / `[install-linux]` / `[uninstall.sh:main]`: Installs or uninstalls Zed across macOS and Linux (`ZED_CHANNEL`, `ZED_VERSION`, `ZED_BUNDLE_PATH`).
  - `[determine-release-channel:GITHUB_REF_NAME]` / `[determine-release-channel.ps1:$env:GITHUB_REF_NAME]`: Computes and exports `RELEASE_CHANNEL` and `RELEASE_VERSION` to `GITHUB_ENV`.
  - `[get-crate-version:$1 / CRATE_NAME]` / `[get-crate-version.ps1:$args[0] / $crateName]`: Queries Cargo metadata for a crate's version.
  - `[get-released-version:$1 / channel]`: Queries `https://cloud.zed.dev/releases/$channel/latest/asset` for the latest released version.
  - `[bump-zed-version:target]`, `[bump-gpui-version:bump_type]`, `[bump-nightly:bump-version.sh]`, `[bump-extension-cli:<message>]`: Bumps versions or triggers corresponding release workflows.
  - `[cherry-pick:<branch-name>]`, `[cherry-pick:<commit-sha>]`, `[cherry-pick:<channel>]`: Cherry-picks a commit onto a release branch and opens a pull request via `gh pr create`.
  - `[create-draft-release:$1]`, `[draft-release-notes:<version> <channel> [--gpui]]`, `[get-pull-requests-since:process.argv[2]]`, `[get-release-notes-since:process.argv[2]]`: Release notes and GitHub draft release tooling.
  - `[trigger-release:$1]`, `[trigger-docs-build:$1 [--from-main]]`, `[upload-nightly:script]`, `[upload-nightly.ps1:-Architecture]`, `[upload-extension-cli:$1]`: Triggers release/docs workflows and uploads artifacts to DigitalOcean Spaces.
- **Deployment, Operations & Sentry Crash Triage**:
  - `[deploy-collab:<production|staging>]`: Fast-forwards `main` and force-pushes the environment deployment git tag.
  - `[what-is-deployed:$1]`: Prints the deployed `collab` container image tag for `production` or `staging` and verifies whether `git rev-parse tags/$tag` matches `$deployed_image_id`.
  - `[kube-shell:$1]`: Opens an interactive `/bin/bash` session inside a Kubernetes `app=zed` pod in namespace `$1`.
  - `[collab-flamegraph:<production|staging>]`: Captures a `perf` profile from a `deployments/collab` Kubernetes pod and renders a flamegraph SVG.
  - `[digital-ocean-db.sh:<database-name>]`: Whitelists the caller's public IP in DigitalOcean database firewall rules and opens `psql`.
  - `[sentry-fetch:issue]`, `[sentry-fetch:format_crash_report]`: Fetches a Sentry issue and its latest event and formats a Markdown crash report.
  - `[select-sentry-crash-candidates:main]`, `[select-sentry-crash-candidates:--output]`, `[select-sentry-crash-candidates:--org]`, `[select-sentry-crash-candidates:--query]`, `[select-sentry-crash-candidates:--top]`, `[select-sentry-crash-candidates:--sample-size]`: Ranks unresolved Sentry crash issues by solvability and frequency and writes candidate JSON.
- **GitHub Issue, PR & Project Board Automation**:
  - `[github_helpers.py:github_rest_api]`, `[github_helpers.py:github_rest_get_paginated]`, `[github_helpers.py:github_graphql]`, `[github_helpers.py:post_github_comment]`, `[github_helpers.py:is_staff_member]`, `[github_helpers.py:is_community_pull_request]`, `[github_helpers.py:fetch_github_project]`, `[github_helpers.py:add_github_project_item]`, `[github_helpers.py:set_github_project_field]`: Shared GitHub REST/GraphQL client and Project v2 mutation helpers.
  - `[github-check-new-issue-for-duplicates.py:issue_number]`, `[github-check-new-issue-for-duplicates.py:--dry-run]`, `[github-check-new-issue-for-duplicates.py:analyze_duplicates]`: Claude-powered duplicate issue and discussion detector for newly opened issues.
  - `[github-find-top-duplicated-bugs.py:--github-token]`, `[github-find-top-duplicated-bugs.py:--issue-number]`: GraphQL scanner that identifies open issues with `>= 2` closed duplicates and updates the duplicate magnets tracking issue.
  - `[github-track-duplicate-bot-effectiveness.py:classify-closed]`, `[github-track-duplicate-bot-effectiveness.py:classify-open]`: Evaluates duplicate-bot comment outcomes (`Success`, `Assist`, `Noise`, `Missed opportunity`) and syncs results to a GitHub Project v2 board.
  - `[github-community-pr-board.py:sync_track_from_labels]`, `[github-community-pr-board.py:recompute_signals]`, `[github-community-pr-board.py:refresh_all_board_items]`, `[community-pr-track-mapping.json:tracks]`: Routes community PRs onto the review board by track and computes size, contributor tier, linked issue type, and upvotes.
  - `[github-guild-board.py:run_event]`, `[github-guild-board.py:run_stale]`, `[github-guild-board.py:run_weekly]`: Manages the Guild GitHub Project v2 board, stale assignee nudges/auto-unassignment, and Slack digests.
  - `[github-triage-queue-board.py:sync_project]`, `[github-triage-queue-board.py:eligibility_window]`: Syncs open issues within the business-day triage window onto the second-line triage Project v2 board.
  - `[github-assign-contributor-issue.py:sys.argv[1] / issue_number]`: Matches `.contrib/good *` issues against Tally contributor submissions, assigns the least-busy contributor, updates Project `#83`, and notifies Slack.
  - `[github-pr-cleanup.py:run]`, `[github-pr-cleanup.py:--dry-run]`: Automates pings and closures for unsigned CLA, stale draft, inactive changes-requested, and unreviewed (`no self-review`) community PRs.
  - `[github-pr-status:main]`: Generates open pull request status reports grouped by assignee.
  - `[github-clean-issue-types.py:replace_labels_with_types]`: Converts legacy `bug`, `feature`, and `crash` labels on open issues into native GitHub issue types.
  - `[github-welcome-first-contribution.py:pr_number]`, `[github-welcome-first-contribution.py:--dry-run]`: Posts a welcome comment on a contributor's first pull request.

## Invariants
- `[bench-agent-model-search:feature_tree]`: `cargo tree --locked -p agent_ui_benchmarks -e normal,build,dev,features` must not include `feature "test-support"`, or execution exits with status `1`.
- `[bundle-linux:RUSTFLAGS]` & `[bundle-linux:LLD]`: `RUSTFLAGS` and `CARGO_ENCODED_RUSTFLAGS` must be unset before invoking `bundle-linux`, `LLD` must be an absolute path to an executable `ld.lld` with major version `>= 12`, and `remote-server` built for `*-musl` (or on FreeBSD in `[bundle-freebsd:remote-server]`) must not dynamically link `libcrypto` or `libssl`.
- `[bundle-mac:SDKROOT]`: Requires macOS SDK major version `>= 26` and target triple `x86_64-apple-darwin` or `aarch64-apple-darwin`.
- `[check-keymaps:pattern]`: Non-macOS keymaps under `assets/keymaps/` must not contain `cmd-`, and `git grep --fixed-strings` checks keymap files against `pattern='super-|win-|fn-'`.
- `[check-licenses:check_license]`, `[check-licenses:check_manifest_for_agpl]`, `[check-licenses:check_no_agpl_license_file]`: Every subdirectory with a tracked `Cargo.toml` (`*/Cargo.toml`) must contain a valid relative symlink named `LICENSE-GPL` or `LICENSE-APACHE` pointing to the root license file, no `Cargo.toml` may match `(agpl|affero)` or `license-file`, and no `LICENSE-AGPL` file may exist.
- `[check-todos:pattern]`: Tracked files (excluding `.github/workflows/ci.yml`, `*criteria.md`, and `*prompt.md`) must not contain `todo!` or `FIXME`.
- `[deploy-collab:branch]`: Must be invoked from the `main` branch with a single environment argument (`production` or `staging`).
- `[github_helpers.py:github_rest_api]` / `[github_helpers.py:github_graphql]`: Requires `GITHUB_TOKEN` in the environment; automatic retries (`MAX_ATTEMPTS = 4`) are restricted to idempotent HTTP methods (`GET`, `HEAD`, `OPTIONS`) and non-mutation GraphQL queries.
- `[github-check-new-issue-for-duplicates.py:should_skip]`: Skips duplicate analysis when the issue author belongs to `zed-industries/staff` or when the issue has a non-empty type other than `Bug` or `Crash`.
- `[github-pr-cleanup.py:SELF_REVIEW_MARKER]`: Must stay synchronized with `SELF_REVIEW_MARKER` in `script/danger/dangerfile.ts`, and PRs with the `never stale` label are exempt from all cleanup rules.

## Side Effects
- **Filesystem & Local Toolchain Mutations**: Scripts create, modify, or delete build artifacts under `target/`, `.blob_store/`, `crates/*/Cargo.toml`, `assets/licenses.*`, `crates/docs_preprocessor/actions.json`, `crates/json_schema_store/src/schemas/`, shell profile configs (`~/.zshrc`, `~/.bashrc`, etc.), `/Applications/Zed*.app`, and `~/.local/`.
- **System Package & Database Mutations**: Bootstrap and OS setup scripts install system packages (`brew`, `apt`, `dnf`, `pkg`, `rustup`, `cargo install`, `pnpm dlx`), modify WSL sysctls, and create, migrate, or force-drop local PostgreSQL databases (`zed`, `zed_llm`, `zed-test-%`).
- **External Network & Cloud Service Calls**: Scripts perform authenticated requests and mutations against the GitHub REST and GraphQL APIs (issues, PRs, releases, workflows, Projects v2), Anthropic Messages API (`api.anthropic.com`), Sentry API (`sentry.io`), Tally API, Slack API/webhooks, DigitalOcean Spaces (`nyc3.digitaloceanspaces.com`), DigitalOcean Managed Databases/Kubernetes (`doctl`, `kubectl`), and Apple notarization services (`xcrun notarytool`).

## Verification
- Run: `./script/shellcheck-scripts`
