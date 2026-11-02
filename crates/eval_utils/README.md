# eval_utils

## Overview

Utilities for evals of agents, providing concurrent evaluation execution, progress reporting, and pass-ratio assertions.

## Interface

- Evaluation types & runner: `[src/eval_utils.rs:OutcomeKind]`, `[src/eval_utils.rs:EvalOutputProcessor]`, `[src/eval_utils.rs:EvalOutput]`, `[src/eval_utils.rs:EvalOutput::passed]`, `[src/eval_utils.rs:EvalOutput::failed]`, `[src/eval_utils.rs:NoProcessor]`, `[src/eval_utils.rs:eval]`

## Invariants

- `eval` executes the first iteration synchronously before spawning remaining (`1..iterations`) evaluations concurrently with a semaphore limit of 32 (`[src/eval_utils.rs:eval]`).
- `eval` panics if `actual_pass_ratio` (`(iterations - failed_count) as f32 / iterations as f32`) is strictly less than `expected_pass_ratio`, and invokes `processor.assert()` afterward (`[src/eval_utils.rs:eval]`).

## Side Effects

- Spawns detached background tasks on `gpui_platform::background_executor()` and writes terminal progress lines and failure summaries to stdout (`[src/eval_utils.rs:eval]`, `[src/eval_utils.rs:report_progress]`).

## Verification

- Run: `cargo test -p eval_utils`
