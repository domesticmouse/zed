# zed-eval

## Overview

Python CLI and installed-agent package for building Zed's headless `eval-cli`
binary, launching benchmark runs on Modal/Harbor/Pier, and fetching results.

This README is for `crates/eval_cli/zed_eval/`. The Rust `eval-cli` binary is
documented in [`../README.md`](../README.md). Commands below assume they are run
from the repository root unless otherwise noted.

Most benchmark users should start with `zed-eval`.

## Quick start

Install the Python CLI as an editable tool:

```sh
crates/eval_cli/script/install-zed-eval
```

This wraps `uv tool install --editable crates/eval_cli/zed_eval --force`, so the
installed `zed-eval` command tracks checkout changes without needing
`PYTHONPATH`. If you prefer not to install globally, run from the checkout with:

```sh
crates/eval_cli/script/zed-eval doctor
```

Check your setup and create the Modal volume if needed:

```sh
zed-eval doctor --create-volume
```

Deploy the Modal app once before launching runs, and again after changing the
Python orchestration code:

```sh
zed-eval deploy
```

Deploying replaces the live app and can cancel in-flight runs, so avoid running
it while evals are active.

Launch a small benchmark run from your current checkout:

```sh
zed-eval run rf --from local --n-tasks 2
```

Monitor and report:

```sh
zed-eval runs                 # recent runs launched from this machine
zed-eval status               # status for the most recent run
zed-eval logs <run-id>        # controller log
zed-eval report <run-id> --fetch
```

After a launch, the local run index remembers the run's namespace and benchmark,
so most commands only need the run id. If the run was launched elsewhere, pass
`--experiment-name` and, if needed, `--namespace`.

## Development

Run the package tests without manually setting `PYTHONPATH`:

```sh
uv run --project crates/eval_cli/zed_eval python -m unittest discover -s crates/eval_cli/zed_eval/tests
```

## Prerequisites

You need:

- access to the Modal workspace used for evals
- a Modal token secret for the controller, default: `agent-evals-modal-token`
- an LLM-provider secret for agent/judge API keys, default:
  `agent-evals-llm-providers`

The controller secret should contain:

- `MODAL_TOKEN_ID`
- `MODAL_TOKEN_SECRET`

The LLM-provider secret should contain the keys your selected models need, such as:

- `ANTHROPIC_API_KEY`
- `OPENAI_API_KEY`
- `BASETEN_API_KEY`

Defaults can be overridden globally:

```sh
zed-eval \
  --volume agent-evals \
  --api-secret agent-evals-llm-providers \
  --modal-token-secret agent-evals-modal-token \
  doctor
```

## Launch benchmarks

Use `zed-eval run` for non-interactive launches:

```sh
# One SWE-Atlas part
zed-eval run rf --from local --n-tasks 5

# Multiple benchmarks share one build when possible
zed-eval run swe-atlas terminal-bench-2.1 --from v0.210.0 --n-tasks 5 --yes

# DeepSWE runs under Pier automatically
zed-eval run deepswe --from local --n-tasks 10

# Inspect what would happen without launching
zed-eval run swe-atlas --from v0.210.0 --plan
zed-eval run deepswe --from local --dry-run
```

Supported benchmark selectors:

| Selector | Meaning | Scoring |
| --- | --- | --- |
| `swe-atlas` | `qna`, `rf`, and `tw` | LLM judge |
| `qna` / `swe-atlas-qna` | SWE-Atlas Codebase Q&A | LLM judge |
| `rf` / `swe-atlas-rf` | SWE-Atlas Refactoring | LLM judge |
| `tw` / `swe-atlas-tw` | SWE-Atlas Test Writing | LLM judge |
| `terminal-bench-2.1` / `tb21` | Terminal-Bench 2.1 | tests |
| `deepswe` | DeepSWE | tests |

For an interactive prompt, use:

```sh
zed-eval swe-atlas --interactive
```

Despite the command name, the interactive flow can launch SWE-Atlas parts,
Terminal-Bench, and DeepSWE.

## Choose source and builds

`--from` controls what source is built into `eval-cli`:

- `--from local` builds current `HEAD` plus tracked changes.
- `--from <ref/tag/sha>` builds a clean git ref, tag, or SHA.

Builds are content-addressed and reused when possible. You can also name or reuse
a build explicitly:

```sh
zed-eval build --from local
zed-eval run rf --build bld-abc123 --n-tasks 2
zed-eval builds --details
```

Untracked files are not included in builds. If they are irrelevant, opt in:

```sh
zed-eval run rf --from local --allow-untracked --n-tasks 2
```

For reproducible runs, use a clean ref/tag/SHA or require a clean tracked tree:

