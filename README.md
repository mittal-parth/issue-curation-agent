# Daily GitHub Issue Curator

An AI agent that finds issues to work on across open source repositories.

## What it does

1. Reads configured repositories from `vars.REPOS`.
2. Fetches issues opened in the last `LOOKBACK_HOURS` (default 24h).
3. Deterministically filters out issues that:
   - are assigned to anyone
   - already have linked open PRs
4. Sends each remaining issue to Cursor CLI (`agent -p`) for quality screening.
5. Appends picks to Google Sheets.
6. Creates a daily GitHub Issue report with a glanceable summary.

## Files

- `.github/workflows/daily-issue-curation.yml` - main workflow
- `AGENTS.md` - curation rubric for the Cursor model

## Required configuration

### Repository Variables

- `REPOS`: comma-separated `owner/name` list  
  Example: `facebook/react,vercel/next.js,denoland/deno`
- `LOOKBACK_HOURS` (optional): default `24`
- `MAX_ISSUES_PER_REPO` (optional): default `20`

### Repository Secrets

- `CURSOR_API_KEY` - required for `agent -p`
- `GOOGLE_SHEETS_ID` - optional but recommended
- `SHEETS_CREDENTIALS` - optional but recommended (Google service-account JSON)
- `GH_PAT` - optional; improves GitHub API rate limits for cross-repo reads

If Sheets secrets are not set, the workflow still runs and creates the GitHub issue report.

## Google Sheet format

The workflow appends to `Sheet1!A:H`:

`date | repo | issue# | title | url | difficulty | reason | status`

`status` is intentionally blank so you can track progress manually.

## Triggering

- Scheduled: daily at `06:00 UTC`
- Manual: `workflow_dispatch` with optional `repos` and `lookback_hours` overrides

## Notes

- This implementation is standalone and does not reuse any existing application code in this repository.
- The report issue uses labels: `daily-picks`, `automation`.
