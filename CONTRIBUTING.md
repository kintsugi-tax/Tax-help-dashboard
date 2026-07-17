# Contributing

Changes to the Tax Help dashboard go through PRs here, not Slack comments.

## How to propose a change

1. Branch from `main`: `git checkout -b your-name/short-description`
2. Make the edit (see "What lives where" below).
3. Open a PR. In the description, say what should change in the dashboard output and why.
4. Yash (or another maintainer) reviews, merges, and syncs the change into the live Cowork artifact / scheduled task.

Small first PRs that are easy to make: add a country to the `COUNTRIES` list, add an alias to `COUNTRY_ALIASES`, fix a wrong region mapping, improve the Slack message format.

## What lives where

| File | What it controls |
|---|---|
| `artifact/txhlp-summary.html` | The live dashboard. Jurisdiction dictionaries (`US_STATES`, `COUNTRIES`, `COUNTRY_ALIASES`), metrics, charts, tables — all inline in the one file. |
| `automation/weekly-slack-summary.md` | Prompt for the Monday Slack post to #tax-research-internal. |
| `README.md` | Field map + methodology. If your PR changes methodology (TAT definition, resolved logic, grouping), update the README in the same PR. |

## Rules

- Never change the TAT or "resolved" methodology without updating the README and flagging the PR to Yash — numbers the team reports to Shardul depend on it.
- Jurisdiction extraction is heuristic by design. Improving the dictionary is welcome; presenting extracted jurisdictions as verified data is not.
- Keep the artifact self-contained (single HTML file, inline CSS/JS). Only whitelisted CDN libraries load in Cowork.

## After a merge

The repo is the source of truth but does not run anything. The live artifact and scheduled task run in Cowork. After merging, ask Yash (or Claude in a Cowork session) to update artifact `txhlp-jurisdiction-summary` and scheduled task `txhlp-weekly-jurisdiction-summary` from the merged files.