```sh
zed-eval run rf --from v0.210.0 --n-tasks 2
zed-eval run rf --from local --require-clean --n-tasks 2
```

## Choose models and judges

Default agent model:

- `sonnet-4.6` → `anthropic/claude-sonnet-4-6`

Common agent model examples:

```sh
zed-eval run rf --model sonnet-4.6 --n-tasks 2
zed-eval run rf --model opus-4.5 --n-tasks 2
zed-eval run rf --model baseten:kimi-k2.7-code --n-tasks 2
zed-eval run rf --model baseten:deepseek-v4-pro --n-tasks 2
```

For SWE-Atlas judge defaults, `--judge auto` uses:

- `qna`: `deepseek-v4-pro`
- `rf` and `tw`: `kimi-k2.7-code`

Override the judge when needed:

```sh
zed-eval run rf --judge leaderboard --n-tasks 2
zed-eval run rf --judge deepseek-v4-pro --judge-model deepseek-ai/DeepSeek-V4-Pro
```

Baseten presets generate the OpenAI-compatible provider settings automatically.
The sandbox secret must include `BASETEN_API_KEY`.

## Monitor, fetch, and report

Useful commands:

```sh
zed-eval runs                         # local, fast recent-run list
zed-eval list --details               # query runs on the Modal volume
zed-eval list -e swe-atlas-rf --details
zed-eval status <run-id>
zed-eval logs <run-id>
zed-eval fetch <run-id>
zed-eval report <run-id> --fetch
```

`fetch` downloads the run archive and extracts it to:

```text
~/.cache/harbor/jobs/<run-id>/
```

`report` prints pass rate plus resource usage such as tokens, tool calls, and
agent steps. Use `--json` for machine-readable output.

## Multi-benchmark suites

Launching more than one benchmark in one command groups the runs under a
`suite_id` stored on each run.

```sh
zed-eval run qna rf terminal-bench-2.1 --from local --n-tasks 2 --yes
```

Suite commands operate on all runs in that group:

```sh
zed-eval suite status <suite-id>
zed-eval suite logs <suite-id>
zed-eval suite fetch <suite-id>
```

## Rejudge a finished run

`rejudge` creates a new derived run by re-running only the judge. It does not redo
the agent's work or modify the parent run.

```sh
zed-eval rejudge <parent-run-id> --judge deepseek-v4-pro
zed-eval rejudge <parent-run-id> --judge kimi-k2.7-code --dry-run
zed-eval report <derived-run-id> --fetch
```

If the parent run is not in your local run index, pass `--experiment-name` and
possibly `--namespace`.

## Baselines

Baseline commands record and inspect baseline-of-record results for completed
clean-commit runs:

```sh
zed-eval baseline record <run-id> --experiment-name swe-atlas-rf
zed-eval baseline list
zed-eval baseline show swe-atlas-rf --model anthropic/claude-sonnet-4-6
```

## Storage model

Runs and builds live on the shared Modal volume:

```text
builds/<build-id>/
  eval-cli
  build-info.json
  source-info.json
  source.patch

runs/<namespace>/<experiment-name>/<run-id>/
  request.json
  state.json
  controller.log
  harbor-command.txt
  run-metadata.json
  harbor-job.tar.gz
  summary.json
```

Namespaces prevent accidental collisions but are not access control. Anyone with
access to the Modal workspace/volume can read run manifests, logs, patches, and
archives.

## Standalone eval-cli

For local or custom harness usage, you can build and invoke the Rust binary
directly.

Build for the current platform:

```sh
cargo build --release -p eval_cli
```

Build the Linux binary used by Harbor/Pier sandboxes:

```sh
crates/eval_cli/script/build-linux
```

Run directly:

```sh
eval-cli \
  --workdir /testbed \
  --model anthropic/claude-sonnet-4-6 \
  --instruction "Fix the bug described in..." \
  --timeout 600 \
  --output-dir /logs/agent
```

`eval-cli` reads provider API keys from environment variables and writes
`result.json`, `thread.md`, and `thread.json` to the output directory.

Exit codes:

| Code | Meaning |
| --- | --- |
| 0 | Agent finished |
| 1 | Error, such as model/auth/runtime failure |
| 2 | Timeout |
| 3 | Interrupted by SIGTERM or SIGINT |

## Harbor/Pier installed agent

The Python package also exposes installed-agent classes used by the remote
orchestrator:

- `zed_eval.agent:ZedAgent` for Harbor
- `zed_eval.pier_agent:ZedPierAgent` for Pier

For manual Harbor experiments with a locally built Linux binary:

