# Weekly Report Generator

Generate a structured weekly leadership report by scanning your team's Slack activity and saving it as a markdown file.

**Usage:** `/weekly-report` or `/weekly-report for March 17-23`

## Step 1: Load Team Configuration

Read `team-config.yaml` from the project root. If the file does not exist, tell the user to run `/setup` first or copy `team-config.yaml.example` and fill it in.

Extract these values (all referenced as `config.*` below):

- `config.team.name` — team name for report title
- `config.team.em_name` — EM name
- `config.slack.members[]` — list of `{name, slack_id}` objects
- `config.slack.channels[]` — list of `{name, id}` objects
- `config.slack.workstreams[]` — list of active workstream strings
- `config.report.sections` — which sections to include
- `config.report.tone` — writing style
- `config.report.output_dir` — local output path template

## Step 2: Determine Date Range

Compute a **trailing 7-day window** from today:

- `end_date` = yesterday (the day before the report is run)
- `start_date` = `end_date` minus 6 days
- `end_date_plus_1` = today (used for Slack `before:` filter, which is exclusive)

If the user specified a date range in the prompt, use that instead. Format all dates as `YYYY-MM-DD`.

Confirm the date range with the user before proceeding:

```
Team: {config.team.name}
Slack scan period: {start_date} → {end_date}
Team members: {count} people
Channels: {count} channels

Proceed?
```

## Step 3: Scan Slack Messages

### 3a. Search per team member

For each member in `config.slack.members`, run a Slack MCP search:

```
from:<@{member.slack_id}> after:{start_date} before:{end_date_plus_1}
```

Use `response_format: "concise"`, `include_context: false`, `limit: 20`.

Paginate with `cursor` if there are more results.

Collect all messages, noting the author name for each.

### 3b. Search key channels

For each channel in `config.slack.channels`, run:

```
in:{channel.name} after:{start_date} before:{end_date_plus_1}
```

Same parameters. This captures messages from non-team-members that are relevant to the team's work.

### 3c. Deduplicate

Messages found in both member search and channel search should be kept once. Use the Slack message timestamp (`ts`) as a dedup key.

## Step 4: Extract and Categorize Content

Analyze all collected messages and extract three categories:

### Highlights

Group achievements by workstream theme. Use `config.slack.workstreams` as a guide for grouping, but create new theme groups for activity that doesn't fit existing workstreams.

For each highlight:
- Lead with the **outcome**, not the activity
- Include precise numbers exactly as stated (never round or approximate)
- Attribute to the person who drove it
- Include PR links, doc links, experiment links when mentioned

**Example format:**
```
**{Workstream or Theme}**
- {Person} shipped {outcome} — {supporting detail with numbers}. PR: [repo/pull/123](url)
- {Person} completed {outcome}, improving {metric} from X to Y.
```

### Callouts / Help Needed

Identify blockers, risks, and items needing leadership attention:
- Handoff issues or alignment gaps
- Resource constraints
- Dependencies on other teams
- Unclear ownership or scope
- Escalation requests

### Media & Links

Collect all shared media from messages:
- Loom recordings
- Figma links
- Google Docs/Slides/Sheets
- GitHub PR links
- Screenshot attachments

For each, preserve the original URL and note the context.

## Step 5: Format the Report

Assemble the report in markdown using this structure:

```markdown
# {config.team.name} Weekly Report

**Period:** {start_date} to {end_date}
**Generated:** {today's date}

---

## Highlights

**{Theme 1}**
- {Outcome with precise numbers}
- {Next outcome}

**{Theme 2}**
- {Outcome with precise numbers}

---

## Callouts / Help Needed

**{Issue Title}**
{Problem statement — what happened, impact, and what's needed.}

---

## Media & Links

- [{Description}]({URL}) — {context}
- [{Description}]({URL}) — {context}
```

**Writing style** (guided by `config.report.tone`):
- First person plural: "We", "Our team"
- Lead with outcomes, back with data
- Confident but not boastful for highlights
- Constructive and specific for callouts
- Preserve all numbers exactly as stated in Slack — never round or approximate
- Show both fractions AND percentages when available: "19/21 done (90%)"
- Flag unclear numbers with `[verify]`

Only include sections listed in `config.report.sections`. Skip any section that has no content for this week.

## Step 6: Save the Report

Create the output directory and write the markdown file:

```
{config.report.output_dir}/reports/weekly_report_{end_date}.md
```

Replace `{start_date}` and `{end_date}` in `output_dir` with actual dates.

## Step 7: Present for Review

Show the full report to the user. Ask:
- Anything to add, remove, or rephrase?
- Any numbers to verify?

If the user requests changes, apply them and re-save the file.

## Step 8: Confirm Completion

Tell the user:
1. Path to the saved report
2. Summary of what was included (number of highlights, callouts, links)

## Error Handling

- If `team-config.yaml` is missing → tell user to run `/setup` or copy the example
- If Slack MCP is not connected → show the `.mcp.json` configuration needed
- If a team member search returns no results → note it but continue with others
- If no messages are found at all → report this and suggest checking the date range or channel names

## Number Formatting

- Percentages: 1 decimal place (e.g., "45.3%")
- Counts: whole numbers
- Never round numbers from Slack messages — preserve exactly as stated
