# danger

## Overview

DangerJS pull-request review automation for the Zed repository that enforces PR title hygiene, release notes formatting (including `[GPUI]` release note requirements), GitHub issue linking guidance, mandatory agent self-review marker removal in `README.md` and preservation in `.rules`, and explicit PR body attestations when modifying database test schemas or edit-agent evaluation fixtures.

## Interface

- `[package.json:scripts.danger]`: Runs `tsc --noEmit && danger` using `danger` (`13.0.8`), `danger-plugin-pr-hygiene` (`0.7.1`), and `typescript` (`5.9.3`).
- `[tsconfig.json:include]`: TypeScript configuration (`ES2022`, `CommonJS`, `strict: true`, `noEmit: true`) targeting `dangerfile.ts`.
- `[dangerfile.ts:prHygiene]`: Configures `danger-plugin-pr-hygiene` with `prefixPattern: /^([a-z\d\(\)_\s]+):(.*)/g`, `useImperativeMood: "off"`, and `noConventionalCommits` banning `["feat", "fix", "style", "refactor", "perf", "test", "chore", "build", "revert"]`.
- `[dangerfile.ts:GPUI_RELEASE_NOTES_PATTERN]`: Regex `/^- \[GPUI\]/im` matching GPUI-prefixed release note bullets when files matching `crates/gpui*/**` are edited or deleted.
- `[dangerfile.ts:ISSUE_LINK_PATTERN]`: Regex matching raw `https://github.com/<owner>/<repo>/issues/<num>` URLs in the PR body not preceded by closing/linking keywords (`Close[sd]?`, `Fixe[sd]`, `Resolve[sd]`, `Implement[sed]`, `Follow-up of`, `Part of`).
- `[dangerfile.ts:SELF_REVIEW_MARKER]`: Marker string `"Remove this line to confirm you've reviewed this PR before submitting."` checked against `README.md` and `.rules` diffs.
- `[dangerfile.ts:SCHEMA_CHANGE_ATTESTATION]`: Required PR body attestation string `"The corresponding database schema migration has been created in the Cloud repo and applied to the production database."`.
- `[dangerfile.ts:MIGRATION_SCHEMA_FILES]`: Protected database schema paths `["crates/collab/migrations/20251208000000_test_schema.sql", "crates/collab/migrations.sqlite/20221109000000_test_schema.sql"]`.
- `[dangerfile.ts:FIXTURE_CHANGE_ATTESTATION]`: Required PR body attestation string `"Changes to test fixtures are intentional and necessary."`.
- `[dangerfile.ts:FIXTURES_PATHS]`: Protected fixture directory prefixes `["crates/assistant_tools/src/edit_agent/evals/fixtures"]`.

## Invariants

- `[dangerfile.ts:hasReleaseNotes]`: Every PR body must contain a non-empty `Release Notes:` section with non-placeholder entries (not empty or `Added/Fixed/Improved ...`), or `fail(...)` is invoked.
- `[dangerfile.ts:gpuiCrates]`: Any PR that edits or deletes files matching `crates/gpui*/**` must include at least one release note entry matching `GPUI_RELEASE_NOTES_PATTERN` (`/^- \[GPUI\]/im`), or `fail(...)` is invoked.
- `[dangerfile.ts:SELF_REVIEW_MARKER]`: `README.md` diffs must not add `SELF_REVIEW_MARKER`, `.rules` must not be deleted, and `.rules` diffs must not remove `SELF_REVIEW_MARKER` without re-adding it, or `fail(...)` is invoked.
- `[dangerfile.ts:modifiedSchemaFiles]`: Any PR modifying a file ending with a path in `MIGRATION_SCHEMA_FILES` must include `SCHEMA_CHANGE_ATTESTATION` in the PR body, or `fail(...)` is invoked.
- `[dangerfile.ts:modifiedFixtures]`: Any PR modifying a file matching `FIXTURES_PATHS` must include `FIXTURE_CHANGE_ATTESTATION` in the PR body, or `fail(...)` is invoked.

## Side Effects

- `[dangerfile.ts:fail]`: Emits Danger failure annotations on the GitHub pull request when release notes, GPUI release notes, `README.md`/`.rules` self-review rules, database schema attestations, or fixture change attestations are violated.
- `[dangerfile.ts:message]`: Emits informational Danger comments on the GitHub pull request when unlinked issue URLs are detected or when a database schema change attestation is present.
- `[dangerfile.ts:schedule]`: Schedules asynchronous Git diff inspections via `danger.git.diffForFile("README.md")` and `danger.git.diffForFile(".rules")`.

## Verification

- Run: `npm --prefix script/danger exec -- tsc --noEmit -p script/danger/tsconfig.json`