```sh
pip install -e crates/eval_cli/zed_eval/
crates/eval_cli/script/build-linux

harbor run -d "swebench_verified@latest" \
  --agent-import-path zed_eval.agent:ZedAgent \
  --ae binary_path=target/eval-cli \
  --ae EVAL_CLI_TIMEOUT=600 \
  -m anthropic/claude-sonnet-4-6
```

## Interface

- **Package & CLI Entrypoints**:
  - `[pyproject.toml:project.scripts.zed-eval]`: Console script entrypoint `zed_eval.cli:main` for the `zed-eval` package (`version = "0.2.0"`).
  - `[cli.py:main]`, `[cli.py:build_parser]`, `[cli.py:command_doctor]`, `[cli.py:command_deploy]`, `[cli.py:command_cleanup]`: Top-level CLI argument parser and command dispatch for `doctor`, `deploy`, `cleanup`, `build`, `builds`, `run`, `swe-atlas`, `runs`, `list`, `status`, `logs`, `fetch`, `report`, `suite`, `rejudge`, and `baseline`.
- **Harbor & Pier Agent Wrappers**:
  - `[agent.py:ZedAgent]`: `BaseInstalledAgent` implementation (`name() == "zed"`) that installs container dependencies, provisions `/usr/local/bin/eval-cli` (from `EVAL_CLI_CONTAINER_PATH`, `binary_path`, or `EVAL_CLI_DOWNLOAD_URL`), and executes `eval-cli`.
  - `[pier_agent.py:ZedPierAgent]`: Pier `BaseInstalledAgent` implementation for air-gapped DeepSWE runs (`install_spec`, `network_allowlist`, `_container_binary_path`).
- **Configuration, Judges & Remote App**:
  - `[config.py:JudgeConfig]`, `[config.py:JUDGES]`, `[config.py:judge_verifier_args]`: Judge model configurations (`leaderboard`, `opus-leaderboard`, `deepseek-v4-pro`, `kimi-k2.7-code`, `gpt55`) and `ZedJudgeProxyVerifier` CLI wiring.
  - `[modal_app.py:app]`: Modal application (`DEFAULT_APP_NAME = "agent-evals"`) and controller functions for remote builds, benchmark execution, and volume management.

## Invariants

- `[pyproject.toml:project.requires-python]` & `[config.py:HARBOR_VERSION]` / `[config.py:MODAL_VERSION]`: Requires Python `>=3.12` with pinned `harbor==0.15.0` and `modal==1.5.0`.
- `[cli.py:command_doctor]`: Requires `git`, `modal`, and `harbor` executables on `PATH` and both Modal secrets (`args.api_secret` default `agent-evals-llm-providers` and `args.modal_token_secret` default `agent-evals-modal-token`) to exist in `modal secret list --json`.
- `[pier_agent.py:ZedPierAgent._container_binary_path]` & `[pier_agent.py:ZedPierAgent.network_allowlist]`: `ZedPierAgent` requires `EVAL_CLI_CONTAINER_PATH` on the mounted Modal volume and restricts sandbox egress to `DEFAULT_PROVIDER_DOMAINS` plus hosts extracted from `BASE_URL_ENV_VARS` and `ZED_OPENAI_COMPATIBLE_PROVIDERS`.
- `[agent.py:EVAL_CLI_FINALIZE_BUFFER_SEC]` & `[agent.py:EVAL_CLI_MIN_TIMEOUT_SEC]`: Reserves a 45-second finalize buffer (`EVAL_CLI_FINALIZE_BUFFER_SEC = 45`) and enforces a minimum timeout of 60 seconds (`EVAL_CLI_MIN_TIMEOUT_SEC = 60`) so `eval-cli` can flush output before Harbor's outer timeout expires.

## Side Effects

- `[cli.py:modal_secret_names]`, `[launch.py]`, `[modal_app.py:app]`: Spawns `git`, `modal`, and `harbor` subprocesses, deploys and invokes remote Modal functions and sandboxes, and reads/writes `builds/<build-id>/` and `runs/<namespace>/<experiment-name>/<run-id>/` artifacts on the shared Modal volume.
- `[volume.py:command_fetch]`: Downloads run archives from the Modal volume and extracts them to `~/.cache/harbor/jobs/<run-id>/`.
- `[agent.py:ZedAgent.install]`: Executes root package installation (`apt-get`, `apk`, `dnf`, or `yum`), installs Node/LSPs/`uv`/`ruff` best-effort, and copies or downloads `/usr/local/bin/eval-cli` inside trial containers.

## Verification

- Run: `uv run --project crates/eval_cli/zed_eval python -m unittest discover -s crates/eval_cli/zed_eval/tests`
