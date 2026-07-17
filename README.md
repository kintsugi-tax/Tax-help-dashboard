# TXHLP Jurisdiction Summary

Jurisdiction-wise summary of Tax Help (TXHLP) service board queries with turnaround time (TAT), in two forms:

1. **Live Cowork artifact** (`artifact/txhlp-summary.html`) — interactive dashboard with Week / Month / Quarter views and period navigation. Pulls fresh Jira data on every open.
2. **Weekly Slack automation** (`automation/weekly-slack-summary.md`) — scheduled Cowork task (Mondays 09:00) that posts last week's summary to #tax-research-internal (C09BG71PK2M).

Requested by Shardul, 2026-07-10 ([Slack thread](https://kintsugi-space.slack.com/archives/C09BG71PK2M/p1783694227743429)).

## Data source

Jira Service Management, project `TXHLP` (Tax Help), Atlassian cloud ID `0ad27751-cd16-4ced-a3da-67aaa28d6b7f`, queried via the Atlassian MCP tool `searchJiraIssuesUsingJql`. Pagination via `pageInfo.endCursor` → `nextPageToken`.

## Field map (verified against live tickets, 2026-07-12)

| Field | ID | Notes |
|---|---|---|
| Relevant Regions | `customfield_11193` | Multi-select: US, EU, Canada, LATAM, APAC, MEA. Populated on intake; 0 nulls in the last quarter's tickets. Confidence: green. |
| Time to done (SLA) | `customfield_10090` | JSM SLA, 48 business-hour goal. Clock pauses in waiting states; weekends excluded. |
| Time to first response (SLA) | `customfield_10046` | JSM SLA, 24h goal, same structure. |
| Resolved | status category `done` | This board never sets a Jira resolution, so `resolutiondate` is always null (verified 2026-07-13 across the last quarter's tickets). Resolved = status category Done; resolution time = "Time to done" SLA `stopTime` (fallback: `statuscategorychangedate`). |

## Methodology

**Resolved** = status category Done, timed by the SLA stop clock (see field map). **Volume** = tickets created in the period. **TAT** = tickets resolved in the period (regardless of when created). Mixing these bases is deliberate: volume answers "what came in", TAT answers "how fast did we close what we closed".

**TAT (headline)** = `completedCycles[last].elapsedTime.millis / 3,600,000` on the Time to done SLA, i.e. business hours with the clock paused while awaiting customer info. Directly comparable to the team's 48-business-hour SLA target. **Within SLA** uses the SLA engine's own `breached` flag, not a recomputation. Tickets without SLA data fall back to calendar created→resolved time (marked `*`) and are excluded from the SLA % denominator.

**Region grouping** (green): from Relevant Regions. A ticket tagged with multiple regions counts once per region, so region rows can sum to more than the total.

**Jurisdiction drill-down** (yellow, heuristic): country and US state names are dictionary-matched from ticket summaries. Standalone uppercase 2-letter tokens are treated as US state codes; `CA` resolves to Canada only when the ticket's region tag is Canada, else California. "multi-state" / "N states" phrasing maps to "Multi-state (US)". False positives are possible; treat as indicative only. If finer, reliable jurisdiction reporting is wanted, the right fix is a country-level field on the intake form.

## Known limitations / ideas for iteration

- Region field is region-level only; country granularity depends on summary text quality.
- SLA fields follow whatever calendar/holiday config the JSM board uses; the dashboard does not re-derive it.
- Multi-region tickets double-count in region rows (documented above).
- Possible v2: satisfaction (customfield_10040), assignee cuts, recurring-question clustering to feed Rushikesh's docs-update process.

## Editing the artifact

The HTML file is self-contained (inline CSS/JS, Chart.js from CDN). To change it: edit `artifact/txhlp-summary.html`, then in Cowork ask Claude to update the artifact `txhlp-jurisdiction-summary` from the file. The jurisdiction dictionaries are the `US_STATES`, `COUNTRIES`, and `COUNTRY_ALIASES` constants near the top of the script block.

## Editing the automation

The scheduled task lives in Cowork (Scheduled sidebar, task `txhlp-weekly-jurisdiction-summary`). `automation/weekly-slack-summary.md` is the canonical copy of its prompt; keep the two in sync when iterating.
