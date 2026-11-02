# update_top_ranking_issues

## Overview
Python CLI utility managed with `uv` and `typer` that queries the GitHub Search API for top-voted open issues in `zed-industries/zed`, groups and ranks them by label/section and net thumbs-up score, and either prints the generated Markdown summary or updates a target reference tracking issue on GitHub.

## Interface
- `[pyproject.toml:project.name]`: Python package `update-top-ranking-issues` (`0.1.0`).
- `[pyrightconfig.json:venvPath]`: Pyright type-checking configuration pointing to `./.venv`.
- `[main.py:app]`: `typer.Typer` CLI application instance.
- `[main.py:main]`: CLI command entrypoint accepting `--github-token` (`Optional[str]`), `--issue-reference-number` (`Optional[int]`), and `--query-day-interval` (`Optional[int]`).
- `[main.py:get_section_to_issues]`: `get_section_to_issues(headers: dict[str, str], start_date: date | None = None) -> dict[str, list[dict[str, Any]]]` for fetching and ranking top issues per section.
- `[main.py:update_reference_issue]`: `update_reference_issue(headers: dict[str, str], issue_number: int, body: str) -> None` for patching the reference GitHub issue body.
- `[main.py:create_issue_text]`: `create_issue_text(section_to_issues: dict[str, list[dict[str, Any]]]) -> str` for formatting ranked issues into numbered Markdown lists under section headings.
- `[main.py:AMERICA_NEW_YORK_TIMEZONE]`: Timezone constant `'America/New_York'`.
- `[main.py:DATETIME_FORMAT]`: Timestamp format constant `'%B %d, %Y %I:%M %p'`.
- `[main.py:ISSUES_PER_SECTION]`: Maximum issues rendered per section (`50`).
- `[main.py:ISSUES_TO_FETCH]`: Number of issues requested per GitHub Search API call (`100`).
- `[main.py:REPO_OWNER]`: Repository owner constant `'zed-industries'`.
- `[main.py:REPO_NAME]`: Repository name constant `'zed'`.
- `[main.py:GITHUB_API_BASE_URL]`: Base URL constant `'https://api.github.com'`.
- `[main.py:EXCLUDE_LABEL]`: Excluded issue label constant `'ignore top-ranking issues'`.

## Invariants
- `[pyproject.toml:requires-python]`: Requires Python `>=3.13`.
- `[main.py:main]`: Requires a GitHub token via `--github-token`, the `GITHUB_ACCESS_TOKEN` environment variable, or `gh auth token`; raises `typer.BadParameter` if none is available.
- `[main.py:ISSUES_TO_FETCH]`: Must not exceed `100` to fit within a single page of GitHub Search API results (`per_page`).
- `[main.py:get_section_to_issues]`: Only open issues in `zed-industries/zed` without the `'ignore top-ranking issues'` label and with a positive net score (`reactions['+1'] - reactions['-1'] > 0`) are included.
- `[main.py:get_section_to_issues]`: Issues within each section are sorted by `(-score, created_at)` and truncated to `ISSUES_PER_SECTION` (`50`).
- `[main.py:get_section_to_issues]`: Empty sections are excluded, and remaining sections are ordered descending by total issue score.

## Side Effects
- `[main.py:main]`: Reads `GITHUB_ACCESS_TOKEN` from the process environment via `os.getenv`.
- `[main.py:main]`: Spawns `gh auth token` via `subprocess.run` when no token is supplied via CLI flag or environment variable.
- `[main.py:main]`: Writes generated Markdown issue tables (when `--issue-reference-number` is omitted) and total execution time to stdout.
- `[main.py:get_section_to_issues]`: Sends HTTP GET requests to `https://api.github.com/search/issues` for each section filter and raises `requests.HTTPError` on non-2xx responses.
- `[main.py:update_reference_issue]`: Sends an HTTP PATCH request to `https://api.github.com/repos/zed-industries/zed/issues/{issue_number}` to overwrite the issue body and raises `requests.HTTPError` on failure.

## Verification
- Run: `uv run --directory script/update_top_ranking_issues python -m py_compile main.py`
