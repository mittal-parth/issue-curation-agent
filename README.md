# Daily GitHub Issue Curator

An AI agent that finds issues to work on across open source repositories.

## What it does

1. Reads configured repositories from `vars.REPOS`.
2. Fetches issues opened in the last `LOOKBACK_HOURS` (default 24h).
3. Deterministically filters issues (see `INCLUDE_ASSIGNED_ISSUES`):
   - by default (`false`), excludes issues assigned to anyone; set to `true` to allow assigned issues as candidates
   - always excludes issues that already have linked open PRs
4. Sends each remaining issue to Cursor CLI (`agent -p`) for quality screening.
5. Appends picks to Google Sheets.
6. Creates a daily GitHub Issue report with a glanceable summary.

## Files

- `.github/workflows/daily-issue-curation.yml` - main workflow
- `AGENTS.md` - curation rubric for the Cursor model

## Required configuration

### Repository Variables

- `REPOS`: comma-separated `owner/name` list to define which repositories are scanned  
  Example: `facebook/react,vercel/next.js,denoland/deno`
- `LOOKBACK_HOURS` (optional): default `24`; controls how far back to look for newly opened issues
- `MAX_ISSUES_PER_REPO` (optional): default `20`; caps candidate volume per repo to keep runs bounded
- `CURATION_MODEL` (optional): default `gpt-5.2`; selects the model used by Cursor CLI for issue grading
- `INCLUDE_ASSIGNED_ISSUES` (optional): default `false`; only unassigned issues become candidates (set to `true` to also include assigned issues). Use the strings `true` or `false`.
- `WRITE_TO_SHEET` (optional): default `true`; toggles whether picked issues are appended to Google Sheets
- `CREATE_REPORT_ISSUE` (optional): default `false`; toggles creation of the daily summary GitHub issue

### Repository Secrets

- `CURSOR_API_KEY` - required so the workflow can use the Cursor CLI to curate issues 
- `GOOGLE_SHEETS_ID` - required only when sheet writing is enabled; identifies the target spreadsheet
- `GOOGLE_SHEETS_CREDENTIALS` - required only when sheet writing is enabled; authenticates to Google Sheets (service-account JSON)
- `GH_PAT` - optional; raises GitHub read API limits for cross-repo querying (falls back to `github.token` if omitted)

If Sheets secrets are not set, the workflow still runs and creates the GitHub issue report.

## Google Sheet format

Use two tabs in the same spreadsheet:

- `Accepted` (picked issues)
- `Rejected` (skipped issues)

If either tab is missing, the workflow auto-creates it.

The workflow appends to `A:G` in both tabs:

`date | repo | title | url | difficulty | reason | status`

`status` is intentionally blank so you can track progress manually.

## Triggering

- Scheduled: daily at `06:00 UTC`
- Manual: `workflow_dispatch` with optional overrides:
  - `repos`
  - `lookback_hours`
  - `write_to_sheet` (default `true`)
  - `create_report_issue` (default `false`)
  - `include_assigned_issues` (default `false`; uses `vars.INCLUDE_ASSIGNED_ISSUES` for scheduled runs)
  - `curation_model` (blank uses `vars.CURATION_MODEL` or `gpt-5.2`)

## Notes

- This implementation is standalone and does not reuse any existing application code in this repository.
- The report issue uses label: `automation`.
