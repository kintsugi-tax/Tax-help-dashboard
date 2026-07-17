# Scheduled task prompt: txhlp-weekly-jurisdiction-summary

Schedule: every Monday 09:00 local (cron `0 9 * * 1`). Canonical copy of the Cowork scheduled-task prompt — keep in sync with the task in the Scheduled sidebar.

---

# Weekly TXHLP jurisdiction summary → Slack #tax-research-internal

Run this every Monday. Report on the LAST COMPLETE ISO week (Monday 00:00 through Sunday end, i.e. the week that ended yesterday). Also compute the week before that for deltas.

## 1. Pull data from Jira

Use the Atlassian connector tool searchJiraIssuesUsingJql with cloudId 0ad27751-cd16-4ced-a3da-67aaa28d6b7f.

IMPORTANT: this board never sets a Jira "resolution", so resolutiondate is always null. Resolved = status category Done. Use statusCategory/statusCategoryChangedDate in JQL, never resolutiondate.

JQL (substitute real dates; PREV_START = Monday two weeks ago, WEEK_END = this Monday):

project = TXHLP AND ((created >= "PREV_START" AND created < "WEEK_END") OR (statusCategory = Done AND statusCategoryChangedDate >= "PREV_START" AND statusCategoryChangedDate < "WEEK_END")) ORDER BY created ASC

Fields: ["summary","status","created","statuscategorychangedate","assignee","customfield_11193","customfield_10090","customfield_10046"]

Paginate: pass pageInfo.endCursor as nextPageToken until hasNextPage is false. Results may be saved to a file if large; read it fully or extract with Grep/jq.

Field meanings (verified July 2026):
- customfield_11193 = "Relevant Regions" multi-select (US, EU, Canada, LATAM, APAC, MEA). Reliable, populated on intake.
- customfield_10090 = "Time to done" JSM SLA, 48 business-hour goal. TAT (business hours) = completedCycles[last].elapsedTime.millis / 3600000. Within SLA = completedCycles[last].breached === false. Resolution timestamp = completedCycles[last].stopTime.epochMillis (fallback: statuscategorychangedate).
- customfield_10046 = "Time to first response" JSM SLA, 24h goal, same structure.

## 2. Compute metrics

A ticket is RESOLVED when its status category key is "done"; its resolution time is the SLA stop time (fallback: statuscategorychangedate). Volume basis = tickets CREATED in the reporting week. TAT basis = tickets RESOLVED in the reporting week (resolution time in window).

Overall: new queries (with delta vs prior week), resolved count, median TAT in business hours, % within 48h SLA (breached=false over resolved tickets that have SLA data), median first response (business hours), still-open count among the week's new tickets.

Per region (from customfield_11193; a ticket with multiple regions counts in each): new, resolved, median TAT, % within 48h.

Jurisdiction drill-down (HEURISTIC): extract country and US state names from ticket summaries by name matching (full country names, full US state names, standalone uppercase 2-letter state codes; "CA" = Canada only if the ticket's region tag is Canada, else California). List top jurisdictions by count. Never present this as verified data.

If a resolved ticket has no "Time to done" SLA data, fall back to calendar time created→resolution and exclude it from the SLA % denominator.

## 3. Post to Slack

Post ONE message to channel C09BG71PK2M (#tax-research-internal) using the Slack connector's slack_send_message with mrkdwn. Format (keep it compact, no preamble):

:world_map: *TXHLP jurisdiction summary — Mon <d> <Mon> – Sun <d> <Mon>*
*Totals:* <N> new (▲/▼ <delta> vs prior wk) · <R> resolved · median TAT <X> bh · <P>% within 48h SLA · median first response <F> bh · <O> still open
*By region:*
• US — <n> new · <r> resolved · median TAT <x> bh · <p>% ≤48h
• EU — ... (one line per region with activity, ordered by volume; skip zero-activity regions)
*Top jurisdictions (heuristic, from summaries):* California 5 · Portugal 3 · ...
*Breached 48h SLA:* <TXHLP-XXX>, <TXHLP-YYY> (or "none")
_TAT = JSM "Time to done" SLA clock: business hours, pauses while awaiting customer info, weekends excluded. Live view with month/quarter cuts: ask Yash for the Cowork artifact._

Use real Slack link formatting for ticket keys: <https://trykintsugi.atlassian.net/browse/TXHLP-XXX|TXHLP-XXX>.

Rules: never fabricate numbers; if the Jira query fails, post nothing and report the failure in your completion summary instead. Round business hours to 1 decimal. If a week has zero tickets, still post the header line saying so.
